---
title: zero copy transfer
date: 2018-08-31 12:00:00
tags: [go,io]
categories: [go]
---

# zero-copy transfer

平时写 http server 会接触到传输文件的问题。最简单的版本大概是这样子的。
``` go
	for {
		n, _ := file.Read(buf)
		if n > 0 {
			w.Write(buf[:n])
		}else{
			break
		}
	}
```
对于一般的程序来说是没有问题的，但是如果需要频繁地传输大文件，那么这种办法效率不够高。因为这里不断地调用两个系统调用，`read()` 和 `write()`。对操作系统稍微熟悉的都会知道系统调用是在**内核态**执行而用户程序是跑在**用户态**。

读取文件就会先从磁盘文件读到内核缓冲区然后再拷贝到用户空间缓冲区，写操作也要先写到内核缓冲区然后再发送。
![read+write](https://upload-images.jianshu.io/upload_images/8053527-1d1055e745391c9e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

系统调用 `sendfile()` 就是用来解决这个底性能问题的。文件可以直接送到socket上，不需要经过用户空间。
![sendfile()](https://upload-images.jianshu.io/upload_images/8053527-dfbdd2afa6e472cd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## sendfile()

```
# include <sys/sendfile.h>
ssize_t sendfile(int out_fd, int in_fd, off_t *offset, size_t count);
```

`in_fd` 是代表输入文件的文件描述符，`out_fd` 是代表输出文件的文件描述符。`out_id` 必须为 socket (linux 2.6.33 开始可以是任何文件)。 `in_fd` 指向的文件必须为可以进行 `mmap()` 操作的，通常为普通文件。

## splice()

```
#define _GNU_SOURCE         /* See feature_test_macros(7) */
#include <fcntl.h>
ssize_t splice(int fd_in, loff_t *off_in, int fd_out,
               loff_t *off_out, size_t len, unsigned int flags);
```

> splice() moves data between two file descriptors without copying between kernel address space and user address space. It transfers up to len bytes of data from the file descriptor fd_in to the file descriptor fd_out, where one of the descriptors must refer to a pipe.

## io.Copy()

当我们需要响应请求并返回文件时，我们可以使用`io.Copy()`，因为底层使用了上面提及的`splice()`和`sendfile()`系统调用。

```
func Copy(dst Writer, src Reader) (written int64, err error) {
	return copyBuffer(dst, src, nil)
}

func CopyBuffer(dst Writer, src Reader, buf []byte) (written int64, err error) {
	if buf != nil && len(buf) == 0 {
		panic("empty buffer in io.CopyBuffer")
	}
	return copyBuffer(dst, src, buf)
}

func copyBuffer(dst Writer, src Reader, buf []byte) (written int64, err error) {
	// If the reader has a WriteTo method, use it to do the copy.
	// Avoids an allocation and a copy.
	if wt, ok := src.(WriterTo); ok {
		return wt.WriteTo(dst)
	}
	// Similarly, if the writer has a ReadFrom method, use it to do the copy.
	if rt, ok := dst.(ReaderFrom); ok {
		return rt.ReadFrom(src)
	}
......
```

上面的源码中可以看到，两个  type assertion。 现在关注一下第二个 `dst.(ReaderFrom)`。

因为我们传惨第一个是 `http.ResponseWriter`，所以实现这个接口的struct是下面这个。

```
// A response represents the server side of an HTTP response.
type response struct {
	conn             *conn
	req              *Request // request for this response
	reqBody          io.ReadCloser
	cancelCtx        context.CancelFunc // when ServeHTTP exits
	wroteHeader      bool               // reply header has been (logically) written
	wroteContinue    bool               // 100 Continue response was written
	wants10KeepAlive bool               // HTTP/1.0 w/ Connection "keep-alive"
	wantsClose       bool               // HTTP request has Connection "close"

	w  *bufio.Writer // buffers output in chunks to chunkWriter
	cw chunkWriter

......
```
找到具体实现`ReadFrom`的地方
```
// ReadFrom is here to optimize copying from an *os.File regular file
// to a *net.TCPConn with sendfile.
func (w *response) ReadFrom(src io.Reader) (n int64, err error) {
	// Our underlying w.conn.rwc is usually a *TCPConn (with its
	// own ReadFrom method). If not, or if our src isn't a regular
	// file, just fall back to the normal copy method.
	rf, ok := w.conn.rwc.(io.ReaderFrom)
	regFile, err := srcIsRegularFile(src)
	if err != nil {
		return 0, err
	}
	if !ok || !regFile {
		bufp := copyBufPool.Get().(*[]byte)
		defer copyBufPool.Put(bufp)
		return io.CopyBuffer(writerOnly{w}, src, *bufp)
	}

	// sendfile path:

	if !w.wroteHeader {
		w.WriteHeader(StatusOK)
	}

	if w.needsSniff() {
		n0, err := io.Copy(writerOnly{w}, io.LimitReader(src, sniffLen))
		n += n0
		if err != nil {
			return n, err
		}
	}

	w.w.Flush()  // get rid of any previous writes
	w.cw.flush() // make sure Header is written; flush data to rwc

	// Now that cw has been flushed, its chunking field is guaranteed initialized.
	if !w.cw.chunking && w.bodyAllowed() {
		n0, err := rf.ReadFrom(src)
		n += n0
		w.written += n0
		return n, err
	}

	n0, err := io.Copy(writerOnly{w}, src)
	n += n0
	return n, err
}
```

函数开头有一段注释已经说明调用的底层是 `*TCPConn` 的 `ReadFrom`。

```
// ReadFrom implements the io.ReaderFrom ReadFrom method.
func (c *TCPConn) ReadFrom(r io.Reader) (int64, error) {
	if !c.ok() {
		return 0, syscall.EINVAL
	}
	n, err := c.readFrom(r)
	if err != nil && err != io.EOF {
		err = &OpError{Op: "readfrom", Net: c.fd.net, Source: c.fd.laddr, Addr: c.fd.raddr, Err: err}
	}
	return n, err
}
```
终于出现了 `splice()` 和 `sendFile()`
```
func (c *TCPConn) readFrom(r io.Reader) (int64, error) {
	if n, err, handled := splice(c.fd, r); handled {
		return n, err
	}
	if n, err, handled := sendFile(c.fd, r); handled {
		return n, err
	}
	return genericReadFrom(c, r)
}
```
再底层的就是go对系统调用的`splice()`和`sendFile()`的封装了。

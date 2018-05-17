---
title: golang 实现文件断点续传 demo
date: 2018-05-17 21:41:58
tags: http
---

# golang 实现文件断点续传 demo

实现http断点续传其中一种办法是使用`范围请求`。
## 涉及头部：
### request
* `Range` 表示获取数据范围。例如：`Range: bytes=0-1023`表示获取下标为`0`到`1023`的字节，一共`1024`个。

### response

* `Content-Range` 表示返回数据的范围。例如`Content-Range: bytes 0-1023/146515`表示当前http报文的body数据是`0-1023`，文件一共有`146515`个字节。

* `Accept-Ranges` 表示接受range的单位，目前只定义了`bytes`。服务器返回这个头部则表明接收`范围请求`。

``` go
package main

import (
	"flag"
	"fmt"
	"github.com/gorilla/mux"
	"io"
	"log"
	"net/http"
	"os"
	"strconv"
	"strings"
)

func main() {
	path := ""
	port := ""
	flag.StringVar(&path,"dir", "", "the dir to server")
	flag.StringVar(&port,"port", "", "server port")
	flag.Parse()
	port = ":" + port
	if strings.LastIndexByte(path,'/') != len(path) - 1{
		path = path + "/"
	}
	r := mux.NewRouter()
	r.HandleFunc("/{file}", func(w http.ResponseWriter, r *http.Request) {
		vars := mux.Vars(r)
		f := vars["file"]
		var start, end int64
		fmt.Sscanf(r.Header.Get("Range"), "bytes=%d-%d", &start, &end)
		file, err := os.Open(path + f)
		if err != nil {
			log.Println(err.Error())
			http.NotFound(w, r)
			return
		}
		info, err := file.Stat()
		if err != nil {
			log.Println(err.Error())
			http.NotFound(w, r)
			return
		}
		if start < 0 ||start >= info.Size() ||end < 0 || end >= info.Size(){
			w.WriteHeader(http.StatusBadRequest)
			w.Write([]byte(fmt.Sprintf("out of index, length:%d",info.Size())))
			return
		}
		if end == 0 {
			end = info.Size() - 1
		}
		w.Header().Add("Accept-ranges", "bytes")
		w.Header().Add("Content-Length", strconv.FormatInt(end-start+1, 10))
		w.Header().Add("Content-Range", "bytes "+strconv.FormatInt(start, 10)+"-"+strconv.FormatInt(end, 10)+"/"+strconv.FormatInt(info.Size()-start, 10))
		w.Header().Add("Content-Disposition", "attachment; filename="+info.Name())
		_, err = file.Seek(start, 0)
		if err != nil {
			log.Println(err.Error())
			w.WriteHeader(http.StatusInternalServerError)
			return
		}
		_, err = io.CopyN(w, file, end-start+1)
		if err != nil {
			log.Println(err.Error())
			return
		}
	})
	http.ListenAndServe(port, r)
}
```

## 待改进
如果在断点后再继续传输并且对象发生变化，那么会出现错误，所以要加上 `Last-Modified`  或者  `ETag`  来判断对象是否已经改变。

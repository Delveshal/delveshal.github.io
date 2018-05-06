---
title: 实现一个简单的http-proxy demo
date: 2018-05-06 17:26:15
tags: [go,http,https,proxy]
categories: http
---

## http-proxy的两种形式
* 第一种是 [RFC 7230 - HTTP/1.1: Message Syntax and Routing](http://tools.ietf.org/html/rfc7230)（即修订后的 RFC 2616，HTTP/1.1 协议的第一部分）描述的普通代理。这种代理扮演的是「中间人」角色，对于连接到它的客户端来说，它是服务端；对于要连接的服务端来说，它是客户端。它就负责在两端之间来回传送 HTTP 报文。
![image.png](https://upload-images.jianshu.io/upload_images/8053527-eebb02c1b2a0f651.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
<!-- more -->

* 第二种是 [Tunneling TCP based protocols through Web proxy servers](https://tools.ietf.org/html/draft-luotonen-web-proxy-tunneling-01)（通过 Web 代理服务器用隧道方式传输基于 TCP 的协议）描述的隧道代理。它通过 HTTP 协议正文部分（Body）完成通讯，以 HTTP 的方式实现任意基于 TCP 的应用层协议代理。这种代理使用 HTTP 的 CONNECT 方法建立连接，但 CONNECT 最开始并不是 RFC 2616 - HTTP/1.1 的一部分，直到 2014 年发布的 HTTP/1.1 修订版中，才增加了对 CONNECT 及隧道代理的描述，详见 [RFC 7231 - HTTP/1.1: Semantics and Content](https://tools.ietf.org/html/rfc7231#section-4.3.6)。实际上这种代理早就被广泛实现。
![image.png](https://upload-images.jianshu.io/upload_images/8053527-8642805e5fc7e678.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

代码中使用了http-proxy认证，详细可以看[这里](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/407)
```
package main

import (
	"bufio"
	"bytes"
	"encoding/base64"
	"fmt"
	"io"
	"log"
	"net"
	"net/http"
	"strings"
)

func main() {
	listener, err := net.Listen("tcp", ":8080")
	if err != nil {
		panic(err)
	}
	for {
		con, err := listener.Accept()
		if err != nil {
			panic(err)
		}
		go handler(con)
	}
}

func handler(client net.Conn) {
	defer client.Close()
	buf := make([]byte, 1<<20)
	n, err := client.Read(buf)
	if err != nil {
		log.Println(err)
		return
	}
	// 解析http报文
	req, err := http.ReadRequest(bufio.NewReader(bytes.NewReader(buf[:n])))
	if err != nil {
		log.Println(err)
		return
	}
	// http-proxy 的认证
	if username, pwd, ok := ProxyBasicAuth(req); !ok || username != "test" || pwd != "123" {
		client.Write([]byte("HTTP/1.1 407 Proxy Authentication Required\r\nProxy-Authenticate: Basic\r\n\r\n"))
	}
	var address string
	// 如果http头的host是默认80，那么就需要手动加上80端口，因为建立TCP连接需要用到。
	if strings.Index(req.Host, ":") == -1 {
		address = req.Host + ":80"
	} else {
		address = req.Host
	}
	server, err := net.Dial("tcp", address)
	if err != nil {
		log.Println(err)
		return
	}
	// 处理 CONNECT 请求，为后续 https 建立连接。
	if req.Method == "CONNECT" {
		fmt.Fprint(client, "HTTP/1.1 200 Connection established\r\n\r\n")
	} else {
		server.Write(buf[:n])
	}
	go io.Copy(server, client)
	io.Copy(client, server)
}

func ProxyBasicAuth(r *http.Request) (username, password string, ok bool) {
	auth := r.Header.Get("Proxy-Authorization")
	if auth == "" {
		return
	}
	return parseProxyBasicAuth(auth)
}

func parseProxyBasicAuth(auth string) (username, password string, ok bool) {
	const prefix = "Basic "
	if !strings.HasPrefix(auth, prefix) {
		return
	}
	c, err := base64.StdEncoding.DecodeString(auth[len(prefix):])
	if err != nil {
		return
	}
	cs := string(c)
	s := strings.IndexByte(cs, ':')
	if s < 0 {
		return
	}
	return cs[:s], cs[s+1:], true
}
```

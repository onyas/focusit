---
title: Golang模拟用户登录
date: 2019-08-05T21:12:20+08:00
categories:
  - ARTS
tags: 
  - tip 
  - golang
---

最近开始学习Go语言，想要拿go来做点东西，首先想到的就是把《极客时间》买的专栏下载下来，这样方便自已搜索并且统一管理。

有了想法之后就开始动手实战，遇到的第一个问题就是怎么模拟用户的登录。<!--more-->有好几种方法可以做到，比如在代码中调用登录的接口，或者调用API的时候带着Cookie。这两种方法都需要我们在代码中配置用户名密码或者Cookie之类的，比较麻烦。

如果用户已经在浏览器中登录了，能不能直接拿着浏览器中的Cookie去登录呢？调研了一翻发现可行。以下是代码，供大家参考。


```golang
package main

import (
	"fmt"
	"io/ioutil"
	"net/http"
	"strings"

	"github.com/onyas/go-browsercookie"
)

func main() {
	var cookieJar, _ = browsercookie.Chrome("https://account.geekbang.org")
	var client = &http.Client{Jar: cookieJar}

	request, _ := http.NewRequest("POST", "https://time.geekbang.org/serv/v1/my/columns", strings.NewReader("{}"))

	request.Header.Set("Origin", "https://time.geekbang.org")
	request.Header.Set("Referer", "https://time.geekbang.org")
	request.Header.Set("User-Agent", "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/59.0.3071.115 Safari/537.36")
	request.Header.Set("X-Real-IP", "211.161.244.200")
	request.Header.Set("Connection", "keep-alive")
	request.Header.Set("Content-Type", "application/json")
	request.Header.Add("Content-Length", "2")

	res, _ := client.Do(request)
	defer res.Body.Close()

	body, _ := ioutil.ReadAll(res.Body)
	fmt.Println(string(body))
}
```

首先要保证在Chrome里面登录极客时间，然后执行go run main.go, 会要求输入电脑的密码，用于取cookie用。然后就输出你已购买的课程。这里主要用到了一个go-browsercookie工具类，大家可以到github上面查看具体的用法[go-browsercookie](https://github.com/onyas/go-browsercookie)
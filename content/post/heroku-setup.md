---
title: heroku部署指南
date: 2018-10-02T23:39:20+08:00
categories:
  - Python
tags: 
  - heroku
---

heroku提供的免费网站可以用于构建个人网站，非常适用于实验性质或是简单的博客系统。但是网上的文档大多数是拷贝的，没有一个完整或者靠谱的文章，本文会详细说明自已在搭建的过程中遇到的各种问题及解决办法。

## 1. 注册帐号

建议用gmail的邮箱去https://www.heroku.com/官网注册，非常简单，不多说了

## 2. 安装CLI

去https://devcenter.heroku.com/articles/heroku-cli#download-and-install 下载安装，后续很多操作都会用到

## 3. 登录heroku

安装完CLI后，需要登录，直接执行heroku login，按照提示输入注册的邮箱与密码即可

## 4. 准备自已的应用

因为部署到heroku是需要跟git相结合的，所以你需要在github上面建一个repo来存放你的代码。如果你还没有准备好，可以先用我做好的拿来当demo, 在当前目录执行 git clone https://github.com/onyas/vip-url-resolve.git

## 5. 用heroku创建用应用

1. cd vip-url-resolveheroku create #appname要求唯一，所以如果重复了会提示创建不成功

2. heroku buildpacks:set heroku/python #显示指定用python来构建打包，这个非常重要，没有这句会报以下错误

```
remote: -----> App not compatible with buildpack: https://buildpack-registry.s3.amazonaws.com/buildpacks/heroku/python.tgz

remote:        More info: https://devcenter.heroku.com/articles/buildpacks#detection-failure

```

3. git push heroku master # 把当前master分支push到远程的heroku分支，如果没有错误，恭喜你马上就成功了

4. heroku open # 这样会在浏览器打开 appname.herokuapp.com，例如我自已建立的就是https://vip-url-resolve.herokuapp.com/

5. heroku logs --tail #如果你能成功打开自已构建的内容，那这条命令就不需要。如果打开的内容跟你期望的不一致，那这条命令就可以帮你排查问题。

我在部署中就遇到了打不开网站的情况，错误日志如下，经排查发现代码中打开端口的时候写死了6789，所以导致报错，heroku推荐用$port的方式绑定端口，而不是写死某一个端口，具体的可以参考代码 https://github.com/onyas/vip-url-resolve，希望以上能帮助你，有疑问欢迎在留言区探讨

```
2018-10-02T08:20:33.496088+00:00 heroku[web.1]: Error R10 (Boot timeout) -> Web process failed to bind to $PORT within 60 seconds of launch 2018-10-02T08:20:33.496277+00:00 heroku[web.1]: Stopping process with SIGKILL2018-10-02T08:20:33.596242+00:00 heroku[web.1]: State changed from starting to crashed2018-10-02T08:20:33.598298+00:00 heroku[web.1]: State changed from crashed to starting2018-10-02T08:20:33.582793+00:00 heroku[web.1]: Process exited with status 1372018-10-02T08:20:37.623592+00:00 heroku[web.1]: Starting process with command `python restAPI.py`
```



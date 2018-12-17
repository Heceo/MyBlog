---
title: "Problems Met When Using Go"
#date: 2018-12-17T16:48:22+08:00
categories : ["dots"]
tags : ["Go"]
draft: false
comments: true
showpagemeta : true 
showcomments : true
slug: ""
description: ""
---

It was extremely annoying when you install Go and set up the environment in mainland of China due to the firewall.

After I installed most of Go extensions in the VSCode, there were two ones left which I couldn't deal with no matter what methods I used. I even didn't know if it's because of the firewall or the problem of Golang itself. So I tried with glide, but it didn't work. "glide up" or "glide install" just left blank in the terminal even if I set proxy for the terminal. Too strange wasn't it? And VSCode neither corresponded nothing after I changed its  proxy. Just as I was forlorn, I found a new tool by Golang -- Go Module.

First, enter your new project outside $GOPATH, and if your Go's version is 1.11:

```bash
set GO111MODULE=on
```

Not sure what's like on versions above 1.11

```bash
go mod init github.com/stamblerre/gocode
```

```bash
go mod init github.com/ianthehat/godef
```

Then in your $GOPATH, you will find two executable programs named like "gocode-gomod.exe", rename them and put them in the /bin, it should work.

What's interesting is, when I was searching on the Internet, I found more useful information by Baidu instead of Google, which seldom happened before.

# Reference

[Visual Studio Code搭建Go环境](https://blog.csdn.net/wyy626562203/article/details/83833592)

[Go1.1.1新功能module的介绍及使用](https://blog.csdn.net/benben_2015/article/details/82227338)

[Go Module](https://github.com/golang/go/wiki/Modules)

[Glide](https://github.com/Masterminds/glide)


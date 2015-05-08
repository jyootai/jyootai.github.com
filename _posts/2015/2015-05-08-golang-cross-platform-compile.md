---
layout: post
title: Golang 跨平台交叉编译 
date: 2015-05-08
author: jyootai
disqus: y
share: y
categories: blog
---

#### **什么是跨平台交叉编译**
解释什么是交叉编译之前，先要明白一个概念：本地编译。

本地编译就是在当前的操作系统下，x86的CPU下，直接编译出来程序，可以运行的程序（或者库文件），其可以直接在当前的环境，即x86的CPU下，当前电脑中，运行。

交叉编译简单的说就是可以编译出在其它操作系统下运行的程序，比如在Linux下编译后在Windows系统中运行，或者说在32位下编译在64位下运行。

### **Golang 的跨平台交叉编译**

Go语言是编译型语言，可以将程序编译后在将其拿到其它操作系统中运行，此过程只需要在编译时增加对其它系统的支持。

1,首先进入`$GOROOT/go/src` 源码所在目录，执行如下命令创建目标平台所需的包和工具文件:

{% highlight bash %}
# 如果你想在Windows 32位系统下运行
$ cd $GOROOT/src
$ CGO_ENABLED=0 GOOS=windows GOARCH=386 ./make.bash
# 如果你想在Windows 64位系统下运行
$ cd $GOROOT/src
$ CGO_ENABLED=0 GOOS=windows GOARCH=amd64 ./make.bash

# 如果你想在Linux 32位系统下运行
$ cd $GOROOT/src
$ CGO_ENABLED=0 GOOS=linux GOARCH=386 ./make.bash
# 如果你想在Linux 64位系统下运行
$ cd $GOROOT/src
$ CGO_ENABLED=0 GOOS=linux GOARCH=amd64 ./make.bash
{% endhighlight%}

2,执行完上面的操作后，现在可以编译将要在目标操作系统上运行的程序了:
{% highlight bash %}
# 如果你想在Windows 32位系统下运行
$ CGO_ENABLED=0 GOOS=windows GOARCH=386 go build test.go
# 如果你想在Windows 64位系统下运行
$ CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build test.go

# 如果你想在Linux 32位系统下运行
$ CGO_ENABLED=0 GOOS=linux GOARCH=386 go build test.go
# 如果你想在Linux 64位系统下运行
$ CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build test.go
{% endhighlight%}

现在你可以在相关目标操作系统上运行编译后的程序了。

**注意**：

* 该方式暂时不支持 CGO。
* 这里并不是重新编译Go，因为安装Go的时候，只是编译了本地系统需要的东西，而需要跨平台交叉编译，需要在Go中增加对其他平台的支持，所以会有 ./make.bash 这么一个过程。

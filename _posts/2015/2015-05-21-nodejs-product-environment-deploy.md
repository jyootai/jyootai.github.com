---
layout: post
title: NodeJS 生产环境部署记 
date: 2015-05-21
author: jyootai
disqus: y
share: y
categories: blog
---

最近用NodeJS写了个跨站登录学校的教务系统，抓取其数据并显示的应用 [Cdquery](https://github.com/jyootai/cdquery)，现在发现没有什么实际的用处，不过反正无聊就当练手:)。

因为之前没有部署过Node项目，所以这次打算把刚写的Cdquery拿来部署一下，走走过程，踩踩坑。果不其然，过程中还是遇到了一些坑，下面就将过程现场还原。

### **Nginx的简单配置**

我是用的 Nginx 来作为服务器，反向代理本地Node应用。这话听起来太绕，没关系，拿具体例子来说明，上代码！

{% highlight vim %}
# /etc/nginx/sites-enabled/default
......
server {
   listen   80;    # -> 1
   server_name IP/Domain name;   # -> 2
   location / {
     proxy_pass http://127.0.0.1:3000;  # -> 3
     ......
   }
}
......
{% endhighlight%}

当在VPS上安装Nginx后，只需要像上面那样更改`/etc/nginx/sites-enabled/default`这个配置文件即可，其余的可以不管，除非你要其它的要求。下面说说上面的配置:

1 处是服务器监听80端口，有过web基础的应该都知道什么用处不多说。

2 处表示你服务使用什么网络地址可以访问。你可以填本地的IP地址，这样你就直接可以通过IP地址访问到的应用；你也可以使用购买的域名，但你得将域名指向你服务器的IP地址，也就是当前正在配置的服务器地址。

3 处就是反向代理的具体体现了，你可以通过执行 `node app.js` 运行你的程序，而你的程序中有`app.listen(3000)`这样的代码，表示在3000端口打开程序，那么你可以在浏览器打开`http://127.0.0.1:3000` 访问你的应用，但此时，只能你一个人完，其他人根本没法和你一起完，多没劲啊！而 3 处的意思就是当别人访问你服务器的IP地址或者指向后的域名时，Nginx就会代理到3000端口，这样别人就可以和你一起玩了。

以上就是Nginx的简单配置，对于运行一个简单的Node项目也足够了。然后重启 Nginx，执行 `sudo service nginx restart`。

### **运行Node的工具**

#### **Node自带运行方式**
首先要运行应用，最基本的 Node 你得安装吧，不过既然你已经走到部署这一步了，那么安装 Node 肯定也不是问题，这里就不在啰嗦了。

这里我说说在生产环境下用什么来运行程序，当然可以直接用自带的node命令`node app.js`，这个当然可以，我第一次部署时就直接这么做的。但是之后问题就出来了，我让班上同学帮忙测试时，没过多久他们就没法访问了，服务器直接挂掉。因为使用 Node 自带的执行方式会有当程序中BUG时，程序就会立即停止，这样服务器也会跟着立即挂掉，所以在生产环境下绝对不能用这种方式。

#### **supervisor**
当然还可以使用 supervisor,我在开发环境下就是使用 supervisor ，这个工具很方便，当项目中文件有变化时它会自动检测到变化然后重启，这样你再也不用修改代码后自己去手动重启服务，因为Node自带的命令是不会检测文件变化而重启的，这一点还真不如 Rails。但这个工具同 Node 自带的运行方式存在一样的问题，当出现BUG时，最后同样会停止程序的运行，服务器也会挂掉，所以这个工具也不能在生产环境中是哦也能够。关于 supervisor 想了解更多可以猛戳[这里](https://github.com/isaacs/node-supervisor)。


#### **Forever**
我最后是选择了 [forever](https://github.com/foreverjs/forever) ,这个工具会创建一个守护进程，一直监听程序的运行，当出现BUG导致程序停止时，这个工具又会自动重新启动程序，并且还可以输出各种日志到指定日志文件中。关于更多用处可以直接到forever项目主页上去了解，这里就不说了，我就说我怎样使用的以及遇到的坑：

首先，我最开始使用最简单的方式：

{% highlight bash %}
$ forever start app.js
warn:    --minUptime not set. Defaulting to: 1000ms
warn:    --spinSleepTime not set. Your script will exit if it does not stay up for at least 1000ms
info:    Forever processing file: www/cdquery/app.js
$ 
{%endhighlight%}

上面虽然有两个警告，但无关紧要，这样确实能正常执行，但不能得到程序执行过程中的信息，如果程序有问题，无法看到错误信息，对程序的改进不友好。

当然forever够强大，能满足这样的要求，还能够像 supervisor 一样检测文件的改变并重启动，看了forever的文档于是写了一个脚本，以便以后方便执行多命令操作：

{% highlight bash %}
$ vim server.sh
#!/bin/sh

export LOG=/home/jyootai/log/nodejs
export APP_PATH=/home/jyootai/www/cdquery
export APP=$APP_PATH/app.js

forever -p $APP_PATH -l $LOG/access.log -e $LOG/error.log -o $LOG/out.log  -aw start $APP
{% endhighlight%}

上面的脚本 `-p` 表示项目的路劲，`-l` 表示输出日志到access.log，`-e` 表示输出控制台错误到error.log，`-o` 表示输出控制台信息到out.log，`-aw`表示追加日志和检测文件改动。保存脚本文件并修改为可执行文件，然后运行：

{% highlight bash %}
$ chmod +x server.sh
$ ./server.sh 
warn:    --minUptime not set. Defaulting to: 1000ms
warn:    --spinSleepTime not set. Your script will exit if it does not stay up for at least 1000ms
info:    Forever processing file: /home/jyootai/www/cdquery/app.js
{% endhighlight%}

和上面一样，显示一样的信息。但是检测运行的进程，发现程序并没有执行，forever也没有子进程。于是打开日志文件查看：

{% highlight bash %}
$ vim /home/jyootai/log/nodejs/access.log
error: Could not read .foreverignore file.
error: ENOENT, open '/.foreverignore'
/home/jyootai/.nvm/versions/node/v0.12.3/lib/node_modules/forever/node_modules/forever-monitor/node_modules/watch/main.js:73
    if (err) throw err;
                       ^
Error: EACCES, scandir '/lost+found'
   at Error (native)
{% endhighlight%}

出现了上面日志文件中的错，于是Google，第一个错是没有在项目文件下添加`.foreverignore`, 这个文件告诉 forever 忽略检测哪些文件的改动，类似与 git 工具中的 .gitignore 文件；对于第二个错，是因为添加`-w`选项时必须也要添加`--watchDirectory`选项，必须要告知检测变动文件的目录，这也算是一个坑吧，因为 forever 文档中没有这个特别的说明。最后根据错误原因添加缺少的的内容，最后的脚本文件如下：

{% highlight bash %}
#!/bin/sh

export LOG=/home/jyootai/log/nodejs
export APP_PATH=/home/jyootai/www/cdquery
export APP=$APP_PATH/app.js

forever -p $APP_PATH -l $LOG/access.log -e $LOG/error.log -o $LOG/out.log  --watchDirectory $APP_PATH  -aw start $APP
{% endhighlight%}

打开配置的IP或者域名，项目正常展现！

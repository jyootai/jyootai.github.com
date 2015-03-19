---
layout: post
title:  关于 Bundle Install 和 Bundle Update
date:   2015-03-19 19:23:45
author: jyootai
disqus: y
categories: blog
---

这两个命令在 Rails 中也算是常客了，特别是 bundle install。以前也没有怎么注意这两个密令的差别，知其然但不知其所以然，上次项目中使用了这两个命令，因为版本的原因遇到了不少问题，于是就索性看看这个命令究竟有什么不同。废话少说，直入正题。

**Bundle Install**

这个命令就是安装 Gemgfile 文件中的 gems，在第一次执行时会比较慢，因为要解决各种依赖问题（心急是吃不了热豆腐的）。当第一次执行完后，会生成一个 Gemfile.lock 这个文件(后面直接称其为 lock 文件)，这个文件就是实际安装的 gem，包括安装的版本都有注明，文件 lock 里面的 gems 比文件 Gemfile 的多，因为还有很多依赖的 gems。

当以后执行 bundle install 时，bundle install 以 lock 文件为优先，为本地系统安装 lock 文件中指定的版本，如果 Gemfile 中有而 Gemfile.lock 中没有的，则会安装，然后更新 lock 文件。这样有了 lock 文件，在不同的开发环境也会安装相同版本的 gem，不会出错。

**Bundle Update**

bundle update 会去检查 Gemfile 里 gem 的更新，然后对比 lock 文件，如果 Gemfile 里没有指定版本或是指定是 `>=` 的版本，那有新版本就会去安装新的版本的 gem，然后更新 lock 文件。

**说明及注意**

>
* 安装的gems默认会到$GEM_HOME下，你可以输入 `echo $GEM_HOME` 查看安装目录
* 删了 lock文件后, bundle install 会先从本地已经安装的 gem 找本地最新版本安装，即使本地的版本落后与source的版本，bundle install也只会安装本地最新的版本而不会安装source源上最新的版本，只有 bundle update 才会更新source源上最新的
* 在本地执行过gem install gem_name之后, bundle install 会复用这个安装的gem, 不会重新安装.

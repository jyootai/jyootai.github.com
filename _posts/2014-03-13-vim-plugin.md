---
layout: post
title: slim在vim下语法高亮显示
date: 2014-03-13 19:12:23 
disqus: y
share: y
---

### vim-slim

这是vim的一个插件，支持slim在vim下高亮显示,网上安装方法很多，这里介绍我自己安装成功的两种方法。

---

### 使用pathogen安装
pathogen是vim的一个插件管理工具，vim的插件都是装在~/.vim目录下的，当装的时间一久插件过多，这个目录就会变得很臃肿，插件也很散乱。而pathogen使每个插件都有一个单独的目录，pathogen在~/.vim里建立一个bundle目录，然后把所有插件放在`~/.vim/bundle/插件名`下面插件的安装过程与没有 pathogen 时类似，但从安装结束开始，一切的插件管理过程都能得到简化。要删除某个插件时把插件的目录删了就行了。

#### 1.安装pathogen(如果你已经安装了pathogen请直接跳到第3步)

        mkdir -p ~/.vim/autoload ~/.vim/bundle; \
        url -so ~/.vim/autoload/pathogen.vim \
        https://raw.github.com/tpope/vim-pathogen/master/autoload/pathogen.vim


#### 2.在`~/.vimrc`这个文件里写入这些内容：

    {% highlight vim %}
    call pathogen#infect()  

    syntax enable
    filetype plugin indent on
    {% endhighlight %}

#### 3.安装slim-vim    

        pushd ~/.vim/bundle; \
        git clone git://github.com/slim-template/vim-slim.git;\
        popd

---

### 使用vundle安装(推荐这种)
  vundle也是vim的一个插件管理工具，它使用类似 Ruby Bundler 的方式来管理插件，只需要在~/.vimrc里面用Bundle声明插件，然后在vim里面用`:BundleInstall` 安装所有插件。

#### 1.安装vundle到`~/.vim/bundle`

        mkdir -p ~/.vim/bundle; pushd ~/.vim/bundle; \
        git clone https://github.com/gmarik/Vundle.vim.git ~/.vim/bundle/vundle  
        popd


#### 2.在`~/.vimrc`中写入下面的内容：

    {% highlight vim %}
    set nocompatible 
    filetype off

    set rtp+=~/.vim/bundle/vundle/
    call vundle#rc()
    Bundle 'slim-template/vim-slim.git'
    syntax enable
    filetype plugin indent on
    {% endhighlight %}

#### 3.打开vim窗口运行`:BundleInstall`,
或者直接`vim +BundleInstall+qall`

其它常用命令：

*   更新插件`:BundleUpdate`
*   清除不再使用的插件`:BundleClean`
*   列出所有插件`:BundleList`
*   查找插件`BundleSearch`

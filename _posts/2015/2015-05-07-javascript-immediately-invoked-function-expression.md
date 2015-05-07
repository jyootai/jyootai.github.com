---
layout: post
title: 关于Javascript中立即执行函数表达式
date: 2015-05-07
author: jyootai
disqus: y
share: y
categories: blog
---


先说说什么是“立即执行函数表达式”，顾名思义，就是在定义后就立即执行的函数表达式。常见的有 `(function (){}())` 和 `(function (){})()` （注意括号的范围,眼睛别看花了o(∩∩)o...）这两种写法。

其实很多同学都在想能不能不加外部的括号直接想 `function (){}()` 这样写呢？其实我之前也有这个疑问，到底又有什么区别，一直没有弄清楚，于是抱着刨根问底的态度进行了一番深入的了解。在解答这个问题之前，我们先来巩固一下 JS 定义函数的两种方式。

---

#### **函数声明、函数表达式**

**函数声明**: 使用 `function` 关键字在指明一个函数名的形式，形如 `function funcName(){}`。

**函数表达式**：使用 `function` 关键字声明一个函数在赋予一个变量的形式，形如 `var func = function (){}`，当然也可以这样写 `var func = function funcName(){}`。

函数声明和函数表达式不同之处在于，一、Javascript引擎在解析javascript代码时会‘函数声明提升’（Function declaration Hoisting）当前执行环境（作用域）上的函数声明，而函数表达式必须等到Javascirtp引擎执行到它所在行时，才会从上而下一行一行地解析函数表达式，二、函数表达式后面可以加括号立即调用该函数，函数声明不可以，只能以funcName()形式调用.下面看一个例子：

{% highlight javascript %}
//这里能正常输出
funcName();
function funcName(){
  console.log("balabalabala");
}

//这里就会报错，因为此时变量 funcName 还并没有得到函数的引用
funcName();
var funcName = function(){
  console.log("balabalabala");
}

//正常运行，变量 funcName 得到函数引用
var funcName = function(){
  console.log("balabalabala");
}
funcName();
{% endhighlight%}

#### **深入"立即执行函数"**

那到底能不能使用 `function (){函数体}()` 这种形式？我们在下面的例子中来说明：
{% highlight javascript %}
//报错，这里js的“语句优先”在作怪，即{}被理解成复合语句块而不是函数的一部分，
//自然前面的function()声明函数的语法不完整导致语法分析期出错
function (){
  console.log("balabalabala");
}()

//报错，()为分组操作符，里面不能为空
function funcName(){
  console.log("balabalabala");
}()

//这里不报错但不会调用，因为()在这里并不是立即调用函数的意思
function funcName(){                
  console.log("balabalabala");         
}("bala")
//上面和下面的相同
function funcName(){                
  console.log("balabalabala");         
}

("bala")

{% endhighlight%}

在一个表达式后面加上括号()，该表达式会立即执行，但是在一个语句后面加上括号()，它就是一个分组操作符。

`(function (){ 函数体}())` 和 `(function (){ 函数体})()` 这两种写法都使用的括号(),目的是将语句转变为表达式，最后在加一个()使之成为“立即执行函数”。

在function前面加一元操作符号,都将函数声明转换成函数表达式,所以同加()的效果一样：
{% highlight javascript %}
(function (arg){
   console.log(arg);   //输出bala,使用()运算符
})("bala");
     
(function(arg){
   console.log(arg);   //输出lala，使用()运算符
}("lala"));
         
!function(arg){
   console.log(arg);   //输出bala,使用！运算符
}("bala");
             
+function(arg){
    console.log(arg);   //输出lala,使用+运算符
}("lala");
             
-function(arg){
    console.log(arg);   //输出byebye,使用-运算符
}("byebye");
{% endhighlight%}












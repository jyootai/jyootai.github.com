---
layout: post
title: AngularJS 中 $http.success() 与 $http.success() 方法
date: 2015-10-13
author: jyootai
disqus: y
share: y
categories: blog
---

$http 是 Angular 中一个核心的服务，主要作用是与远程服务进行数据的传输，比如一个 ajax 服务请求的实现。关于 $http 想了解更多的可以不停的点击[这里](https://docs.angularjs.org/api/ng/service/$http)。

$http 服务中比较常用的是`$http.then()`与`$http.success()`方法， 比如我们想从远程服务获取一些数据并打印出来我们可以这样写：

{% highlight js %}
$http.post('jyootai.com/api', params).success(function(response) {
    console.log(response);// 获得服务相应后执行
});
{% endhighlight%}

当然，我们也可以这样写：

{% highlight js %}
$http.post('jyootai.com/api', params).then(function(response) {
    console.log(response);// 获得服务相应后执行
});
{% endhighlight%}

二者不同就是上面使用的`success()`,下面使用的是`then()`。两种方法都能实现我们的需求，但是这两种方法是否有不同？答案是肯定的。

下面我们来看看`success()`方法的核心部分，如果你想看 Angular 的源码可以点击[这里](https://github.com/angular/angular.js/blob/master/src/ng/http.js#L974):

{% highlight js %}
promise.success = function(fn) {
    // ...
    promise.then(function(response) {
        fn(response.data, response.status, response.headers, config);
    });
    return promise;
};
{% endhighlight%}

从上面可以看出`success()`方法其实就是和`then()`方法差不多，只是在方法内部调用了`then()`方法，并且解构了 then 方法传递给回调方法的参数，也就是说使用`success()`方法得到的 response 比`then()`精简，少一些数据，这是一个语法糖。

注意，还有一个重要的区别，就是使用`success()`方法无法使外部获取到最后回调函数中返回的数据：

{% highlight js %}
var getData = function () {
    return $http.get('jyootai.com/api', {params: params}).success(function (res) {
        return res.data;
    });
};
console.log(getData()); // 这里是无法打印出来数据的
{% endhighlight%}

上面的数据是无法打印出来，因为数据并没有直接返回出来，这是`success()`内部实现机制造成的，如果想要打印出返回的数据这时就要使用`then()`方法。

为了避免造成不可遇见bug，还是建议在项目中都使用`then()`方法，保持整个项目代码风格的一致性。
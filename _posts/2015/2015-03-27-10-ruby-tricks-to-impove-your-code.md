---
layout: post
title: 10个Ruby技巧提升你的代码 
date: 2015-03-27
author: jyootai
disqus: y
share: y
categories: blog
---

本文向你展示10个Ruby技巧（或许这些你都知道），但不管怎样，这些都是值得分享的东西，并且只需要一点时间就能浏览完。

---

#### **1.构造Hash**

你可以用一个列表一系列的值构造Hash 通过`Hash[...]`方法，它将会像下面的方式创建一个Hash：

{% highlight ruby %}
Hash['key1', 'value1', 'key2', 'value2']
# => {"key1" => "value1", "key2" => "value"}
{% endhighlight %}

#### **2.Lambda字面量`->`**

定义一个lambda可以使用在Rails中用的比较流行的`->`字面量，它可以很容易的定义一个lambda对象：

{% highlight ruby %}
a = -> {1 + 1}
a.call
# => 2

a = ->(arg) {arg + 1} #含参
a.call(2) #传递参数
# => 3
{% endhighlight %}

#### **3.双星** **

双星在Ruby中算是一个小魔法，来看看下面的方法：

{% highlight ruby %}
def my_method(a, *b, **c)

end
{% endhighlight %}

`a`是一个常规的参数，`b`将会得到第一个参数后面的所有参数并把他们放入一个数组中，`**c`将会得到传人方法的任何格式为 `key: value` 的参数。

来看看下面的例子：
一个参数

{% highlight ruby %}
my_method(1)
# => [1, [], {}]
{% endhighlight %}

超过一个参数
{% highlight ruby %}
my_method(1, 2, 3, 4)
# => [1, [2, 3, 4], {}]
{% endhighlight %}

超过一个参数 + hash-style 参数

{% highlight ruby %}
my_method(1, 2, 3, 4, a: 1, b: 2)
# => [1, [2, 3, 4], {:a => 1, :b => 2}]
{% endhighlight %}

#### **4.操作单个对象和数组用同样的方式**

有时你可能会先给出一个选项去接受单个对象或者一个数组对象，而不是最后去检查你接受到对象的类型，这时你可以用`[*type]`或者`Array(type)`

我们先设置两个变量，第一个是单对象，第二个是数字数组：

{% highlight ruby %}
stuff = 1
stuff_arr = [1, 2, 3]
{% endhighlight %}

在下面的例子，通过任何被给定的对象用`[*...]`去循环：

{% highlight ruby %}
[*stuff].each { |s| s }  # => [1]
[*stuff_arr].each { |s| s }  # => [1, 2, 3]
{% endhighlight %}

和上面效果一样，这次用`Array(...)`：

{% highlight ruby %}
Array(stuff).each { |s| s }  # => [1]
Array(stuff_arr).each { |s| s }  # => [1, 2, 3]
{% endhighlight %}

#### **5.`||=` 操作符号**

利用`||=`操作符能写出很简洁的代码

它实际上和下面的表示方式相同：

{% highlight ruby %}
a || a = b # 正确
{% endhighlight %}

大多数人都认为它和下面的表示方式一样，但实际不是：

{% highlight ruby %}
a = a || b # 错误
{% endhighlight %}

因为如果a已经存在，第二种方法的表示就是再给a进行分配赋值，但这样其实是没有意义的。

#### **6.强制的哈希参数**

这个是在Ruby 2.0被引进的，你可以在定义方法时指定接受的参数为hash类型，像这样：

{% highlight ruby %}
def my_method({})
end
{% endhighlight %}

你可以指定想要接受值的键，或者为键指定一个默认的值。下面的a和b是指定要接受值的键：

{% highlight ruby %}
def my_method(a:, b:, c: 'default')
  return a, b, c
end
{% endhighlight %}

我们不给b传值来调用它，这时会报错：

{% highlight ruby %}
my_method(a: 1)
# => ArgumentError: missing keyword: b
{% endhighlight %}

由于c有一个默认的值，我们可以仅仅只给a和b传值：

{% highlight ruby %}
my_method(a: 1, b: 2)
# => [1, 2, "default"]
{% endhighlight %}

或者是给所有键传值：

{% highlight ruby %}
my_method(a: 1, b: 2, c: 3)
# => [1, 2, 3]
{% endhighlight %}

我们大多时候的做法是直接传入一个hash，这样的做法只是看起来更明显，就像下面的方式：

{% highlight ruby %}
hash = { a: 1, b: 2, c: 3 }
my_method(hash)
# => [1, 2, 3]
{% endhighlight %}

#### **7.生成字符或者字符数组**

你可能想要生成一个数字列表或者把所有的字符放入一个数组，你可以用Ruby的ranges去做这样的事。

A到Z：

{% highlight ruby %}
('a'..'z').to_a
# => ["a", "b", "c", "d", "e", "f", "g", "h", "i", "j", "k", "l", "m", "n", "o", "p", "q", "r", "s", "t", "u", "v", "w", "x", "y", "z"]
{% endhighlight %}

1到10：

{% highlight ruby %}
(1..10).to_a
# => [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
{% endhighlight %}

#### **8.Tap**

`tap`是一个很好的小方法去提高代码的可读性，我们拿下面的类作为例子：

{% highlight ruby %}
class User
  attr_accessor :a, :b, :c
end
{% endhighlight %}

现在去实例化一个对象并为对象的属性去赋值，你可能会这么做：

{% highlight ruby %}
def my_method
  o = User.new
  o.a = 1
  o.b = 2
  o.c = 3
  o
end
{% endhighlight %}

或者你可以用`tap`使用下面的方式去做：

{% highlight ruby %}
def my_method
  User.new.tap do |o|
    o.a = 1
    o.b = 2
    o.c = 3
  end
end
{% endhighlight %}

基本上，`tap`方法引入被调用的对象到代码块中并最后返回它。

#### **9.Hash默认的值**

默认的，当我们试图接受一个在hash中未定义的值，你将会得到nil。但你可以改变这个结果在进行初始化时。
（提示：一般不要这样做除非你知道自己在干什么）

在第一个例子中，我们定义了一个默认的值为0，所以当我们`a[:a]`时会得到0而不是nil：

{% highlight ruby %}
a = Hash.new(0)
a[:a]
# => 0
{% endhighlight %}

我们能够传入任何形式类型到Hash初始函数中：

{% highlight ruby %}
a = Hash.new({})
a[:a]
# => {}
{% endhighlight %}

或者是一个字符串：

{% highlight ruby %}
a = Hash.new('lolcat')
a[:a]
# => "lolcat"
{% endhighlight %}

#### **10.here 文档**

here文档有一个缺点就是它会影响代码流畅和缩进问题，由于`HERE`会缩进两格，但有时为了最后内容连续性，你可能会把每行内容都靠左写，像这样：

{% highlight ruby %}
def my_method
  <<-HERE
Ruby
stricks
Interesting
Right
  HERE
end
{% endhighlight %}

这有一个技巧可以避免它，通过用`gsub`方法加一个正则表达式。你可以自动的去除前面的空格，这样你就能保持缩进。

{% highlight ruby %}
def my_method
  <<-HERE.gsub(/^\s+/, '')
    Ruby
    stricks
    Interesting
    Right
  HERE
end
{% endhighlight %}


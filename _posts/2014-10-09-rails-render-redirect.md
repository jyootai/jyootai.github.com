---
layout: post
title: Rails 中的渲染与重定向
date: 2014-10-03 
disqus: y
share: y
---

Rails 应用程序的生命周期是按照每个请求独立的。当一个请求到来时，Rails 便开始处理，处理完了也就完了，再来一个请求时，又再开始处理。

渲染： 通过 render 方法实现，无论是渲染默认的或者指定的模板，或者是其他的局部模板,渲染都是处理一个请求的最后一步。

重定向： 通过 redirect_to 方法实现，终止当前的请求，再启动另外一个新的请求。

---

下面举例具体说明二者的不同:

先看一个处理表单的 create 方法。

	{% highlight ruby %}
	def create
    @topic = Topic.new topic_params
    if  @topic.save
      flash[:success] = "创建成功"
      redirect_to root_path
    else
      render 'new'
    end 
end
	{% endhighlight %}

如果保存成功，就会给予提示，然后 redirect_to 到一个新的请求上去。在上面的例子中，root 页面指的是 index 页面，对应的是 Topics#index 这个动作。

这里的逻辑是如果新建的 Topic 记录保存成功，下一步业务逻辑是把用户定向到上一级，那么是否可以直接渲染 Topics/index.html 这个模板呢 ? 例如：

{% highlight ruby %}
def create
  @topic = Topic.new topic_params
  if @topic.save
    flash[:success] = "创建成功"
    render :controller => "Topics", :action => "index" # ->这里改变了
  else
	……
	……
  end
end
{% endhighlight %}

上面的结果是 Topics/index.html 页面将会被渲染，但是这其中可能会存在一些你意想不到的陷阱，例如 Topics#index 方法是这样的：

{% highlight ruby %}
def index
  @topics = Topic.find(:all)    
end
{% endhighlight %}

如果在 Topics#create 后直接渲染了 index.html，那么 Topics#index 将不会执行，这样就导致 @topics 不会被初始化，这就意味着，在 index.html 中应用这个变量 @topics 时出现错误。

{% highlight erb %}
<p>All Topics<p>
<% @topics.each do |topic| %>
    ……
<% end %>
{% endhighlight %}

这就是为什么需要重定向到 Topics/index.html,而不是简单地渲染其模板。这也就是重定向的意义所在，创建一个新的请求，从头开始这个请求以确定需要被渲染的模板。



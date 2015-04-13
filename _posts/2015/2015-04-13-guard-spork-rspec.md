---
layout: post
title: Rspec+Guard+Spork 实现自动化测试 
date: 2015-04-13
author: jyootai
disqus: y
share: y
categories: blog
---

Rspec 是 rails 的一个测试框架。

Guard 的作用是监听项目中文件的更新，如果项目中的文件发生了改变，便会根据配置去启动相应的测试脚本。

Spork 通过它内部加载文件的机制来提升测试的时间。

---

### **1.添加 Rspec**

添加 Rspec 到项目中，只需要在 Gemfile 中加入对应的 gem：

{% highlight ruby%}
group :development, :test do
  gem 'rspec-rails'
end
{% endhighlight %}

执行`bundle install` ,然后在用如下命令初始化`spec/`文件夹：

{% highlight ruby%}
rails generate rspec:install
{% endhighlight %}

这时会生成如下相关文件：

{% highlight ruby %}
create  .rspec
create  spec
create  spec/spec_helper.rb
create  spec/rails_helper.rb
{% endhighlight %}

之后就可以直接用 `bundle exec rspec` 运行测试，但这样每次更新测试文件后都需要重新执行此命令，不太方便。

### **2.在项目中使用 Guard**

在`Gemfile`中添加 guard-rspec 这个 gem：
{% highlight ruby%}
group :development, :test do
  gem 'rspec-rails'
  gem 'guard-rspec' #这里
end
{% endhighlight %}

在 Guardfile 中添加对应 Rspec 的配置,执行如下命令：

{% highlight ruby%}
guard init rspec
{% endhighlight %}

最后执行`bundle exec guard`即可。

### **3.使用 Spork 加速测试**

首先在 Gemfile 中加入如下 Gem：

{% highlight ruby%}
group :development, :test do
  gem 'rspec-rails'
  gem 'guard-rspec' 
  gem 'guard-spork' #这里
  gem 'spork'       #这里     
  gem 'spork-rails' #这里
end
{% endhighlight %}

在 Guardfile 中添加 Spork 的配置，执行如下命令：

{% highlight ruby%}
guard init spork
{% endhighlight %}

此时还需要要对spork进行配置,需要spork加载的文件的机制可根据自己项目情况自行配置，如下是我自己的配置，首先改`spec_helper.rb`：

{% highlight ruby%}
# spec_helper.rb文件
require 'rubygems'
require 'spork'
require 'rspec/rails'
Spork.prefork do
  ENV['RAILS_ENV'] ||= 'test'
  require File.expand_path('../../config/environment', __FILE__)
  ActiveRecord::Migration.maintain_test_schema!
  RSpec.configure do |config|
    config.fixture_path = "#{::Rails.root}/spec/fixtures"
    config.use_transactional_fixtures = true
    config.infer_spec_type_from_file_location!
    config.expect_with :rspec do |expectations|
      expectations.include_chain_clauses_in_custom_matcher_descriptions = true
    end
    config.mock_with :rspec do |mocks|
      mocks.verify_partial_doubles = true
    end
  end
end
Spork.each_run do
end
{% endhighlight %}

然后修改`rails_helper.rb`文件：

{% highlight ruby%}
# rails_helper.rb文件
require 'spec_helper'
{% endhighlight %}

加入此行至`.rspec` ，让 RSpec 缺省使用 spork

{% highlight ruby%}
--drb
{% endhighlight %}

最后运行`bundle exec guard `,出现如下显示表示大功告成：

{% highlight ruby%}
jyootai@YT:~/project/my_test$ bundle exec guard
19:30:00 - INFO - Guard::RSpec is running
19:30:00 - INFO - Starting Spork for RSpec
Using RSpec, Rails
Preloading Rails environment
Loading Spork.prefork block...
Spork is ready and listening on 8989!
19:30:03 - INFO - Spork server for RSpec successfully started

19:30:03 - INFO - Guard is now watching at '/home/jyootai/project/my_test'
[1] guard(main)> 
{% endhighlight %}

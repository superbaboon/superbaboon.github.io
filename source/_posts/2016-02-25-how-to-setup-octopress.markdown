---
layout: post
title: "how-to-setup-octopress"
date: 2016-02-25 10:00:01 +0800
comments: true
categories: 
---


参考octopress官方[安装文档](http://octopress.org/docs/setup/)

### 名字解释

* rvm：ruby version manager
* gem：package manager for the Ruby programming language

### 系统环境

* OSX EI Capitan 10.11.1

### install rvm

```
curl -L https://get.rvm.io | bash -s stable --ruby

```

### install ruby

安装最新OSX EI Caption最新版本的ruby版本2.3.0

```
rvm install 2.3.0
```

出现下面的错误

```
Error running '__rvm_make install',
showing last 15 lines of /Users/junjiewu/.rvm/log/1456294025_ruby-2.3.0/install.log
    from ./tool/rbinstall.rb:686:in `block in <class:Installer>'
    from ./tool/rbinstall.rb:754:in `block (2 levels) in <main>'
    from /Users/junjiewu/.rvm/src/ruby-2.3.0/lib/rubygems/specification.rb:821:in `block in each_spec'
    from /Users/junjiewu/.rvm/src/ruby-2.3.0/lib/rubygems/specification.rb:743:in `block (2 levels) in each_gemspec'
    from /Users/junjiewu/.rvm/src/ruby-2.3.0/lib/rubygems/specification.rb:742:in `each'
    from /Users/junjiewu/.rvm/src/ruby-2.3.0/lib/rubygems/specification.rb:742:in `block in each_gemspec'
    from /Users/junjiewu/.rvm/src/ruby-2.3.0/lib/rubygems/specification.rb:741:in `each'
    from /Users/junjiewu/.rvm/src/ruby-2.3.0/lib/rubygems/specification.rb:741:in `each_gemspec'
    from /Users/junjiewu/.rvm/src/ruby-2.3.0/lib/rubygems/specification.rb:819:in `each_spec'
    from ./tool/rbinstall.rb:751:in `block in <main>'
    from ./tool/rbinstall.rb:801:in `block in <main>'
    from ./tool/rbinstall.rb:798:in `each'
    from ./tool/rbinstall.rb:798:in `<main>'
make: *** [do-install-nodoc] Error 1
```

解决方式，退回到安装2.2.1版本

```
rvm install 2.2.1
rvm use 2.2.1
rvm rubygems latest
```

顺利执行完毕，然后执行以下命令

```
gem install bundler
bundle install
```

执行bundle install各种类似以下的错误

```
An error occurred while installing sass-globbing (1.0.0), and Bundler cannot continue.
Make sure that `gem install sass-globbing -v '1.0.0'` succeeds before bundling.
```

解决的方式是不断的按照它的提示执行命令，然后不断的执行bundle install

最后执行

```
rake install
```

Done!



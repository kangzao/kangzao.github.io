---
layout: post
title: jekyll安装及配置
date: 2017-07-13
description: jekyll安装及配置
tag: jekyll
published: true
---

参考主页：http://jekyll.com.cn/


	gem install jekyll

ERROR:  Error installing jekyll:
liquid requires Ruby version >= 2.1.0.

查看ruby版本

{% highlight shell linenos %}
ruby —version
{% endhighlight %}

ruby 2.0.0p648 (2015-12-16 revision 53162) [universal.x86_64-darwin16]

升级ruby

使用RVM也就是Ruby Version Manager,Ruby版本管理器来升级ruby，RVM包含了Ruby的版本管理和Gem库管理(gemset)。

	curl -L get.rvm.io | bash -s stable

会在~/.profile文件中配置环境变量

Add RVM to PATH for scripting. Make sure this is the last PATH variable change.
export PATH="$PATH:$HOME/.rvm/bin"

[[ -s "$HOME/.rvm/scripts/rvm" ]] && source "$HOME/.rvm/scripts/rvm" # Load RVM into a shell 	session *as a function*

测试是否安装正常

	rvm -v
    
rvm 1.29.2 (latest) by Michal Papis, Piotr Kuczynski, Wayne E. Seguin [https://rvm.io/]

列出已知ruby的版本

	rvm list known

安装ruby2.4.1

	rvm install ruby-2.4.1

中间报错：

and make sure brew update works before continuing.\n'
Failed to update Homebrew, follow instructions here:
https://github.com/Homebrew/homebrew/wiki/Common-Issues

重新安装homebrew(官网：https://brew.sh/index_zh-cn.html)

	ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

Fatal: unable to access 'https://github.com/Homebrew/brew/': Failed to connect to github.com 	port 443: Operation timed out
Failed during: git fetch origin master:refs/remotes/origin/master --tags --force --depth=1

发现是由于绑定了host所致。。。解除绑定安装成功：

==> Cleaning up /Library/Caches/Homebrew...

==> Migrating /Library/Caches/Homebrew to /Users/Ian/Library/Caches/Homebrew...

==> Deleting /Library/Caches/Homebrew...

Already up-to-date.

==> Installation successful!

	rvm install ruby-2.4.1
    
Install of ruby-2.4.1 - #complete
Ruby was built without documentation, to build it run: rvm docs generate-ri

	gem install jekyll
    
Done installing documentation for public_suffix, addressable, colorator, sass, jekyll-sass-		converter, rb-fsevent, ffi, rb-inotify, listen, jekyll-watch, kramdown, liquid, mercenary, 		forwardable-extended, pathutil, rouge, safe_yaml, jekyll after 32 seconds 18 gems installed

	jekyll
    
/Users/Ian/.rvm/rubies/ruby-			 2.4.1/lib/ruby/site_ruby/2.4.0/rubygems/core_ext/kernel_require.rb:55:in require: cannot load such file -- bundler (LoadError)

	gem install bundler
    
one installing documentation for bundler after 4 seconds
1 gem installed


	jekyll
    
Could not find proper version of jekyll (3.1.1) in any of the sources
Run bundle install to install missing gems.

	bundle install
    
Bundle complete! 3 Gemfile dependencies, 17 gems now installed.
Use "bundle info [gemname]" to see where a bundled gem is installed.

	jekyll server
    
Configuration file: /Users/Ian/jekyll-blog/_config.yml
Dependency Error: Yikes! It looks like you don't have jekyll-sitemap or one of its dependencies installed. In order to use Jekyll as currently configured, you'll need to install this gem. The full error message from Ruby is: 'cannot load such file -- jekyll-sitemap' If you run into trouble, you can find helpful resources at http://jekyllrb.com/help/!
jekyll 3.1.1 | Error:  jekyll-sitemap

	bundle update jekyll
    
Using jekyll 3.5.0 (was 3.1.1)
Bundle updated!

	jekyll server
    
Configuration file: /Users/Ian/jekyll-blog/_config.yml
       Deprecation: The 'gems' configuration option has been renamed to 'plugins'. Please update your config file accordingly.
Dependency Error: Yikes! It looks like you don't have jekyll-sitemap or one of its dependencies installed. In order to use Jekyll as currently configured, you'll need to install this gem. The full error message from Ruby is: 'cannot load such file -- jekyll-sitemap' If you run into trouble, you can find helpful resources at https://jekyllrb.com/help/!

修改_config.yml，去掉jekyll-sitemap

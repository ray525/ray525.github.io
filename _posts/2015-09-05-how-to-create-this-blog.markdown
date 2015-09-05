---
layout: post
title:  "How to create this Blog"
date:   2015-09-05 21:07:07
categories: jekyll
---

## The final purpose
1. add jekyll folders and files on my git page [repository](https://github.com/ray525/ray525.github.io)
2. add markdown file in `_posts`, then your article will show on my [github page](http://ray525.github.io/)

## How to setup jekyll on windows
1. install ruby
2. install DevKit
3. setup mirror image
<<<<<<< HEAD:_posts/2015-09-05-how-to-create-this-blog.markdown
	{% highlight ruby %}
	$ gem sources --remove https://rubygems.org/
=======

    ```
    $ gem sources --remove https://rubygems.org/
>>>>>>> bccf982dff2a3a78c895897913e0ab6becf9cc0b:_posts/2015-08-24-how-to-create-this-blog.markdown
    $ gem sources -a https://ruby.taobao.org/
    $ gem sources -l
    *** CURRENT SOURCES ***
    
    https://ruby.taobao.org
    # 请确保只有 ruby.taobao.org
<<<<<<< HEAD:_posts/2015-09-05-how-to-create-this-blog.markdown
	{% endhighlight %}
=======
    ```
    
>>>>>>> bccf982dff2a3a78c895897913e0ab6becf9cc0b:_posts/2015-08-24-how-to-create-this-blog.markdown
4. install package using  `gem` 
	{% highlight ruby %}
	gem install bundler
    gem install github-pages
    gem install jekyll
    {% endhighlight %}
    
5. the i start myblog
<<<<<<< HEAD:_posts/2015-09-05-how-to-create-this-blog.markdown
	{% highlight ruby %}
	jekyll new myblog
    cd myblog
    jekyll serve
    {% endhighlight %}
    then there will be some folders and files are created by jekyll. and this folders and files will be holded by my github [repository](https://github.com/ray525/ray525.github.io)
	
---
=======

    ```
    jekyll new myblog
    cd myblog
    jekyll serve
    ```
    
    then there will be some folders and files are created by jekyll. and this folders and files will be holded by my github [repository](https://github.com/ray525/ray525.github.io)
    
___
>>>>>>> bccf982dff2a3a78c895897913e0ab6becf9cc0b:_posts/2015-08-24-how-to-create-this-blog.markdown

6. encounter a problem:

    > Generating... Liquid Exception: No such file or directory - python c:/Ruby200-x64/lib/ruby/gems/2.0.0/gems/pygments.rb-0.4.2/lib/pygments/mentos.py in 2013-04-22-yizeng-hello-world.md

    - install pygments

        * it seems pygments is installed by default when i install ruby. the folder is: `C:\Ruby22-x64\lib\ruby\gems\2.2.0\gems\pygments.rb-0.6.3\lib`
        * add `highlighter: pygments` at the end of _config.yml
        * but i have run the following command: `gem install pygments.rb`, and restart the computer

7. 
    > Please add the following to your Gemfile to avoid polling for changes:
 gem 'wdm', '>= 0.1.0' if Gem.win_platform?
    
    - instal wdm
    - gem install wdm
    
## reference
1. [Setup Jekyll on Windows](http://yizeng.me/2013/05/10/setup-jekyll-on-windows/#install-ruby) (*there is a troubleshooting section*)
2. [Jekyll 搭建静态博客](http://gaohaoyang.github.io/2015/02/15/create-my-blog-with-jekyll/)(*this is the target blog i want to make*)
        

 



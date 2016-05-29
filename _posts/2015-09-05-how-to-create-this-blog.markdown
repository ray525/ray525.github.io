---
layout: post
title:  "How to create this Blog"
date:   2015-09-05 21:07:07l
categories: jekyll
---

* content
{:toc}




## The final purpose
1. add jekyll folders and files on my git page [repository](https://github.com/ray525/ray525.github.io)
2. add markdown file in `_posts`, then your article will show on my [github page](http://ray525.github.io/)

## How to setup jekyll on windows
1. install ruby
2. install DevKit
3. setup mirror image

	> gem sources -a https://ruby.taobao.org/
	>  
    > gem sources -l
    >
    >*** CURRENT SOURCES ***
    >
    >https://ruby.taobao.org
    >
    >请确保只有 ruby.taobao.org

4. install package using `gem` 

	>gem install bundler
	>
	>gem install github-pages
	>
	>gem install jekyll
    
5. the i start myblog

	> jekyll new myblog
	> 
	> cd myblog
	> 
	> jekyll serve
	
	then there will be some folders and files are created by jekyll. and this folders and files will be holded by my github [repository](https://github.com/ray525/ray525.github.io)

6. encounter a problem:

    > Generating... Liquid Exception: No such file or directory - python c:/Ruby200-x64/lib/ruby/gems/2.0.0/gems/pygments.rb-0.4.2/lib/pygments/mentos.py in 2013-04-22-yizeng-hello-world.md

    - install pygments

        * it seems pygments is installed by default when i install ruby. the folder is: `C:\Ruby22-x64\lib\ruby\gems\2.2.0\gems\pygments.rb-0.6.3\lib`
        * add `highlighter: pygments` at the end of _config.yml
        * but i have run the following command: `gem install pygments.rb`, and restart the computer

	> Please add the following to your Gemfile to avoid polling for changes:
 	gem 'wdm', '>= 0.1.0' if Gem.win_platform?
    
    - instal wdm
		<pre><code>gem install wdm</code></pre>

## Test C code

	#! /usr/bin/env python
	#coding=utf-8
	class Link:
	    def __init__(self,value,next = None):
	        self.value = value
	        self.next = next
	
	    def show(self):
	        while self is not None:
	            print(self.value)
	            self = self.next
	            
	    def insert(self,value):
	        while self.next is not None:
	            self = self.next
	        self.next = Link(value)
	
	node1 = Link(0)
	for x in range(5):
	    node1.insert(x)    
	node1.show()
    
## Reference
1. [Setup Jekyll on Windows](http://yizeng.me/2013/05/10/setup-jekyll-on-windows/#install-ruby) (*there is a troubleshooting section*)
2. [Jekyll 搭建静态博客](http://gaohaoyang.github.io/2015/02/15/create-my-blog-with-jekyll/)(*this is the target blog i want to make*)
---
layout: post
title:  "How to create this Blog"
date:   2015-08-23 21:07:07
categories: jekyll
---
## The final purpose
1. add jekyll folders and files on my git page [repository](https://github.com/ray525/ray525.github.io)
2. add markdown file in `_posts`, then your article will show on my [github page](http://ray525.github.io/)

## How to setup jekyll on windows
1. install ruby
2. install DevKit
3. setup mirror image
    ```
    $ gem sources --remove https://rubygems.org/
    $ gem sources -a https://ruby.taobao.org/
    $ gem sources -l
    *** CURRENT SOURCES ***
    
    https://ruby.taobao.org
    # 请确保只有 ruby.taobao.org
    ```
4. install package using  `gem` 

    ```
    gem install bundler
    gem install github-pages
    gem install jekyll
    ```
    
5. the i start myblog
    ```
    jekyll new myblog
    cd myblog
    jekyll serve
    ```
    then there will be some folders and files are created by jekyll. and this folders and files will be holded by my github [repository](https://github.com/ray525/ray525.github.io)
___
6. encounter a problem:
    > Generating... Liquid Exception: No such file or directory - python c:/Ruby200-x64/lib/ruby/gems/2.0.0/gems/pygments.rb-0.4.2/lib/pygments/mentos.py in 2013-04-22-yizeng-hello-world.md

- install pygments
        * it seems pygments is installed by default when i install ruby. the folder is: `C:\Ruby22-x64\lib\ruby\gems\2.2.0\gems\pygments.rb-0.6.3\lib`
        * add `highlighter: pygments` at the end of _config.yml
        * but i have run the following command: `gem install pygments.rb`, and restart the computer

## reference
1. [Setup Jekyll on Windows](http://yizeng.me/2013/05/10/setup-jekyll-on-windows/#install-ruby) (*there is a troubleshooting section*)
2. [Jekyll 搭建静态博客](http://gaohaoyang.github.io/2015/02/15/create-my-blog-with-jekyll/)(*this is the target blog i want to make*)
        

 



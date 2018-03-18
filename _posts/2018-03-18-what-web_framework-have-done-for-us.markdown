---
layout: post
title:  "What web_framework have done for us?"
date:   2018-03-18 15:33:00
categories: web_server
tags: web backend
---

* content
{:toc}

![framework](http://static.zybuluo.com/ranger-01/g2nmyy9lxf7soes9gf1jin0q/image.png)

[我在哪？ 我在干什么？](https://www.zybuluo.com/ranger-01/note/1077107)




## 1. Web work flow

在[上一篇文章中](https://www.zybuluo.com/mdeditor#948727-full-reader)，我们知道web工作刘如下：

> the web client <-> the web server <-> the socket <-> (uwsgi protocol)uWSGI(WSGI) <-> Python

并用例子说明了，application server（uWSGI）要做的事情：
![image_1c8pfcqt750j25vcogdo13c02g.png-43.8kB][1]

1. 从输入中获取数据构造env字典（`handle_one_request, parse_request`），里面一般包括：
	- PATH_INFO：
	- HTTP_HOST，
	- SERVER_PORT
	- REQUEST_METHOD
	- QUERY_STRING(GET), 
	- CONTENT_LENGTH(POST),
	- wsgi.input(POST请求中，框架会从这个句柄中获取POST请求携带的data)
    - GET request    
		
		![image_1c8pei3921tgb1gc96pb771hei9.png-79.9kB][2]
    - POST request
        
		![image_1c8pept201b2b20eubas1g1f3km.png-66.8kB][3]

2. 根据application返回的output，构造http response
    - application先调用`start_response`构造response header
        ![image_1c8pfu82u15ucd651d38id289u3a.png-24kB][4]
    - application server在把output，添加在后面，行成response
        ![image_1c8pfrupn16be48n1j0nho313bf2t.png-63kB][5]

## 2. 框架做的事情
1. URL mapping
    - 创建URL map
    
		![image_1c8rqodng15rj1pmulrr1fmb1mto9.png-28.3kB][6]
    
	- 添加路由
		
		![image_1c8rqsq9tffji15tavv5asu4m.png-45.5kB][7]
    
	- URL map详情
	
		![image_1c8s2kpfq2c0mnsiajj5i1d5713.png-22.5kB][16]
		![image_1c8rr1rlr16m91vd1shg1f6k107g13.png-99.8kB][8]
		
    - URL match
		
		![image_1c8s2hb411lvn1c3b14ehh6514m4m.png-40.1kB][15]
		

2. Request and Response的封装
    - Request
    ![image_1c8rs9g1m1bsgiik635bep3vk9.png-69.5kB][10]
    - Response
    ![image_1c8rsbtlb1i0v1mt7129ua8i1drtm.png-51kB][11]
3. template 引擎
    - 模板引擎：相当于构造了一个函数，输入是模板需要的变量，输出是html
    ![image_1c8rv999c1a24i7nm6q91d1se713.png-60.7kB][12]
4. 错误，异常处理
    - 设置错误处理函数
     ![image_1c8rvocj2jpou8b6k51p3i18ll1t.png-70.3kB][14]
    - 根据错误的不同，output显示不一样的信息：例如：500则显示traceback；其他显示有用的msg
    ![image_1c8rvncdf16q536a1pc31ehr1b7b1g.png-48kB][13]
   
    
## 3. 框架还可以做的事情
1. 安全支持：放CSRF, XSS, SQL\命令行注入
2. session支持： token
3. 权限管控： has_permission
3. ORM支持
4. admin后台管理


  [1]: http://static.zybuluo.com/ranger-01/esz38qvkd251c4itiauhearo/image_1c8pfcqt750j25vcogdo13c02g.png
  [2]: http://static.zybuluo.com/ranger-01/ctfae2b9n08vj1qc9m3neke7/image_1c8pei3921tgb1gc96pb771hei9.png
  [3]: http://static.zybuluo.com/ranger-01/zykd60vi3zwk3stpgnkcxlpx/image_1c8pept201b2b20eubas1g1f3km.png
  [4]: http://static.zybuluo.com/ranger-01/oxx6vvhhmocil912enpdlrkj/image_1c8pfu82u15ucd651d38id289u3a.png
  [5]: http://static.zybuluo.com/ranger-01/aj2j8u6ayi3s2ypt9nkiwpy5/image_1c8pfrupn16be48n1j0nho313bf2t.png
  [6]: http://static.zybuluo.com/ranger-01/mhclfs9tdx9umpp7rge5a6pw/image_1c8rqodng15rj1pmulrr1fmb1mto9.png
  [7]: http://static.zybuluo.com/ranger-01/8csyfmbzk3vjfcqgszqyshdj/image_1c8rqsq9tffji15tavv5asu4m.png
  [8]: http://static.zybuluo.com/ranger-01/331n1dqgwh6qlnskdr2ppx7e/image_1c8rr1rlr16m91vd1shg1f6k107g13.png
  [9]: http://static.zybuluo.com/ranger-01/esz38qvkd251c4itiauhearo/image_1c8pfcqt750j25vcogdo13c02g.png
  [10]: http://static.zybuluo.com/ranger-01/hqfgiy0y7wt7era7u6qavm8k/image_1c8rs9g1m1bsgiik635bep3vk9.png
  [11]: http://static.zybuluo.com/ranger-01/8mlexq0njws18swnfvnikujm/image_1c8rsbtlb1i0v1mt7129ua8i1drtm.png
  [12]: http://static.zybuluo.com/ranger-01/rdd0zjml70oo6hizwx2rozr0/image_1c8rv999c1a24i7nm6q91d1se713.png
  [13]: http://static.zybuluo.com/ranger-01/ode2d9z0992kf9v10570r054/image_1c8rvncdf16q536a1pc31ehr1b7b1g.png
  [14]: http://static.zybuluo.com/ranger-01/wyknex0st1opwja0m4skm8rl/image_1c8rvocj2jpou8b6k51p3i18ll1t.png
  [15]: http://static.zybuluo.com/ranger-01/sp5rxvas2865gtesgapxlxea/image_1c8s2hb411lvn1c3b14ehh6514m4m.png
  [16]: http://static.zybuluo.com/ranger-01/n77jfosjntvnptdnmpq1kj52/image_1c8s2kpfq2c0mnsiajj5i1d5713.png
  
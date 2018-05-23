---
layout: post
title:  "HTTP Cache"
date:   2018-05-23 16:43:00
categories: web 
tags: cache
---

* content
{:toc}

![intro](http://static.zybuluo.com/ranger-01/m7nb9miarzk7yrcnz1xiucb6/image_1ce61ke1j19ffus1vhe4q1kes5q.png)

[作业部落记录](https://www.zybuluo.com/ranger-01/note/1157187)




## 1. HTTP缓存简介

-  HTTP 的缓存机制不仅可以极大地减少服务器负载， 更重要的是加速页面的载入，以及减少用户的流量消耗。
-  HTTP 中最基本的缓存机制， 涉及到的 HTTP 头字段， 包括 `Cache-Control, Last-Modified, If-Modified-Since, Etag, If-None-Match` 等。

## 2. HTTP header 介绍

### 2.1 Cache-Control

- request

``` javascript
Cache-Control: max-age=<seconds>
Cache-Control: max-stale[=<seconds>]
Cache-Control: min-fresh=<seconds>
Cache-control: no-cache 
Cache-control: no-store
Cache-control: no-transform
Cache-control: only-if-cached
```

- response

```javascript
Cache-control: must-revalidate
Cache-control: no-cache
Cache-control: no-store
Cache-control: no-transform
Cache-control: public
Cache-control: private
Cache-control: proxy-revalidate
Cache-Control: max-age=<seconds>
Cache-control: s-maxage=<seconds>
```

### 2.2 Last-Modified/If-Modified-Since

- Last-Modified：标示这个响应资源的最后修改时间。web服务器在响应请求时，告诉浏览器资源的最后修改时间。

- If-Modified-Since：当资源过期时（使用Cache-Control标识的max-age），发现资源具有Last-Modified声明，则再次向web服务器请求时带上头 If-Modified-Since，表示请求时间。web服务器收到请求后发现有头If-Modified-Since 则与被请求资源的最后修改时间进行比对。若最后修改时间较新，说明资源又被改动过，则响应整片资源内容（写在响应消息包体内），HTTP 200；若最后修改时间较旧，说明资源无新修改，则响应HTTP 304 (无需包体，节省浏览)，告知浏览器继续使用所保存的cache。


### 2.3 Etag/If-None-Match

- Etag：web服务器响应请求时，告诉浏览器当前资源在服务器的唯一标识（生成规则由服务器觉得）。Apache中，ETag的值，默认是对文件的索引节（INode），大小（Size）和最后修改时间（MTime）进行Hash后得到的。

- If-None-Match：当资源过期时（使用Cache-Control标识的max-age），发现资源具有Etage声明，则再次向web服务器请求时带上头If-None-Match （Etag的值）。web服务器收到请求后发现有头If-None-Match 则与被请求资源的相应校验串进行比对，决定返回200或304。


### 2.4 why Etag

- Last-Modified标注的最后修改只能精确到秒级，如果某些文件在1秒钟以内，被修改多次的话，它将不能准确标注文件的修改时间

- 如果某些文件会被定期生成，当有时内容并没有任何变化，但Last-Modified却改变了，导致文件没法使用缓存

- 有可能存在服务器没有准确获取文件修改时间，或者与代理服务器时间不一致等情形

Etag是服务器自动生成或者由开发者生成的对应资源在服务器端的唯一标识符，能够更加准确的控制缓存。Last-Modified与ETag是可以一起使用的，服务器会优先验证ETag，一致的情况下，才会继续比对Last-Modified，最后才决定是否返回304。

![image_1ce61ke1j19ffus1vhe4q1kes5q.png-107.6kB][5]

## 3. 实验

### 3.1 nginx 配置

![image_1ce5sqrq69f2fla1ef61lgg1l3h2t.png-8.8kB][1]

### 3.2 第一次加载

![image_1ce5sjbui1k6e128u17m31tfg1fpq9.png-63.7kB][2]

### 3.3 超过max-age

![image_1ce5smsb0slpeq19om1qrosfl1m.png-59.4kB][3]

### 3.4 没有超过max-age

![image_1ce5soigsfcd17j18qo11sq138s23.png-64.5kB][4]


## 4. 参考

1. [HTTP 缓存机制 -- 客户端缓存](https://juejin.im/entry/58c3d451da2f6056096b0dd6)

  [1]: http://static.zybuluo.com/ranger-01/g4wuvz50pyk4y266zu6x44vr/image_1ce5sqrq69f2fla1ef61lgg1l3h2t.png
  [2]: http://static.zybuluo.com/ranger-01/z2r93m4z8vb3j5jekmj3hjju/image_1ce5sjbui1k6e128u17m31tfg1fpq9.png
  [3]: http://static.zybuluo.com/ranger-01/yojy0a2j2rm8umhvwbms7lcn/image_1ce5smsb0slpeq19om1qrosfl1m.png
  [4]: http://static.zybuluo.com/ranger-01/stjg0kbtu6omrx7tifheg5ij/image_1ce5soigsfcd17j18qo11sq138s23.png
  [5]: http://static.zybuluo.com/ranger-01/m7nb9miarzk7yrcnz1xiucb6/image_1ce61ke1j19ffus1vhe4q1kes5q.png
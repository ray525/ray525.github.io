---
layout: post
title:  "浏览器如何加载页面"
date:   2018-05-27 23:17:00
categories: web
tags: js
---

* content
{:toc}

![intro](http://static.zybuluo.com/ranger-01/1j82kbz871onzpww19t3vqap/image_1cegu6i9ap9mkmp2om1nhu1f0t1g.png)

[作业部落](https://www.zybuluo.com/ranger-01/note/1162312)




## 1. 浏览器的多进程与多线程

![image_1cegra5dm63r10og1vg922qu4f9.png-101.9kB][1]

相比于单进程浏览器，多进程有如下优点：

- 避免单个page crash影响整个浏览器
- 避免第三方插件crash影响整个浏览器
- 多进程充分利用多核优势
- 方便使用沙盒模型隔离插件等进程，提高浏览器稳定性

### 1.1 浏览器有哪些进程

- Browser进程：浏览器的主进程（负责协调、主控），只有一个。作用有
    - 负责浏览器界面显示，与用户交互。如前进，后退等
    - 负责各个页面的管理，创建和销毁其他进程
    - 将Renderer进程得到的内存中的Bitmap，绘制到用户界面上
    - 网络资源的管理，下载等 
- GPU进程：最多一个，用于3D绘制等
- 第三方插件进程：每种类型的插件对应一个进程，仅当使用该插件时才创建
- 浏览器渲染进程（Renderer进程，内部是多线程的）：：默认每个Tab页面一个进程，互不影响。主要作用为
    - 页面渲染，
    - 脚本执行，
    - 事件处理等

### 1.2 渲染进程包含哪些线程

![image_1cegrl011lnf781omt1udm158am.png-18.4kB][2]

- GUI渲染线程
    - 负责渲染浏览器界面，解析HTML，CSS，构建DOM树和RenderObject树，布局和绘制等。
    - 当界面需要重绘（Repaint）或由于某种操作引发回流(reflow)时，该线程就会执行
    - 注意，**GUI渲染线程与JS引擎线程是互斥的**，当JS引擎执行时GUI线程会被挂起（相当于被冻结了），GUI更新会被保存在一个队列中等到JS引擎空闲时立即被执行
    
- JS引擎线程
    - 也称为JS内核，负责处理Javascript脚本程序。（例如V8引擎）
    - JS引擎线程负责解析Javascript脚本，运行代码。
    - JS引擎一直等待着任务队列中任务的到来，然后加以处理，一个Tab页（renderer进程）中无论什么时候都只有一个JS线程在运行JS程序
    - 同样注意，GUI渲染线程与JS引擎线程是互斥的，所以如果JS执行的时间过长，这样就会造成页面的渲染不连贯，导致页面渲染加载阻塞。
    
- 事件触发线程
    - 归属于浏览器而不是JS引擎，用来控制事件循环（可以理解，JS引擎自己都忙不过来，需要浏览器另开线程协助）
    - 当JS引擎执行代码块如setTimeOut时（也可来自浏览器内核的其他线程,如鼠标点击、AJAX异步请求等），会将对应任务添加到事件线程中
    - 当对应的事件符合触发条件被触发时，该线程会把事件添加到待处理队列的队尾，等待JS引擎的处理
    - 注意，由于JS的单线程关系，所以这些待处理队列中的事件都得排队等待JS引擎处理（当JS引擎空闲时才会去执行）
    
- 定时触发器线程
    - 传说中的setInterval与setTimeout所在线程
    - 浏览器定时计数器并不是由JavaScript引擎计数的,（因为JavaScript引擎是单线程的, 如果处于阻塞线程状态就会影响记计时的准确）
    - 因此通过单独线程来计时并触发定时（计时完毕后，添加到事件队列中，等待JS引擎空闲后执行）
    - 注意，W3C在HTML标准中规定，规定要求setTimeout中低于4ms的时间间隔算为4ms。
    
- 异步http请求线程
    - 在XMLHttpRequest在连接后是通过浏览器新开一个线程请求
    - 将检测到状态变更时，如果设置有回调函数，异步线程就产生状态变更事件，将这个回调再放入事件队列中。再由JavaScript引擎执行。

## 2. GUI线程与JS线程的互斥

![image_1cegt8d511i6a1tks75j12m31ude13.png-38.1kB][3]

- JS 会阻塞后续 DOM 解析以及其它资源(如 CSS，JS 或图片资源)的加载。
    - 因为js有可能会修改dom，如果不阻塞后续的资源下载，dom的操作顺序不可控。
- CSS 不会阻塞后续 DOM 结构的解析，不会阻塞其它资源(如图片)的加载，但是会阻塞 JS 文件的运行。
    - 前面的 CSS 样式可能会覆盖 JS 文件中定义的元素样式。如果在修改这些元素属性同时渲染界面（即JS线程和UI线程同时运行），那么渲染线程前后获得的元素数据就可能不一致了。
- 外链的js如果含有defer="true"属性，将会并行加载js，到页面全部加载完成后才会执行，会按顺序执行
    - defer属性的作用是，告诉浏览器，等到DOM加载完成后，再执行指定脚本。
- 外链的js如果含有async="true"属性，将不会依赖于任何js和css的执行，此js下载完成后立刻执行，不保证按照书写的顺序执行。
    - async属性的作用是，使用另一个进程下载脚本，下载时不会阻塞渲染。

## 3. 页面渲染步骤

### 3.1 渲染过程
![image_1cegu6i9ap9mkmp2om1nhu1f0t1g.png-38.7kB][4]

- 解析HTML，构建DOM树（这里遇到外链，此时会发起请求）
- 解析CSS，生成CSS规则树
- 合并DOM树和CSS规则，生成render树
- 布局render树（Layout/reflow），负责各元素尺寸、位置的计算
- 绘制render树（paint），绘制页面像素信息
- 浏览器会将各层的信息发送给GPU，GPU将各层合成（composite），显示在屏幕上

### 3.2 reflow & repaint

![image_1cegv0jbg10tn5hlb83br6jhl1t.png-70.9kB][5]
 
- reflow(回流): 
    根据Render Tree布局(几何属性)，意味着元素的内容、结构、位置或尺寸发生了变化，需要重新计算样式和渲染树；
- repaint(重绘): 
    意味着元素发生的改变只影响了节点的一些样式（背景色，边框颜色，文字颜色等），只需要应用新样式绘制这个元素就可以了；

- tips
    - 在上述渲染过程中，前3点可能要多次执行，比如js脚本去操作dom、更改css样式时，浏览器又要重新构建DOM、CSSOM树，重新render，重新layout、paint；
    - Layout在Paint之前，因此每次Layout重新布局（reflow 回流）后都要重新出发Paint渲染，这时又要去消耗GPU；
    - Paint不一定会触发Layout，比如改个颜色改个背景；（repaint 重绘）
    - 图片下载完也会重新出发Layout和Paint；
    - reflow回流的成本开销要高于repaint重绘，一个节点的回流往往回导致子节点以及同级节点的回流；

## 4. JS运行机制

这里就直接引用一张图片来协助理解：（参考自Philip Roberts的演讲[《Help, I’m stuck in an event-loop》](http://vimeo.com/96425312)）

![image_1ceh22rbg1ta4js21ual1f6i6a22a.png-28.5kB][6]

上图大致描述就是：

- 主线程运行时会产生执行栈，栈中的代码调用某些api时，它们会在事件队列中添加各种事件（当满足触发条件后，如ajax请求完毕）
- 而栈中的代码执行完毕，就会读取事件队列中的事件，去执行那些回调
- 如此循环
- 注意，总是要等待栈中的代码执行完毕后才会去读取事件队列中的事件

### 4.1 macrotask与microtask

```javascript
console.log('script start');

setTimeout(function() {
    console.log('setTimeout');
}, 0);

Promise.resolve().then(function() {
    console.log('promise1');
}).then(function() {
    console.log('promise2');
});

console.log('script end');

// output
script start
script end
promise1
promise2
setTimeout
```

- macrotask（又称之为宏任务），
    - 可以理解是每次执行栈执行的代码就是一个宏任务（包括每次从事件队列中获取一个事件回调并放到执行栈中执行）
    - 主代码块，setTimeout，setInterval等（可以看到，事件队列中的每一个事件都是一个macrotask）

- microtask（又称为微任务），
    - 可以理解是在当前 task 执行结束后立即执行的任务
    - Promise，process.nextTick等

![image_1ceh2e3dq4tk1lf819c91kdml1b2n.png-25.5kB][7]

## 5. 参考

1. [JS 和 CSS 的位置对资源加载顺序的影响(有示例)](https://zhuanlan.zhihu.com/p/24944905)
2. [html,css,js加载顺序](https://blog.csdn.net/brink_compiling/article/details/72825750)
3. [DOM 操作成本到底高在哪儿？](https://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651553940&idx=1&sn=114080b1cee1a7e72c7581934622ba1d&chksm=80255755b752de43f9b58e70423944f6096e8427c38fca6900e8da49eb8ca8a0131d0907eb09&mpshare=1&scene=1&srcid=040719DvzU4no41xbTGr0g3F#rd)
4. [从浏览器多进程到JS单线程，JS运行机制最全面的一次梳理](http://www.dailichun.com/2018/01/21/js_singlethread_eventloop.html)


  [1]: http://static.zybuluo.com/ranger-01/7uvpipr1pd7nvlsga04mr2gw/image_1cegra5dm63r10og1vg922qu4f9.png
  [2]: http://static.zybuluo.com/ranger-01/3ak0ic9dm3hjzkjewpoailde/image_1cegrl011lnf781omt1udm158am.png
  [3]: http://static.zybuluo.com/ranger-01/lx3qxmjo8byybi5zfwhl8vce/image_1cegt8d511i6a1tks75j12m31ude13.png
  [4]: http://static.zybuluo.com/ranger-01/1j82kbz871onzpww19t3vqap/image_1cegu6i9ap9mkmp2om1nhu1f0t1g.png
  [5]: http://static.zybuluo.com/ranger-01/mkzvgmr9ftmwwa3k5eemukpc/image_1cegv0jbg10tn5hlb83br6jhl1t.png
  [6]: http://static.zybuluo.com/ranger-01/zryvw2gts3i40lwnh3wxo6tl/image_1ceh22rbg1ta4js21ual1f6i6a22a.png
  [7]: http://static.zybuluo.com/ranger-01/ydg2yawa1htsts6yr4r233hz/image_1ceh2e3dq4tk1lf819c91kdml1b2n.png
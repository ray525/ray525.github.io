---
layout: post
title:  "精通Docker：容器概述"
date:   2018-05-14 16:00:00
categories: deploy
tags: web deploy docker
---

* content
{:toc}

![intro](http://static.zybuluo.com/ranger-01/h14duilk27ip7r4wtoya7q2j/image_1cdf6eufl1najlg51lrnv9q1phi9.png)

[作业部落记录](https://www.zybuluo.com/ranger-01/note/1141121)



## 1. 容器生态系统

### 1.1 容器核心技术

![image_1cd20vvd3603hod71h15kp1mfcp.png-107.5kB][1]

1. 容器规范
容器不管只是Docker， 还有CoreOS的 rkt。**有了容器规范，不同组织和厂商的容器能够在不同的runtime上运行**。保证容器的可移植性
![image_1cd212e7h1hpsj461f34o041kvp16.png-56.7kB][2]
    - runtime spec
    - image format spec
    
2. 容器runtime

    ![image_1cd21boamgpciv4advds1efop.png-48.2kB][3]
    
    **runtime是容器真正运行的地方**，runtime需要跟OS kernel紧密协作，为容器提供运行环境。
    
3. 容器管理工具
    
    ![image_1cd21cau5r316u6c751a091vdu16.png-72.4kB][4]
    
    光有 runtime 还不够，用户得有工具来管理容器啊。**容器管理工具对内与 runtime 交互，对外为用户提供 interface**，比如 CLI。

4. 容器定义工具

    ![image_1cdesvtifc2r1nko128a8fmhumm.png-87.1kB][5]
    
    容器定义工具允许用户定义容器的内容和属性，这样容器就能够被保存，共享和重建。

5. Registries

    ![image_1cdet0mt01d4d14qs1sd1i1619me13.png-43.2kB][6]

    容器是通过 image 创建的，需要有一个仓库来统一存放 image，这个仓库就叫做 Registry。

6. 容器OS

    ![image_1cdet18to1s2s1fplbqk10li9ve1g.png-55.1kB][7]

    容器 OS 是专门运行容器的操作系统。与常规 OS 相比，容器 OS 通常体积更小，启动更快。因为是为容器定制的 OS，通常它们运行容器的效率会更高。

### 1.2 容器平台技术

容器平台技术能够让容器作为集群在分布式环境中运行。

![image_1cdet1od1nnqm7t116n1tms19c01t.png-66.8kB][8]

1. 容器编排引擎

    ![image_1cdet2b0e197q1clcgdp156tb92a.png-89.8kB][9]

    微服务架构中，应用被分为不同的组件，每个组件可能会运行多个相同的容器。这些容器组成集群，集群中的容器会根据业务需要被动态的创建，迁移，销毁
    所谓编排（orchestration），通常包括容器管理、调度、集群定义和服务发现等。通过容器编排引擎，容器被有机的组合成微服务应用，实现业务需求

2. 容器管理平台

    ![image_1cdet32rt995hnm4iq14hi10k737.png-59.7kB][10]

    容器管理平台是架构在容器编排引擎之上的一个更为通用的平台。抽象了编排引擎的底层实现细节，为用户提供更方便的功能。

3. 基于容器的PaaS

### 3. 容器支持技术

下面这些技术被用于支持基于容器的基础设施。

![image_1cdet4arg1rlhrqa15f4984a7l44.png-96.8kB][11]

1. 容器网络

    ![image_1cdet7dhs196r213q911rg1lij71.png-90.4kB][12]

    容器的出现使网络拓扑变得更加动态和复杂。用户需要专门的解决方案来管理容器与容器，容器与其他实体之间的连通性和隔离性。

2. 服务发现

    ![image_1cdet97alhi7177s1dgapf91u507u.png-57.8kB][13]

    动态变化是微服务应用的一大特点。当负载增加时，集群会自动创建新的容器；负载减小，多余的容器会被销毁。容器也会根据 host 的资源使用情况在不同 host 中迁移，容器的 IP 和端口也会随之发生变化。
    在这种动态的环境下，必须要有一种机制让 client 能够知道如何访问容器提供的服务。这就是服务发现技术要完成的工作。
3. 监控

    ![image_1cdetamc81tuedfg4ad12vd1skn8b.png-126.4kB][14]

4. 数据管理

    ![image_1cdetbdbo151n1p0mt58jnf1apo8o.png-22.6kB][15]

    容器经常会在不同的 host 之间迁移，如何保证持久化数据也能够动态迁移

5. 日志管理

    ![image_1cdetbu7k92v1o9k1ipjhj13d995.png-46.3kB][16]

    日志为问题排查和事件管理提供了重要依据

6. 安全性

    ![image_1cdetcgar591ek3gk81lk81dpa9i.png-26.9kB][17]

    OpenSCAP 能够对容器镜像进行扫描，发现潜在的漏洞。

## 2. 容器是什么

容器是一种轻量级，可移植，自包含的软件打包技术，使应用程序可以在任何地方以相同的方式运行。build once， run anywhere。

### 2.1 容器与虚拟机区别

- 容器
容器在 Host 操作系统的用户空间中运行，与操作系统的其他进程隔离。
容器由两部分组成：
    - 应用程序本身
    - 依赖：比如应用程序需要的库或其他软件

- 虚拟机
传统的虚拟化技术，比如 VMWare, KVM, Xen，目标是创建完整的虚拟机。为了运行应用，除了部署应用本身及其依赖（通常几十 MB），还得安装整个操作系统（几十 GB）。

![image_1cdetd5d61o48bhu43a5i5t1a9v.png-165.2kB][18]

### 2.2 容器轻量级的原因

- 由于所有的容器共享同一个 Host OS，这使得容器在体积上要比虚拟机小很多。
- 另外，启动容器不需要启动整个操作系统，所以容器部署和启动速度更快，开销更小，也更容易迁移。

## 3. 为什么要用容器
简要的答案是：容器使软件具备了超强的可移植能力。

### 3.1 容器解决了什么问题
- 一方面应用包含多种服务，这些服务有自己所依赖的库和软件包
- 另一方面存在多种部署环境，服务在运行时可能需要动态迁移到不同的环境中。
如何让每种服务能够在所有的部署环境中顺利运行？
![image_1cd41jhp2h3su6add31fq57362g.png-76.6kB][19]

## 4. 容器时如何工作的
![](http://static.zybuluo.com/ranger-01/h14duilk27ip7r4wtoya7q2j/image_1cdf6eufl1najlg51lrnv9q1phi9.png)

Docker 采用的是 Client/Server 架构。客户端向服务器发送请求，服务器负责构建、运行和分发容器。客户端和服务器可以运行在同一个 Host 上，客户端也可以通过 socket 或 REST API 与远程的服务器通信。


  [1]: http://static.zybuluo.com/ranger-01/7yvnd0wrwnizq76zy3f8926f/image_1cd20vvd3603hod71h15kp1mfcp.png
  [2]: http://static.zybuluo.com/ranger-01/quzf1954ddzpaifz2tt455z4/image_1cd212e7h1hpsj461f34o041kvp16.png
  [3]: http://static.zybuluo.com/ranger-01/mam6sofukdrjpg49u8th0vvg/image_1cd21boamgpciv4advds1efop.png
  [4]: http://static.zybuluo.com/ranger-01/0z4cc4ut4wdza585eix2jxom/image_1cd21cau5r316u6c751a091vdu16.png
  [5]: http://static.zybuluo.com/ranger-01/p9pbp2fdcl6zl9cw6gwo1rrm/image_1cdesvtifc2r1nko128a8fmhumm.png
  [6]: http://static.zybuluo.com/ranger-01/pghrfqzwuoj401q8oyh3kool/image_1cdet0mt01d4d14qs1sd1i1619me13.png
  [7]: http://static.zybuluo.com/ranger-01/mlqdux4gl00il5gmou1syo3e/image_1cdet18to1s2s1fplbqk10li9ve1g.png
  [8]: http://static.zybuluo.com/ranger-01/8xf4vkymuhe7ulvbu1gm6ypr/image_1cdet1od1nnqm7t116n1tms19c01t.png
  [9]: http://static.zybuluo.com/ranger-01/u8krdxcisvhbgyj1ojmgjfx7/image_1cdet2b0e197q1clcgdp156tb92a.png
  [10]: http://static.zybuluo.com/ranger-01/yczu8o3vvh0x8b31i102j8el/image_1cdet32rt995hnm4iq14hi10k737.png
  [11]: http://static.zybuluo.com/ranger-01/xzoj437yonqyibpx7wpqlx7z/image_1cdet4arg1rlhrqa15f4984a7l44.png
  [12]: http://static.zybuluo.com/ranger-01/wcsksoahm1ktm1bls2ytio32/image_1cdet7dhs196r213q911rg1lij71.png
  [13]: http://static.zybuluo.com/ranger-01/abkkmfyxcr2x7hposi7udf4h/image_1cdet97alhi7177s1dgapf91u507u.png
  [14]: http://static.zybuluo.com/ranger-01/vn0o1g9nsou7pb7lncpmkh6d/image_1cdetamc81tuedfg4ad12vd1skn8b.png
  [15]: http://static.zybuluo.com/ranger-01/8caeufq8ydm0pc07zv206bq9/image_1cdetbdbo151n1p0mt58jnf1apo8o.png
  [16]: http://static.zybuluo.com/ranger-01/t5s5s62xzl6b0cxoiw62pz54/image_1cdetbu7k92v1o9k1ipjhj13d995.png
  [17]: http://static.zybuluo.com/ranger-01/5fm7jlxp4pxeszh6qol95q70/image_1cdetcgar591ek3gk81lk81dpa9i.png
  [18]: http://static.zybuluo.com/ranger-01/bswtsqqwpk5ripzyco29ic1y/image_1cdetd5d61o48bhu43a5i5t1a9v.png
  [19]: http://static.zybuluo.com/ranger-01/e8p9f7ok8e8pw66k0wu1idiy/image_1cd41jhp2h3su6add31fq57362g.png
  
---
title: RESTful API
tags: [API]
toc: true
date: 2019-03-31 21:22:08
category: Web
---

前后端分离之后，前后端人员通过接口数据进行交互，目前大部分都是通过RESTful API来进行的，双方各司其职，各发挥长处，但是同时也带来了沟通成本的增加，那么你了解REST吗？你的API是RESTful的吗？本文将和工作中内容关联起来阐述我眼中的REST。
<!-- more -->

>REST即表述性状态传递（英文：Representational State Transfer，简称REST）是Roy Fielding博士在2000年他的博士论文中提出来的一种软件架构风格。它是一种针对网络应用的设计和开发方式，可以降低开发的复杂性，提高系统的可伸缩性。

看了REST的定义，什么也没明白，下面我来说说什么是REST，如何让你的API设计成RESTful。

REST把所有的东西都看成资源，RESTful的接口为 动词 + 名词，比如我们订单接口，那么订单就可以认为是一种资源，可以通过URI接口来获取，那么获取资源接口就可以设计成 `GET` `/orders`，这就是动词 + 名词。

动词常用有四种： `GET` `POST` `PUT` `DELETE` 
名词就是特定的资源了，和具体的业务有关，是名词就行。

- `GET` /orders 获取订单列表
- `GET` /orders/{id} 获取某个订单
- `POST` /orders 新增订单
- `PUT` /orders/{id} 修改某个订单
- `DELETE` /orders/{id} 删除某个订单

还有一个就是状态码的管理,但是这个工作中用的不怎么规范。
【成功状态】
- `GET` `200` OK
- `POST` `201` Created
- `PUT` `200` OK
- `DELETE` `204` No Contents
【错误状态】
- `500` 服务器内部错误
- `503` 服务器无法处理请求
- `400` 错误请求，请求体不对
- `405` 错误请求，请求方法不对
- `401` 错误请求，身份认证通不过
- `403` 错误请求，未授权


这就是常用的接口设计方案，这样子的接口就是符合RESTful的，看起来挺简单，但是这个跨越却是历史性，这样子接口的设计就会有一个规范，大家都默认这种描述资源的方式比较直观，接口都是通过json来传递数据，比较轻量级，这样子就统一起来，不像之前只有GET和POST，而且接口都是开发人员随便起的，因为这个接口只有他一个人用，还有可能通过xml数据传输，即使提供出去，那也是一团糟。



说完了REST和RESTful，来讲讲工作中都怎么用。

现在常用的REST框架有Jersey和CXF，当然SpringMVC也是支持RESTful API的，目前来说,SpringMVC是主流，之前项目中有用Jersey，我觉得Jersey也挺好用，而且内置jetty来处理Web请求，有点类似目前主流的SpringBoot的一部分功能。

最近在和一个第三方厂商对接接口，我开发完成了接口，想要做一个模拟对接服务，代码肯定是不能写在项目中，只能另外写一个项目来提供对接接口，当时首先想到的就是SpringBoot来提供一站式解决方案，简单快捷，但是后来想了想觉得有点重了，只是提供一个模拟接口而已，只需要提供要给接口出来返回json就行，想到了之前用过Jersey，就用Jersey+Gson来完成了，今天也开了个仓库来记录，以后可能还会用到。

<div class="github-widget" data-repo="ruanzz/Api"></div>

把项目clone下来之后，运行`ApiApplication`的`main()`方法就可以把一个REST服务跑起来，借助的是内嵌的Jetty服务器，看到这里是不是觉得和SpringBoot有点类似？是的，软件的思想其实都是殊途同归的，触类旁通很重要。

接下来访问增删改查接口

curl -H 'Content-Type:application/json' -XPOST http://127.0.0.1:8080/demos -d '{"id":5,name:"demo5"}'
{"id":5,"name":"demo5","createTime":"2019-03-31 56:22:37"}

curl -XGET http://localhost:8080/demos/5 
{"id":5,"name":"demo5","createTime":"2019-03-31 56:22:37"}

 curl -H 'Content-Type:application/json' -XPUT http://127.0.0.1:8080/demos/5 -d '{name:"demo5-1"}'
{"id":5,"name":"demo5-1","createTime":"2019-03-31 56:22:37"}

 curl -XDELETE http://127.0.0.1:8080/demos/5
"sucess"


  


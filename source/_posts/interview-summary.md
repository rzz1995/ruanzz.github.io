---
title: Java后端面试题整理
date: 2018-12-18 21:47:55
tags: [interview,Java]
toc: true
---
![惠灵顿](/asset/img/interview-summary/Mac.jpeg)

<!-- more -->
这个冬天有点冷，冬天了还是储藏点过冬的粮食，用倒推法来总结相关知识，这篇博客将会长期更新，收集网上的面试题，然后自己整理思路，以后自己肯定会用的上，也为以后博客积累素材。

## 1. Java基础
### 1.1 语法糖
>1.String类有看过源码吗？和StringBuffer，StringBuilder有什么区别？

String类是一个final不可修改的类,说明String类不能被继承，StringBuffer和StringBuilder都是用来拼接字符串用的，只有调用toString()方法的时候才会真正的new String()对象，StringBuffer是线程安全的，StringBuilder是线程不安全的，因此StringBuilder的性能会更好一点，有并发场景的话还是使用StringBuffer来比较合适。

>2.Throwable、Error、Exception区别和联系。受查异常和非受查异常都用过哪些，谈一下对他们的理解

Throwable 是异常的顶层类，Error和Exception都继承了这个类，当然还有其他一些子类，在我们开发过程中常接触到也就是这两个类的子类，算是两种比较常用的异常处理。Error代表出现了非常严重的错误，JVM虚拟机无法处理，只能Crash，比如OutOfMemoryError。Exception是一般性的异常，这个一般性指的是不会造成虚拟机宕机，但是对开发人员来说，最复杂的也就是这块，对异常的处理是根据业务来的，是抛出去还是try起来，怎么对异常分类？这个都要根据具体的业务来决定，跟业务相关复杂性肯定不会小，因此如何处理好异常也是很有门道的。
Exception和Error本来是互不相关的，但是Exception中有一个叛徒RuntimeException，这个叛徒却是最受开发人员欢迎的，基本上自定义业务一样都会继承它，为什么说他是叛徒呢？它和Error一样，都是都是unchecked的，程序都会进行中止处理，除RuntimException及其子类外的Exception都是checked的，受查异常和非受查异常在在编译过程中就有体现出来，受查异常必须在代码层面做处理，不然编译不通过。常用的非受查异常有OutOfMemoryError,ClassCastException,NullPointerException等，常用的受查异常有：FileNotFoundException，NumberFormatException，NoSuchMethodException，IOException，ClassNotFoundException，InterruptedException等

> 3.父类和子类之间加载的时候，代码块，构造块，构造方法，普通方法调用的顺序

static静态成员变量或者静态代码块是JVM启动的时候加载，所以会优先执行这个，大概的顺序是代码块优先于构造方法，静态优先于非静态。执行的顺序如下：
1.父类静态成员变量和静态代码块
2.子类静态成员变量和静态代码块
3.父类普通成员变量和普通代码块
4.父类构造方法
5.子类普通成员变量和普通代码块
6.子类构造方法

> 4.Java泛型有什么好处？是怎么实现的?

泛型的的好处就是在编译的时候检查类型安全，把运行时异常提前到编译时异常，并且所有的强制转换都是自动和隐式转换，提高了代码的重用，不用到处都是显式的强制转换，让代码更加优雅。
Java泛型的实现原理是类型擦除，是在编译器这个层次来实现。什么是类型擦除呢？就是使用泛型的时候加上的类型参数，在编译的时候去掉，在生成的Java字节码文件中是不包含泛型中的类型信息的。比如，定义了一个`List<String>`类型，在编译之后都会变成`List`类型，JVM看到的只是`List`，而由泛型附加的类型信息是对JVM来说是透明的。

### 1.2 集合
### 1.3 多线程
### 1.4 IO
### 1.5 JVM
>1.ClassLoader的种类，父子关系(不一定是继承)，双亲委派机制，Java为什么要用双亲委派机制?

> 2.如何打破双亲委派机制?什么时候需要打破双亲委派机制？有哪些框架打破了双亲委派机制？

## 2. 数据库
### 2.1 MySQL
> 1.MySQL数据库的底层原理

MySQL数据库是C/S架构模式，我们关心的是Server端，也就是mysqld这个进程。MySQL服务端是二层架构：
1.SQL层(SQL Layer),在MySQL数据库系统处理底层数据之前的所有工作都是在这一层完成，包括权限判断，SQL解析，执行计划优化，查询缓存的处理等等。具体模块及职责如下:
- 初始化模块: MySQL Server启动的时候做初始化操作
- 核心API: 提供一些非常高效的底层操作功能的优化实现
- 网络交互模块: 抽象底层网络交互所使用的api，实现底层网络数据的接收与发送
- Client&Server交互协议模块: MySQL 客户端和服务端交互的协议实现，基于TCP/IP,Socket等
- 用户模块: 用户管理模块，包括用户的登录连接权限以及用户的授权管理
- 访问限制模块: 根据用户模块的授权信息来控制用户对数据的访问
- 连接管理模块: 负责监听对MySQL Server的各种请求，接收连接请求，转发所有的连接请求到线程管理模块，每一个连接上MySQL Server的客户端请求都会被分配或者创建一个连接线程为其单独服务
- Query解析和转发模块: 连接线程接收到客户端的一个Query之后，将Query分类后转发给各个对应的处理模块
- Query Cache模块: 将客户端提交给MySQL的Select类Query请求的返回结果集缓存到内存中，与该Query的一个Hash值做一个对应。当取数据的基表发生变化之后，缓存失效
- Query 优化器模块: 优化客户端请求的Query，根据一些算法来得出最优的策略，告诉程序如何去取这个Query语句的结果
- 表变更管理模块: 负责完成DDL、DML的Query，比如update，delete，insert，create table，alter table等语句的处理
- 表维护模块: 表的状态检查，错误修复
- 系统状态管理模块: 在客户端请求系统状态的时候，将各种系统状态返回给用户，比如show status,show variables等
- 表管理器: 维护.frm文件，以及一个Cache,缓存各个表的结构信息。
- 日志记录模块: 负责的整个系统的日志记录
- 复制模块: 分为Master模块和Slave模块，Master模块主要负责在Replication环境下读取Master节点的binary日志，以及和Slave端的IO线程交互。Slave模块有两个线程(IO线程和SQL线程)，IO线程从Master请求和接受binary日志，并写入本地relay log。SQL线程从relay log中读取日志事件，解析成可以执行的SQL语句，然后顺序执行，这样子从节点就和主节点的数据基本上保持一致。
- 存储引擎接口模块: 将各种数据处理高度抽象化，各种存储引擎产品实现接口即可，实现了MySQL特有的可插拔存储引擎。

2.存储引擎层(Storage Engine Layer),底层数据的存取都是在这一层做的，常用的存储引擎有InnoDB引擎和MyISAM引擎。


> 2.InnoDB引擎和MyISAM引擎的区别和应用场景

从这两种存储引擎的优缺点来谈区别以及应用场景
【InnoDB】
优点：支持事务，支持外键
缺点：占用磁盘多，读效率慢于MyISAM
【MyISAM】
优点：查询较快（读取数据不加锁），支持多种存储方式（静态表，压缩表等）
缺点：写入较慢（锁表），没有事务。

结论：支持事务选InnoDB,对读有要求的选MyISAM。

> 3.数据库索引的原理，分类

> 4.B+树有了解吗？为什么MySQL选用B+树做主要存储结构，为什么常用索引推荐使用B+树？为什么B+树可以减少磁盘IO?

> 4.如何避免索引失效？

> 5.有处理过MySQL优化吗？如何优化？

## 3. 框架
### 3.1 SpringFramework
> 1.简单谈谈Spring的IOC和AOP,都在哪些场景用到过？

> 2.Spring中Bean生命周期过程，以及两种作用域Singleton和Prototype有什么区别，应用场景有哪些？

> 3.AOP动态代理模式的两种类型,区别是什么？

> 4.Spring的事务的隔离级别

### 3.2 SpringMVC
> 1.SpringMVC的作用，与Struct的区别是什么？

> 2.SpringMVC的工作原理，都涉及到哪些类？

## 4. 计算机网络

> 1.谈谈TCP/IP模型

> 2.TCP的三次握手和四次挥手，为什么要这样子设计？

> 4.TCP协议传输的吞吐量，丢包问题

## 5. 算法

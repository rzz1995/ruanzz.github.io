---
title: Java后端面试题整理
date: 2018-12-18 21:47:55
tags: [interview,Java]
toc: true
---
![惠灵顿](/asset/img/interview-summary/Mac.jpeg)

<!-- more -->
这个冬天有点冷，冬天了还是储藏点过冬的粮食，用倒推法来总结相关知识，这篇博客将会长期更新，收集网上的面试题，然后自己整理思路，以后自己肯定会用的上，也为以后博客积累素材。

## Java基础
### 语法糖
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

### 集合
### 多线程
### IO
### JVM
>1.ClassLoader的种类，父子关系(不一定是继承)，双亲委派机制，Java为什么要用双亲委派机制?

> 2.如何打破双亲委派机制?什么时候需要打破双亲委派机制？有哪些框架打破了双亲委派机制？

> 3.JVM内存结构

JVM内存结构主要有三大块：堆内存、方法区和栈。
堆内存是JVM中最大的一块由年轻代和老年代组成，而年轻代内存又被分成三部分，Eden空间、From Survivor空间、To Survivor空间,默认情况下年轻代按照8:1:1的比例来分配
方法区存储类信息、常量、静态变量等数据，是线程共享的区域，为与Java堆区分，方法区还有一个别名Non-Heap(非堆)
栈又分为java虚拟机栈和本地方法栈,主要用于方法的执行

> 4.对象是否可GC

这个问题其实就是JVM如何判断对象是否需要被回收，JVM使用可达性算法来计算一个对象是否可达。
算法思路：通过一些被列为”GC Roots“的对象作为起始点，从这些点开始向下搜索，搜索走过的路径称为引用链，当一个对象到GC Roots没有任何引用链时，则说明对象需要被回收。
可以作为GC Roots对象包括以下几种：
- 虚拟机栈中引用的对象
- 方法区中静态属性引用的对象
- 方法区中常量引用的对象
- 本地方法栈中JNI引用的对象

> 5.Minor GC、Major GC和 Full GC

堆内存由年轻代和老年代组成。
从年轻代空间（包括 Eden 和 Survivor 区域）回收内存被称为 Minor GC。
Major GC 是清理老年代。
Full GC 是清理整个堆空间—包括年轻代和老年代。


## 数据库
### MySQL
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

## 框架
### SpringFramework
> 1.简单谈谈Spring的IOC和AOP,都在哪些场景用到过？

IOC是控制反转，就是对象的创建工作交给Spring容器来做，程序员不需要new出来，需要什么就跟Spring容器要，前提是容器里边有这个对象，底层实现是反射。AOP是面向切面编程，在记录日志，权限控制，事务方面都有应用，底层实现是动态代理。

> 2.Spring中Bean生命周期过程，以及两种作用域Singleton和Prototype有什么区别，应用场景有哪些？

生命周期:
1.Bean的建立：有BeanFactory读取Bean定义文件，并生成各个实例

2.Setter注入：执行Bean的属性依赖注入

3.BeanNameAware的setBeanName()：如果Bean实现了org.springframework.beans.factory.BeanNameAware接口，则执行其setBeanName()方法

4.BeanFactoryAware的setBeanFactory()：如果Bean实现了org.springframework.beans.factory.BeanFactoryAware接口，则执行其setBeanFactory()方法

5.BeanPostProcessors的processBeforeInitialization()：容器中如果有实现org.springframework.beans.factory.BeanPostProcessors接口的实例，则任何Bean在初始化之前都会执行这个实例的processBeforeInitializaton()方法

6.InitializingBean的afterPropertiesSet()：如果Bean类实现了org.springframework.beans.factory.InitailizingBean接口，则执行去afterPropertiesSet()方法

7.Bean定义文件中定义init-method：如果在xml文件中定义一个Bean的时候指定了init-methond，则执行这个方法，并且这个初始化方法是不能带有参数。

8.BeanPostProcessors的processAfterInitializaton()：容器中如果有实现org.springframework.beans.factory.BeanPostProcessors接口的实例，则去执行processAfterInitialization()方法

9.DisposableBean的destroy()方法：在容器关闭时，如果Bean实现了org.springframework.beans.factory.DisposableBean接口，则执行它的destroy()方法

10.Bean定义文件中定义destroy-method：在容器关闭时，执行destroy-method()方法，并且这个方法是不带参数的

作用域：
【singleton】：单例，每次访问都是返回同一个实例，Spring默认，无状态的Bean都应该使用Singleton，比如需要回收的重要资源(数据库连接池，线程池)
【prototype】：多例，每次访问都会进行new操作，Spring不会对Bean的整个生命周期进行负责，具有prototype的作用域的Bean创建后交由调用者负责销毁对象回收资源，有状态的Bean应该配置成prototype



> 3.AOP动态代理模式的两种类型,区别是什么？

AOP动态代理有两种：JDK代理和CGLIB代理，JDK代理只能对实现了接口的类进行代理，而不能针对未实现接口的类。CGLIB代理是针对类(未使用final修饰)实现代理,主要是对指定的类生成一个子类，覆盖其中的方法，底层是使用ASM字节码生成框架，使用字节码技术生成代理类。
Spring是怎么选择使用哪种代理的呢？如果一个Bean实现了接口，那么默认使用JDK代理，当Bean没有实现接口时，Spring默认使用CGLIB代理。

> 4.Spring事务的特性，隔离级别，传播行为

特性：
1.原子性(Atomicity)： 事务的不可分割性
2.一致性(Consistency)：事务执行前后数据的完整性保持一致
3.隔离性(Isolation)：事务执行过程中，不应该受到其他事务的干扰
4.持久性(Durability)：事务一旦结束，数据就持久到数据库

如果事务不考虑隔离性，就会引发安全性问题。比如：
1.脏读：一个事务读到了另一个事务未提交的数据
2.不可重复读：一个事务读到了另一个事务已经提交的update数据导致多次查询结果不一致
3.虚幻读：一个事务读到了另一个事务已经提交的insert的数据导致多次查询结果不一致

为了解决这个问题，引出事务隔离级别：
1.默认(default)：默认的隔离级别，使用数据库的默认隔离级别
2.未提交读(read uncommited)：脏读，不可重复读，虚幻读都有可能发生
3.已提交读(read commited)：避免脏读，但是不可重复读和虚幻读还是有可能发生
4.可重复读(repeatable read)：避免脏读和不可重复读，但是虚幻读还是有可能发生
5.串行化(Serializable)：避免上述所有读问题

MySQL默认的隔离级别是可重复读，Oracle默认的隔离级别是已提交读

事务的传播行为：
1.保证同一个事务中
PROPAGATION_REQUIRED：支持当前事务，如果不存在就新建一个
PROPAGATION_SUPPORTS：支持当前事务，如果不存在，就不使用事务
PROPAGATION_MANDATORY：支持当前事务，如果不存在就抛异常
2.保证没有在同一个事务中
PROPAGATION_REQUIRED_NEW：如果有事务存在，挂起当前事务，创建一个新的事务
PROPAGATION_NOT_SUPPORTED：以非事务方式运行，如果有事务存在，挂起当前事务
PROPAGATION_NEVER：已非事务方式运行，如果有事务存在，则抛出异常
PROPAGATION_NESTED：如果当前事务存在，则已嵌套事务执行

### SpringMVC
> 1.SpringMVC的作用，与Struct的区别是什么？

SpringMVC是一个基于请求驱动的Web框架，使用了前端控制器模式来设计，再根据请求映射规则分发给相应的页面控制器进行处理。简单点就是处理http请求和响应。
区别：
- SpringMVC是基于Servlet来实现，Struct是基于Filter来实现的。
- SpringMVC是基于方法级别的拦截，一个方法对应一个request上下文，Controller Bean默认是sigleton的，只会创建一个Controller，但是没有共享的属性，所以是线程安全。Struct是基于类级别的拦截，每次请求都会创建一个Action，Action Bean在Spring容器中是prototype的，通过setter方法将request注入到属性当中。

> 2.SpringMVC的工作原理，都涉及到哪些类？

1.用户发送请求至前端控制器DispatherServlet

2.DispatherServlet接收到请求之后调用HandlerMapping处理映射器

3.处理映射器找到具体的处理类(xml配置，注解)，生成处理器对象及处理拦截器一并返回给
DispatherServlet

4.DispatherServlet调用HandlerAdapter处理适配器

5.HandlerAdapter经过适配调用具体的处理器(Controller)

6.Controller执行完成返回ModelAndView对象

7.HandlerAdapter将ModelAndView返回给DispatherServlet

8.DispatherServlet将ModelAndView对象返回给ViewResolver视图解析器

9.ViewResolver解析后返回具体的View

10.DispatherServlet根据View进行渲染视图

11.DispatherServlet响应用户

## 计算机网络

> 1.谈谈TCP/IP模型

> 2.TCP的三次握手和四次挥手，为什么要这样子设计？

> 3.TCP协议传输的吞吐量，丢包问题

> 4.浏览器的一个HTTP请求到后端的一个大致过程是怎样的？

1.利用DNS进行域名解析(先找本地hosts再找运营商的DNS服务器)
2.发起TCP三次握手
3.建立TCP连接之后发起HTTP请求
4.服务器响应HTTP请求，浏览器得到HTML代码
5.浏览器解析HTML代码，并请求HTML代码中的资源(csc,js,image)
6.浏览器对页面进行渲染

## 算法

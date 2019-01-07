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

> 5.说说你对面向对象、封装、继承、多态的理解。

- 封装 隐藏具体的实现细节，并且明确标识允许外部使用的成员方法和成员变量，防止数据被破坏。
- 继承 子类继承父类，拥有父类除private修饰的所有成员变量和成员方法，并且可以在父类基础上进行扩展，实现了代码的重用。
- 多态 一个接口有多个子类或者多个实现类，在运行期间(非编译期间)才决定所引用的对象的实际类型，再根据其实际的类型调用其对应的方法，也就是常说的”动态绑定“。有三个条件：继承、重写、向上转型。因此面试的时候往往就是说说多态的理解，这样子面向对象编程的精髓基本都会涉及了。
(1) 继承： 子类继承父类或者实现父类
(2) 重写： 子类重写从父类继承过来的方法
(3) 向上转型： 父类引用指向子类

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

这个问题可以从OSI七层模型谈起引出TCP/IP五层模型，OSI是学术定义，但是不实用，工业界使用的是TCP/IP。
OSI七层模型:
- 应用层 针对特定的协议，为应用程序做服务,比如SMP,POP3,SSH,FTP等协议。
- 表示层 负责数据格式的转换，把不同表现形式的信息转换成适合网络传输的格式。
- 会话层 负责建立和断开通信连接，什么时候建立连接，什么时候断开连接以及保持多久的连接。
- 传输层 在两个通信节点之间负责数据的传输，起着可靠传输的作用。(运行在这一层的设备有四层交换机，四层路由器)
- 网络层 路由选择，在多个网络之间转发数据包，负责将数据包传送到目的地址。(运行在这一层的设备有路由器，三层交换机)
- 数据链路层 负责物理层面互联设备之间的通信传输，比如一个以太网相连的两个节点之间的通信，是数据帧与1、0比特流之间的转换。(运行在这一层的设备是网桥、以太网交换机、网卡)
- 物理层 主要是1、0比特流与电子信号的高低电平之间的转换。(运行在这一层的设备是中继器、双绞线)

TCP/IP五层模型:
七层模型中的应用层、表示层、会话层在这个模型中都属于应用层，其他四层不变。

> 2.TCP的三次握手和四次挥手，为什么要这样子设计？

【三次握手】
1.客户端发起连接请求，发送SYN=1和客户端序号c给服务器端，同时进入SYN_SENT状态。
2.服务端收到SYN后需要作出确认，于是发送ACK=1,同时自己也发送SYN=1,服务端序号s，同时发送确认号c+1(表示下一个接受序号),进入SYN_RCVD状态。
3.客户端收到服务端的SYN和ACK,作出确认，发送ACK=1，以及序号c+1的数据，同时发送确认号s+1(表示客户端下一个要接收的序号)。此时客户端和服务端进入ESTABLISHED状态，确认过眼神，状态已经建立。

【四次挥手】
1.客户端发起断开请求，发送FIN=1和序号c给服务端，客户端进入FIN-WAIT-1状态。
2.服务端收到FIN后作出确认，发送ACK=1和服务端序号s,确认号c+1(表示下一个接收序号),服务端此时还可以向客户端发送数据,服务端进入CLOSE-WAIT状态，客户端进入FIN-WAIT-2状态。
3.服务端没有数据发送时，它向客户端发送FIN=1,ACK=1,请求断开连接，同时发送服务端序号s，确认号c+1,服务端进入LAST-ACK状态。
4.客户端收到之后进行确认，发送ACK=1,以及序号c+1和确认号s+1。客户端进入TIME-WAIT状态，客户端需要等待2MSL,确保服务端收到了ACK，若这期间没有服务端的消息，便可认为服务端收到了确认，此时可以断开连接。服务端和客户端进入CLOSED状态。

![TCP三次握手四次挥手](/asset/img/interview-summary/tcp_shake_hand.jpg)

为什么这样子设计呢？握手两次行不行？

两次握手的话，只要服务端发出确认就建立连接了。有一种情况是客户端发出了两次连接请求，但由于某种原因，使得第一次请求被滞留了。第二次请求先到达后建立连接成功，此后第一次请求终于到达，这是一个失效的请求了，服务端以为这是一个新的请求于是同意建立连接，但是此时客户端不搭理服务端，服务端一直处于等待状态，这样就浪费了资源。假设采用三次握手，由于服务端还需要等待客户端的确认，若客户端没有确认，服务端就可以认为客户端没有想要建立连接的意思，于是这次连接不会生效。

四次握手，为什么客户端发送确认后还需要等待2MSL?

因为第四次握手客户端发送ACK确认后，有可能丢包了，导致服务端没有收到，服务端就会再次发送FIN = 1，如果客户端不等待立即CLOSED，客户端就不能对服务端的FIN = 1进行确认。等待的目的就是为了能在服务端再次发送FIN = 1时候能进行确认。如果在2MSL内客户端都没有收到服务端的任何消息，便认为服务端收到了确认。此时可以结束TCP连接。

> 3.有了传输层为什么还要网络层？

网络层是针对主机与主机之间的服务。而传输层针对的是不同主机进程之间的通信。网络层负责将数据包从源IP地址转发到目标IP地址，而传输层负责将数据包再递交给主机中对应端口的进程。

> 4.TCP序号的作用，怎么样保证可靠传输？

序号和确认号是实现可靠传输的关键。
序号-当前数据包的首个字节的顺序号，确认号-表示下一个想要接收的字节序号，并且已经正确收到确认号之前的所有字节。
通信双方通过序号和确认号来判断数据是否丢失，是否按顺序到达，是否冗余等等，如果丢失了就重传，如果冗余了就丢弃，换句话说，序号，确认号和重传机制保证了数据不丢失、不重复。

> 5.TCP和UDP的区别

- TCP面向连接，传输数据之前需要建立会话，UDP是无连接的
- TCP是可靠传输，UDP只负责发送数据，不保证接收方是否接收，不保证可靠
- TCP面向字节流，UDP面向报文
- TCP只支持点到点通信，UDP支持点到点、点到面、面到面的通信


> 6.浏览器的一个HTTP请求到后端的一个大致过程是怎样的？

1.利用DNS进行域名解析(先找本地hosts再找运营商的DNS服务器)
2.发起TCP三次握手
3.建立TCP连接之后发起HTTP请求
4.服务器响应HTTP请求，浏览器得到HTML代码
5.浏览器解析HTML代码，并请求HTML代码中的资源(csc,js,image)
6.浏览器对页面进行渲染

## 算法

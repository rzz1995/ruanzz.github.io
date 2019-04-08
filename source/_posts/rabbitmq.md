---
title: RabbitMQ
tags: [MQ]
toc: true
date: 2019-04-01 22:42:42
category: Web
---
目前大部分的后台系统都引入了MQ，不管是ActiveMQ还是RabbitMQ，亦或是RocketMQ、Kafka等等，MQ的主要作用是对系统进行解耦、异步化以及削峰，解耦应该是最常用的场景，本文将围绕这一业务场景以RabbitMQ作为案例来进行阐述。
<!-- more -->

## 认识MQ
MQ的全称为Message Queue，中文翻译为消息队列，Java中定义了一系列的接口规范，这套规范叫做JMS，像ActiveMQ就是遵循JMS规范，我之前也用过一段时间的ActiveMQ，由于系统没有什么并发压力，数据量也不大，ActiveMQ确实也挺适合的，但是ActiveMQ更新速度太慢了，我想主要的原因是官方觉得比较成熟了吧。

接下来讲讲今天的主角，RabbitMQ也属于消息队列，但是它是用Erlang写的，所以谈不上遵循JMS规范，RabbitMQ遵循的是AMQP协议，AMQP协议是一个提供统一消息服务的的应用层标准的消息队列协议，不受语言限制，由于底层使用的是专门面向并发的Erlang语言，所以RabbitMQ的吞吐量非常优秀，而且目前社区非常活跃，基本上现在的微服务系统都是通过集成RabbitMQ来作为消息传递的。

这两者有什么区别呢？JMS定义了统一的接口来对消息操作，而且限定语言为Java；AMQP只是协议，不规定实现方式，不受语言限制，通过规定协议来统一数据交互的格式。

至于MQ的选型本文不做讨论，本文的主角是RabbitMQ，前面说过，MQ的常用场景是用来解耦，下面来看看两个我工作中的真实场景。

我是做云平台业务的，当底层对云资源做了修改之后，云平台是不知道底层做了修改，这时候怎么能让云平台知道呢？一般有两种方案，一种是底层对云资源做了修改之后，调云平台的修改接口，还有一种就是底层做了修改之后发布消息到消息队列，云平台去订阅消息。第二种方案很好的将云平台和底层进行了解耦，这种场景不是很频繁，底层的东西不会经常变动，这个是不符合原来的平台的设计原则的，消息吞吐量不大，ActiveMQ完全能够胜任。

还有一个业务场景是的云平台部署了一个云资源，部署完成了之后底层需要告知云平台已经部署完成，云平台才能做后续的事情，比如结束相关订单，生成服务实例来进行计费，报表开始将该资源纳入统计，通知监控平台纳管该资源进行监控，发送邮件，短信通知等等一系列相关的事情，如果都是在单体系统中还好，大不了都调一遍接口，或者和上一个场景中一样直接使用ActiveMQ，分不同的主题来订阅到不同的消息触发不同的逻辑操作，如果使用ActiveMQ的话这个时候消息吞吐量是相对比较大的，特别是目前架构已经演变成微服务架构了，不应该还采用之前的技术，应该全面拥抱微服务全套解决方案，微服务主要用的MQ就是RabbitMQ，而且RabbitMQ中的高级消息模型就可以满足我们上面的第二个业务场景，吞吐量也是完全Hold的住。

## 安装配置

因为RabbitMQ是有Erlang写的，安装之前需要安装相关依赖，这个可以参考官方文档来进行安装，我的是OS X系统，官方推荐直接通过`brew install rabbitmq`即可，这个最好要打开VPN，因为有些资源是挂在国外，brew已经处理好了依赖关系，等待安装完成即可。

安装完成之后通过命令`brew services start rabbitmq`，启动成功之后访问`http://127.0.0.1:15672`，使用默认用户名密码`guest/guest`登录，guest这个账户是RabbitMQ默认的管理员账户，拥有最高的权利，生产环境一般会新添加一个用户，赋予新用户管理员权限，然后把guest这个默认用户删除掉，这里只是我本地的环境，不用考虑安全因素。

接下来我们创建一个demo用户，并且给demo用户一个虚拟主机，这个虚拟主机待会试验的时候会用到。

![添加用户](/asset/img/rabbitmq/add_user.png)
![添加host](/asset/img/rabbitmq/add_host.png)

然后去用户详情或者host详情里边做好两者的关联，这里从用户这里进入

![添加关联](/asset/img/rabbitmq/user_host.png)

这样子我们基本就完成了前期准备，下面来介绍消息模型。

## 消息模型

### 基本消息模型

基本消息模型如下图所示，生产者将消息发送到消息队列中，消费者从消息队列获取消息，消息存储在队列里。
![基本消息模型](/asset/img/rabbitmq/model1.png)

Consumer:
```Java
public class Consumer {

  private final static String QUEUE_NAME = "simple_queue";

  public static void main(String[] argv) throws Exception {
    // 1.获取到连接
    Connection connection = ConnectionUtil.getConnection();
    // 2.创建通道
    Channel channel = connection.createChannel();
    // 3.声明队列
    channel.queueDeclare(QUEUE_NAME, false, false, false, null);
    // 4.定义队列的消费者
    DefaultConsumer consumer = new DefaultConsumer(channel) {
      // 获取消息，并且处理，这个方法类似事件监听，如果有消息的时候，会被自动调用
      @Override
      public void handleDelivery(String consumerTag, Envelope envelope, BasicProperties properties,
          byte[] body) throws IOException {
        // body 即消息体
        String msg = new String(body);
        System.out.println(" [Simple] received : " + msg + "!");
        // 手动进行ACK
        //channel.basicAck(envelope.getDeliveryTag(), false);
      }
    };
    // 监听队列，第二个参数：是否自动进行消息确认,false要进行手动确认，保证消息的可靠性
    channel.basicConsume(QUEUE_NAME, true, consumer);
  }

}
```


Producer：
```Java
public class Producer {

  private final static String QUEUE_NAME = "simple_queue";

  public static void main(String[] argv) throws Exception {
    // 1.获取到连接
    Connection connection = ConnectionUtil.getConnection();
    // 2.创建通道
    Channel channel = connection.createChannel();
    String message = "Hello World!";
    // 3.向队列发送消息
    channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
    System.out.println(" [Simple] Sent '" + message + "'");
    // 4.关闭通道和连接
    channel.close();
    connection.close();
  }

}
```
Console:
![发送](/asset/img/rabbitmq/model1_send.png)

![接收](/asset/img/rabbitmq/model1_recv.png)

RabbitMQ中为了确保消息已经被消费了，会有一个确认机制来控制，可以设置为自动确认，如果想自己控制，可以设置为false之后手动进行确认，总之，消息确认机制从消费者端保证了消息被成功消费。

### work消息模型

当生产者产生消息比较多的时候，这就造成队列里边堆积了大量消息，这对系统来说是一个潜在的负担，这个时候就可以考虑work消息模型，多个消费者订阅同一个消息队里，多个消费者共享消息队列里的任务，但是一个消息只能由一个消费者消费。
![work消息模型](/asset/img/rabbitmq/model2.png)

代码基本上和基本消息类型大同小异，完整代码见仓库中work包，生产者发送了50个任务，两个消费者消费消息,但是不是轮流来消费，看处理能力，只保证一个消息只会被消费一次！
Consumer-1:
![Consumer-1](/asset/img/rabbitmq/model2_recv1.png)
Consumer-2:
![Consumer-2](/asset/img/rabbitmq/model2_recv2.png)
Producer:
![Producer](/asset/img/rabbitmq/model2_send.png)

### 发布/订阅模型

生产者产生消息，将消息发送到交换机，每个消费者都有自己的队列，队列连接到交换机，由交换机决定发送到哪个队列中，这样子可以实现一条消息可以被多个消费者消费。
![发布订阅模型](/asset/img/rabbitmq/model3.png)

完整代码见仓库中publish包，两个消费者所在的队列绑定到同一个交换机上，生产者发送消息到交换机的时候两个队列都会收到消息，如果没有队列连接到交换机，那么消息将丢失。
Consumer-1:
![Consumer-1](/asset/img/rabbitmq/model3_recv1.png)
Consumer-2:
![Consumer-2](/asset/img/rabbitmq/model3_recv2.png)
Producer:
![Producer](/asset/img/rabbitmq/model3_send.png)

 
### 路由模型
  
生产者产生消息，发送消息到交换机，交换机类为direct，这样子交换机就会根据配置的路由向指定的队列发送消息，而不会像所有连接到交换机的队列都发送消息，所以消息队列需要绑定一个routing key,生产发送消息的时候需要指定routing key，这样子就可以匹配上了。
![路由模型](/asset/img/rabbitmq/model4.png)
 
完整代码见仓库中route包，两个消费者所在队列绑定到同一个交换机上，并且指定了routingKey，如图所示
![绑定形式](/asset/img/rabbitmq/model4_bind.png)
生产者先发送routingKey为update，然后发送routingKey为delete,如图所示，与我们预期的结果一样。
Consumer-1:
![Consumer-1](/asset/img/rabbitmq/model4_recv1.png)
Consumer-2:
![Consumer-2](/asset/img/rabbitmq/model4_recv2.png) 

 
### 主题模型
    
路由模型是主题模型的一个特例，路由模型中的routing key是一个全匹配的key值，主题模型中key支持通配符，其中`#`匹配一个或多个值，`*`匹配不多不少一个词。
![主题模型](/asset/img/rabbitmq/model5.png)
完整代码见仓库中topic包，两个消费者所在队列绑定到同一个交换机，Consumer-1的routingKey为item.#,Consumer-2的routingKey为item.*，如图所示
![绑定形式](/asset/img/rabbitmq/model5_bind.png)
生产者先发送routingKey为item.delete，然后发送routingKey为item.delete.test,如图所示，与我们预期的结果一样。
Consumer-1:
![Consumer-1](/asset/img/rabbitmq/model5_recv1.png)
Consumer-2:
![Consumer-2](/asset/img/rabbitmq/model5_recv2.png) 

## SpringBoot集成

SpringBoot是有AMQP-starter的，引入SpringBoot项目即可，然后在application.yml中配置RabbitMQ的信息，接下来就是写一个listener即可，真的是太方便了，完整代码见仓库spring包，运行ListenerTest即可验证。
```Java
@Component
public class Listener {

  @RabbitListener(bindings = @QueueBinding(
      value = @Queue(value = "exchange.spring.queue", durable = "true"),
      exchange = @Exchange(
          value = "exchange.spring",
          ignoreDeclarationExceptions = "true",
          type = ExchangeTypes.TOPIC
      ),
      key = {"#.#"}))
  public void listen(String msg) {
    try {
      TimeUnit.MINUTES.sleep(1);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
    System.out.println("接收到消息：" + msg);
  }

}
```


最后，附上仓库地址
<div class="github-widget" data-repo="ruanzz/RabbitMQ-Tutorial"></div>


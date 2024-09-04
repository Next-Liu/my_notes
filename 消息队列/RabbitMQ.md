### 一.功能

削峰填谷：用户----->MQ------> 系统------>MySQL(MQ能够接受的并发请求比MySQL多)

任务解耦

提高吞吐量、并发量



生产者不需要等待消费者的响应

允许短暂不一致

#### AMQP、JMS

AMQP是一个提供统一消息服务的应用层标准**高级消息队列协议**,是一个通用的**应用层协议。**

消息发送与接受的双方遵守这个协议可以实现异步通讯。这个协议约定了消息的格式和工作方式。

JMS是**Java消息服务应用程序接口**，是一个Java平台中面向消息中间件的API。

### 二.RabbitMQ简介

RabbitMQ是一个实现了AMQP（Advanced Message Queuing Protocol）高级消息队列协议的消息队列服务，用**Erlang**语言。

生产者先绑定channel，一个通道有不同的消息队列，消费者指定对应通道

![](D:\学习笔记\消息队列\pictures\Snipaste_2024-03-10_13-52-25.png)

**Exchange**
 交换器，用来接收生产者发送的消息并将这些消息路由给服务器中的队列。

**Binding**
 绑定，用于消息队列和交换器之间的关联。一个绑定就是基于路由键将交换器和消息队列连接起来的路由规则，所以可以将交换器理解成一个由绑定构成的路由表。

**Virtual Host**
 虚拟主机，表示一批交换器、消息队列和相关对象。虚拟主机是共享相同的身份认证和加密环境的独立服务器域。每个 vhost 本质上就是一个 mini 版的 RabbitMQ 服务器，拥有自己的队列、交换器、绑定和权限机制。vhost 是 AMQP 概念的基础，必须在连接时指定，RabbitMQ 默认的 vhost 是 / 。	

提供了6种模式：

**简单模式**：一个生产者一个消费者一个消息队列

**work模式**：一个生产者多个消费者一个消息队列（共同消费）

![](D:\学习笔记\消息队列\pictures\Snipaste_2024-03-10_16-42-28.png)

**发布订阅模式：**一个生产者，一个交换机，生成多个相同的消费队列，多个消费者

消息发送给所有绑定到交换机的队列

![](D:\学习笔记\消息队列\pictures\Snipaste_2024-03-10_16-42-36.png)

**路由模式**：通过交换机定制规则把消息传到不同的消息队列中

![](D:\学习笔记\消息队列\pictures\Snipaste_2024-03-10_16-42-42.png)

队列与交换机之间的绑定不再是任意的了，而是要指定一个RoutingKey

**主题模式：**跟路由模式差不多，规则更详细

topic类型交换器让队列绑定Routing key时使用通配符

多个单词之间以"."分割 例如"item.insert"

"#":匹配一个或多个词

"*":匹配恰好一个词

如：

item.*:可以匹配item.insert或者item.insert.update

item.#:只能匹配item.insert

![](D:\学习笔记\消息队列\pictures\Snipaste_2024-03-10_16-42-46.png)

### 三.保证任务不重复执行、任务不丢失

### 四.安装RabbitMQ

使用docker

```
docker pull rabbitmq:management
docker run -d --hostname my-rabbit --name rabbit -p 15672:15672 -p 5672:5672 rabbitmq:management
--hostname：为容器定义的主机名
15672：在网址上访问
5672：java工程连接的端口
```

![](D:\学习笔记\消息队列\pictures\Snipaste_2024-03-10_14-44-00.png)

![](D:\学习笔记\消息队列\pictures\Snipaste_2024-03-10_14-44-52.png)

![Snipaste_2024-03-10_14-45-05](D:\学习笔记\消息队列\pictures\Snipaste_2024-03-10_14-45-05.png)

![Snipaste_2024-03-10_14-45-13](D:\学习笔记\消息队列\pictures\Snipaste_2024-03-10_14-45-13.png)

选择可操作的虚拟机

### 五.RabbitMQ入门

创建maven项目

导入jar包

```java
    <dependency>
      <groupId>com.rabbitmq</groupId>
      <artifactId>amqp-client</artifactId>
      <version>5.6.0</version>
    </dependency>
```

SpringBoot整合RabbitMQ

生产者工程：

1.在application.yml文件配置RabbitMQ相关信息

2.编写生产者配置类

![](D:\学习笔记\消息队列\pictures\Snipaste_2024-03-10_21-58-24.png)

![Snipaste_2024-03-10_22-00-45](D:\学习笔记\消息队列\pictures\Snipaste_2024-03-10_22-00-45.png)

消费者工程：

配置监听类，使用注解配置监听的队列

![](D:\学习笔记\消息队列\pictures\Snipaste_2024-03-10_22-50-18.png)
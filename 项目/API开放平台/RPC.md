### 问题

1.api-gateway项目中需要操作数据库，拿到accesskey对应的secretKey

2.api-gateway项目要调用之前写过的代码

### RPC

如何调用其他项目的方法？

1.http请求

2.RPC

3.公共代码打个jar包，其他项目去引用



![](D:\学习笔记\项目\pictures\Snipaste_2024-07-17_01-27-02.jpg)

### Dubbo框架(RPC实现)

[3 - 基于 Spring Boot Starter 开发微服务应用 | Apache Dubbo](https://cn.dubbo.apache.org/zh-cn/overview/mannual/java-sdk/quick-start/spring-boot/)

```java
<dependency>
       <groupId>org.apache.dubbo</groupId>
       <artifactId>dubbo</artifactId>
       <version>3.0.9</version>
 </dependency>
 <dependency>
       <groupId>com.alibaba.nacos</groupId>
       <artifactId>nacos-client</artifactId>
       <version>2.1.0</version>
 </dependency>
```


**单体架构：**

将业务所有功能集中在一个项目中开发，打成一个包部署

**微服务架构：**

首先是服务化，就是将单体架构中的功能模块从单体应用中拆分出来，独立部署为多个服务。

### 服务拆分原则：

**大多数小型项目来说，一般是先采用单体架构**，随着用户规模扩大、业务复杂后**再逐渐拆分为**微服务架构**。这样初期成本会比较低，可以快速试错。但是，这么做的问题就在于后期做服务拆分时，可能会遇到很多代码耦合带来的问题，拆分比较困难。

而对于一些大型项目，在立项之初目的就很明确，为了长远考虑，在架构设计时就直接选择微服务架构。虽然前期投入较多，但后期就少了拆分服务的烦恼（**前难后易**）

#### 怎么拆：

- **高内聚**：每个微服务的职责要尽量单一，包含的业务相互关联度高、完整度高。
- **低耦合**：每个微服务的功能要相对独立，尽量减少对其它微服务的依赖，或者依赖接口的稳定性要强。

横向拆分：按照项目的功能模块拆分：用户、支付、购物车

纵向拆分：抽取各个功能模块的公共服务

### 服务调用

把原本本地方法调用，改造成跨微服务的远程调用（RPC，即**R**emote **P**roduce **C**all）。

使用RestTemplate实现HTTP请求的发送。

```java
// TODO 1.获取商品id
    Set<Long> itemIds = vos.stream().map(CartVO::getItemId).collect(Collectors.toSet());    
// List<ItemDTO> items = itemService.queryItemByIds(itemIds);
    // 2.1.利用RestTemplate发起http请求，得到http的响应
    ResponseEntity<List<ItemDTO>> response = restTemplate.exchange(
            "http://localhost:8081/items?ids={ids}",
            HttpMethod.GET,
            null,
            new ParameterizedTypeReference<List<ItemDTO>>() {  //返回值类型
            },
            Map.of("ids", CollUtil.join(itemIds, ","))  //url占位符的参数 ids是key
    );
// 2.2.解析响应
    if(!response.getStatusCode().is2xxSuccessful()){
        // 查询失败，直接结束
        return;
    }
    List<ItemDTO> items = response.getBody();
    if (CollUtils.isEmpty(items)) {
        return;
    }
```

### 服务治理

- item-service这么多实例，cart-service如何知道每一个实例的地址？

- http请求要写url地址，`cart-service`服务到底该调用哪个实例呢？

- 如果在运行过程中，某一个`item-service`实例宕机，`cart-service`依然在调用该怎么办？

- 如果并发太高，`item-service`临时多部署了N台实例，`cart-service`如何知道新实例的地址？

  ![](D:\学习笔记\微服务\pictures\Snipaste_2024-03-25_20-21-57.png)


- 服务提供者会定期向注册中心发送请求，报告自己的健康状态（心跳请求）
- 当注册中心长时间收不到提供者的心跳时，会认为该实例宕机，将其从服务的实例列表中剔除Nacos

### Nacos注册中心

启动Nacos

#### 服务注册

引入依赖

```java
<!--nacos 服务注册发现-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

配置Nacos

```java
spring:
  application:
    name: item-service # 服务名称
  cloud:
    nacos:
      server-addr: 192.168.150.101:8848 # nacos地址
```

启动服务实例（同一个服务可以启动多个，即有多个实例）

#### 服务发现和负载均衡

**引入依赖**

```java
<!--nacos 服务注册发现-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

**配置Nacos地址**

```java
spring:
  cloud:
    nacos:
      server-addr: 192.168.150.101:8848
```

**服务发现**

服务调用者订阅服务提供者提供的服务

服务提供者有多个实例，而真正发起调用时只需要知道一个实例的地址。

因此，服务调用者必须利用负载均衡的算法，从多个实例中挑选一个去访问。常见的负载均衡算法有：

- 随机

- 轮询

- IP的hash

- 最近最少访问

  使用DiscoveryClient来获取服务

DiscoveryClient、RestTemplate具体使用看笔记

![](D:\学习笔记\微服务\pictures\Snipaste_2024-03-25_21-16-51.png)

### OpenFeign

声明式的http客户端

**引入依赖**

```java
  <!--openFeign-->
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-openfeign</artifactId>
  </dependency>
  <!--负载均衡器-->
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-loadbalancer</artifactId>
  </dependency>
```

**启用OpenFeign**

![](D:\学习笔记\微服务\pictures\Snipaste_2024-03-26_11-07-43.png)

**编写OpenFeign客户端**

![](D:\学习笔记\微服务\pictures\Snipaste_2024-03-26_11-08-10.png)

Feign底层发起http请求，依赖于其它的框架。其底层支持的http客户端实现包括：

- HttpURLConnection：默认实现，不支持连接池
- Apache HttpClient ：支持连接池
- OKHttp：支持连接池

通常会使用带有连接池的客户端来代替默认的HttpURLConnection。使用OK Http.

**引入依赖**

```
<!--OK http 的依赖 -->
<dependency>
  <groupId>io.github.openfeign</groupId>
  <artifactId>feign-okhttp</artifactId>
</dependency>
```

开启连接池

```
feign:
  okhttp:
    enabled: true # 开启OKHttp功能
```

### 服务拆分问题

服务地址过多，前端不清楚请求谁、秘钥泄露风险

使用网关解决这些问题：

网关就是**网**络的**关**口。数据在网络间传输，从一个网络传输到另一网络时就需要经过网关来做数据的**路由和转发以及数据安全的校验**。

网关也相当于是一个微服务

![](D:\学习笔记\微服务\pictures\Snipaste_2024-03-26_13-43-18.png)

前端请求不能直接访问微服务，而是要请求网关：

- 网关可以做安全控制，也就是登录身份校验，校验通过才放行
- 通过认证后，网关再根据请求判断应该访问哪个微服务，将请求转发过去

新建服务

引入依赖

```java
 <!--网关-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>
        <!--nacos discovery-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <!--负载均衡-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-loadbalancer</artifactId>
        </dependency>
```

配置路由

```java
server:
  port: 8080
spring:
  application:
    name: gateway
  cloud:
    nacos:
      server-addr: 192.168.150.101:8848
    gateway:
      routes:
        - id: item # 路由规则id，自定义，唯一
          uri: lb://item-service # 路由的目标服务，lb代表负载均衡，会从注册中心拉取服务列表
          predicates: # 路由断言，判断当前请求是否符合当前规则，符合则路由到目标服务
            - Path=/items/**,/search/** # 这里是以请求路径作为判断规则
        - id: cart
          uri: lb://cart-service
          predicates:
            - Path=/carts/**
        - id: user
          uri: lb://user-service
          predicates:
            - Path=/users/**,/addresses/**
        - id: trade
          uri: lb://trade-service
          predicates:
            - Path=/orders/**
        - id: pay
          uri: lb://pay-service
          predicates:
            - Path=/pay-orders/**
```
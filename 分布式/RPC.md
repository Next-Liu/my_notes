### RPC的概念

​	RPC的全称是Remote Procedure Call，即远程过程调用。是指要跨机器而非本机，所以需要用到网络编程才能实现，但是不是只要通过网络通信访问到另一台机器的应用程序

​	**RPC框架能够帮助我们解决系统拆分后的通信问题，并且能让我们像调用本地一样去调用远程方法。**利用RPC我们不仅可以很方便地将应用架构从“单体”演进成“微服务化”，而且还能解决实际开发过程中的效率低下、系统耦合等问题，这样可以使得我们的系统架构整体清晰、健壮，应用可运维度增强。

一次RPC调用，本质就是服务消费者与服务提供者间的一次网络信息交换的过程。

### 对象在网络的传输

![](D:\学习笔记\分布式\pictures\Snipaste_2024-04-18_11-30-24.png)

#### 序列化方式

![](D:\学习笔记\分布式\pictures\Snipaste_2024-04-18_11-48-40.png)

序列化过程就是在读取对象数据的时候，不断加入一些特殊分隔符，这些特殊分隔符用于在反序列化过程中截断用。

- 头部数据用来声明序列化协议、序列化版本，用于高低版本向后兼容
- 对象数据主要包括类名、签名、属性名、属性类型及属性值，当然还有开头结尾等数据，除了属性值属于真正的对象值，其他都是为了反序列化用的元数据
- 存在对象引用、继承的情况下，就是递归遍历“写对象”逻辑

1.实现Serializable

2.JSON

3.Hession

Hessian协议要比JDK、JSON更加紧凑，性能上要比JDK、JSON序列化高效很多，而且生成的字节数也更小。

```java
Student student = new Student();
student.setNo(101);
student.setName("HESSIAN");

//把student对象转化为byte数组
ByteArrayOutputStream bos = new ByteArrayOutputStream();
Hessian2Output output = new Hessian2Output(bos);
output.writeObject(student);
output.flushBuffer();
byte[] data = bos.toByteArray();
bos.close();

//把刚才序列化出来的byte数组转化为student对象
ByteArrayInputStream bis = new ByteArrayInputStream(data);
Hessian2Input input = new Hessian2Input(bis);
Student deStudent = (Student) input.readObject();
input.close();

System.out.println(deStudent);
```

序列化方式的选择

1.序列化协议的通用性和兼容性：一个具有高通用性的序列化协议应该能够支持在不同的操作系统和编程语言之间传输数据。一个具有良好兼容性的序列化协议应该能够支持向后兼容，即新版本的代码能够理解和处理旧版本的数据格式。

2.空间开销：序列化之后的二进制数据占用的空间大小

3.安全性：JDK原生序列化方式有以下漏洞：**远程代码执行（RCE）**: 攻击者可以通过精心构造的恶意序列化数据，在反序列化时执行任意代码。这种情况下，攻击者可以完全控制程序的行为，可能导致敏感数据泄露、系统崩溃或其他严重后果。

### 常见的网络IO模型

阻塞IO：

​	应用进程发起IO系统调用后，应用进程被阻塞，转到内核空间处理。之后，内核开始等待数据，等待到数据之后，再将内核中的数据拷贝到用户内存中

IO多路复用：

​	多路指多个网络连接IO，复用指多个通道在一个复用器上

​	多个网络连接的IO可以注册到一个复用器（select）上，当用户进程调用了select，那么整个进程会被阻塞。同时，内核会“监视”所有select负责的socket，当任何一个socket中的数据准备好了，select就会返回。这个时候用户进程再调用read操作，将数据从内核中拷贝到用户进程。

RPC调用在大多数的情况下，**是一个高并发调用的场景**，考虑到系统内核的支持、编程语言的支持以及IO模型本身的特点，在RPC框架的实现中，在网络通信的处理上，选择IO多路复用的方式
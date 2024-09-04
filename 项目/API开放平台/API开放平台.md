# 项目背景

1.前端开发需要调用后端接口，否则只能mock数据

2.使用现成的系统的功能

  

防攻击

不能随便使用

统计调用次数

计费



# 项目介绍

做一个提供API接口调用的平台，用户可以注册登录，开通接口调用权限，用户可以使用接口，并且每次调用会进行统计，管理员可以发布接口，下线接口，以及可视化接口的使用情况。



![](D:\学习笔记\项目\pictures\Snipaste_2024-07-01_01-46-30.jpg)

# 数据库设计

![](D:\学习笔记\项目\pictures\Snipaste_2024-07-05_02-09-56.jpg)

![](D:\学习笔记\项目\pictures\Snipaste_2024-07-05_02-10-56.jpg)

![Snipaste_2024-07-05_02-11-14](D:\学习笔记\项目\pictures\Snipaste_2024-07-05_02-11-14.jpg)

# MybatisX-Generator插件的使用

![](D:\学习笔记\项目\pictures\Snipaste_2024-07-06_03-46-34.jpg)

![Snipaste_2024-07-06_03-46-41](D:\学习笔记\项目\pictures\Snipaste_2024-07-06_03-46-41.jpg)

# 前端和后端交互

前端接口调用：oneapi插件自动生成调用后端接口的前端代码

openapi的规范

![](D:\学习笔记\项目\pictures\Snipaste_2024-07-06_17-33-37.jpg)

![Snipaste_2024-07-06_17-36-15](D:\学习笔记\项目\pictures\Snipaste_2024-07-06_17-36-15.jpg)

![Snipaste_2024-07-06_17-36-32](D:\学习笔记\项目\pictures\Snipaste_2024-07-06_17-36-32.jpg)

生成文件如下：

![](D:\学习笔记\项目\pictures\Snipaste_2024-07-06_17-37-09.jpg)

![](D:\学习笔记\项目\pictures\Snipaste_2024-07-06_19-50-25.jpg)

F12勾选Fetch/XHR

![](D:\学习笔记\项目\pictures\Snipaste_2024-07-06_19-54-00.jpg)


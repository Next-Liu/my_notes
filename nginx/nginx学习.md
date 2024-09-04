### 一.安装

安装gcc环境

yum install gcc-c++

安装zlib

yum install -y zlib zlib-devel

安装openSSL

yum install -y openssl openssl-devel

**下载ngnix安装包**

**解压缩**

tar -zxvf ngnix-1.24.0.tar.gz

**进入 ngnix-1.24.0目录**

**执行./configure,创建Makefile文件**

编译 make

安装 make install

#### 启动和访问

上述操作将临时文件目录指定为/var/temp/nginx/client

故需要先创建 mkdir /var/temp/nginx/client -p

进入nginx目录下的sbin目录

cd /usr/local/nginx/sbin

启动nginx 

./nginx

./nginx -s stop

./nginx -s reload

查看

ps -aux | grep nginx

注意关闭防火墙

systemctl status firewalld.service
systemctl stop firewalld.service

### 二.静态网站的部署

在cd /usr/local/nginx/html目录下上传，若有多个项目，创建文件夹

### 三.配置虚拟主机

把一台物理服务器划分为多个虚拟服务器。

配置文件/usr/local/ngnix/conf/ngnix.conf

```nginx
 server {
        listen       80;
        server_name  localhost;
        location / {
            root   html/可以加上项目文件夹名;   #root代表ngnix的根路径 /usr/local/ngnix
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
  }
#可以配置多个server
```

### 四.反向代理和负载均衡

正向代理针对客户端，反向代理针对服务器

(1)正向代理和代理服务器

​	正向代理即通常所说的代理，用于代表**内部网络用户**向Internet上的服务器(或**称**外部服务器，通常为Web服务器)发出连接请求，并接收响应结果，执行该代理功能的服务器称为代理服务器。使用代理服务器访问外部网络时，客户端必须在局域网设置中指明代理服务器的地址以及要代理的服务的端口号。

(2)反向代理和代理服务器

​	反向代理的方向与正向代理相反，指代表外部网络用户向内部服务器发出请求**，即接收来自Internet上用户的连接请求**，**并将这些请求转发给内部网络上的服务器**，然后将从内部服务器上得到的响应返回给Internet上请求连接的客户：执行反向代理服务的服务器称为反向代理服务器，反向代理服务器对外部用户表现为一个服务器。

![](D:\学习笔记\java\pictures\Snipaste_2024-03-14_16-08-52.png)

项目部署到tomcat的root目录

启动tomcat sh bin/startup.sh

配置文件/usr/local/ngnix/conf/ngnix.conf

```nginx
#配置tomcat中项目的网址
upstream tomcat-travel(自己配置的项目名){
   server    ip:port;  #真实访问的网址
}
 server {
        listen       80;
        server_name  localhost;  #访问的网址，找proxy_pass中的内容
        location / {
            # root   html/可以加上项目文件夹名;   #root代表ngnix的根路径 /usr/local/ngnix
        	proxy_pass http://tomcat-travel; #不用通过root，通过代理转发
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
  }
```



负载均衡是指突然到达很多请求，一个tomcat压力太大，导致崩溃。可以将tomcat复制多份，增大吞吐量，将请求分摊到多个服务器。

把刚刚tomcat复制三份，修改端口为8080,8081,8082

修改tomcat配置文件的端口

配置文件

```nginx
upstream tomcat-travel(自己配置的项目名){
   server    ip:port weight=2; #weight相当于可以让更高性能的服务器更大概率处理请求，默认都是1
   server    ip:port;
   server    ip:port;
}
```


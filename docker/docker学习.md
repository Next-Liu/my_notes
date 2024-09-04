### 一.docker简介

docker可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何Linux机器上，也可以实现虚拟化

LXC(Linux Container容器)是一种内核虚拟化技术，可以提供轻量级的虚拟化，以便隔离进程和资源。

**每个容器只保留了linux系统运行当前项目的环境**

传统虚拟机，虚拟出一条硬件，运行一个完整的操作系统

容器内的应用直接运行在宿主机的内容

每个容器间相互隔离，每个容器内部都有一个属于自己的文件系统，互不影响



通过仓库，下载镜像。通过镜像，创建容器。通过容器安装应用程序。在容器中，进行操作。

```linux
 sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine       //卸载
sudo yum install -y yum-utils

sudo yum-config-manager  --add-repo    https://download.docker.com/linux/centos/docker-ce.repo
sudo yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
    
sudo yum install docker-ce docker-ce-cli containerd.io
sudo systemctl start docker //启动docker
```

### 二.容器操作

镜像是只读的，不可修改。

容器是可读可写的，可以修改。

**1.查看容器**

docker ps [-a]

不带 -a，显示正在运行的容器

带 -a  ,显示所有容器

**2.创建容器**

docker  run   -itd   --name   容器名称    镜像名   /bin/bash    

-p 指定容器的端口

创建一个新容器，运行容器。

 d代表后台启动，不带d直接进入容器

**3.关闭容器**

docker   stop    容器的id或name

**4.启动容器**

docker   start   容器的id或name

**5.重启docker**

docker   restart   容器的id或name

**6.进入容器**

docker  exec   -it   容器名称或id      /bin/bash

进入容器，在容器中进行你的操作（沙箱环境）。

**7.退出容器**

exit

**8.删除容器**

docker   rm   -f    容器名或id

### 三.镜像操作

**1.查看已有镜像**

docker  images

**2.查找中央仓库的镜像**

docker  search  镜像名

**3.拉取镜像**

docker  pull  镜像名:版本

**4.删除本地镜像**

docker  rmi  -f   镜像名

**5.拷贝操作**

docker cp 容器id:容器内路径 目的主机路径

docker cp 目的主机路径  容器id:容器内路径

### 四.网络映射

默认宿主机和容器网络是不通的。

在创建容器时，可以通过映射联通。

想要使得外界项目的请求使用容器内的tomcat发布

docker   run   -itd    **-p    宿主机ip:宿主机端口:容器端口**    --name  容器名   镜像名  /bin/bash

项目发布到tomcat下的webapp目录下

![](D:\学习笔记\docker\pictures\Snipaste_2024-03-07_16-01-33.png)

### 五.部署项目

**1.数据卷Volume：目录映射**

将宿主机的目录和容器的目录进行映射，共享存储空间。

**2.操作**

docker   run   -itd    -p    宿主机ip:宿主机端口:容器端口   **-v  宿主机目录:容器目录**  --name  容器名   镜像名  /bin/bash

![](D:\学习笔记\docker\pictures\Snipaste_2024-03-07_16-13-30.png)

#### 3、部署项目

第一步，创建容器，映射端口和目录

第二步，将项目传入宿主机对应的目录。

第三步，启动容器的tomcat

进入bin目录

sh.startup.sh

第四步，外网访问测试

### 六.安装镜像

#### 安装JDK

先 docker search jdk8

![](D:\学习笔记\docker\pictures\Snipaste_2024-03-08_19-28-07.png)

#### 安装MySQL

docker pull mysql

docker run -itd --name mysql8 -p 3307:3306 **-e MYSQL_ROOT_PASSWORD=123456** mysql

sudo docker exec -it mysql8 bash 进入容器

mysql -uroot -p123456连接数据库

#### 安装Nginx

sudo docker pull nginx

sudo docker run -itd -p 80:80 --name nginx1 nginx

curl:80 //访问测试

**查看nginx资源信息**

sudo docker exec -it nginx1 /bin/bash

/etc/nginx               #nginx的配置文件处

/usr/share/nginx   #nginx的共享资源处

**修改配置文件**

![](D:\学习笔记\docker\pictures\Snipaste_2024-03-08_20-07-19.png)

容器的配置文件拷贝到宿主机

sudo docker cp 51019c52ab4a:/etc/nginx/nginx.conf /usr/nginx

添加这些：

```javascript
    #gzip  on;

    #include /etc/nginx/conf.d/*.conf;
    server {
    listen       80;
    server_name  localhost;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
   }
server { ... }：这是一个 server 块，用于定义一个虚拟主机配置。在该 server 块中配置了一个 HTTP 服务，监听在 80 端口，并且将请求发送到 localhost 主机
location / { ... }：定义了一个 location 块，用于配置请求的处理规则。这里配置了当请求的 URI 为 / 时，将请求映射到指定的目录和默认文件。
index index.html index.htm;：指定了当请求的路径对应的目录时，Nginx 将首先查找 index.html 文件，如果不存在则查找 index.htm 文件作为默认文件返回给客户端。
```

共享卷

sudo docker run -itd -p 80:80 -v /usr/nginx/nginx.conf:/etc/nginx/nginx.conf -v /usr/nginx/html:/usr/share/nginx/html --name nginx2 nginx

### 七.发布springboot项目

构建springboot项目，数据库ip改为服务器ip

打包运行测试成功

编写DockerFile

```dockerfile
# DOCKER命令全部大写

FROM java:8

#MAINTAINER 维护者信息，可不写

MAINTAINER xiao

#ADD 文件放在当前目录下，拷过去会自动解压

ADD [上传的jar包名称] /自定义.jar

#EXPOSE 项目暴露的端口号

EXPOSE 80

ENTRYPOINT ["java"，"-jar","/自定义.jar"]
```

将打包好的jar包和DockerFile文件存到同一个文件夹下

构建镜像

docker build -t xx(镜像名):xx(版本)

发布运行

docker run -itd -p 8080(宿主):8080(项目绑定的端口) --name xx xx(镜像名):xx(版本) /bin/bash



`docker load`命令用于从文件加载镜像或容器。

包含了要加载的镜像或容器的归档文件（通常是以`.tar`为扩展名的文件）。

```linux
docker load -i /path/to/image_or_container.tar
此时镜像或者容器被加载到了本地Docker引擎，需要docker run执行
```


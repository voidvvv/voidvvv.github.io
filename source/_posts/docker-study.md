---
title: docker学习记录
date: 2023-12-18 19:19:31
categories:
- docker
tags:
- docker
- container
---

docker 学习
<!-- more -->

教程参考[B站狂神视频](https://www.bilibili.com/video/BV1og4y1q7M4?p=11)：
# 环境
linux CentOs7 系统内核3.0以上
# 安装
[参照官方文档](https://docs.docker.com/engine/install/centos/)
[Fedora] (https://docs.docker.com/engine/install/fedora/)

```shell
## 首先卸载老版本（如果有的话） 移除老版本
 $ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
               
 ## 安装需要的安装包
sudo yum install -y yum-utils
## 设置镜像仓库（国外 -- 不建议）
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
    
## 国内阿里云镜像(推荐)
sudo  yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/fedora/docker-ce.repo

## 或者
sudo dnf config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/fedora/docker-ce.repo


## 更细yum软件包索引
yum makecache fast

## 安装docker相关内容 docker-ce 社区办--推荐  ee--企业版 不推荐
sudo yum install docker-ce docker-ce-cli containerd.io

## 启动docker
sudo systemctl start docker
## 查看版本 可以查看是否安装成功
docker version
## 启动镜像并设置为开机自启
systemctl start docker.service
systemctl enable docker.service

## helloworld
sudo docker run hello-world
## 查看下载镜像
docker images

## 卸载docker
sudo yum remove docker-ce docker-ce-cli containerd.io
## 删除资源
sudo rm -rf /var/lib/docker
sudo rm -rf /var/lib/containerd

## 配置阿里云镜像加速

```

# 镜像基本命令
```shell
docker version
docker info
docker --help

```

[docker帮助文档官网](https://docs.docker.com/reference/)

# 镜像命令
---
docker images:查看当前所有的镜像
 docker images [OPTIONS] [REPOSITORY[:TAG]]
```shell
## 展示所有镜像
docker images -a 
## 只展示id
docker images -q
```
![在这里插入图片描述](demo1.png)
docker search -- 搜索镜像
```shell
## 搜索mysql镜像
docker search mysql

## 搜索
```
docker pull 下载镜像
```shell
docker pull mysql ## 下载mysql镜像

docker pull mysql:5.0 ## 指定版本下载mysql

## docker 镜像分层下载
```

docker rmi: 删除镜像
```shell
docker rmi imageId|imageName  根据镜像id或者名称删除镜像

docker rmi -f ${docker images -aq}  ## 批量删除。${} 是先查询出来，然后删除所查询的镜像

docker rmi -f name1 name2 name3  ## 删除多个镜像

```
# 容器命令
---
有了镜像才可以创建容器,这里下载centos容器并启动
```shell
## 下载镜像
docker pull centos

## 启动容器
docker run [可选参数] image

## 参数说明
--name 	名称
-d 		以后台方式运行
-i		使用交互方式运行，进入容器查看内容
-t
-p 		指定容器的端口  -p 8080:8080
	主机端口:容器端口
	容器端口
-P(大写) 随机指定端口
--rm 	运行完退出后会删除容器

docker run -it centos /bin/bash ## 启动并进入容器

exist ## 在容器中使用可以退出当前容器


## 列出所有正在运行的容器
docker ps
docker ps -a ## 包含所有当前运行以及曾经运行过得容器
docker ps -n=7  ## 显示最近创建的容器列表
docker ps -q ## 只显示容器编号

## 退出容器
exist ## 退出并停止
ctrl + P + Q ## 容器不停止，但是退出

## 删除容器
docker rm 容器id ## 根据id删除容器 加-f强制删除

docker rm  -f  [参数]  ## 类似于镜像删除
# 启动 停止 容器
docker start 容器id  ## 通过容器id启动容器
docker restart ## 重启容器
docker stop  ## 容器停止
docker kill ## 强制容器停止
```
# 其他常用命令
---
```shell
docker run -d centos ## 后台启动centos ，但是会自动停止。因为centos需要作为前台。像tomcat这种就不需要，直接后台就可以

## 查看日志
docker logs [OPTIONS] CONTAINER
-t ## 显示时间戳
-f ## 跟随输出显示
--details ## 指定显示行数 后面需要跟数字

## 查看容器中的进程命令
docker top CONTAINER [ps OPTIONS]

## 查看容器元数据
docker inspect

## 进入当前正在运行的容器
docker exec -it 容器id  # 进入容器后开启一个新的终端

docker attach 容器id ## 进入容器正在执行的终端

## 将容器内的文件拷贝到当前服务器主机上
docker cp 容器id:/home/text.java /home
## 将容器中/home/text.java的文件拷贝到当前主机上的/home文件夹,在容器停止时也可以拷贝

```
commit镜像
```shell
	## 跟git类似，相当于从镜像生成容器，然后更改容器，若想将容器此时的状态记录成镜像则需要执行以下命令
	docker commit -a "作者" -m "嗯update" 容器id  镜像名称:版本号
	## 几乎完全照搬git思路即可
	
```

# 容器数据卷
将容器内部的文件与当前系统中的文件绑定共享
```shell
## 将主机文件与容器内部某路径文件共享。两个对文件的操作均会同步，类似于双向绑定
docker run -it -v  主机目录文件:容器内部文件  mysql  /bin/bash 

## 查看
docker inspect 容器id

## 其中有一部分 mounts 就是挂载信息

## 挂载相关命令
docker volume --help

## 查看当前都有哪些挂载
docker volume ls  

## 想看某个挂载的具体数据
docker volume inspect 挂载名

## 其中有个数据为mountpoint，代表挂载的路径。
## 路径一般为：/var/lib/docker/volumes/挂载名/_data

## 尽量使用具名挂载
## 设置容器对于当前文件的权限
docker run -it -v  主机目录文件:容器内部文件:ro|rw
# ro: 只读
# rw: 可写

## 还可以指定容器挂载
docker run -it -name test --volumes-from 另一个容器名称  要启动的容器名称:版本号
```
## 具名挂载
```shell
## 指定主机目录
docker run -it -v  主机目录文件:容器内部文件  mysql  /bin/bash 
```
## 匿名挂载
```shell
## 不指定主机目录,会生成一串hash串代替文件名
docker run -it -v  主机目录文件:容器内部文件  mysql  /bin/bash 
```

# Dockerfile
用来构建docker镜像的构建文件.

![在这里插入图片描述](https://img-blog.csdnimg.cn/dea64458ca24474b89e2abfe7434bfc3.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAdm9pZHZ2dg==,size_20,color_FFFFFF,t_70,g_se,x_16)
需要知道，dockerfile中，#开头的语句默认为注释，不会被编译执行

文件字段：
```shell
FROM			#基础镜像
MAINTAINER		#维护者（邮箱）
RUN				# 需要执行什么东西
ADD				# 需要添加什么文件，如果是tar.gz会自动解压
WORKDIR			# 工作目录
VOLUME			# 挂载的目录，默认匿名挂载
EXPOSE			# 暴露的端口
CMD				# 指定容器启动时执行的命令，会跟在ENTRYPOINT 后面当做其参数
ENTRYPOINT 		# 启动时执行的命令
ENV				# 设置当前镜像内的环境变量
```
### 构建自己的镜像
1. 编写Dockerfile文件
```shell
FROM			JAVA:8
MAINTAINER		VOIVVVV(voidvvv@git.com)
WORKDIR			/usr/local
ADD				springboot_image_demo.jar /app.jar
CMD				["--server.port=8080"]
ENTRYPOINT 		["java","-jar","app.jar"]
```
springboot项目从github源码直接打包构建镜像
```shell
FROM maven:3.3-jdk-8

VOLUME /tmp

WORKDIR /code

# Prepare by downloading dependencies
ADD pom.xml /code/pom.xml  
#RUN ["mvn", "dependency:resolve"]
#RUN ["mvn", "verify"]

# Adding source, compile and package into a fat jar
ADD src /code/src
RUN ["mvn", "clean", "install"]

RUN ["ls", "/code/target"]
RUN ["pwd"]
RUN ["ls", "-ltrh", "/code/target/myspringboot.jar"]

EXPOSE 8080

ENTRYPOINT [ "java", "-jar", "/code/target/myspringboot.jar" ]
```


2. build 镜像
```shell
docker build -f Dockerfile文件名 -t 镜像名称:版本号 .(这个点代表构建所需的环境)

docker build -f Dockerfile文件名 -t 镜像名称:版本号 .

## 其中构建所需的环境使用一个点，代表使用当前文件夹下作为环境。Dockerfile中的命令比如文件添加复制什么的也都基于这里来拿.Dockerfile文件也是从这里读取的.
## 这个环境可以为 
	. 			当前文件夹下
	tar.gz		压缩包内的所有文件
	git仓库		指定某个仓库，还可以指定分支，文件夹，就以该仓库指定分支指定文件夹下的内容为环境构建镜像
	
```

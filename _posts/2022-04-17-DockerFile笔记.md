---
layout: mypost
title: DockerFile笔记
categories: [Docker Linux]
---

*#音量注意：没有调节音量的功能，播放前请注意设备音量*

<iframe src="//music.163.com/outchain/player?type=2&id=95475&auto=1&height=66" frameborder="0" width="100%" height="86px" ></iframe>


## 序

> 学习一下DockerFile怎么写

## 构建步骤

DockerFile是用来构建Docker镜像的构建文件，是由一系列命令和参数构成的脚本。

构建步骤：

- 编写DockerFile文件
- docker build 构建镜像
- docker run 运行镜像

DockerFile每条指令都是大写字母，后面至少跟随一个参数，每条指令都创建一个新的镜像层，然后 commit，堆叠成一层层的结构。

## DockerFile和镜像、容器的区别

- DockerFile是构建镜像的原材料，build 后会产生一个镜像

- 镜像是一个构建好的交付品，准备好了拿到各处去 run

- 容器是运行的状态，涉及到部署与运维，是直接提供服务的

## DockerFile的指令关键字

> FROM # 基础镜像，当前新镜像是基于哪个镜像的
>
> MAINTAINER # 镜像维护者的姓名混合邮箱地址
>
> RUN  # 容器构建时需要运行的命令
>
> EXPOSE # 当前容器对外保留出的端口
>
> WORKDIR  # 指定在创建容器后，终端默认登录的进来工作目录，一个落脚点
>
> ENV  # 用来在构建镜像过程中设置环境变量
>
> ADD  # 将宿主机目录下的文件拷贝进镜像且ADD命令会自动处理URL和解压tar压缩包
>
> COPY # 类似ADD，拷贝文件和目录到镜像中！
>
> VOLUME # 容器数据卷，用于数据保存和持久化工作
>
> CMD  # 指定一个容器启动时要运行的命令，dockerFile中可以有多个CMD指令，但只有最后一个生效！
>
> ENTRYPOINT # 指定一个容器启动时要运行的命令！和CMD一样，也有区别
>
> ONBUILD  # 当构建一个被继承的DockerFile时运行命令，父镜像在被子镜像继承后，父镜像的ONBUILD被触发

## 改写一个DockerFile

```bash
[root@ home]# mkdir dockerfile-test
[root@ home]# ls
dockerfile-test docker-test-volume
[root@ home]#
[root@ home]# vim mydockerfile-centos
[root@ home]# cat mydockerfile-centos
FROM centos
MAINTAINER test<a@a.com>

ENV MYPATH /usr/local
WORKDIR $MYPATH

RUN yum -y install vim
RUN yum -y install net-tools

EXPOSE 80 

CMD echo $MYPATH 
CMD echo "----------end--------" 
CMD /bin/bash
[root@ home]#
[root@ home]#
[root@ home]#
[root@ home]# docker build -f mydockerfile-centos -t mycentos:0.1 .
Sending build context to Docker daemon 6 .144kB
Step 1 /10 : FROM centos
---> 470671670cac
Step 2 /10 : MAINTAINER test<a@a.com>
---> Running in ac052943c151
Removing intermediate container ac052943c151
---> 9d37c7174860
Step 3 /10 : ENV MYPATH /usr/local
---> Running in a9d43e0b41bb
Removing intermediate container a9d43e0b41bb
---> 7a89b945c3a6
Step 4 /10 : WORKDIR $MYPATH
---> Running in b41f085b06bc
Removing intermediate container b41f085b06bc
---> 022384682f07
Step 5 /10 : RUN yum -y install vim
---> Running in 8a8d351ee43b
##省略##
Complete!
Removing intermediate container 8a8d351ee43b
---> 6f6449a55974
##省略##
Successfully built 6f6449a55974
Successfully tagged mycentos:0.1
[root@ home]#
[root@ home]#
[root@ home]#
[root@ home]# docker images
##省略##
[root@ home]# docker run -it mycentos:0.1
[root@ 6f6449a55974 local]# pwd
/usr/local
[root@ 6f6449a55974 local]# ifconfig

```

## CMD 和 ENTRYPOINT 的区别

假设编写了两个 DockerFile，一个最后一句是`CMD [ "ls", "-a" ]`，另一个最后一句是`ENTRYPOINT [ "ls", "-a" ]`，这时候我们运行时想加个`-l`参数，把指令变成`ls -al`，尝试`docker run mytest -l`，会发现情况一报错，而情况二正常。

这是因为 CMD 会被 docker run 之后的参数替换，而 ENTRYPOINT 会接收 docker run 之后的参数，形成新的命令组合。

## 自定义一个 docker 镜像

1. mkdir -p admin/build/tomcat

2. 在上述目录下 touch read.txt

3. 将 JDK 和 tomcat 安装的压缩包拷贝进上一步目录

4. 在 admin/build/tomcat 目录下新建一个Dockerfile文件

   ```dockerfile
   # vim Dockerfile
   
   FROM centos
   MAINTAINER tomcat-test<a@a.com>
   #把宿主机当前上下文的read.txt拷贝到容器/usr/local/路径下
   COPY read.txt /usr/local/cincontainer.txt
   #把java与tomcat添加到容器中
   ADD jdk-8u11-linux-x64.tar.gz /usr/local/
   ADD apache-tomcat-9.0.22.tar.gz /usr/local/
   #安装vim编辑器
   RUN yum -y install vim
   #设置工作访问时候的WORKDIR路径，登录落脚点
   ENV MYPATH /usr/local
   WORKDIR $MYPATH
   #配置java与tomcat环境变量
   ENV JAVA_HOME /usr/local/jdk1.8.0_11
   ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
   ENV CATALINA_HOME /usr/local/apache-tomcat-9.0.22
   ENV CATALINA_BASE /usr/local/apache-tomcat-9.0.22
   ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin
   #容器运行时监听的端口
   EXPOSE 8080
   #启动时运行tomcat
   # ENTRYPOINT ["/usr/local/apache-tomcat-9.0.22/bin/startup.sh" ]
   # CMD ["/usr/local/apache-tomcat-9.0.22/bin/catalina.sh","run"]
   CMD /usr/local/apache-tomcat-9.0.22/bin/startup.sh && tail -F
   /usr/local/apache-tomcat-9.0.22/bin/logs/catalina.out
   ```

5. 构建镜像

   ```bash
   [root@ tomcat]# docker build -f Dockerfile -t tomcat-test:0.1 .
   .....
   Successfully built ffdf6529937d
   Successfully tagged tomcat-test:latest  # 构建完成
   
   # 查看确定构建完毕！
   [root@ tomcat]# docker images
   REPOSITORY TAG IMAGE ID CREATED SIZE
   tomcat-test latest ffdf6529937d 20 seconds ago 636MB
   ```

6. 启动容器

   -d 后台启动

   -v 宿主机绝对路径目录:容器内目录 添加了两个容器卷

   --privileged=true 解决权限问题（cannot open directory: Permission denied）

   ```bash
   docker run -d -p 9090 :8080 --name tomcat-test -v /home/admin/build/tomcat/test:/usr/local/apache-tomcat-9.0.22/webapps/test -v /home/admin/build/tomcat/tomcat9logs/:/usr/local/apache-tomcat-9.0.22/logs --privileged=true tomcat-test
   ```

7. 验证是否正确安装 jdk 及 拷贝文件

   ```bash
   docker exec tomcat-test java -version
   java version "1.8.0_171"
   
   docker exec tomcat-test ls
   cincontainer.txt
   ```

8. 验证访问

   ```bash
   curl localhost:9090
   ```

9. 容器卷配置

   容器之间配置信息的传递，数据卷的生命周期一直持续到没有容器使用它为止。

   存储在本机的文件则会一直保留！

   ```bash
   [root@ test]#  pwd
   /home/admin/build/tomcat/test
   [root@ test]# mkdir WEB-INF
   [root@ test]#  vim a.jsp
   [root@ test]#  cd WEB-INF/
   [root@ WEB-INF]#  vim web.xml
   ```

   ```html
   ##########  web.xml
   
   <?xml version="1.0" encoding="UTF-8"?>
   
   <web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xmlns="http://java.sun.com/xml/ns/javaee"
   xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
   http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
   id="WebApp_ID" version="2.5">
   
   <display-name>test</display-name>
   
   </web-app>
   
   ###########  a.jsp
   
   <%@ page language="java" contentType="text/html; charset=UTF-8"
   pageEncoding="UTF-8"%>
   <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN"
   "http://www.w3.org/TR/html4/loose.dtd">
   <html>
   <head>
   <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
   <title>hello</title>
   </head>
   <body>
   -----------welcome------------
   <%=" my docker tomcat hello666 "%>
   <br>
   <br>
   <% System.out.println("-------my docker tomcat-------");%>
   </body>
   </html>
   ```

   

10. 验证访问

    ```bash
    curl localhost:9090/test/a.jsp
    ```

11. 查看日志

    ```bash
    [root@ tomcat]# cd tomcat9logs/
    [root@ tomcat9logs]# ll
    total 24
    -rw-r----- 1 root root 6993 May 12 12 :50 catalina.2020-05-12.log
    -rw-r----- 1 root root 7024 May 12 12 :53 catalina.out
    -rw-r----- 1 root root 0 May 12 12 :47 host-manager.2020-05-12.log
    -rw-r----- 1 root root 408 May 12 12 :47 localhost.2020-05-12.log
    -rw-r----- 1 root root 150 May 12 12 :53 localhost_access_log.2020-05-12.txt
    -rw-r----- 1 root root 0 May 12 12 :47 manager.2020-05-12.log
    [root@ tomcat9logs]# cat catalina.out
    ....
    -------my docker tomcat-------  # 搞定
    ```

## 发布镜像到镜像仓库

```shell
# 1、登录
[root@ tomcat]# docker login --username=xxxxxxxx 镜像仓库网站
Password:
WARNING! Your password will be stored unencrypted in
/root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded

# 2、设置 tag
docker tag [ImageId] 镜像仓库地址:[镜像版本号]
[root@ tomcat]# docker tag 251ca4419332 镜像仓库地址:[镜像版本号]

# 3、推送命令
docker push 镜像仓库地址:[镜像版本号]

[root@ tomcat]# docker push 镜像仓库地址:[镜像版本号]
```

## 唠叨环节

下次学容器网络。

再见👋

## TODO LIST

1. docker 网络
1. k8s


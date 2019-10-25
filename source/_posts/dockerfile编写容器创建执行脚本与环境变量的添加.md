---
title: 通过dockerfile添加自起服务与环境变量
date: 2019-04-7 10:50:02
tags: docker
---
&#160;&#160;&#160;&#160;&#160;&#160;docker可以将不同应用的环境进行打包，整理成镜像，在需要使用的时候，轻松的将镜像以容器的形式进行部署。相同的镜像部署的容器完全相同，容器也可以根据实际环境的需要，进行定制化配置，同时容器也能被再次打包成镜像，方便后续的部署。
&#160;&#160;&#160;&#160;&#160;&#160;本文将着重讲解dockerfile的简单实用，dockerfile是用于镜像制作，可以将一个已经存在的镜像按照dockerfile文件的“配置”重新进行打包，成为适用于自己项目部署的新镜像。本文也将在后续工程积累中持续更新dockerfile的使用，下面将分以下几个部分进行事例讲解：
<!--more-->

## dockerfile的基本结构
### 1.FROM：指定基础镜像
  该命令为dockerfile的第一条命令，不可缺少，用于指定基础镜像
#### 用法：
```
  FROM <image>:<tag>
例:
  FROM node:latest //用于指定以标签为latest的node镜像为基础镜像开始制作新的镜像
  
```
### 2.MAINTAINER：维护人
  指定镜像的维护人，通常也就是镜像的制作人
#### 用法：
```
  MAINTAINER <-name->
例：
  MAINTAINER X-D-Y <邮箱>  //指定为何人的信息，例如人的名字和邮箱等的信息
```
### 3.RUN:构建容器时需要执行的命令
  RUN命令有两种书写方式，一种以&lt;command&gt;的形式进行配置；一种是以["executable","param1","param2"]
#### 用法：
```
  RUN &lt;linux shell command&gt;
  RUN [executable","param1","param2"]
例：
  RUN vim dockerfile
  RUN ["vim" "dockerfile"]
```
### 4.CMD:运行容器时需要执行的命令
  CMD命令的书写方式有三种

#### 用法：
```
  1. CMD["excutable","param1","param2"]
  2. CMD["param1","param2"]
  3. CMD command param1 param2
例：
  CMD ["tar","-czvf","docker.tar"]
  CMD ["echo","$PWD"]
  CMD tar -czvf docker.tar
```
<font color=#ff0000>RUN和CMD的区别</font>

RUN命令是在容器构建的时候运行，大多为容器载后续需要用一些工具或者文件的准备，如apt-get install ，yum install一些工具或者提前解压好需要使用的文件。而CMD命令则是在容器启动的时候运行的命令，一般为环境变量的设置等等

### 5.ADD:将本地的文件添加到容器中
  ADD命令可以使本地的文件自动的添加到容器内部，如果是压缩包则自动实现解压
#### 用法
```
  ADD <src> ... <dest>
  ADD ["<src>",...,"<dest>"]
例：
  ADD home* /dir
```
### 6.COPY:将本地文件添加到容器的指定路径下
  COPY命令和ADD命令是类似的，但是与ADD命令不同点在于，ADD命令可以自动解压亚索文件，而COPY命令不会自动解压
&#160;&#160;&#160;&#160;&#160;&#160;
### 7.LABEL:用于为镜像添加元数据
  用来标明dockerfile的标签 可以使用Label代替Maintainer 最终都是在docker image基本信息中可以查看
#### 用法
```
  LABEL <key>=<value> <key>=<value> <key>=<value> 
例：
  LABEL version="1.0" auther=X-D-Y
```

### 8.EXPOSE：用于对外暴露端口
  将docker容器内的端口暴露出去，在宿主机上也需要指定相应的端口实现容器内端口的映射

```
  EXPOSE <hostPort> <containerPort>
例：
  EXPOSE 80:8080
```
### 9.VOLUME
  创建文件挂载点，在docker运行时，docker会创建一个匿名的volume，并将此volume绑定到container的指定目录中，如果container的指定目录下已经有内容，则会将内容拷贝的volume中
#### 用法
```
  VOLUME <容器内的路径>
用法：
  VOLUME /foo
```
### 10.WORKDIR: 进入到工作目录
  指定好工作目录，然后在该工作目录下执行预先设定好的各种指令如”RUN、CMD、ENTRYPOINT、ADD、COPY“等
用法：
```
  WORKDIR: <workspace path>
例：
  WORKDIR /root
```

### 本文的重点内容为docker镜像拉取的容器中环境变量的配置，有两种方式实现DOCKER容器内环境变量的设置：1.运行脚本通过脚本加载环境变量；2.直接配置环境变量。
### 1.dockerfile的entrypoint使用
&#160;&#160;&#160;&#160;&#160;&#160;entrypoint可以用于docker容器启动时，自动运行其中的命令，如果有多条需要执行的命令，可以使用多条entrypoint，然后在容器启动时，逐条进行加载并执行。该命令通常为被使用于执行shell脚本，然后再shell脚本内填写需要执行的命令。使用格式如下：
```bash
ENTRYPOINT ["/bin/echo", "Hello"]  ；
ENTRYPOINT ["./test.sh"]  
```
执行的根目录通常为容器的根目录。但是对于自己编写的程序，启动时需要加载相关环境变量，从根目录执行时，不存在该环境变量对应的相关文件，会使得程序无法启动或者报错。此时就需要在fockerfile文件中写入环境变量的相关配置。
### 2.dockerfile中环境变量的添加
&#160;&#160;&#160;&#160;&#160;&#160;通常环境变量的添加如下形式：
```bash
ENV LD_LABRIRY_PATH /home/xdy/xxx/lib
```

最后借鉴一张网上的图片加深并巩固理解(如有侵权，请联系本人删除)：
<div align=center>
<img src="https://github.com/x-d-y/blog/blob/master/source/_posts/dockerfile%E7%BC%96%E5%86%99%E5%AE%B9%E5%99%A8%E5%88%9B%E5%BB%BA%E6%89%A7%E8%A1%8C%E8%84%9A%E6%9C%AC%E4%B8%8E%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F%E7%9A%84%E6%B7%BB%E5%8A%A0/911490-20171208222222062-849020400.png?raw=true" width = 600>
</div>

镜像批量打包:
    ```
    docker save $(docker images | grep -v REPOSITORY | awk 'BEGIN{OFS=":";ORS=" "}{print $1,$2}') -o image.tar
    ```
### 持续更新中

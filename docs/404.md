# Docker简单介绍

![img](http://www.ruanyifeng.com/blogimg/asset/2018/bg2018020901.png)

## 一、认识docker

### 1. 什么是docker

- docker是一项相对新颖的技术，对比96年出生的Java来说相当于一个刚出生的宝宝，到目前为止的最新版本还仍在1字头纠缠，但这不妨碍docker的广泛应用。
- 简单的说，docker是一个大货轮，货轮运行在系统的海洋里，货轮上的集装箱就是一个个容器，这些容器就是一个独立的系统，这些容器各自独立，互不干扰。



### 2. 为什么是docker

#### 2.1 优势

- 解决了环境配置的麻烦：运行Java程序就配一个支持Java运行的jre。

  > 永兴元的服务由两部分组成：Java封装服务暴露对外接口；python实现AI分析事故车等。在这个系统中应当由两个容器：Java-container、Python-container，python的环境不应当和Java在一个机子中。

- 环境隔离：容器之间相互独立，互不干扰

  > 物理服务器的能力十分强大，只跑一个程序太可惜，但许多程序整合在一起，先不说环境的冲突，如果一个程序内存泄漏，GG，大家都得倒霉

- 像虚拟机，但比虚拟机更强大：虚拟机是一个完整的系统，而docker是对进程的一个隔离。

  > 1. 启动快：几秒钟就能完成启动
  > 2. 资源需求低 ：一台主机可以同时运行几千个Docker容器
  > 3. 体积小：虚拟机一般要几GB到几十GB的空间，而容器只需要MB级甚至KB级（yxy不算）



#### 2.2 趋势

无论是深度学习还是微服务集群，或是其他分布式服务，无不涉及到大量系统的配合，对大量机器的采购难免会消耗大量资源，docker则正好能够解决这个问题。



### 3. 简单概念

#### 3.1 容器（container）

上文说到，容器就像是一个个集装箱，那么集装箱里面就是系统，当容器运行的时候，可以看作一个系统正在运行。

```shell
$ docker container ps

CONTAINER ID    IMAGE        COMMAND       CREATED         STATUS         PORTS                     NAMES
0290ba3fe1c3    yxy:1.00     "bash"        4 days ago      Up 4 days      0.0.0.0:8090->8090/tcp    yxy
```



#### 3.2 镜像（image）

镜像可以看作容器的模板，可以看作是快照，但和VMware快照有一定区别。

```shell
$ docker image ls

REPOSITORY                                       TAG                 IMAGE ID            CREATED             SIZE
registry.cn-qingdao.aliyuncs.com/sanbaiyun/yxy   v1                  681f3b34f46a        4 days ago          3.39 GB
yxy                                              1.00                681f3b34f46a        4 days ago          3.39 GB
yxy                                              0.9                 25cc57912c91        9 days ago          1.21 GB
docker.io/centos                                 centos7.4.1708      9f266d35e02c        21 months ago       197 MB
```



#### 3.3 TAG

给镜像打上tag就相当于给程序标上版本号。



#### 3.4 仓库

仓库是存放镜像的地方，一个名为base-data的仓库中存放了各个版本的base-data镜像；类比来说，某公司可以有许多仓库，A仓库堆放的是化学品A，B仓库堆放的是肉产品，仓库中的产品有不同的生产批次，其中，产品就是镜像，产品批次就是镜像的Tag。





## 二、docker的初步使用

### 1. 使用docker

#### 1.1 拉取镜像

可以直接在仓库中拉取需要的镜像：

```shell
$ docker pull image1:tag
```



#### 1.2 运行镜像

将本地的镜像作为容器的模板运行，镜像还是那个镜像，只是多了一个容器。

1. 使用image:tag（名称为image的镜像，版本为tag）以后台模式启动一个名为【containername】的容器，将容器的8080端口映射到宿主机的80端口，将容器的/data目录映射到宿主机的/data目录下

    ```shell
    $ docker run -p 80:8080 -v /data:/data -d --name containername image1:tag
    ```

2. 对正在运行的容器进行某些操作（在containername中交互式执行runoob.sh）

    ```shell
    $ docker exec -it containername /bin/sh /root/runoob.sh
    ```

3. 生命周期

    - **docker start** :启动一个或多个已经被停止的容器

    - **docker stop** :停止一个运行中的容器

    - **docker restart** :重启容器



### 2. 创建镜像

#### 2.1 方法一：Dockerfile

Dockerfile相当于从基础镜像到完整的系统的步骤：

示例

```dockerfile
#基于CentOS-7.4.1708的基础镜像
FROM centos:centos7.4.1708

#维护人
LABEL MAINTAINER="zye@che300.com"

#基础环境配置
RUN  yum -y install zlib-devel bzip2-devel libffi-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gcc make wget

#配置1:Java
ADD /java/jre-8u271-linux-x64.tar.gz /opt/java

ENV JAVA_HOME=/opt/java/jre1.8.0_271
ENV PATH=$JAVA_HOME/bin:$PATH

#配置2:Python
ADD /python/Python-3.7.4.tgz opt/python

RUN cd /opt/python/Python-3.7.4 \
    && ./configure prefix=/usr/local/python3 \
    && make && make install \
    && ln -s /usr/local/python3/bin/python3.7 /usr/bin/python3 \
    && ln -s /usr/local/python3/bin/pip3 /usr/bin/pip3

#配置3:Python环境拓展
RUN pip3 install pandas -i https://pypi.tuna.tsinghua.edu.cn/simple

#APP安装     
ADD /app/yxy-vha-c.jar /opt/app/java/vha-c.jar
ADD /app/myche.tar /opt/app/db/
ADD /app/accidentcarapi_cython.tar /opt/app/python/

#对外暴露端口
EXPOSE 8090
    
ENTRYPOINT ["/opt/app/python/accidentcarapi_cython/start_gunicorn.sh"]

```

构建：

```shell
$ docker build -t name .	
```



#### 2.2 方法二：基于操作的镜像提交

当已有一个镜像之后，可以运行此容器，进行一些修改之后，将此容器的快照作为镜像提交，这样就可以得到新镜像

 ```shell
$ docker commit -a "Marno" -m "Marno想提交就提交" a404c6c174a2  image1:newtag
 ```

> Marno将容器【a404c6c174a2】提交为image:newtag，并附带说明：Marno想提交就提交



想要比较两个镜像的区别，可以

```shell
$ docker diff image1
```

> 将该容器与创建该容器的镜像进行比较，其结果是忽略中间过程的。
>
> 当正在运行的容器出现问题，紧急修复后提交为新镜像，可以用此命令判断是否仅修改了需要修改的文件



### 3. 提交仓库

#### 3.1 登录

```shell
$ docker login 
```

#### 3.2 提交

```shell
$ docker image tag imagename username/repository:tag
$ docker image tag image:0.0.1 marno/image:0.0.1
$ docker image push username/repository:tag
```

> 一般来说，仓库和镜像名相同



## 三、docker和微服务

微服务：软件把任务外包出去，让各种外部服务完成这些任务，软件本身只是底层服务的调度中心和组装层。

很适合用 Docker 容器实现，每个容器承载一个服务。一台计算机同时运行多个容器，从而就能很轻松地模拟出复杂的微服务架构。



> ​	Docker 可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。

## Docker是什么

**Docker 属于 Linux 容器的一种封装，提供简单易用的容器使用接口。**它是目前最流行的 Linux 容器解决方案。

Docker 将应用程序与该程序的依赖，打包在一个文件里面。运行这个文件，就会生成一个虚拟容器。程序在这个虚拟容器里运行，就好像在真实的物理机上运行一样。有了 Docker，就不用担心环境问题。总体来说，Docker 的接口相当简单，用户可以方便地创建和使用容器，把自己的应用放入容器。容器还可以进行版本管理、复制、分享、修改，就像管理普通的代码一样。Doker是基于GO语言实现的云开源项目

通过对应用组件的封装、分发、部署、运行等生命周期的管理，达到应用组件级别的“一次封装，到处运行”这里的应用组件 既可以是web应用 也可以是一套数据库服务 甚至是一个操作系统或编译器

## Docker的优点

Docker 是一个用于开发，交付和运行应用程序的开放平台。Docker 使您能够将应用程序与基础架构分开，从而可以快速交付软件。借助 Docker，您可以与管理应用程序相同的方式来管理基础架构。通过利用 Docker 的方法来快速交付，测试和部署代码，您可以大大减少编写代码和在生产环境中运行代码之间的延迟。

- 可以将程序及其依赖、运行环境一起打包为一个镜像，可以迁移到任意Linux操作系统
- 运行时利用沙箱机制形成隔离容器，各个应用互不干扰
- 启动、移除等都可以通过一行命令完成，方便快捷

## Docker的需求

我们在开发的时候，对微服务等大型项目进行部署的时候，因为他们的组件多而复杂，运行环境也较为复杂，部署的会遇到很多问题

- 依赖关系复杂，容易出现兼容性问题
- 开发、测试、生产环境有差异

### 快速，一致地交付您的应用程序

Docker 允许开发人员使用您提供的应用程序或服务的本地容器在标准化环境中工作，从而简化了开发的生命周期。容器非常适合持续集成和持续交付（CI / CD）工作流程。

### 响应式部署和扩展

Docker 是基于容器的平台，允许高度可移植的工作负载。Docker 容器可以在开发人员的本机上，数据中心的物理或虚拟机上，云服务上或混合环境中运行。Docker 的可移植性和轻量级的特性，还可以使您轻松地完成动态管理的工作负担，并根据业务需求指示，实时扩展或拆除应用程序和服务。

### 在同一硬件上运行更多工作负载

Docker 轻巧快速。它为基于虚拟机管理程序的虚拟机提供了可行、经济、高效的替代方案，因此您可以利用更多的计算能力来实现业务目标。Docker 非常适合于高密度环境以及中小型部署，而您可以用更少的资源做更多的事情。

## Docker如何解决依赖的兼容问题

我的理解是Docker会将应用的Libs（函数库）、Deps（依赖）、配置与应用一起打包，这样就会保证应用的依赖兼容与版本问题。并且Docker会将每个应用放到一个独立的容器中去隔离运行，避免互相干扰。

![image-20230103145413300](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/blog/202301031454393.png)

## Docker如何解决不同系统环境的问题

我们都知道，不同的操作系统的具体环境是不同的，Docker是如何来解决这方面呢，我们先来了解下操作系统的结构组成。用linux来举个例子。首先我们日常接触的就是**用户程序**，比如QQ，微信这些。再而就是**系统应用**，系统应用就是将内核进行封装，方便程序员来进行调用。最后是Linux内核，与硬件进行交互，提供操作硬件的指令。

<img src="https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/blog/202301031510918.png" alt="image-20230103151037738" style="zoom:80%;" />

Ubuntu和CentOS都是基于Linux内核，只是系统应用不同，提供的函数库有差异。此时，如果将一个Ubuntu版本的MySQL应用安装到CentOS系统，MySQL在调用Ubuntu函数库时，会发现找不到或者不匹配，就会报错了。

<img src="https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/blog/202301031512856.png" alt="image-20230103151201650" style="zoom: 67%;" />

### Docker如何解决不同系统环境的问题？

* Docker将用户程序与所需要调用的系统(比如Ubuntu的)函数库一起打包。
* Docker运行到不同操作系统时，借助于已经打包好的库函数，直接调用Linux内核来运行
* 也就是说，Docker打包好的程序包可以运行在任何一个基于Linux内核的操作系统上

### 总结

Docker如何解决大型项目依赖关系复杂，不同组件依赖的兼容性问题？

- Docker允许开发中将应用、依赖、函数库、配置一起打包，形成可移植镜像
- Docker应用运行在容器中，使用沙箱机制，相互隔离

Docker如何解决开发、测试、生产环境有差异的问题

- Docker镜像中包含完整运行环境，包括系统函数库，仅依赖系统的Linux内核，因此可以在任意Linux操作系统上运行

## Docker与虚拟机

**虚拟机**（virtual machine）是在操作系统中模拟硬件设备，然后运行另一个操作系统，比如在 Windows 系统里面运行Ubuntu 系统，这样就可以运行任意的Ubuntu应用了。而**Docker仅仅是封装函数库**，并没有模拟完整的操作系统。

![image-20230103151858000](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/blog/202301031518117.png)

### Docker 与虚拟机的区别

- docker是一个系统进程；虚拟机是在操作系统中的操作系统
- docker体积小、启动速度快、性能好；虚拟机体积大、启动速度慢、性能一般

## 镜像与容器

- 镜像（Image）：Docker将应用程序及其所需的依赖、函数库、环境、配置等文件打包在一起，称为镜像。
- 容器（Container）：镜像中的应用程序运行后形成的进程就是容器，只是Docker会给容器做隔离，对外不可见。

![image-20230103154100671](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/blog/202301031541790.png)

一切应用最终都是代码组成，都是硬盘中的一个个的字节形成的文件。只有运行时，才会加载到内存，形成进程。而镜像，就是把一个应用在硬盘上的文件、及其运行环境、部分系统函数库文件一起打包形成的文件包。这个文件包是只读的。容器呢，就是将这些文件中编写的程序、函数加载到内存中运行，形成进程，只不过要隔离起来。因此一个镜像可以启动多次，形成多个容器进程。例如你下载了一个QQ，如果我们将QQ在磁盘上的运行文件及其运行的操作系统依赖打包，形成QQ镜像。然后你可以启动多次，双开、甚至三开QQ，跟多个妹子聊天。
## Docker与DockerHub

开源应用程序非常多，打包这些应用往往是重复的劳动。为了避免这些重复劳动，人们就会将自己打包的应用镜像，例如Redis、MySQL镜像放到网络上，共享使用，就像GitHub的代码共享一样。那么DockerHub就是一个Docker镜像的托管平台。这样的平台称为Docker Registry。国内也有类似于DockerHub 的公开服务，比如 网易云镜像服务、阿里云镜像库 等。我们一方面可以将自己的镜像共享到DockerHub，另一方面也可以从DockerHub拉取镜像。

![image-20230103155825483](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/blog/202301031558574.png)

## Docker架构

Docker是一个Cs架构的程序，分别由**客户端**和**服务端**组成，服务端主要用来负责处理Docker指令，管理镜像，容器等。客户端是用来通过命令或RestAPI向Docker服务端发送指令。可以在本地或远程向服务端发送指令。

![image-20230103160145307](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/blog/202301031601403.png)

## 安装Docker

> 企业部署一般都是采用Linux操作系统，而其中又数CentOS发行版占比最多，因此我们在CentOS下安装Docker。

Docker 分为 CE 和 EE 两大版本。CE 即社区版（免费，支持周期 7 个月），EE 即企业版，强调安全，付费使用，支持周期 24 个月。Docker CE 分为 stable test 和 nightly 三个更新频道。官方网站上有各种环境下的 安装指南，这里主要介绍 Docker CE 在 CentOS上的安装。

### Centos安装Docker

Docker CE 支持 64 位版本 CentOS 7，并且要求内核版本不低于 3.10， CentOS 7 满足最低内核的要求，所以我们在CentOS 7安装Docker。

#### 卸载

如果之前安装过旧版本的Docker，可以使用下面命令卸载

~~~powershell
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine \
                  docker-ce

~~~

#### 安装

首先需要大家虚拟机联网，并安装yum工具

~~~powershell
yum install -y yum-utils \
           device-mapper-persistent-data \
           lvm2 --skip-broken
~~~

然后更新本地镜像源

~~~powershell
# 设置docker镜像源
yum-config-manager \
    --add-repo \
    https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
    
sed -i 's/download.docker.com/mirrors.aliyun.com\/docker-ce/g' /etc/yum.repos.d/docker-ce.repo

yum makecache fast
~~~

然后输入命令

~~~powershell
yum install -y docker-ce
~~~

docker-ce为社区免费版本。稍等片刻，docker即可安装成功。

#### 启动Docker

Docker应用需要用到各种端口，逐一去修改防火墙设置。非常麻烦，因此建议大家直接关闭防火墙！启动docker前，一定要关闭防火墙后！！

~~~powershell
# 关闭
systemctl stop firewalld
# 禁止开机启动防火墙
systemctl disable firewalld
~~~

通过命令启动docker

~~~powershell
systemctl start docker  # 启动docker服务

systemctl stop docker  # 停止docker服务

                                                                systemctl restart docker  # 重启docker服务
~~~

然后输入命令，可以查看docker版本

~~~powershell
docker -v
~~~

#### 配置镜像仓库加速

针对Docker客户端版本大于 1.10.0 的用户。您可以通过修改daemon配置文件/etc/docker/daemon.json来使用加速器

~~~powershell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://exwt4woy.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
~~~

## Docker的基本操作

### 镜像操作

镜像名称一般分两部分组成：`[repository]:[tag]`，在没有指定的tag时，默认是latest，代表最新版的镜像。

<img src="https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/blog/202301031705762.png" alt="image-20230103170505691" style="zoom:50%;" />

#### 镜像操作命令

从仓库中拉取一个nginx的镜像

~~~shell
docker pull nginx
~~~

通过命令查看镜像

~~~shell
docker images
~~~

将nginx的镜像导出磁盘

~~~sh
#导出 OPTIONS：文件名称 
docker save [OPTIONS] IMAGE [IMAGE...]
#举个例子
docker save -o nginx.tar nginx:latest
~~~

删除掉docker中的nginx镜像

~~~sh
docker rmi nginx:latest
~~~

将刚才导出来的nginx镜像进行上传

~~~sh
docker load -i nginx.tar
~~~

<img src="https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/blog/202301031705358.png" alt="image-20230103170536263" style="zoom:67%;" />

### 容器操作

Docker 容器是一个开源的应用容器引擎，让开发者可以以统一的方式打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何安装了docker引擎的服务器上（包括流行的Linux机器、windows机器），也可以实现虚拟化。容器是完全使用沙箱机制，相互之间不会有任何接口（类似 iPhone 的 app）。几乎没有性能开销,可以很容易地在机器和数据中心中运行。最重要的是,他们不依赖于任何语言、框架包括系统。

#### Docker容器保护三个状态

* 运行：进程正常运行
* 暂停：进程暂停，CPU不再运行，并不释放内存
* 停止：进程终止，回收进程占用的内存、CPU等资源

#### 容器操作命令

创建一个Nginx容器，先去docker hub 查看Nginx的容器运行命令 然后进行创建

~~~sh
docker run --name containerName -p 80:80 -d nginx
~~~

`docker run`：创建并运行一个容器

`--name` ：给容器起一个名字，比如叫做containerName

`-d `：后台运行容器

`nginx`：镜像名称，例如nginx

这里的前面的80指的是服务器的端口，而后面的80指的是docker内部的端口，也就是说我们需要用服务器的端口和docker的端口进行映射。

查看我们创建好的容器

```sh
docker ps
```

查看指定容器的日志

~~~sh
#这里的name指的是容器的名称
docker logs name
~~~

动态查看容器的日志

~~~sh
docker logs -f name
~~~

删除容器，但是不能删除运行中的容器 使用`-f`参数可以强制删除

~~~sh
docker rm [容器名称]
~~~

进行容器内部，exec进入容器内部可以修改文件，但是在容器内修改文件是不推荐的。

~~~sh
docker exec -it [容器名] [要执行的命令]
~~~

容器内部会模拟一个独立的Linux文件系统，看起来如同一个linux服务器一样，nginx的环境、配置、运行文件全部都在这个文件系统中，包括我们要修改的html文件。但是容器内没有vi命令【没有多余的命令】，无法直接修改

停止指定容器

~~~sh
docker stop mn
~~~

查看所有容器包括挂掉的

~~~sh
docker ps -a
~~~

### 数据卷操作

有这样几个问题，Docker容器删除之后，在容器中产生的数据也会随之销毁吗，Docker容器可以与外部机器直接交换文件吗，容器之间如何进行数据交互。这个时候我们就要用到数据卷了，数据卷是宿主机中的一个文件或者目录。当容器目录和数据卷目录绑定之后，对方的修改会立即同步。一个数据卷可以同时被多个容器同时挂载。

#### 数据卷的作用

* 容器持久化   
* 外部机器和容器间接通信
* 容器之间数据交互

#### 数据卷操作

创建数据卷

~~~sh
docker volume create [数据卷名称]
~~~

查看全部的数据卷

~~~sh
docker volume ls		
~~~

查看数据卷存在宿主机中的位置

~~~sh
docker volume inspect [数据卷的名称]
~~~

删除本地所有的数据卷

~~~sh
docker volume prune
~~~

删除指定的卷

~~~sh
docker volume rm [数据卷名称]
~~~

#### 挂载数据卷

> 我们可以将数据卷挂载在容器上去，这样就可以通过数据卷来修改容器内部的数据了。

创建启动容器时，使用`-v`参数挂载数据卷

~~~sh
docker run --name [容器名称] -p [宿主端口]:[容器端口] -v [数据卷名称]:[容器内部的数据目录] -d [docker hub的容器名称]
#举个例子 将html挂载到nginx上去
docker run --name nginx -p 39002:80 -v html:/usr/share/nginx/html -d nginx
~~~

我们尝试修改nginx的html页面，先查看数据卷在宿主机的位置

~~~sh
docker inspect html
~~~

会打印出如下信息

~~~sh
[root@VM-4-8-centos ~]# docker inspect html
[
    {
        "CreatedAt": "2023-01-04T16:06:47+08:00",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/html/_data",
        "Name": "html",
        "Options": {},
        "Scope": "local"
    }
]

~~~

其中的`Mountpoint`就是地址，然后我们进入这个地址，并打印目录信息

~~~sh
[root@VM-4-8-centos ~]# cd /var/lib/docker/volumes/html/_data
[root@VM-4-8-centos _data]# ls
50x.html  index.html
~~~

然后我们使用vim对html进行修改并且保存，在公网中访问nginx，会发现已经修改成功了。

![image-20230104162857976](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/blog/202301041628042.png)

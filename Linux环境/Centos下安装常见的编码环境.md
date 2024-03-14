## 安装宝塔面板

centon终端输入

```Bash
yum install -y wget && wget -O install.sh http://download.bt.cn/install/install_6.0.sh && sh install.sh ed8484bec
```

Ubuntu/Deepin安装脚本

```Bash
wget -O install.sh http://download.bt.cn/install/install-ubuntu_6.0.sh && sudo bash install.sh ed8484bec
```

## Redis

#### 宝塔面板安装

宝塔面板安装，直接点击安装即可。

#### 终端安装

先下载安装所需要的包

```Bash
wget http://download.redis.io/releases/redis-4.0.2.tar.gz
```

解压文件 我们在解压文件前最好新建一个统一管理文件的目录 当然 没有也可以

```Bash
tar -zxvf redis-4.0.2.tar.gz 
```

进入该解压后的目录

```Bash
cd redis-4.0.2  
```

编译及安装

```Bash
make
make install 
```

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/image.png)

安装成功了。

## Nginx

#### 宝塔面板安装

选择版本点击即可

#### 终端

先安装gcc 依赖

```Bash
yum -y install gcc gcc-c++ make libtool zlib zlib-devel openssl openssl-devel pcre pcre-devel
```

使用wget命令下载安装包 当然也可以下载好上传

```Bash
wget -c https://nginx.org/download/nginx-1.18.0.tar.gz
```

解压好安装包

```Bash
tar -zxvf nginx-1.18.0.tar.gz
cd nginx-1.18.0
```

不需要ssl

```Bash
./configure --prefix=/usr/local/nginx
```

需要ssl

```Bash
./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module
```

编译安装

```Bash
make
make install
```

**安装完成**

## JDK11

访问官网下载页面

https://www.oracle.com/java/technologies/javase-jdk11-downloads.html

下载对应的linux包

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/image.png)

接下来将压缩包上传到linux服务器中（使用宝塔面板可以直接上传到固定的文件夹） 我在www/server下新建了JDK11的安装目录

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/image.png)

上传（可直接拖曳）成功之后进行解压缩

这里建议使用终端解压缩，当然也可以使用宝塔面板右键解压缩

```Bash
tar -zxvf jdk-11.0.14_linux-x64_bin.tar.gz 
```

解压完成只需要配置环境变量即可

进入目录etc下面的profile文件夹

```Bash
cd /etc/profile
```

可以使用vim（需要提前下载vim编辑器）打开 如果有宝塔，可以直接双击该文件

```Bash
vim profile   使用vim 指令打开
```

在profile文件里追加以下环境配置

```Bash
export JAVA_HOME=/www/server/JDK11/jdk-11.0.17
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
```

配置完成之后刷新以下(需要注意路径）

```Bash
 source profile
```

输入以下指令查询版本是否安装成功

```Bash
java -version
```

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/image.png)

出现如图便是安装成功。

## 安装MySQL并配置远程连接用户

### 安装

#### 使用宝塔面板安装Mysql一步到位

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/image.png)

#### 终端安装Mysql

复制下载连接下载

```Bash
wget https://cdn.mysql.com//Downloads/MySQL-8.0/mysql-8.0.29-1.el8.x86_64.rpm-bundle.tar
```

解压rpm bundle包

```Bash
tar xvf mysql-8.0.29-1.el8.x86_64.rpm-bundle.tar 
```

### 配置远程连接用户

先进入root权限的mysql中

```Bash
mysql -uroot -proot账号的密码
```

进入到mysql 库中

```Bash
use mysql;
```

创建用户 这里的syh@db 是用户名 syh@123是密码  %是通配符 也就是说指定的主机是全部可以。当然你也可以指定主机ip

```Bash
create user 'syh@dbs' identified by 'syh@123';
```

这个是直接创建了远程连接。然后查看下用户和权限。

```Bash
select host,user from user;
```

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/image.png)

刷新权限

```Bash
flush privileges;
```

使用远程连接测试即可。

## 安装RocketMq

步骤1:下载压缩包 官网下载太慢了 自己找链接下载。

步骤2：解压

```Bash
unzip rocketmq-all-4.5.2-bin-release.zip
```

步骤3：修改目录名称

```Bash
mv rocketmq-all-4.5.2-bin-release rocketmq
```

**启动服务器**

步骤1：启动命名服务器（bin目录下）

```Bash
sh mqnamesrv
```

步骤2：启动消息服务器（bin目录下）

```Bash
sh mqbroker -n localhost:9876
```

修改runbroker.sh文件中有关内存的配置（调整的与当前虚拟机内存匹配即可，推荐256m-128m）

其实这里就可以使用了 但是如果你的jdk版本不是8的话 会发生问题，因为rocketmq创建初始就是jdk8的环境基础。所以我们需要修改jdk版本

打开 `runserver.sh` 和 `runbroker.sh`（Windows 下则是相应的 .cmd 后缀），做如下修改（修改前记得备份）：

1. 找到包含 `-Djava.ext.dirs` 参数的行，将该行删除或注释掉；
2. 找到包含 `-Xloggc` 参数的行，将该行删除或注释掉；
3. 找到包含 `-XX:+UseGCLogFileRotation` 参数的行，将该行删除或注释掉；
4. 找到包含 `export CLASSPATH=` 的行（在文件的前面部分），将等号后面的内容改为 `.:${BASE_DIR}/conf:${BASE_DIR}/lib/*:${CLASSPATH}`
5. 找到包含 `-XX:-UseLargePages` 参数的行，在它的下面添加一行 `JAVA_OPT="${JAVA_OPT} --add-exports java.base/jdk.internal.ref=ALL-UNNAMED"`

**如果是jdk8我们直接启动就成了**
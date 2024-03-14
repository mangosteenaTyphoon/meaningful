>Docker拉取mysql8.0的镜像，并且创建容器进行配置，使用远程可视化窗口进行连接。

###  拉取镜像

拉取mysql8.0的镜像

~~~bash
docker pull mysql:8.0
~~~

查看镜像

~~~bash
docker images	
~~~

### 创建容器

这里创建容器的name是名称  -p 3306:3306 前面的3306指的是宿主机端口 后面的3306是指mysql在docker里面的端口号， -e 后面的配置是配置root的密码 我这里写的密码是password -d 后面为mysql的镜像名称 也就是我们刚刚拉取的mysql:8.0

~~~bash
docker run --name mysql8.0 -p 3306:3306 -e MYSQL_ROOT_PASSWORD=password -d mysql:8.0
~~~

创建完容器后进行查看

~~~bash
docker ps -a
~~~

此时可以看到我们的mysql8.0的容器已经创建成功

### 进入容器

先进入到mysql8.0的容器内部

~~~bash
docker exec -it mysql8.0 bash
~~~

然后我们就和平常使用mysql一样了 输入账号密码登录进去尽可

~~~bash
mysql -uroot -ppassword
~~~

先进入到mysql这个库里面去

~~~bash
user mysql;
~~~

修改root用户的配置

~~~bash
alert user 'root'@'%' identified with mysql_native_password by 'password';
~~~

这个时候查看一个root用户的权限以及密码规则

~~~bash
select host,user,plugin from user;
~~~

这个时候远程连接就可以了。
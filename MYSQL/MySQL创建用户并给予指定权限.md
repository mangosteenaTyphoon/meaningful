---
title: MySQL创建用户并给予指定权限
date: 2022-12-16
tags:
 - MySQL
categories:
 - MySQL
---

> 在实际项目中，为了安全起见，我们需要对多个数据库进行安全管理，因此要创建不同的用户，给予他们不同的权限。

### 创建用户

```sql
create user 用户名 identified by '密码';
```

例如：

```sql
create user test identified by '123456';
```

### 分配权限

新创建的用户，默认情况下是没有任何权限的。

```sql
grant 权限 on 数据库.数据表 to '用户'@ '主机名';
```

例如，给我们上面创建的用户分配权限

~~~sql
grant all on dbtest.* to 'test'@'%';
~~~

这样，test这个用户就拥有了datest这个数据库所有的权限了。

### 给用户分配更加精准的权限

给test这个用户有查询dbtest数据库tmp1表的权限。

```sql
grant select on dbtest.temp1 to 'test'@'%'; 
```

给来自10.163.225.87的用户joe分配可对数据库vtdc的employee表进行select,insert,update,delete,create,drop等操作的权限，并设定口令为123.

~~~sql
grant all privileges on vtdc.* to joe@10.163.225.87 identified by '123';
~~~

###  回收权限

收回test所有的权限

```sql
revoke all on *.* from 'test'@'%';
```

### 修改用户密码策略

~~~sql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '新的密码';
~~~


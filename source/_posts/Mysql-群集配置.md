---
title: Mysql 群集配置
date: 2016-10-11 10:52:54
tags: mysql 
---

>Mysql 配置读写分离, 降低读服务器压力


## 主 Mysql 配置

### 编辑主 Mysql 服务器的MySQL配置文件my.cnf
> 一般位置（/etc/my.cnf / /usr/local/Cellar/mysql/5.6.25/my.cnf）
> 在[mysqld]下面添加以下参数：
<!-- more -->

``` conf
log-bin=mysql-bin #开启MYSQL二进制日志
server-id=1 #服务器ID不能重复
binlog-do-db=brpg #需要做主从备份的数据库名字
expire-logs-days = 7 #只保留7天的二进制日志，以防磁盘被日志占满
```

### 在 A 服务器添加一个用于主从复制的帐号
>登陆mysql命令行，执行

``` bash
GRANT REPLICATION SLAVE ON *.* TO '帐号'@'从服务器IP' IDENTIFIED BY '密码';
```

### 在主MySQL服务器上执行命令，把数据库设置成只读状态

``` bash
FLUSH TABLES WITH READ LOCK;
```

### 导出数据文件

``` bash
mysqldump -uroot -p brpg > brpg.sql
```

### 回到 Mysql，解封数据库只读状态

``` bash
UNLOCK TABLES;
```

### 重新启动 Mysql 让主 Mysql 配置生效

``` bash
service mysqld restart
```

### 记下file及position的值

``` bash
show master status;
```

## 从 Mysql 配置

### 编辑从 Mysql 服务器的MySQL配置文件my.cnf
``` conf
server-id=2 #服务器ID不能重复
replicate-do-db=brpg #需要做复制的数据库名
#replicate-ignore-table=brpg.xx #需要自动跳过的表
slave-skip-errors = 1032,1062,126,1114,1146,1048,1396 #自动跳过的错误代码，以防复制出错被中断
```

>Mysql 5.5 或更高版本不支持 master- 相关配置

### 将主库上备份的数据库恢复到从库

### 重启从库MYSQL

### 登录从库的MySQL命令行

```bash
change master to master_host='192.168.0.2', master_user='mysqlrun', master_password='123', master_log_file='file的值', master_log_pos=position的值;
```

> 设置成主库的连接信息，file及position的值是之前记录下来，position的值没有单引号，其他的值要单引号

### 启动从库连接
```bash
start slave; 
```

### 查看从库状态

```bash
show slave status\G; #查看连接情况，是不是两个YES，两个YES为成功
```

```bash
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
```

### 如果从服务器启动出现报错
>Slave_IO_Running: Connect


>>需要先检查主 Mysql 的账号密码是否正确
>>主从的 my.cnf bind_ip 一栏设置的 ips 是否正确
>>主服务器与从服务器的防火墙 是否已经允许 3306 进出 (telnet ip:3306)

#### iptables 设置
``` bash
service iptables stop
iptables -A INPUT -p tcp -m state --state NEW -m tcp --dport 3306 -j ACCEPT
service iptables save
service iptables start
```

### ufw 设置
```bash
ufw allow 3306/tcp # 允许 3306 端口通过
 
# 其他设置
ufw disable # 禁用
ufw enable # 启动
# 查看状态
ufw status numbered
 
:# ufw status numbered
[ 1] 3306/tcp                   ALLOW IN    Anywhere
[ 2] 3306/tcp (v6)              ALLOW IN    Anywhere (v6)
 
# 根据序号删除
ufw delete 1
ufw delete 2
```

> bash Slave_IO_Running: No
> Last_IO_Error: Fatal error: The slave I/O thread stops because master and slave have equal MySQL server UUIDs; these UUIDs must be d

>> 那可能是因为2台服务器是复制过去的，登录任意一台服务器，重新生成 Mysql 的 auto.cnf

```bash
mv /usr/local/mysql/var/auto.cnf /usr/local/mysql/var/auto.bak.cnf #重命名该名称
```

```bash
service mysql restart # 重新启动 mysql ，Mysql 会重新生成 auto.cnf
```
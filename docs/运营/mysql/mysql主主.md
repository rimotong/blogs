## 1.系统环境
<br>

<span id="busuanzi_container_site_pv" style='display:none'>
    本站总访问量：<span id="busuanzi_value_site_pv"></span> 次
</span>
<span id="busuanzi_container_site_uv" style='display:none'>
    本站总访客数：<span id="busuanzi_value_site_uv"></span> 人
</span>

<br>

### 1.Linux : centos 7
  Mysql ：mysql5.7
### 2.卸載mariadb-lib
CentOS7默认自带mariadb, 安装MySQL之前将其卸载.
    [root@mysql-lh1 ~]# rpm -qa | grep mariadb
    mariadb-libs-5.5.52-1.el7.x86_64
    [root@mysql-lh1 ~]# rpm -e --nodeps mariadb-libs-5.5.52-1.el7.x86_64

### 3.下载安装

去官网下载最新版MySQL的rpm集合包并解压(由于MGR是MySQL-5.7.17新加的特性, 确保下载的是MySQL5.7.17或者更高的版本)

    [root@mysql-lh1 ~]# wget https://cdn.mysql.com//Downloads/MySQL-5.7/mysql-5.7.19-1.el7.x86_64.rpm-bundle.tar
    [root@mysql-lh01 ~]# mkdir software
    [root@mysql-lh01 ~]# tar -xf mysql-5.7.19-1.el7.x86_64.rpm-bundle.tar -C software
    [root@mysql-lh01 ~]# cd software/
    [root@mysql-lh01 software]# ls -ltr

依次安装

    [root@mysql-lh01 software]# rpm -ivh mysql-community-common-5.7.19-1.el7.x86_64.rpm
    Preparing...                          ################################# [100%]
    Updating / installing...
    1:mysql-community-common-5.7.19-1.e################################# [100%]
    [root@mysql-lh01 software]# rpm -ivh mysql-community-libs-5.7.19-1.el7.x86_64.rpm 
    Preparing...                          ################################# [100%]
    Updating / installing...
    1:mysql-community-libs-5.7.19-1.el7################################# [100%]
    [root@mysql-lh01 software]# rpm -ivh mysql-community-client-5.7.19-1.el7.x86_64.rpm 
    Preparing...                          ################################# [100%]
    Updating / installing...
    1:mysql-community-client-5.7.19-1.e################################# [100%]
    [root@mysql-lh01 software]# rpm -ivh mysql-community-server-5.7.19-1.el7.x86_64.rpm 
    Preparing...                          ################################# [100%]
    Updating / installing...
    1:mysql-community-server-5.7.19-1.e################################# [100%]
    [root@mysql-lh01 software]#

### 4.初始化MySQL

    [root@mysql-lh1 ~]# mysqld --initialize --user=mysql
    其中: --initialize 选项默认以安全模式来初始化, 会为root生成一个随机密码并将该密码标记为过期, 登录后需要重新设置一个新的密码, 而使用—initialize-insecure命令则不适用安全模式, 不会为root生成一个密码. 生成的密码在mysql的log文件里:
    [root@mysql-lh1 ~]# vi /var/log/mysqld.log
    …
    2017-09-02T09:23:10.247283Z 0 [Warning] InnoDB: New log files created, LSN=45790
    2017-09-02T09:23:10.323075Z 0 [Warning] InnoDB: Creating foreign key constraint system tables.
    2017-09-02T09:23:10.387801Z 0 [Warning] No existing UUID has been found, so we assume that this is the first time that this server has been started. Generating a new UUID: 56627e36-8fc0-11e7-8f0b-005056b60f66.
    2017-09-02T09:23:10.390070Z 0 [Warning] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
    2017-09-02T09:23:10.390663Z 1 [Note] A temporary password is generated for root@localhost: glq?p&dvd0MJ
    上面标注黄色的即为生成的root用户的密码.
### 5.启动MySQL服务器.

    [root@mysql-lh1 ~]# systemctl start mysqld

### 6.修改root密码
使用root用户登录MySQL, 并修改密码

    [root@mysql-lh1 ~]# mysql -uroot -p
    Enter password: glq?p&dvd0MJ
    Welcome to the MySQL monitor.  Commands end with ; or \g.
    Your MySQL connection id is 63
    Server version: 5.7.19

    Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

    Oracle is a registered trademark of Oracle Corporation and/or its
    affiliates. Other names may be trademarks of their respective
    owners.

    Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

    mysql> show databases;   -- 当前root用户的密码被标记为过期, 使用之前要修改密码
    ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.
    mysql>
    修改root用户的密码
    mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'beaconAdmin';
    Query OK, 0 rows affected (0.00 sec)

    mysql> FLUSH PRIVILEGES;
    Query OK, 0 rows affected (0.00 sec)

### 7.重启MySQL服务器
    [root@mysql-lh1 ~]# systemctl stop mysqld
    [root@mysql-lh1 ~]# 	
    inactive
    [root@mysql-lh1 ~]# systemctl start mysqld
    [root@mysql-lh1 ~]# systemctl is-active mysqld
    active
    [root@mysql-lh1 ~]# ps -ef | grep mysqld
    mysql     5818     1  1 17:43 ?        00:00:00 /usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid
    root      5850  3110  0 17:43 pts/1    00:00:00 grep --color=auto mysqld
注意: 这里不要使用systemctl restart mysqld来重启MySQL服务, 因为第一次我们是安全模式启动的, 用stop再restart启动能让mysql以正常模式启动.

### 8.设置MySQL开机启动
    [root@mysql-lh1 ~]# systemctl enable mysqld.service
    [root@mysql-lh1 ~]# systemctl is-enabled mysqld
    enabled
    [root@mysql-lh1 ~]#

### 9.防火墙设置
3306为MySQL的默认端口, 不能被其他进程占用.

    查看防火墙当前规则
    [root@mysql-lh01 ~]# firewall-cmd --list-ports
    [root@mysql-lh01 ~]# firewall-cmd --permanent --list-ports

    设置firewall允许3306端口通过防火墙.
    [root@mysql-lh01 ~]# firewall-cmd --permanent --zone=public --add-port=3306/tcp
    success
    [root@mysql-lh01 ~]# firewall-cmd --reload
    success
    [root@mysql-lh01 ~]# firewall-cmd --zone=public --permanent --list-ports
    3306/tcp 3306/udp 

## 2.Mysql双主设置
### 1.修改mysql配置文件
两台mysql均要开启binlog日志功能，开启方法：在/etc/my.cnf配置文件中加上一下内容。两台mysql的server-id不能一样，所以我这边设置的是Master的server-id = 1,Master2的Server-id = 2,

    Master1:
    server-id = 1
    log-bin = mysql-bin
    auto-increment-increment = 2
    auto-increment-offset = 1

    Master2:
    server-id = 2
    log-bin = mysql-bin
    auto-increment-increment = 2
    auto-increment-offset = 1

设置好之后重启Mysql : systemctl restart mysqld
### 2.将master1设为master2的主服务器
在Master1和master2中同时设置如下命令：

#### 1.在Master1和Master2上新建授权用户
    mysql -u root -p
    password:
    mysql>grant replication slave on *.* to 数据库用户名@'%' identified by '数据库密码';
    mysql>show master status;
注：如图所示，其中File的值从表需要用到

#### 2.在master1 上将master2设为自己的主服务器,在master2 上将master1设为自己的主服务器
    Change000master000to000master_host='x.x.x.x',master_user='xxxx',master_password='xxxxx',master_log_file='mysql-bin.000002',master_log_pos=442;（去掉0）
    注：其中 master_host：如果是Master1上设置，那么就写master2的IP，反之一样
    Master_user: 数据库的用户名
    master_password：数据库的密码
    master_log_file：如果是Master1上设置，那么就写master2中查出来的File的值，反之也一样的
    Master_log_pos:如果是Master1上设置，那么就写master2中查出来的postion的值，反之也一样的

#### 3.启动
    start slave;
    show slave status\G‘’

#### 3.同步测试
登录其中一台数据库，建一个库，登录另一台，查看是否存在刚才新建的库，如果存在，则证明双主已经安装完成了
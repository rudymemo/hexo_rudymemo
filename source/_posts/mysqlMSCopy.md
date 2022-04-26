---
title: mysql主主复制
date: 2022-03-24 10:40:00
mp3: /music/Sunny Choi - Happily Ever After.mp3
cover: 'https://cdn.jsdelivr.net/gh/rudymemo/picx.xpoet.cn@main/20220424/qpdSJoD2zZt6YQ9.zne9bs9ijls.jpg'
typora-root-url: ..\..\themes\diaspora\source


---



<h4>1. 介绍</h4>

+ ###### 优点

  > 两台服务器互为主从，可以互相数据同步数据。

+ ###### 缺点

  > 同时操作相同内容时可能导致数据不一致。



<h4>2. 环境搭建</h4>

+ ###### 准备两台虚拟机，环境保持一致

  ###### 我选择的服务器是 CentOS 8.3，数据库版本 MySQL 5.7。

  ###### 下载 CentOS8.3 镜像，使用 VMware Workstation Pro 安装，只需要安装一次就可以了，另外一个可以直接复制已安装好的虚拟机。
  
+ ###### 下载并安装MySQL官方的 Yum Repository

  ```bash
  [root@localhost ~]# wget -i -c http://dev.mysql.com/get/mysql57-community-release-el7-10.noarch.rpm
  
  [root@localhost ~]# yum -y install mysql57-community-release-el7-10.noarch.rpm
  
  # 安装命令之前需要先关闭 mysql 模块
  [root@localhost ~]# yum module disable mysql
  
  [root@localhost ~]# yum -y install mysql-community-server
  
  ```

+ ###### 安装完成之后设置开机启动，然后启动数据库

  ```bash
  [root@localhost ~]# systemctl enable mysqld.service
  
  [root@localhost ~]# systemctl start mysqld.service
  ```

+ ###### 启动之后登录修改密码，root 密码可以通过 mysqld.log 查看到一串临时密码

  ```shell
  [root@localhost ~]# grep "password" /var/log/mysqld.log
  # 登录 mysql
  [root@localhost ~]# mysql -uroot -p
  [root@localhost ~]# Enter password: 这里粘贴刚才临时密码
  # 修改 root 密码，需要字母数字大小写特殊字符才可以修改成功
  mysql>ALTER USER 'root'@'localhost' IDENTIFIED BY 'passwd';
  # 开启远程访问 '%' 代表所有用户，也可以是某一个 IP 地址
  mysql>grant all privileges on *.* to 'root'@'%' identified by 'passwd' with grant option;
  # 执行刷新命令
  mysql>flush privileges;
  ```

+ ###### 数据库调整之后，还需要防火墙放开3306端口

  ```shell
  [root@localhost ~]# firewall-cmd --zone=public --add-port=3306/tcp --permanent
  
  [root@localhost ~]# firewall-cmd --reload
  ```


- ###### 修改数据库编码方式修改，编辑 my.cnf 文件，增加开头两行以及结尾两行代码

  ```shell
  [root@localhost ~]# vi /etc/my.cnf
  [client]
  default-character-set=utf8
  # For advice on how to change settings please see
  # http://dev.mysql.com/doc/refman/5.7/en/server-configuration-defaults.html
  
  [mysqld]
  #
  # Remove leading # and set to the amount of RAM for the most important data
  # cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
  # innodb_buffer_pool_size = 128M
  #
  # Remove leading # to turn on a very important data integrity option: logging
  # changes to the binary log between backups.
  # log_bin
  #
  # Remove leading # to set options mainly useful for reporting servers.
  # The server defaults are faster for transactions and fast SELECTs.
  # Adjust sizes as needed, experiment to find the optimal values.
  # join_buffer_size = 128M
  # sort_buffer_size = 2M
  # read_rnd_buffer_size = 2M
  datadir=/var/lib/mysql
  socket=/var/lib/mysql/mysql.sock
  character-set-server=utf8
  collation-server=utf8_general_ci
  
  [root@localhost ~]# systemctl restart mysqld
  ```

  ###### 测试连接数据库可以连接成功，至此一台环境已经搭建完成，然后复制虚拟机即可完成另外一台。

<h4>3. 主主复制实现</h4>

+ ###### 第一台主服务器配置

  ```shell
  [root@localhost ~]# vi /etc/my.cnf
  server-id=1/ / 注意两台服务器 server-id 不一样
  auto_increment_increment=2 // 增长幅度
  auto_increment_offset=1 // 开始点，第一台从 1 开始，第二台从 2 开始。
  ```

  ```shell
  # 登录到 mysql
  mysql>grant replication slave on *.* to 'root'@'192.168.20.134' identified by 'passwd';
  mysql>show master status;
  +------------------+----------+--------------+------------------+-------------------+
  | File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
  +------------------+----------+--------------+------------------+-------------------+
  | mysql-bin.000004 |      154 |              |                  |                   |
  +------------------+----------+--------------+------------------+-------------------+
  1 row in set (0.00 sec)
  mysql>change master to    
  	->master_host='192.168.1.134'
  	->master_user='root'
  	->master_password='passwd'
  	->master_log_file='mysql-bin.000004'
  	->master_log_pos=154;
  mysql>start slave;
  ```

+ ###### 第二台主服务器配置

  ```shell
  [root@localhost ~]# vi /etc/my.cnf
  server-id=2 // 注意两台服务器 server-id 不一样
  auto_increment_increment=2 // 增长幅度
  auto_increment_offset=2 // 开始点
  ```

  ```shell
  # 登录到 mysql
  mysql>grant replication slave on *.* to 'root'@'192.168.20.128' identified by 'passwd';
  mysql>show master status;
  +------------------+----------+--------------+------------------+-------------------+
  | File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
  +------------------+----------+--------------+------------------+-------------------+
  | mysql-bin.000004 |      154 |              |                  |                   |
  +------------------+----------+--------------+------------------+-------------------+
  1 row in set (0.00 sec)
  mysql>change master to    
  	->master_host='192.168.1.128'
  	->master_user='root'
  	->master_password='passwd'
  	->master_log_file='mysql-bin.000004'
  	->master_log_pos=154;
  mysql>start slave;
  ```

  ###### 到此两台服务器主主复制配置完成。




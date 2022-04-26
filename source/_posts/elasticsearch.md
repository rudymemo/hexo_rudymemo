---
title: elasticsearch单机版安装
date: 2019-03-12 12:38:45
mp3: /music/Memories - East Root.mp3
cover: 'https://cdn.jsdelivr.net/gh/rudymemo/picx.xpoet.cn@main/20220424/X8jUcWsLPM4ivrb.b2ljpodym3c.jpg'
typora-root-url: ..\..\themes\diaspora\source
---

本文记录一下Elasticsearch单机版安装中遇到的问题：

- elasticsearch不可以用root用户启动

  ```java
  adduser 用户名 //设置用户名
  passwd 用户名  //设置密码
  chown -R 用户名：用户名 路径 //设置权限
  su 用户名 //可切换用户
  ```

- 下载安装包，解压安装

  [官网下载链接](https://www.elastic.co/cn/start?elektra=home&amp;storm=banner)

  linux利用wget下载安装包

  ```java
  tar -xvf elasticsearch-6.6.1.tar.gz
  ```

  [安装帮助文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started-install.html)

  进入bin目录

  ```
  cd elasticsearch-6.6.1/bin
  ```

  然后执行

  ```
  ./elasticsearch
  ```

  启动之后第一个问题：

  ```
  OpenJDK 64-Bit Server VM warning: If the number of processors is expected to increase from one, then you should configure the number of parallel GC threads appropriately using -XX:ParallelGCThreads=N
  ```

  在GitHub中搜索到了解决方案：[elasticsearch-issues](https://github.com/elastic/elasticsearch/issues/22245)

  ```
  cd /elasticsearch-6.6.1/config/
  vi jvm.options
  ```

  添加参数：-XX:-AssumeMP即可解决。

- 第二个问题

  ```
  max file descriptors [65535] for elasticsearch process is too low, increase to at least [65536]
  ```

  切换到root用户

  ```
  vi /etc/security/limits.conf
  ```

  修改 

  \* soft nofile 65536

  \* hard nofile 65536

  保存之后切换到原用户

- 第三个问题

  单机版内存大小问题如下：

  ```
  max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
  ```

  解决：

  切换到root用户

  执行命令：

  sysctl -w vm.max_map_count=262144

  查看结果：

  sysctl -a|grep vm.max_map_count

  显示：

  vm.max_map_count = 262144

  在此切换到之前的用户然后启动即可。

- 外网访问问题

  单机版安装启动成功之后根据端口号输入网址无法访问，经查原来默认只有本机可以访问，解决方法如下：

  修改elasticsearch.yml文件

  ```
  cd /elasticsearch-6.6.1/config/
  vi elasticsearch.yml
  ```

  #network.host: 192.168.0.1

  修改为本机IP，去掉注释即可。


![](https://cdn.jsdelivr.net/gh/rudymemo/picx.xpoet.cn/202204261112444.png)

至此单机版完成。
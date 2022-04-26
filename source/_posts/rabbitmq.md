---
title: rabbitMQ安装
date: 2018-10-23 12:38:45
mp3: /music/Olimpica - Roberto Cacciapaglia.mp3
cover: 'https://cdn.jsdelivr.net/gh/rudymemo/picx.xpoet.cn@main/20220424/sc9MOp2L3RH4i6q.2ssih7b6cg00.jpg'
typora-root-url: ..\..\themes\diaspora\source
---





系统版本：

![](/img/rabbitMQ/微信图片_20190225153504.png)



[官网安装说明](http://www.rabbitmq.com/install-rpm.html)

按照官网说明安装的是最新版本的，中间踩过不少坑，缺少的gcc以及socat就不说了，安装就行了，我的最终结果如下图：

![](/img/rabbitMQ/微信图片_20190225154006.png)

![](/img/rabbitMQ/微信图片_20190225154010.png)

我是很无语，最终选择放弃。

------

改用下面两个版本进行安装：

- wget <http://www.rabbitmq.com/releases/rabbitmq-server/v3.5.0/rabbitmq-server-3.5.0-1.noarch.rpm>
- wget <http://www.rabbitmq.com/releases/erlang/erlang-19.0.4-1.el7.centos.x86_64.rpm>
- rpm -ivh erlang-19.04-1.e17.centos.x86_64.rpm
- rmp -ivh rabbitmq-server-3.5.0-1.noarch.rpm

以上安装顺利，下面就是检查安装以及启动等。

- service rabbitmq-server start 启动
- service rabbitmq-server stop 停止
- 其他快捷方式可自行查阅

然后访问 ip:15672，发现无法打开，官网有说明：

![](/img/rabbitMQ/微信图片_20190225155615.png)

打开 http://www.rabbitmq.com/management.html 可以看到命令如下：

```
rabbitmq-plugins enable rabbitmq_management
```

执行之后即可。



默认用户名密码：guest/guest，只不过guest用户只能在localhost下访问（为了安全），可以更改配置文件将ebin目录下rabbit.app中loopback_users里的<<"guest">>删除（不建议，现在我主要是以启动进入为目的，后面再好好研究）。

![](/img/rabbitMQ/微信截图_20190225160240.png)



可以看到已经成功进入。



待续。。。


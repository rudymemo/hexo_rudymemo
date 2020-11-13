---
title: springBoot连接rabbitMQ
date: 2018-10-24
mp3: /music/Again - Robert J. P. Oberg.flac
cover: /img/wallhaven-341874.jpg
typora-root-url: ..\..\themes\diaspora\source
---

上篇文章讲述了安装rabbitmq的过程，下面介绍一下springBoot连接rabbitmq。

首先引入dependence：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

连接配置：

```properties
# rabbitMQ
spring.rabbitmq.username=rudy
spring.rabbitmq.password=124512
spring.rabbitmq.host=ip
spring.rabbitmq.port=5672
spring.rabbitmq.virtual-host=/
# 手动ACK 默认 none
spring.rabbitmq.listener.simple.acknowledge-mode=manual
```


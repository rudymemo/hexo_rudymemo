---
title: spring boot 日志原理
date: 2018-11-02
mp3: /music/Blue - Rob Simonsen.mp3
cover: 'https://cdn.jsdelivr.net/gh/rudymemo/picx.xpoet.cn@main/20220424/yduzlIMYmfB248a.79lzzqc8mi40.jpg'
typora-root-url: ..\..\themes\diaspora\source
---

[官方文档](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#boot-features-logging)

介绍spring boot 日志之前先说一下spring 的日志原理，下面的介绍基于spring 5。

通过研究spring源码可以得知使用的是jcl门面日志，原始jcl默认使用log4j-jdk14-jdk13-simplelog。而spring对jcl进行了重写，默认值jul，通过static方法判断使用log4j2或者slf4j从而更改默认值。

![](https://cdn.jsdelivr.net/gh/rudymemo/picx.xpoet.cn/img/202204261109098.png)

spring boot 则采用slf4j + logback(默认)来记录日志。

![部分依赖截图](https://cdn.jsdelivr.net/gh/rudymemo/picx.xpoet.cn/img/202204261109173.png)

如果不想使用logback需要先排除掉logback启动类，然后加入log4j2的启动类，配置如下：

![更换日志](https://cdn.jsdelivr.net/gh/rudymemo/picx.xpoet.cn/img/202204261109354.png)

spring boot 之所以不直接采用spring的日志是因为jcl官网已经不提供更新了，slf4j是一个很全面的门面日志，现阶段一直更新。
---
title: GitHub个人网站修改DNS，解决国内访问慢的问题
date: 2019-09-26 15:08:06
mp3: /music/Letting Go - Paul Cardall.mp3
cover: 'https://cdn.jsdelivr.net/gh/rudymemo/picx.xpoet.cn@main/20220424/9qHIBlxL4vQdKyi.2fff1x6i5tus.jpg'
typora-root-url: ..\..\themes\diaspora\source
---



首先你要有一个GitHub的个人网站（^_^），这个后续写一下利用GitHub创建个人网站以及域名解析。

DNS我选的是[cloudflare](https://www.cloudflare.com/)，具体注册流程就不写了。

- 首先添加站点：

![](https://cdn.jsdelivr.net/gh/rudymemo/picx.xpoet.cn/img/202204261111781.jpg)

![](https://cdn.jsdelivr.net/gh/rudymemo/picx.xpoet.cn/img/202204261111226.jpg)

- 选择Free，然后Confirm plan

![](https://cdn.jsdelivr.net/gh/rudymemo/picx.xpoet.cn/img/202204261111757.png)



- 添加解析，如下图：



![](https://cdn.jsdelivr.net/gh/rudymemo/picx.xpoet.cn/img/202204261111082.png)

- 然后continue

![](https://cdn.jsdelivr.net/gh/rudymemo/picx.xpoet.cn/img/202204261111280.jpg)

这个就是需要去域名注册商删除现在的第一步提示的DNS记录，然后替换成Nameserver1和Nameserver2。

去阿里云修改DNS，[具体方式请点击阿里官方链接参考](https://help.aliyun.com/document_detail/54157.html?spm=a2c4g.11186623.2.13.4aac538faj1K3B)。

修改完成之后等待结果，一般修改正确很快就可以了，成功入下图所示：

![](https://cdn.jsdelivr.net/gh/rudymemo/picx.xpoet.cn/img/202204261111835.jpg)

- 问了几个朋友访问都很快

![](https://cdn.jsdelivr.net/gh/rudymemo/picx.xpoet.cn/img/202204261111263.png)

![](https://cdn.jsdelivr.net/gh/rudymemo/picx.xpoet.cn/202204261111756.png)

缓存后更快，几乎秒开。










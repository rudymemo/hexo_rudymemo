---
title: windows脚本拷贝文件到共享文件夹
date: 2020-06-03 17:09:41
mp3: /music/Abundance of the Heart - Rachel Currea.mp3
cover: /img/th.jpg
typora-root-url: ..\..\themes\diaspora\source

---



最近有一个需求：把用户上传的附件备份到局域网共享文件夹中。

由于不是实时备份所以就打算每天凌晨把前一天上传的附件copy到共享文件夹中。具体思路就是先写bat脚本，然后创建计划任务。如果熟悉bat脚本的话写起来应该很简单，windows通过界面创建计划任务也不麻烦（没有Linux简单）。好了，思路有了，脚本写好了，计划任务也创建了，结果却失败了。。。

​	开始我的做法是：

1. 编写bat脚本
2. 创建计划任务
3. 添加映射网络驱动器，把共享文件夹映射到本地磁盘目录

找一下原因吧，搜索关键字（windows创建计划任务没有执行脚本），然后找到了原因。bat脚本在执行的时候需要利用NET USE连接共享文件夹得到授权才可以执行copy，手动映射的找不到路径，于是增加了两行代码如下：

```
// 每次连接前需要先断开连接
NET USE 本地地址 /delete
// 取得授权保持连接
NET USE 共享地址 本地地址 /PERSISTENT YES 
```










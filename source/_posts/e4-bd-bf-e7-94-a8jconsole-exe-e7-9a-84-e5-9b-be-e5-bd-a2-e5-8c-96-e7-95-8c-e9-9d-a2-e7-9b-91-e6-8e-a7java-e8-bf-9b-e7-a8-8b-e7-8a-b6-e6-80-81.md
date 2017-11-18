---
title: 使用jconsole.exe的图形化界面监控Java进程状态
tags:
  - Java
  - jconsole
  - 内存
  - 进程
id: 62
categories:
  - 笔记
date: 2014-08-22 12:06:47
---

启动程序时加控制台参数：

<pre class="lang:default decode:true ">-Dcom.sun.management.jmxremote.port=9112 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false</pre>

注意参数**-Dcom.sun.management.jmxremote.port=9112**  表示监控的端口
启动程序后，使用jdk路径下的bin下的jconsole.exe工具
![](http://img.my.csdn.net/uploads/201304/22/1366612298_1333.png)
点击连接
![](http://img.my.csdn.net/uploads/201304/22/1366612393_1244.png)
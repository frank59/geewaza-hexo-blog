---
title: Java常用自带工具介绍
tags:
  - Java
  - JVM
id: 210
categories:
  - 笔记
date: 2015-01-08 11:27:03
---

> jps
>   jinfo
>   jmap
>   jstack
>   JConsole
>   Visual VM
>   <!--more-->

## 一、jps

jps(Java Virtual Machine Process Status Tool)是JDK 1.5提供的一个显示当前所有java进程pid的命令，简单实用，非常适合在linux/unix平台上简单察看当前java进程的一些简单情况。
jps存放在JAVA_HOME/bin/jps，使用时为了方便请将JAVA_HOME/bin/加入到Path。
基本使用方式：
![](http://ww3.sinaimg.cn/mw690/3d6ce2f1jw1enyieq1osmj20bd01vaa3.jpg)
**不过，jps命令有个地方很不好，似乎只能显示当前用户的java进程，要显示其他用户的还是只能用unix/linux的ps命令。**

### 参数使用

#### -q

只显示pid，不显示class名称,jar文件名和传递给main方法的参数
![](http://ww1.sinaimg.cn/mw690/3d6ce2f1jw1enyid8zv6qj205g01rwec.jpg)

#### -m

输出传递给main方法的参数，在嵌入式jvm上可能是null
![](http://ww2.sinaimg.cn/mw690/3d6ce2f1jw1enyihb2g86j214003bq67.jpg)

#### -l

输出应用程序main class的完整package名 或者 应用程序的jar文件完整路径名
![](http://ww4.sinaimg.cn/mw690/3d6ce2f1jw1enyj5li69dj20gq01jjrq.jpg)

#### -v

输出传递给JVM的参数
![](http://ww3.sinaimg.cn/mw690/3d6ce2f1jw1enyj71t6d6j20p001lt9f.jpg)

## 二、jinfo

jinfo可以输出并修改运行时的java进程的JVM参数。用处比较简单，用于输出JAVA系统参数及命令行参数。用法是jinfo -opt  pid
**另外，jinfo可能在未来的jvm中移除**
![](http://ww3.sinaimg.cn/mw690/3d6ce2f1jw1enyq5b982tj20je0fpgnd.jpg)

#### -flag _name_

<span class="lang:default decode:true  crayon-inline ">-flag &lt;name&gt;</span>  打印该Java进程的指定的JVM参数的值
如：查看13092的MaxPerm大小可以用
![](http://ww2.sinaimg.cn/mw690/3d6ce2f1jw1enypkcbjbtj209f01cdfw.jpg)

#### -flag [+|-]_name_

<span class="lang:default decode:true  crayon-inline ">-flag [+|-]&lt;name&gt;</span>  控制生效或不生效某些JVM参数
如生效PrintGCDetails参数
![](http://ww4.sinaimg.cn/mw690/3d6ce2f1jw1enyprermkvj20ah00s0sp.jpg)

#### -flag _name_=_value_

<span class="lang:default decode:true  crayon-inline ">-flag &lt;name&gt;=&lt;value&gt;</span>  设定名为name的参数为指定值
如果出现下面这样的情况，说明参数不支持修改
![](http://ww1.sinaimg.cn/mw690/3d6ce2f1jw1enypyxsy50j20iq03f3zt.jpg)

#### -flags

查看进程的所有JVM参数
![](http://ww2.sinaimg.cn/mw690/3d6ce2f1jw1enyq5au199j20k503kdg2.jpg)

## 三、jmap

打印出某个Java进程（使用pid）内存内的，所有‘对象’的情况（如：产生那些对象，及其数量）。
使用方法：

> jmap [ option ] pid
>   jmap [ option ] executable core
>   jmap [ option ] [server-id@]remote-hostname-or-IP

![](http://ww1.sinaimg.cn/mw690/3d6ce2f1jw1enzu9p6heoj20gg07uwid.jpg)

### 参数

#### -histo

打印每个class的实例数目,内存占用,类全名信息。VM的内部类名字开头会加上前缀“*”。 如果live子参数加上后，只统计活的对象数量。
![](http://ww2.sinaimg.cn/mw690/3d6ce2f1jw1enztov6ebtj20gx0dkn0u.jpg)

#### -dump:[live,]format=b,file=_filename_

使用hprof二进制形式,输出jvm的heap内容到文件。live子选项是可选的，假如指定live选项,那么只输出活的对象到文件。
![](http://ww4.sinaimg.cn/mw690/3d6ce2f1jw1enztwyyfpzj20fh01djrs.jpg)

#### -finalizerinfo

打印正等候回收的对象的信息。
![](http://ww1.sinaimg.cn/mw690/3d6ce2f1jw1enzu51muwcj20ci02raar.jpg)

#### -heap

打印heap的概要信息，GC使用的算法，heap的配置及wise heap的使用情况。
![](http://ww3.sinaimg.cn/mw690/3d6ce2f1jw1enzu2wg77cj20an0kpafu.jpg)

#### -permstat

打印classload和jvm heap长久层的信息。包含每个classloader的名字，活泼性，地址，父classloader和加载的class数量。另外，内部String的数量和占用内存数也会打印出来。
![](http://ww1.sinaimg.cn/mw690/3d6ce2f1jw1enzujom5k3j20p50ieao8.jpg)

#### -J

传递参数给jmap启动的jvm。

#### -F

强迫。在pid没有相应的时候使用-dump或者-histo参数。在这个模式下，live子参数无效。

## 四、jstack

jstack用于打印出给定的java进程ID或core file或远程调试服务的Java堆栈信息，如果是在64位机器上，需要指定选项"-J-d64"，Windows的jstack使用方式只支持以下的这种方式：

> jstack [-l] pid

使用方法：

> jstack [-l] &lt;pid&gt;
>   jstack -F [-m] [-l] &lt;pid&gt;
>   jstack [-m] [-l] &lt;executable&gt; &lt;core&gt;
>   jstack [-m] [-l] [server_id@]&lt;remote server IP or hostname&gt;

![](http://ww3.sinaimg.cn/mw690/3d6ce2f1jw1eo00k23qu2j20pd0hqgyf.jpg)

### 参数

#### -F

当’jstack [-l] pid’没有相应的时候强制打印栈信息

#### -l

长列表。打印关于锁的附加信息，例如属于java.util.concurrent的ownable synchronizers列表。
![](http://ww2.sinaimg.cn/mw690/3d6ce2f1jw1eo00ogtd0cj20m20cx0wo.jpg)

#### -m

打印java和native c/c++框架的所有栈信息。
![](http://ww1.sinaimg.cn/mw690/3d6ce2f1jw1eo00r6coruj20fk0jtgtw.jpg)

## 五、JConsole

JConsole是一个基于JMX的GUI工具，用于连接正在运行的JVM，不过此JVM需要使用可管理的模式启动。如果要把一个应用以可管理的形式启动，可以在启动是设置`com.sun.management.jmxremote`。例如，启动一个可以在本地监控的J2SE的应用Java2Demo ，需输入以下命令：

    JDK_HOME/bin/java -Dcom.sun.management.jmxremote -jar JDK_HOME/demo/jfc/Java2D/Java2Demo.jar
    `</pre>

    概述界面：
    ![](http://ww2.sinaimg.cn/mw690/3d6ce2f1jw1eo0staiyr3j20p00kumzt.jpg)
    内存界面：
    ![](http://ww2.sinaimg.cn/mw690/3d6ce2f1jw1eo0stayj6bj20p00kun0f.jpg)
    线程界面：
    ![](http://ww4.sinaimg.cn/mw690/3d6ce2f1jw1eo0stbw00yj20p00kujuv.jpg)
    类界面：
    ![](http://ww2.sinaimg.cn/mw690/3d6ce2f1jw1eo0stcatxoj20p00kuq5g.jpg)
    VM摘要界面：
    ![](http://ww4.sinaimg.cn/mw690/3d6ce2f1jw1eo0stco580j20p00kutfv.jpg)
    MBean界面：
    ![](http://ww2.sinaimg.cn/mw690/3d6ce2f1jw1eo0std5fx3j20p00ku78b.jpg)

    ### 远程监控

    启动程序时加控制台参数：

    <pre>`-Dcom.sun.management.jmxremote.port=9112 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false

注意参数`-Dcom.sun.management.jmxremote.port=9112`表示监控的端口
启动程序后，使用jdk路径下的bin下的jconsole.exe工具
![](http://ww2.sinaimg.cn/mw690/3d6ce2f1jw1enyqc7vr87j20bk0cxjt3.jpg)
点击链接后就可以远程监控Java进程的运行状态了：
![](http://ww2.sinaimg.cn/mw690/3d6ce2f1jw1enyqc7z89sj20ol0juadt.jpg)

## 六、Visual VM

VisualVM 是一款免费的性能分析工具。它通过 jvmstat、JMX、SA（Serviceability Agent）以及 Attach API 等多种方式从程序运行时获得实时数据，从而进行动态的性能分析。同时，它能自动选择更快更轻量级的技术尽量减少性能分析对应用程序造成的影响，提高性能分析的精度。
![](http://ww4.sinaimg.cn/mw690/3d6ce2f1jw1eo1xvxp74cj20p00ge0x1.jpg)

### 安装插件

![](http://ww3.sinaimg.cn/mw690/3d6ce2f1jw1eo1xvy6xenj20a104hjs8.jpg)
![](http://ww3.sinaimg.cn/mw690/3d6ce2f1jw1eo1xvydtx5j20pb0fi795.jpg)
**参考：[使用 VisualVM 进行性能分析及调优](http://www.ibm.com/developerworks/cn/java/j-lo-visualvm/)**
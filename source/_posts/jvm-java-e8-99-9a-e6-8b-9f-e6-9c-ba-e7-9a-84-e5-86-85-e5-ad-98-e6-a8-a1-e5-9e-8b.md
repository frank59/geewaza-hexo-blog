---
title: JVM——(1)Java虚拟机的内存模型
tags:
  - JVM
  - 虚拟机
id: 84
categories:
  - 文章
date: 2014-08-25 19:57:56
---

## 运行时数据区域

![Java虚拟机运行时数据区](http://ww4.sinaimg.cn/large/3d6ce2f1jw1ejp24zzba7j209e0ag0ti.jpg)

<!--more-->

### 程序计数器（Programme Counter Register）

每个虚拟机的实现方式可能不同，虚拟机通过这个计数器来选区下一条需要执行的字节码指令以及其他一些基础功能，如异常处理和跳转功能。每个线程都有独立的程序计数器，并且相互不影响，是线程私有的。对于正在执行的Native方法，这个计数器值则为空（Undefined）。这个区域不会出现内存溢出等问题。

### Java虚拟机栈（Java Virtual Machine Stacks）

线程私有的，生命周期与线程相同。每个方法在执行的同时，会创建一个栈帧（Stack Frame）用于存储局部变量、操作数栈、动态链接、方法出口等信息。方法开始执行时，压入这个栈帧，方法执行完成，这个栈帧就出栈。
递归调用方法如果递归的深度过深，就会出现栈溢出（StackOverflowError）异常，值得就是这个地方的栈溢出。如果这个区域允许动态扩展，但是无法申请到足够的内存，就会内存溢出（OutOfMemoryError）。
设置参数：
-Xss 调整栈内存容量

### 本地方法栈（Native Method Stack）

线程私有。与上面提到的Java虚拟机栈（Java Virtual Machine Stacks）相对应，Java虚拟机栈为Java方法服务，本地方法栈为Native方法服务。Sun HotSpot将这两个栈合二为一。栈溢出和内存溢出规则也一样。
设置参数：
-Xoss 调整栈内存容量（Hot spot虚拟机无效）
虚拟机栈和本地方法栈的-Xss（-Xoss）参数影响了栈的深度，当抛出StackOverflowError异常说明-Xss参数设置过小。不停递归调用会抛出此异常。定义大量本地变量增加方法帧本地变量表的长度也会抛出此异常。

> 每个线程分配到的栈容量越大，可以建立的线程数量自然就减少，建立线程时就越容易吧剩下的内存耗尽。
>   如果是建立过多线程导致的内存溢出，在不能减少线程数或者更换64位虚拟机的情况下，就只能通过减少最大堆和减少栈容量来换取更多的线程。

### Java堆（Heap）

线程共享的一块内存区域，在虚拟机启动时创建。唯一的目的是存放对象实例。垃圾回收主要发生在这一区域。
Java堆还可以细分为：新生代，老生代。还可以细分如下图：![Java虚拟机堆内存细分](http://ww4.sinaimg.cn/large/3d6ce2f1jw1ejp2611qt8j20ku0cqdgr.jpg)
设置参数：
-Xmx 堆最大大小
-Xms 堆初始大小
理论上只要不断创建对象，并且保证GC Roots到对象之间有可达路径就能避免垃圾回收机制清除这些对象。当内存无法再分配时，就抛出OOM异常。

### 方法区（Method Area）

线程共享的一块内存区域，用于存储已被虚拟机即在的类信息、常量、静态变量、即时编译器编译后的代码等数据。常被称为“永生区”，但这种说法只限于Hotspot，BEA JRockit、IBM J9等虚拟机不存在“永生区”的概念，Hotspot也在计划使用Native Memory来替代“永生区”，JDK1.7已经将原本放在永生区的字符串常量池移出。
设置参数：
-XX:MaxPermSize 设置方法区上限

#### 运行时常量池（Runtime Constant Pool）

是上面提到的方法区的一部分。用于存放编译器生成的各种字面量和符号引用。

> String.intern() 是一个Native方法，它的作用是：如果字符串常量池中已经包含一个等于此String对象的字符串，则返回代表池中这个字符串的String对象；否则，将此String对象包含的字符串添加到常量池中，并且返回此String对象的引用。在JDK1.6及之前的版本中，由于常量池分配在永久代内，我们可以通过-XX:PermSize和-XX:MaxPermSize限制方法区大小，从而间接限制其中常量池的容量。
>   而在JDK1.7（以及部分其他虚拟机，例如JRockit）的intern()实现不会再复制实例，只是在常量池中记录首次出现的实例引用。

在是同CGLib工具时（如 Spting, Hibernate等框架），可能会动态生成大量的类，可能会造成PermGen的OOM。

### 直接内存（Direct Memory）

> 在JDK1.4中心加入了NIO（New Input/Output）类，银土了一种基于通道（Channel）与缓冲区（Buffer）的I/O方式，它可以使用Native函数库直接分配堆外内存然后通过一个存储在Java堆中的DirectByteBuffer对象作为这块内存的引用进行操作

设置参数：
-XX:MaxDirectMemorySize 指定直接内存的大小
使用rt.jar中的Unsafe的功能如：

<pre class="lang:default decode:true ">public class DirectMemoryOOM {
    private static final int _1MB = 1024 * 1024;
    public static void main(String[] args) throws Exception {
        Field unsafeField = Unsafe.class.getDeclaredFields()[0];
        unsafeField.setAccessible(true);
        Unsafe unsafe = (Unsafe) unsafeField.get(null);
        while (true) {
            //分配内存
            unsafe.allocateMemory(_1MB);
        }
    }
}</pre>

<span style="line-height: 1.714285714; font-size: 1rem;">使用NIO时要注意这种情况。</span>
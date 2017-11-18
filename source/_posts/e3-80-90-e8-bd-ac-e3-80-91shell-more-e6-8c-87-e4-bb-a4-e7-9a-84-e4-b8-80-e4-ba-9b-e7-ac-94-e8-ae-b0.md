---
title: 【转】shell more指令的一些笔记
tags:
  - Linuex
  - more
  - shell
  - 转载
id: 245
categories:
  - 笔记
date: 2015-04-15 10:17:21
---

原帖：[http://blog.csdn.net/cneaglelee/article/details/6216589](http://blog.csdn.net/cneaglelee/article/details/6216589)
more在查看文件内容时很常用。有同学问我，使用more怎样到文件尾，怎样从文件尾开始查看。这些类似的技巧可能大家不怎么关注，的确有时也很有必要。现做个简单的总结。
<!--more-->

## more基本使用

类似 cat ，不过会以一页一页的显示方便使用者逐页阅读，而最基本的指令就是按空白键（space）就往下一页显示，按 b 键就会往回（back）一页显示，而且还有搜寻字串的功能（与 vi 相似），使用中使用帮助按 h 。

## more后面跟的参数

-num 一次显示的行数
-d 提示使用者，在画面下方显示 [Press space to continue, 'q' to quit.] ，如果使用者按错键，则会显示 [Press 'h' for instructions.] 而不是 '哔' 声
-l 取消遇见特殊字元 ^L（送纸字元）时会暂停的功能
-f 计算行数时，以实际上的行数，而非自动换行过后的行数（有些单行字数太长的会被扩展为两行或两行以上）
-p 不以卷动的方式显示每一页，而是先清除萤幕后再显示内容
-c 跟 -p 相似，不同的是先显示内容再清除其他旧资料
-s 当遇到有连续两行以上的空白行，就代换为一行的空白行
-u 不显示下引号 （根据环境变数 TERM 指定的 terminal 而有所不同）
+/ 在每个档案显示前搜寻该字串（pattern），然后从该字串之后开始显示
+num 从第 num 行开始显示

## more文件后的参数

f [K]有行数K向前（forward）移动K行，没有K，向前移动一屏
b [K]有行数K向后（backward）移动K行，没有K，向后移动一屏
d [K]有行数K向前（forward）移动K行，没有K，向前移动半屏
u [K]有行数K向后（backward）移动K行，没有K，向后移动半屏
j [K]有行数K向前（forward）移动K行，没有K，向前移动1行
k [K]有行数K向后（backward）移动K行，没有K，向后移动1行
g [K]有行数K向前（forward）移动K行，没有K，移动到文件首
G [K]有行数K向后（backward）移动K行，没有K，移动到文件首
p [K]%向后移动的百分比
更详细的帮助你可以在linux/unix下使用man more。

## more问题解决

（1）如何查看到文件尾，more文件后直接输入G字符。
（2）怎样从文件尾开始查看，more -p G 文件名，前翻使用b或u就可以了。
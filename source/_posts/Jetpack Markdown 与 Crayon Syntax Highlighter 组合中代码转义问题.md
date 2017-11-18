---
title: Jetpack Markdown 与 Crayon Syntax Highlighter 组合中代码转义问题
tags:
  - Wordpress
id: 232
categories:
  - 笔记
date: 2015-01-15 11:20:00
---
使用Jetpack Markdown和Crayon Syntax Highlighter插件组合时常常出现这样的问题：
代码：
```shell
# Redis示例配置文件

# 注意单位问题：当需要设置内存大小的时候，可以使用类似1k、5GB、4M这样的常见格式：
#
# 1k => 1000 bytes
# 1kb => 1024 bytes
# 1m => 1000000 bytes
# 1mb => 1024*1024 bytes
# 1g => 1000000000 bytes
# 1gb => 1024*1024*1024 bytes
... 
```
提交过后变成了：
![](http://ww2.sinaimg.cn/mw690/3d6ce2f1jw1eoa0nll80aj20lu04ewfe.jpg)
<!--more-->


造成问题的原因大概是：在后台编辑框中提交的文本被保存到数据库中，在前台展示时才会经过Markdown转码。但是做的是先由Markdown根据语法转码后交由Crayon Syntax Highlighter进行代码高亮的渲染。而Markdown会将代码中的特殊符号经由HTML进行转义，而Highlighter会原封不动得显示`<pre>`标签中的代码，于是转义过后的代码就被原封不动地展示出来了。所以Highlighter必须在渲染时将转义过后的代码再转义回来。后台选项如下：
![](http://ww4.sinaimg.cn/mw690/3d6ce2f1jw1eoa0cinyymj20ck0erdj3.jpg)
对于行内代码，Markdown依然会转义，取消下面选项后，需要重新提交文章才能生效：
![](http://ww1.sinaimg.cn/mw690/3d6ce2f1jw1eoa0nlv300j2088023wet.jpg)
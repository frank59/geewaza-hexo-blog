---
title: Java插件式框架PF4J
tags:
  - Java
  - PF4J
  - 插件框架
id: 65
categories:
  - 笔记
date: 2014-08-22 15:09:21
---

原文地址：[http://www.oschina.net/p/pf4j](http://www.oschina.net/p/pf4j "http://www.oschina.net/p/pf4j")

PF4J 是一个 Java 的插件框架，为第三方提供应用扩展的渠道。使用 PF4J 你可以轻松将一个普通的 Java 应用转成一个模块化的应用。PF4J 本身非常轻量级，只有 50KB 左右，目前只依赖了 slf4j。Gitblit 项目使用的就是 PF4J 进行插件管理。
Maven：

<pre class="lang:default decode:true ">&lt;dependency&gt;
    &lt;groupId&gt;ro.fortsoft.pf4j&lt;/groupId&gt;
    &lt;artifactId&gt;pf4j&lt;/artifactId&gt;
    &lt;version&gt;${pf4j.version}&lt;/version&gt;
&lt;/dependency&gt;</pre>

实例代码：

<pre class="lang:java decode:true ">public static void main(String[] args) {
    ...

    PluginManager pluginManager = new DefaultPluginManager();
    pluginManager.loadPlugins();
    pluginManager.startPlugins();

    ...
}</pre>

[repo path="decebals/pf4j"]
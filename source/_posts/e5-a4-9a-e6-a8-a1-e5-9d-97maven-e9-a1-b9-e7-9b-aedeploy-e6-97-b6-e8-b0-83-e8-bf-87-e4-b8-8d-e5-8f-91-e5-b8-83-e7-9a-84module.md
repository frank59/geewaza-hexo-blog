---
title: 多模块Maven项目deploy时调过不发布的Module
tags:
  - Maven
id: 268
categories:
  - 笔记
date: 2016-12-07 10:11:45
---

在不需要发布的Module的pom.xml文件中使用以下插件

                &lt;plugin&gt;
                    &lt;groupId&gt;org.apache.maven.plugins&lt;/groupId&gt;
                    &lt;artifactId&gt;maven-deploy-plugin&lt;/artifactId&gt;
                    &lt;version&gt;2.8.2&lt;/version&gt;
                    &lt;configuration&gt;
                        &lt;skip&gt;true&lt;/skip&gt;
                    &lt;/configuration&gt;
                &lt;/plugin&gt;
    
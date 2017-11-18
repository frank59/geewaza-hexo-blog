---
title: Maven配置不同环境使用不同的配置文件
tags:
  - Maven
id: 260
categories:
  - 笔记
date: 2016-05-20 18:35:56
---

代目目录结构：
![http://ww1.sinaimg.cn/mw690/3d6ce2f1jw1f4215vf124j20g503vq3o.jpg](http://ww1.sinaimg.cn/mw690/3d6ce2f1jw1f4215vf124j20g503vq3o.jpg)
<!--more-->

pom.xml

        &lt;profiles&gt;
            &lt;profile&gt;
                &lt;!-- 开发环境 --&gt;
                &lt;id&gt;dev&lt;/id&gt;
                &lt;activation&gt;
                    &lt;activeByDefault&gt;true&lt;/activeByDefault&gt;
                &lt;/activation&gt;
                &lt;properties&gt;
                    &lt;deploy.type&gt;dev&lt;/deploy.type&gt;
                &lt;/properties&gt;
            &lt;/profile&gt;
            &lt;profile&gt;
                &lt;!-- 测试环境 --&gt;
                &lt;id&gt;test&lt;/id&gt;
                &lt;properties&gt;
                    &lt;deploy.type&gt;test&lt;/deploy.type&gt;
                &lt;/properties&gt;
            &lt;/profile&gt;
            &lt;profile&gt;
                &lt;!-- 生产环境 --&gt;
                &lt;id&gt;online&lt;/id&gt;
                &lt;properties&gt;
                    &lt;deploy.type&gt;online&lt;/deploy.type&gt;
                &lt;/properties&gt;
            &lt;/profile&gt;
        &lt;/profiles&gt;
        &lt;build&gt;
            &lt;resources&gt;
                &lt;!-- 配两个资源目录 --&gt;
                &lt;resource&gt;
                    &lt;directory&gt;src/main/resources&lt;/directory&gt;
                &lt;/resource&gt;
                &lt;resource&gt;
                    &lt;directory&gt;src/main/resources.${deploy.type}&lt;/directory&gt;
                &lt;/resource&gt;
            &lt;/resources&gt;
            &lt;plugins&gt;
                &lt;plugin&gt;
                    &lt;groupId&gt;org.apache.maven.plugins&lt;/groupId&gt;
                    &lt;artifactId&gt;maven-compiler-plugin&lt;/artifactId&gt;
                    &lt;configuration&gt;
                        &lt;source&gt;1.6&lt;/source&gt;
                        &lt;target&gt;1.6&lt;/target&gt;
                        &lt;encoding&gt;UTF-8&lt;/encoding&gt;
                    &lt;/configuration&gt;
                &lt;/plugin&gt;
            &lt;/plugins&gt;
        &lt;/build&gt;
    
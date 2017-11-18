---
title: 使用Maven根据不同环境替换web.xml
tags:
  - Maven
id: 275
categories:
  - 笔记
date: 2017-01-03 13:52:43
---

在pom.xml增加使用`maven-war-plugin`插件：

        ......
        &lt;profiles&gt;
            &lt;profile&gt;
                &lt;id&gt;product&lt;/id&gt;
                &lt;activation&gt;
                    &lt;activeByDefault&gt;true&lt;/activeByDefault&gt;
                &lt;/activation&gt;
                &lt;properties&gt;
                    &lt;deploy.type&gt;product&lt;/deploy.type&gt;
                &lt;/properties&gt;
            &lt;/profile&gt;
            &lt;profile&gt;
                &lt;id&gt;dev&lt;/id&gt;
                &lt;properties&gt;
                    &lt;deploy.type&gt;dev&lt;/deploy.type&gt;
                &lt;/properties&gt;
            &lt;/profile&gt;
            &lt;profile&gt;
                &lt;id&gt;test&lt;/id&gt;
                &lt;properties&gt;
                    &lt;deploy.type&gt;test&lt;/deploy.type&gt;
                &lt;/properties&gt;
            &lt;/profile&gt;
        &lt;/profiles&gt;
        ......
        &lt;build&gt;
        ......
            &lt;resources&gt;
                &lt;resource&gt;
                    &lt;filtering&gt;true&lt;/filtering&gt;
                    &lt;directory&gt;src/main/resources&lt;/directory&gt;
                &lt;/resource&gt;
                &lt;resource&gt;
                    &lt;filtering&gt;true&lt;/filtering&gt;
                    &lt;directory&gt;src/main/resources.${deploy.type}&lt;/directory&gt;
                    &lt;excludes&gt;
                        &lt;exclude&gt;WEB-INF&lt;/exclude&gt;
                        &lt;exclude&gt;WEB-INF/**.*&lt;/exclude&gt;
                    &lt;/excludes&gt;
                &lt;/resource&gt;
            &lt;/resources&gt;
            ......
            &lt;plugins&gt;
                ......
                &lt;plugin&gt;
                    &lt;groupId&gt;org.apache.maven.plugins&lt;/groupId&gt;
                    &lt;artifactId&gt;maven-war-plugin&lt;/artifactId&gt;
                    &lt;version&gt;2.4&lt;/version&gt;
                    &lt;configuration&gt;
                        &lt;webResources&gt;
                            &lt;resource&gt;
                                &lt;directory&gt;src/main/resources.${deploy.type}/WEB-INF&lt;/directory&gt;
                                &lt;filtering&gt;true&lt;/filtering&gt;
                                &lt;targetPath&gt;WEB-INF&lt;/targetPath&gt;
                            &lt;/resource&gt;
                        &lt;/webResources&gt;
                    &lt;/configuration&gt;
                &lt;/plugin&gt;
                ......
            &lt;/plugins&gt;
        ......
        &lt;/build&gt;

![](http://ww1.sinaimg.cn/mw690/3d6ce2f1jw1fbdebl5h7lj208j04v3yq.jpg)
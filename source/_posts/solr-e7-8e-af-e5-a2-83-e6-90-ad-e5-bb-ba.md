---
title: Solr 环境搭建
tags:
  - Solr
id: 249
categories:
  - 笔记
date: 2016-03-28 16:44:11
---

## 系统环境

- Windows 10
- Tomcate 8.0.24
- JDK 1.7.0.79(x64)
- Solr 4.7.2

<!--more-->

## 搭建步骤

假设JDK以及Tomcate已经安装完成，环境变量也已经配置完成。Tomcate的安装路径D:\tomcat-8.0.24
1. 下载solr：[http://mirrors.hust.edu.cn/apache/lucene/solr/](http://mirrors.hust.edu.cn/apache/lucene/solr/)
2. 解压solr(这里解压到了D:\opt\tmp\solr-4.7.2)，找到D:\opt\tmp\solr-4.7.2\example\webapps中的solr.war，将其拷贝到tomcat的webapps目录下D:\tomcat-8.0.24\webapps
3. 启动一次tomcat，tomcat会在D:\tomcat-8.0.24\webapps下生成solr目录D:\tomcat-8.0.24\webapps\solr，生成之后就关闭tomcat，则可删除solr.war了
![](http://ww3.sinaimg.cn/large/3d6ce2f1jw1f2cnmhcuhoj20hp05cab9.jpg)
4. 打开编辑文件D:\tomcat-8.0.24\webapps\solr\WEB-INF\web.xml, 在文件末尾&lt;web-app/&gt;内添加如下代码

    &lt;env-entry&gt;
    &lt;env-entry-name&gt;solr/home&lt;/env-entry-name&gt;
    &lt;env-entry-value&gt;d:optsolr&lt;/env-entry-value&gt;
    &lt;env-entry-type&gt;java.lang.String&lt;/env-entry-type&gt;
    &lt;/env-entry&gt;
    `</pre>

    ![](http://ww2.sinaimg.cn/mw690/3d6ce2f1jw1f2cnmhfnchj20bn04a3za.jpg)
    如果不存在d:\opt\solr这个目录，则需要手动创建
    5. 回到之前解压的目录中，找到D:\opt\tmp\solr-4.7.2\example\solr目录，将该目录下所有文件拷贝到d:\opt\solr中
    ![](http://ww3.sinaimg.cn/mw690/3d6ce2f1jw1f2cnmhqskrj20h904mmxz.jpg)
    6. 将解压目录下D:\opt\tmp\solr-4.7.2\example\lib\ext 所有的jar文件放到 tomcat的D:\tomcat-8.0.24\webapps\solr\WEB-INF\lib
    ![](http://ww2.sinaimg.cn/mw690/3d6ce2f1jw1f2cnmhz1oxj20cc04jmxx.jpg)
    7. 重启Tomcat,访问本地的solr管理页面：http://localhost:8080/solr
    ![](http://ww1.sinaimg.cn/mw690/3d6ce2f1jw1f2cnmiddopj20ob0hagnl.jpg)

    ## 检验solr是否能正常工作-创建一个索引

    1. 在d:/opt/solr目录下创建一个mycore的目录
    2. 将解压文件中的 D:\opt\tmp\solr-4.7.2\example\multicore\core0 目录下的conf目录拷贝到mycore目录中
    ![](http://ww1.sinaimg.cn/mw690/3d6ce2f1jw1f2cnmhzzncj20et0583z3.jpg)
    3. 在d:/opt/solr目录下创建一个mydocs的目录
    4. 将 D:\opt\tmp\solr-4.7.2\example\exampledocs下的post.jar文件拷贝到mydocs目录中，再将 example\multicore\exampledocs下的ipod_other.xml文件拷贝到mydocs中
    ![](http://ww2.sinaimg.cn/mw690/3d6ce2f1jw1f2cnmiv7rij20be04m3yr.jpg)
    5. 通过solr的管理页面创建一个core:
    ![](http://ww1.sinaimg.cn/mw690/3d6ce2f1jw1f2cnmj4wyrj20js0clta9.jpg)
    6. 重启Tomcat
    7. 使用cmd在mydocs目录下执行以下命令：

    <pre>`java -Durl=http://localhost:8080/solr/mycore/update -Ddata=files -jar post.jar ipod_other.xml

![](http://ww4.sinaimg.cn/mw690/3d6ce2f1jw1f2cnmjce0tj20i4061dgw.jpg)
8. 在solr 的web管理节目中执行以下图示操作，即可看到查询结果，表示solr已能正常工作。
![](http://ww3.sinaimg.cn/mw690/3d6ce2f1jw1f2cnmjsjkyj20ur0n10w5.jpg)
或者使用URL访问查询：
![](http://ww3.sinaimg.cn/mw690/3d6ce2f1jw1f2cnmk0krgj20ko06574t.jpg)
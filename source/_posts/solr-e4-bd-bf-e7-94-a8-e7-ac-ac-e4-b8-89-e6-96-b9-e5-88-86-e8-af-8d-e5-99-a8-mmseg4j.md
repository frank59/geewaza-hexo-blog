---
title: Solr 使用第三方分词器-mmseg4j
tags:
  - Solr
id: 256
categories:
  - 笔记
date: 2016-03-29 19:38:06
---

*   首先确定的是分词器：这里使用的是mmseg4j这个分词器，github地址是：[https://github.com/chenlb/mmseg4j-solr](https://github.com/chenlb/mmseg4j-solr)
注意下载的版本，我的solr是4.7版本的所以下载的包是mmseg4j-solr-2.0.0.jar
*   将下载的包解压，将解压包中的.jar文件全部复制到solr的Web应用目录，我这里是D:\tomcat-8.0.24\webapps\solr\WEB-INF\lib

<!--more-->

![](http://ww4.sinaimg.cn/mw690/3d6ce2f1jw1f2dyezqli5j20ir02imxl.jpg)
- 在solr的目录D:\opt\solr下创建videos目录，这将是我们创建的Core的名字，并创建目录D:\opt\solr\videos\conf，在该目录下拷贝两个文件过来schema.xml, solrconfig.xml：
![](http://ww4.sinaimg.cn/mw690/3d6ce2f1jw1f2dyf0aouij20ls05ymyg.jpg)
![](http://ww4.sinaimg.cn/mw690/3d6ce2f1jw1f2dyf0o6s6j20ii02kjro.jpg)

solrconfig.xml代码:

    &lt;?xml version="1.0" encoding="UTF-8" ?&gt;
    &lt;config&gt;
      &lt;luceneMatchVersion&gt;4.7&lt;/luceneMatchVersion&gt;
      &lt;directoryFactory name="DirectoryFactory" class="${solr.directoryFactory:solr.StandardDirectoryFactory}"/&gt;

      &lt;dataDir&gt;${solr.core0.data.dir:}&lt;/dataDir&gt;
      &lt;schemaFactory class="ClassicIndexSchemaFactory"/&gt;

      &lt;updateHandler class="solr.DirectUpdateHandler2"&gt;
        &lt;updateLog&gt;
          &lt;str name="dir"&gt;${solr.core0.data.dir:}&lt;/str&gt;
        &lt;/updateLog&gt;
      &lt;/updateHandler&gt;
      &lt;requestHandler name="/get" class="solr.RealTimeGetHandler"&gt;
        &lt;lst name="defaults"&gt;
          &lt;str name="omitHeader"&gt;true&lt;/str&gt;
        &lt;/lst&gt;
      &lt;/requestHandler&gt;  

      &lt;requestHandler name="/replication" class="solr.ReplicationHandler" startup="lazy" /&gt; 

      &lt;requestDispatcher handleSelect="true" &gt;
        &lt;requestParsers enableRemoteStreaming="false" multipartUploadLimitInKB="2048" formdataUploadLimitInKB="2048" /&gt;
      &lt;/requestDispatcher&gt;

      &lt;requestHandler name="standard" class="solr.StandardRequestHandler" default="true" /&gt;
      &lt;requestHandler name="/analysis/field" startup="lazy" class="solr.FieldAnalysisRequestHandler" /&gt;
      &lt;requestHandler name="/update" class="solr.UpdateRequestHandler"  /&gt;
      &lt;requestHandler name="/admin/" class="org.apache.solr.handler.admin.AdminHandlers" /&gt;

      &lt;requestHandler name="/admin/ping" class="solr.PingRequestHandler"&gt;
        &lt;lst name="invariants"&gt;
          &lt;str name="q"&gt;solrpingquery&lt;/str&gt;
        &lt;/lst&gt;
        &lt;lst name="defaults"&gt;
          &lt;str name="echoParams"&gt;all&lt;/str&gt;
        &lt;/lst&gt;
      &lt;/requestHandler&gt;
      &lt;admin&gt;
        &lt;defaultQuery&gt;solr&lt;/defaultQuery&gt;
      &lt;/admin&gt;

    &lt;/config&gt;
    `</pre>

    schema.xml代码:

    <pre>`&lt;?xml version="1.0" ?&gt;
    &lt;schema name="videos" version="1.1"&gt;
      &lt;types&gt;
        &lt;fieldtype name="string"  class="solr.StrField" sortMissingLast="true" omitNorms="true"/&gt;
        &lt;fieldType name="long" class="solr.TrieLongField" precisionStep="0" positionIncrementGap="0"/&gt;
        &lt;fieldType name="int" class="solr.TrieIntField" precisionStep="0" positionIncrementGap="0"/&gt;
        &lt;fieldType name="text_general" class="solr.TextField" positionIncrementGap="100"&gt;
          &lt;analyzer&gt;
            &lt;tokenizer class="com.chenlb.mmseg4j.solr.MMSegTokenizerFactory" mode="complex" dicPath="dic"/&gt;
          &lt;/analyzer&gt;
        &lt;/fieldType&gt;
      &lt;/types&gt;

     &lt;fields&gt;   
      &lt;!-- general --&gt;
      &lt;field name="id"  type="string"   indexed="true"  stored="true"  multiValued="false" required="true"/&gt;
      &lt;field name="title" type="text_general"   indexed="true"  stored="true"  multiValued="false" /&gt; 
      &lt;field name="keyword" type="text_general"   indexed="true"  stored="true"  multiValued="true" /&gt; 
      &lt;field name="thumburl"  type="string"   indexed="false"  stored="true"  multiValued="false" /&gt;
      &lt;field name="playurl" type="string"   indexed="false"  stored="true"  multiValued="false" /&gt;
      &lt;field name="tag_1" type="text_general"   indexed="true"  stored="true"  multiValued="false" /&gt; 
      &lt;field name="tag_2" type="text_general"   indexed="true"  stored="true"  multiValued="false" /&gt; 
      &lt;field name="tag_3" type="text_general"   indexed="true"  stored="true"  multiValued="false" /&gt;
      &lt;field name="_version_" type="long"     indexed="true"  stored="true"/&gt;

      &lt;dynamicField name="*_i" type="int" indexed="true" stored="true"/&gt;
      &lt;dynamicField name="*_s" type="string" indexed="false" stored="true"/&gt;
      &lt;dynamicField name="*_t" type="text_general" indexed="true" stored="true"/&gt;

      &lt;copyField source="title" dest="keyword"/&gt;
      &lt;copyField source="tag_1" dest="keyword"/&gt;
      &lt;copyField source="tag_2" dest="keyword"/&gt;
      &lt;copyField source="tag_3" dest="keyword"/&gt;

     &lt;/fields&gt;

     &lt;!-- field to use to determine and enforce document uniqueness. --&gt;
     &lt;uniqueKey&gt;id&lt;/uniqueKey&gt;

     &lt;!-- field for the QueryParser to use when an explicit fieldname is absent --&gt;
     &lt;defaultSearchField&gt;keyword&lt;/defaultSearchField&gt;

     &lt;!-- SolrQueryParser configuration: defaultOperator="AND|OR" --&gt;
     &lt;solrQueryParser defaultOperator="OR"/&gt;
    &lt;/schema&gt;
    `</pre>

    注意这段代码，定义了text_general类型的字段使用了mmseg4j的分词器，自定义词典放在了dic目录下。

    <pre>`    &lt;fieldType name="text_general" class="solr.TextField" positionIncrementGap="100"&gt;
          &lt;analyzer&gt;
            &lt;tokenizer class="com.chenlb.mmseg4j.solr.MMSegTokenizerFactory" mode="complex" dicPath="dic"/&gt;
          &lt;/analyzer&gt;
        &lt;/fieldType&gt;
    `</pre>

*   创建一个新的core，这里我起名为videos
    ![](http://ww3.sinaimg.cn/mw690/3d6ce2f1jw1f2dyf0zbonj20ao0auwfe.jpg)
*   重启solr，测试分词器：
    尝试分析字符串`求助，哪位大神知道移动端自频道这个剧集是怎么添加的？`
    ![](http://ww3.sinaimg.cn/mw690/3d6ce2f1jw1f2dyf1f077j21790fw45a.jpg)
*   接下来，在D:\opt\solr\videos\dic中增加一个文件words-my.dic（分词器会识别words开头.dic结尾的文件作为自定义字典），文件内容如下:

    <pre>`自频道
    大神
    移动端
    剧集
    `</pre>

*   重启solr，再次测试分词效果：
    ![](http://ww4.sinaimg.cn/mw690/3d6ce2f1jw1f2dyf1toimj212x0fbjww.jpg)</p>
*   附上程序建立索引数据的代码：

    <p>需要用到的工具：

    <pre>`&lt;dependency&gt;
       &lt;groupId&gt;org.apache.solr&lt;/groupId&gt;
       &lt;artifactId&gt;solr-solrj&lt;/artifactId&gt;
       &lt;version&gt;4.7.2&lt;/version&gt;
    &lt;/dependency&gt;
    `</pre>

    Java代码：

    <pre>`public class InsertVideoData {

        public static void main(String[] args) throws IOException, SolrServerException {
            test01();
        }

        private static void test01() throws IOException, SolrServerException {

            String fileName = "/tmp/videos.dat";

            List&lt;String&gt; lines = FileUtils.readLines(new File(fileName));

            HttpSolrServer server = new HttpSolrServer("http://localhost:8080/solr/videos");
            for (String line : lines) {
                JSONObject videoJSON = new JSONObject(line);

                int id = videoJSON.getInt("id");
                String title = videoJSON.getString("title");
                String thumburl = videoJSON.getString("thumburl");
                String playurl = videoJSON.getString("playurl");
                String tag1 = videoJSON.getString("tag1");
                String tag2 = videoJSON.getString("tag2");
                String tag3 = videoJSON.getString("tag3");

                SolrInputDocument doc = new SolrInputDocument();
                doc.addField("id", id);
                doc.addField("title", title);
                doc.addField("thumburl", thumburl);
                doc.addField("playurl", playurl);
                doc.addField("tag_1", tag1);
                doc.addField("tag_2", tag2);
                doc.addField("tag_3", tag3);
                server.add(doc);
            }
            server.commit();
            System.out.println("OK!");
            System.exit(0);
        }

    }
    
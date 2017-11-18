---
title: Lucene 索引基础知识学习(Lucene3.6)
tags:
  - Java
  - Lucene
  - 索引
id: 200
categories:
  - 笔记
date: 2014-12-03 23:03:46
---

学习要点：

> *   执行基本的索引操作
> *   对文档和域进行加权操作
> *   索引日期和数字
> *   优化索引
> *   并发线程及安全机制

<!--more-->

## 索引的基本操作

### 对索引进行“增删改查”

<pre class="lang:java decode:true">public class Pro2_1 {

    protected static final String direct_path = "./indexDir/cp2/pro2_1";
    protected static Version version = Version.LUCENE_36;

    protected String[] ids = {"1", "2"};
    protected String[] unindexed = {"Netherlands", "Italy"};
    protected String[] unstored = {"Amsterdam has lots of bridges", "Venice has lots of canals"};
    protected String[] text = {"Amsterdam", "Venice"};
    protected Directory directory;
    protected Analyzer analyzer;

    /**
     * 完成一些初始化的操作
     * @throws IOException
     */
    public void setUp() throws IOException {
        File indexFile = new File(direct_path);
        if (!indexFile.exists()) {
            indexFile.mkdirs();
        }
        //使用RAMDreictory对象来存放索引，索引存放在内存中
        directory = new RAMDirectory();    
        //构造analyzer
        analyzer = new WhitespaceAnalyzer(version);
    }

    /**
     * 获取IndexWriter
     * @return
     * @throws CorruptIndexException
     * @throws LockObtainFailedException
     * @throws IOException
     */
    private IndexWriter getWriter() throws CorruptIndexException, LockObtainFailedException, IOException {
        return new IndexWriter(directory, new IndexWriterConfig(version, analyzer));
    }

    /**
     * 获取IndexReader
     * @return
     * @throws CorruptIndexException
     * @throws IOException
     */
    public IndexReader getReader() throws CorruptIndexException, IOException {
        return IndexReader.open(directory);
    }

    /**
     * 向索引添加文档并查询
     * @throws CorruptIndexException
     * @throws IOException
     */
    @Test
    public void test001() throws CorruptIndexException, IOException {
        setUp();
        writeIndex();
        search("contents", "lots");
    }

    /**
     * 创建索引并保存新的文档
     * @throws CorruptIndexException
     * @throws LockObtainFailedException
     * @throws IOException
     */
    public void writeIndex() throws CorruptIndexException, LockObtainFailedException, IOException {
        IndexWriter writer = getWriter();
        for (int i = 0; i &lt; ids.length; i++) {
            Document doc = new Document();
            doc.add(new Field("id", ids[i], 
                    Store.YES, 
                    Index.NOT_ANALYZED));
            doc.add(new Field("country", unindexed[i], 
                    Store.YES, 
                    Index.NO));
            doc.add(new Field("contents", unstored[i], 
                    Store.NO, 
                    Index.ANALYZED));
            doc.add(new Field("city", text[i], 
                    Store.YES, 
                    Index.ANALYZED));
            writer.addDocument(doc);
        }
        writer.close();
    }

    /**
     * 通过自定字段查询
     * @param fieldName
     * @param searchString
     * @throws CorruptIndexException
     * @throws IOException
     */
    public void search(String fieldName, String searchString) throws CorruptIndexException, IOException {
        IndexSearcher searcher = new IndexSearcher(getReader());
        Term t= new Term(fieldName, searchString);
        Query query = new TermQuery(t);
        TopDocs result  = searcher.search(query, 10);    //返回最多为10条记录
        ScoreDoc[] hits = result.scoreDocs;
        if (hits.length &gt; 0) {    
            System.out.println("找到:" + hits.length + " 个结果!");    
            for (int i = 0; i &lt; hits.length; i++) {
                ScoreDoc sdoc = hits[i];
                System.out.println(sdoc);
            }
        } else {
            System.out.println("找到:" + hits.length + " 个结果!");    
        }
        searcher.close();
    }

    /**
     * 删除索引中的文档
     * @throws CorruptIndexException
     * @throws LockObtainFailedException
     * @throws IOException
     */
    @Test
    public void test002() throws CorruptIndexException, LockObtainFailedException, IOException {
        setUp();
        writeIndex();
        deleteIndex("id", "2");
        search("contents", "lots");
    }

    /**
     * 删除指定的索引
     * @param fieldName
     * @param searchString
     * @throws CorruptIndexException
     * @throws LockObtainFailedException
     * @throws IOException
     */
    private void deleteIndex(String fieldName, String searchString) throws CorruptIndexException, LockObtainFailedException, IOException {
        IndexWriter writer = getWriter();
        Term term = new Term(fieldName, searchString);
        writer.deleteDocuments(term);
        search("contents", "lots");
        writer.commit();
        search("contents", "lots");
        writer.close();
    }

    /**
     * 更新索引中的文档
     * @throws CorruptIndexException
     * @throws LockObtainFailedException
     * @throws IOException
     */
    @Test
    public void test003() throws CorruptIndexException, LockObtainFailedException, IOException {
        setUp();
        writeIndex();
        search("contents", "museums");
        update("id", "1");
        search("contents", "museums");
    }

    /**
     * 更新索引信息
     * @param fieldName
     * @param searchString
     * @throws CorruptIndexException
     * @throws LockObtainFailedException
     * @throws IOException
     */
    private void update(String fieldName, String searchString) throws CorruptIndexException, LockObtainFailedException, IOException {
        Document doc = new Document();
        doc.add(new Field("id", searchString, 
                Store.YES, 
                Index.NOT_ANALYZED));
        doc.add(new Field("country", "Netherlands", 
                Store.YES, 
                Index.NO));
        doc.add(new Field("contents", "Den Haag has a lot of museums", 
                Store.NO, 
                Index.ANALYZED));
        doc.add(new Field("city", "Den Haag", 
                Store.YES, 
                Index.ANALYZED));

        Term term = new Term(fieldName, searchString);

        IndexWriter writer = getWriter();
        //更新信息过程是先删除老的文档，在保存新的文档
        writer.updateDocument(term, doc);
        writer.close();
    }
}</pre>

- 对索引的修改需要通过调用writer的`commit()`或是`close()`方法向索引提交更改。
- 删除操作不会马上释放文档的存储，Lucene只是将该文档标记为“删除”。

### 一些域的选项

- 域索引选项
- **Index.ANALYZED**:使用分析器将域值分解成独立的语汇单元流，并使每个语汇单元能被搜索。该选项适用于普通文本域（如正文、标题、摘要等）。
- **Index.NOT_ANALYZED**:对域进行索引，但不对String值进行分析。该操作实际上将域值作为单一语汇单元并使之能被搜索。该选项适用于索引那些不被分解的域值，如URL、文件路径、日期、人名、社保号码和电话号码等。该选项尤其适用于“精准匹配”搜索。
- **Index.ANALYZED_NO_NORMES**:这是Index.ANALYZED选项的一个变体，它不会再索引中存储norms信息。norms记录了索引中的index-time boost信息，但是当你进行搜索时可能会比较耗费内存。
- **Index.NOT_ANALYZED_NO_NORMS**:与Index.NOT_ANALYZED选项类似，但也是不存储norms。该选项常用于在搜索期间节省索引空间和减少内存消耗，因为single-token域并不需要norms信息，除非它们已被进行加权操作。
- **Index.NO**:使对应的域值不被搜索。
- 域存储选项
- **Store.YES**:指定存储域值。该情况下，原始的字符串值会全部被保存在索引中，并可以由IndexReader类恢复。该选项对于需要展示搜索结果的一些域很有用（如URL、标题或数据库主键）。如果索引的大小在搜索程序考虑之列的话，不要存储太大的域值，因为存储这些域值会消耗掉索引的存储空间。
- **Store.NO**:指定不存储域值。该选项通常跟Index.ANALYZED选项共同用来索引打的文本域值，通常这些域值不用回复为初始格式，如Web页面的正文，或其他类型的文本文档。

### 选项组合

<table>
<thead>
<tr>
  <th>索引选项</th>
  <th>存储选项</th>
  <th>项向量</th>
  <th>使用范例</th>
</tr>
</thead>
<tbody>
<tr>
  <td>NOT_ANALYZED_NO_NORMS</td>
  <td>YES</td>
  <td>NO</td>
  <td>标识符（文件名，主键），电话号码和社会安全号码、URL、姓名、日期、用于排序的文本域</td>
</tr>
<tr>
  <td>ANALYZED</td>
  <td>YES</td>
  <td>WITH_POSITIONS_OFFSETS</td>
  <td>文档标题、摘要</td>
</tr>
<tr>
  <td>ANALYZED</td>
  <td>NO</td>
  <td>WITH_POSITIONS_OFFSETS</td>
  <td>文档正文</td>
</tr>
<tr>
  <td>NO</td>
  <td>YES</td>
  <td>NO</td>
  <td>文档类型、数据库主键（如果没有用于搜索）</td>
</tr>
<tr>
  <td>NOT_ANALYZED</td>
  <td>NO</td>
  <td>NO</td>
  <td>隐藏的关键词</td>
</tr>
</tbody>
</table>

### 其它的Directory类

- **SimpleFSDirectory**:简单的Directory子类，使用java.io._API将文件存储入文件系统。不能很好支持多线程操作。
- **NIOFSDirectory**:使用java.nio._API将文件保存至文件喜用。能很好支持除Microsoft Windows之外的多线程操作，原因是Sun的JRE在Windows平台上长期存在问题。
- **MMapDirectory**:使用内存映射I/O进行文件访问。对于64位JRE来说是一个很好的选择，对于32位JRE并且索引尺寸相对较小时也可以使用该类。
- **RAMDirectory**:将所有文件都存入RAM
- **FileSwitchDirectory**:使用两个文件目录，根据文件扩招名在两个目录之间切换使用。

## 索引过程中的加权和多值域

加权操作可以在索引期间完成页可以在搜索期间完成。这里讨论在索引期间完成的加权操作。
默认情况下，所有文档都没有加权值，或者说它们都具有同样的加权因子1.0。

<pre class="lang:java decode:true">    private void addBoost() throws CorruptIndexException, LockObtainFailedException, IOException, ParseException {
        IndexWriter writer = getWriter();
        for (String[] info : email_datas) {
            Document doc = new Document();
            doc.add(new Field("senderName", info[0], 
                    Field.Store.YES, Field.Index.NOT_ANALYZED));
            doc.add(new Field("senderEmail", info[1],
                    Field.Store.YES, Field.Index.NOT_ANALYZED));
            doc.add(new Field("hostName", info[2],
                    Field.Store.YES, Field.Index.ANALYZED));
            Field body = new Field("body", info[3],
                    Field.Store.NO, Field.Index.ANALYZED);
            body.setBoost(1.2F);
            doc.add(body);                                    //给body域加权
            String[] phones = info[4].split(",");
            for (String phone : phones) {                    //多值域
                doc.add(new Field("phone", phone.trim(),
                        Field.Store.YES, Field.Index.NOT_ANALYZED));
            }

            doc.add(new NumericField("timestamp").setLongValue(formatDate(info[5]).getTime()));        //将日期按照Long型处理

            if (info[2].equalsIgnoreCase("geewaza")) {
                doc.setBoost(1.5F);                            //给这个文档加权
            } else if(info[2].equalsIgnoreCase("163")) {
                doc.setBoost(0.1F);                            //默认权值是1.0， 这里相当于降权
            }
            writer.addDocument(doc);
        }
        writer.close();
    }</pre>

- 改变索引中文档或是域的加权需要先删除文档再重新加入索引。

## 优化索引

### 索引段

Lucene索引包含一个或多个段，每个段都是一个独立的索引，它包含整个文档索引的一个子集。每当writer刷新缓冲区增加的文档，以及挂起目录删除操作时，索引文件都会建立一个新段。段的数量过多会影响搜索的效率，所以合并段操作就是优化索引的一个方式。
每个段都包含多个文件，文件格式为`_X.&amp;lt;ext&amp;gt;`，这里X代表段名称，`&amp;lt;ext&amp;gt;`为扩展名，用来标识该文件对应索引的某个部分。如果使用混合文件格式，那么上述索引文件都会被压缩成一个单一的文件：`_X.cfs`。
![](http://ww2.sinaimg.cn/mw690/3d6ce2f1gw1emwujedvbtj20d50a9jsp.jpg)
还有一个特殊文件，叫段文件，用段`_&amp;lt;N&amp;gt;`标识，该文件指向所有激活的段。
优化索引就是将索引的多个段合并成一个或者少量的段。优化只能提高搜索速度，不能提高索引速度。
使用`optimize()`进行优化。
![](http://ww2.sinaimg.cn/mw690/3d6ce2f1gw1emwurp0yyqj20gc06i3zb.jpg)
优化会消耗大量的CPU和I/O资源。还会临时占用磁盘空间。

## 并发、线程安全及锁机制

这里讨论索引的并发和线程安全的主题是：**索引文件的并发访问**、**IndexReader和IndexWriter的线程安全性**以及**Lucene用于实现前面两项的锁机制**。

### 线程安全和多虚拟机安全

- 任意熟练的只读属性的IndexReader类都可以同时打开一个索引。无论Reader是否属于同一个JVM，以及是否属于同一台计算机都无关紧要。Reader是线程安全的，所以可以再同一个JVM中多个线程持有同一个Reader实例。
- 对于一个索引来说，一次只能打开一个Writer。Lucene采用文件锁来提供保障。一旦建立起IndexWriter对象，系统立即会分配一个锁给它。该锁只有当IndexWriter对象被关闭时才会释放。当你使用IndexReader对象改变索引，这时IndexReader对象会作为Writer使用：它必须在修改上述内容之前成功获取Write锁，并在被关闭时释放该锁。
- IndexReader对象甚至可以再IndexWriter对象正在修改索引时打开。每个IndexReader对象将向索引展示自己被打开的时间点。该对象只有在IndexWriter对象提交修改或自己被重新打开后才能获知索引的修改情况。所以一个更好的选择是，在已有IndexReader对象被打开的情况下，打开新IndexReader时采用参数create=true:这样，新的IndexReader会持续检查索引的情况。
- 任意多个线程都可以共享同一个IndexReader类或IndexWriter类。这些类不仅是线程安全的，而且是线程友好的，即是说它们能够很好地扩展到新增线程。

### 索引锁机制

- **单一Writer**：Lucene采用了基于文件的锁，如果锁文件（默认为write.lock）存在于你的索引所在目录内，writer会马上打开该索引。此时若企图对同一索引创建其他writer的话，将产生一个LockObtainFailedException异常。
- IndexWriter类的`isLocked(Directory)`方法——该方法会返回参数目录所指定的索引是否已被锁住。
- IndexWriter类的`unlock(Directory)`方法——该方法使你能够在任意时刻对任意的Lucene索引进行解锁，贸然使用它是很危险的。
Writer锁的测试程序：

<pre class="lang:java decode:true">    private void testLock() throws CorruptIndexException, LockObtainFailedException, IOException {
        IndexWriter writer1 = getWriter();
        IndexWriter writer2 = null;

        try {
            writer2 = getWriter();
            System.out.println("We should never reach this point.");
        } catch (LockObtainFailedException e) {
            e.printStackTrace();
        } finally {
            writer1.close();
            if (writer2 != null) {
                writer2.close();
            }
        }
    }</pre>

异常：

<pre class="lang:java decode:true ">org.apache.lucene.store.LockObtainFailedException: Lock obtain timed out: NativeFSLock@D:\opt\lucene\indexDir\cp2\pro2_4\write.lock
    at org.apache.lucene.store.Lock.obtain(Lock.java:84)
    at org.apache.lucene.index.IndexWriter.&lt;init&gt;(IndexWriter.java:1098)
    at com.geewaza.study.luceneinaction.cp2.Pro2_4.getWriter(Pro2_4.java:56)
    at com.geewaza.study.luceneinaction.cp2.Pro2_4.testLock(Pro2_4.java:147)
    at com.geewaza.study.luceneinaction.cp2.Pro2_4.test003(Pro2_4.java:139)
    at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
    at sun.reflect.NativeMethodAccessorImpl.invoke(Unknown Source)
    at sun.reflect.DelegatingMethodAccessorImpl.invoke(Unknown Source)
    at java.lang.reflect.Method.invoke(Unknown Source)
    ......</pre>

&nbsp;
---
title: Redis数据结构
tags:
  - Redis
id: 216
categories:
  - 笔记
date: 2015-01-13 19:41:29
---

![](http://ww1.sinaimg.cn/mw690/3d6ce2f1jw1eo6xmvijf7j206803f0sq.jpg)
Redis是一个开源的使用ANSI C语言编写、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API。从2010年3月15日起，Redis的开发工作由VMware主持。

Redis提供五种数Value据类型：

> [string](#string)
>    [hash](#hash)
>    [list](#list)
>    [set](#set)
>    [zset](#zset)

Redis对Key的基本操作：

> [key](#key)

<!--more-->

## string

[原文资料](http://www.cnblogs.com/stephen-liu74/archive/2012/03/14/2349815.html)

### 概述

字符串类型是Redis中最为基础的数据存储类型，它在Redis中是二进制安全的，这便意味着该类型可以接受任何格式的数据，如JPEG图像数据或Json对象描述信息等。在Redis中字符串类型的Value最多可以容纳的数据长度是512M。

### 相关命令列表

<table>
<thead>
<tr>
  <th align="left">命令原型</th>
  <th align="left">时间复杂度</th>
  <th align="left">命令描述</th>
  <th align="left">返回值</th>
</tr>
</thead>
<tbody>
<tr>
  <td align="left">**APPEND** key value</td>
  <td align="left">O(1)</td>
  <td align="left">如果该Key已经存在，APPEND命令将参数Value的数据追加到已存在Value的末尾。如果该Key不存在，APPEND命令将会创建一个新的Key/Value。</td>
  <td align="left">追加后Value的长度。</td>
</tr>
<tr>
  <td align="left">**DECR** key</td>
  <td align="left">O(1)</td>
  <td align="left">将指定Key的Value原子性的递减1。如果该Key不存在，其初始值为0，在decr之后其值为-1。如果Value的值不能转换为整型值，如Hello，该操作将执行失败并返回相应的错误信息。注意：该操作的取值范围是64位有符号整型。</td>
  <td align="left">递减后的Value值。</td>
</tr>
<tr>
  <td align="left">**INCR** key</td>
  <td align="left">O(1)</td>
  <td align="left">将指定Key的Value原子性的递增1。如果该Key不存在，其初始值为0，在incr之后其值为1。如果Value的值不能转换为整型值，如Hello，该操作将执行失败并返回相应的错误信息。注意：该操作的取值范围是64位有符号整型。</td>
  <td align="left">递增后的Value值。</td>
</tr>
<tr>
  <td align="left">**DECRBY** key decrement  </td>
  <td align="left">O(1)  </td>
  <td align="left">将指定Key的Value原子性的减少decrement。如果该Key不存在，其初始值为0，在decrby之后其值为-decrement。如果Value的值不能转换为整型值，如Hello，该操作将执行失败并返回相应的错误信息。注意：该操作的取值范围是64位有符号整型。</td>
  <td align="left">减少后的Value值。</td>
</tr>
<tr>
  <td align="left">**INCRBY** key increment  </td>
  <td align="left">O(1)</td>
  <td align="left">将指定Key的Value原子性的增加increment。如果该Key不存在，其初始值为0，在incrby之后其值为increment。如果Value的值不能转换为整型值，如Hello，该操作将执行失败并返回相应的错误信息。注意：该操作的取值范围是64位有符号整型。</td>
  <td align="left">增加后的Value值。</td>
</tr>
<tr>
  <td align="left">**GET** key</td>
  <td align="left">O(1)</td>
  <td align="left">获取指定Key的Value。如果与该Key关联的Value不是string类型，Redis将返回错误信息，因为GET命令只能用于获取string Value。</td>
  <td align="left">与该Key相关的Value，如果该Key不存在，返回nil。</td>
</tr>
<tr>
  <td align="left">**SET** key value</td>
  <td align="left">O(1)</td>
  <td align="left">设定该Key持有指定的字符串Value，如果该Key已经存在，则覆盖其原有值。</td>
  <td align="left">总是返回"OK"。</td>
</tr>
<tr>
  <td align="left">**GETSET** key value</td>
  <td align="left">O(1)</td>
  <td align="left">原子性的设置该Key为指定的Value，同时返回该Key的原有值。和GET命令一样，该命令也只能处理string Value，否则Redis将给出相关的错误信息。</td>
  <td align="left">返回该Key的原有值，如果该Key之前并不存在，则返回nil。</td>
</tr>
<tr>
  <td align="left">**STRLEN** key</td>
  <td align="left">O(1)</td>
  <td align="left">返回指定Key的字符值长度，如果Value不是string类型，Redis将执行失败并给出相关的错误信息。</td>
  <td align="left">返回指定Key的Value字符长度，如果该Key不存在，返回0。</td>
</tr>
<tr>
  <td align="left">**SETEX** key seconds value</td>
  <td align="left">O(1)</td>
  <td align="left">原子性完成两个操作，一是设置该Key的值为指定字符串，同时设置该Key在Redis服务器中的存活时间(秒数)。该命令主要应用于Redis被当做Cache服务器使用时。</td>
  <td align="left">** **</td>
</tr>
<tr>
  <td align="left">**SETNX** key value</td>
  <td align="left">O(1)</td>
  <td align="left">如果指定的Key不存在，则设定该Key持有指定字符串Value，此时其效果等价于SET命令。相反，如果该Key已经存在，该命令将不做任何操作并返回。</td>
  <td align="left">1表示设置成功，否则0。</td>
</tr>
<tr>
  <td align="left">**SETRANGE** key offset value</td>
  <td align="left">O(1)</td>
  <td align="left">替换指定Key的部分字符串值。从offset开始，替换的长度为该命令第三个参数value的字符串长度，其中如果offset的值大于该Key的原有值Value的字符串长度，Redis将会在Value的后面补齐(offset - strlen(value))数量的0x00，之后再追加新值。如果该键不存在，该命令会将其原值的长度假设为0，并在其后添补offset个0x00后再追加新值。鉴于字符串Value的最大长度为512M，因此offset的最大值为536870911。最后需要注意的是，如果该命令在执行时致使指定Key的原有值长度增加，这将会导致Redis重新分配足够的内存以容纳替换后的全部字符串，因此就会带来一定的性能折损。</td>
  <td align="left">修改后的字符串Value长度。</td>
</tr>
<tr>
  <td align="left">**GETRANGE** key start end</td>
  <td align="left">O(1)</td>
  <td align="left">如果截取的字符串长度很短，我们可以该命令的时间复杂度视为O(1)，否则就是O(N)，这里N表示截取的子字符串长度。该命令在截取子字符串时，将以闭区间的方式同时包含start(0表示第一个字符)和end所在的字符，如果end值超过Value的字符长度，该命令将只是截取从start开始之后所有的字符数据。</td>
  <td align="left">子字符串</td>
</tr>
<tr>
  <td align="left">**SETBIT** key offset value</td>
  <td align="left">O(1)</td>
  <td align="left">设置在指定Offset上BIT的值，该值只能为1或0，在设定后该命令返回该Offset上原有的BIT值。如果指定Key不存在，该命令将创建一个新值，并在指定的Offset上设定参数中的BIT值。如果Offset大于Value的字符长度，Redis将拉长Value值并在指定Offset上设置参数中的BIT值，中间添加的BIT值为0。最后需要说明的是Offset值必须大于0。</td>
  <td align="left">在指定Offset上的BIT原有值。</td>
</tr>
<tr>
  <td align="left">**GETBIT** key offset</td>
  <td align="left">O(1)</td>
  <td align="left">返回在指定Offset上BIT的值，0或1。如果Offset超过string value的长度，该命令将返回0，所以对于空字符串始终返回0。</td>
  <td align="left">在指定Offset上的BIT值。</td>
</tr>
<tr>
  <td align="left">**MGET** key [key ...]</td>
  <td align="left">O(N)</td>
  <td align="left">N表示获取Key的数量。返回所有指定Keys的Values，如果其中某个Key不存在，或者其值不为string类型，该Key的Value将返回nil。</td>
  <td align="left">返回一组指定Keys的Values的列表。</td>
</tr>
<tr>
  <td align="left">**MSET** key value [key value ...]</td>
  <td align="left">O(N)</td>
  <td align="left">N表示指定Key的数量。该命令原子性的完成参数中所有key/value的设置操作，其具体行为可以看成是多次迭代执行SET命令。</td>
  <td align="left">该命令不会失败，始终返回OK。</td>
</tr>
<tr>
  <td align="left">**MSETNX** key value [key value ...]</td>
  <td align="left">O(N)</td>
  <td align="left">N表示指定Key的数量。该命令原子性的完成参数中所有key/value的设置操作，其具体行为可以看成是多次迭代执行SETNX命令。**然而这里需要明确说明的是，如果在这一批Keys中有任意一个Key已经存在了，那么该操作将全部回滚，即所有的修改都不会生效。**</td>
  <td align="left">1表示所有Keys都设置成功，0则表示没有任何Key被修改。</td>
</tr>
</tbody>
</table>

### 命令示例

#### 1、SET、GET、APPEND、STRLEN

<pre class="lang:sh decode:true">/&gt; redis-cli   #执行Redis客户端工具。
redis 127.0.0.1:6379&gt; exists mykey                   #判断该键是否存在，存在返回1，否则返回0。
(integer) 0
redis 127.0.0.1:6379&gt; append mykey "hello"      #该键并不存在，因此append命令返回当前Value的长度。
(integer) 5
redis 127.0.0.1:6379&gt; append mykey " world"    #该键已经存在，因此返回追加后Value的长度。
(integer) 11
redis 127.0.0.1:6379&gt; get mykey                      #通过get命令获取该键，以判断append的结果。
"hello world"
redis 127.0.0.1:6379&gt; set mykey "this is a test" #通过set命令为键设置新值，并覆盖原有值。
OK
redis 127.0.0.1:6379&gt; get mykey
"this is a test"
redis 127.0.0.1:6379&gt; strlen mykey                  #获取指定Key的字符长度，等效于C库中strlen函数。
(integer) 14</pre>

#### 2、INCR、DECR、INCRBY、DECRBY

<pre class="lang:sh decode:true ">redis 127.0.0.1:6379&gt; set mykey 20     #设置Key的值为20
OK
redis 127.0.0.1:6379&gt; incr mykey         #该Key的值递增1
(integer) 21
redis 127.0.0.1:6379&gt; decr mykey        #该Key的值递减1
(integer) 20
redis 127.0.0.1:6379&gt; del mykey          #删除已有键。
(integer) 1
redis 127.0.0.1:6379&gt; decr mykey        #对空值执行递减操作，其原值被设定为0，递减后的值为-1
(integer) -1
redis 127.0.0.1:6379&gt; del mykey   
(integer) 1
redis 127.0.0.1:6379&gt; incr mykey        #对空值执行递增操作，其原值被设定为0，递增后的值为1
(integer) 1
redis 127.0.0.1:6379&gt; set mykey hello #将该键的Value设置为不能转换为整型的普通字符串。
OK
redis 127.0.0.1:6379&gt; incr mykey        #在该键上再次执行递增操作时，Redis将报告错误信息。
(error) ERR value is not an integer or out of range
redis 127.0.0.1:6379&gt; set mykey 10
OK
redis 127.0.0.1:6379&gt; decrby mykey 5 
(integer) 5
redis 127.0.0.1:6379&gt; incrby mykey 10
(integer) 15</pre>

#### 3、GETSET

<pre class="lang:sh decode:true ">redis 127.0.0.1:6379&gt; incr mycounter      #将计数器的值原子性的递增1
(integer) 1
#在获取计数器原有值的同时，并将其设置为新值，这两个操作原子性的同时完成。
redis 127.0.0.1:6379&gt; getset mycounter 0  
"1"
redis 127.0.0.1:6379&gt; get mycounter       #查看设置后的结果。
"0"</pre>

#### 4、SETEX

<pre class="lang:sh decode:true ">redis 127.0.0.1:6379&gt; setex mykey 10 "hello"   #设置指定Key的过期时间为10秒。
OK    
#通过ttl命令查看一下指定Key的剩余存活时间(秒数)，0表示已经过期，-1表示永不过期。
redis 127.0.0.1:6379&gt; ttl mykey                       
(integer) 4
redis 127.0.0.1:6379&gt; get mykey                      #在该键的存活期内我们仍然可以获取到它的Value。
"hello"
redis 127.0.0.1:6379&gt; ttl mykey                        #该ttl命令的返回值显示，该Key已经过期。
(integer) 0
redis 127.0.0.1:6379&gt; get mykey                      #获取已过期的Key将返回nil。
(nil)</pre>

#### 5、SETNX

<pre class="lang:sh decode:true ">redis 127.0.0.1:6379&gt; del mykey                      #删除该键，以便于下面的测试验证。
(integer) 1
redis 127.0.0.1:6379&gt; setnx mykey "hello"        #该键并不存在，因此该命令执行成功。
(integer) 1
redis 127.0.0.1:6379&gt; setnx mykey "world"       #该键已经存在，因此本次设置没有产生任何效果。
(integer) 0
redis 127.0.0.1:6379&gt; get mykey                      #从结果可以看出，返回的值仍为第一次设置的值。
"hello"</pre>

#### 6、SETRANGE、GETRANGE

<pre class="lang:sh decode:true ">redis 127.0.0.1:6379&gt; set mykey "hello world"       #设定初始值。
OK
redis 127.0.0.1:6379&gt; setrange mykey 6 dd          #从第六个字节开始替换2个字节(dd只有2个字节)
(integer) 11
redis 127.0.0.1:6379&gt; get mykey                         #查看替换后的值。
"hello ddrld"
redis 127.0.0.1:6379&gt; setrange mykey 20 dd        #offset已经超过该Key原有值的长度了，该命令将会在末尾补0。
(integer) 22
redis 127.0.0.1:6379&gt; get mykey                           #查看补0后替换的结果。
"hello ddrld\x00\x00\x00\x00\x00\x00\x00\x00\x00dd"
redis 127.0.0.1:6379&gt; del mykey                         #删除该Key。
(integer) 1
redis 127.0.0.1:6379&gt; setrange mykey 2 dd         #替换空值。
(integer) 4
redis 127.0.0.1:6379&gt; get mykey                        #查看替换空值后的结果。
"\x00\x00dd"   
redis 127.0.0.1:6379&gt; set mykey "0123456789"   #设置新值。
OK
redis 127.0.0.1:6379&gt; getrange mykey 1 2      #截取该键的Value，从第一个字节开始，到第二个字节结束。
"12"
redis 127.0.0.1:6379&gt; getrange mykey 1 20   #20已经超过Value的总长度，因此将截取第一个字节后面的所有字节。
"123456789"</pre>

#### 7、SETBIT、GETBIT

<pre class="lang:default decode:true ">redis 127.0.0.1:6379&gt; del mykey
(integer) 1
redis 127.0.0.1:6379&gt; setbit mykey 7 1       #设置从0开始计算的第七位BIT值为1，返回原有BIT值0
(integer) 0
redis 127.0.0.1:6379&gt; get mykey                #获取设置的结果，二进制的0000 0001的十六进制值为0x01
"\x01"
redis 127.0.0.1:6379&gt; setbit mykey 6 1       #设置从0开始计算的第六位BIT值为1，返回原有BIT值0
(integer) 0
redis 127.0.0.1:6379&gt; get mykey                #获取设置的结果，二进制的0000 0011的十六进制值为0x03
"\x03"
redis 127.0.0.1:6379&gt; getbit mykey 6          #返回了指定Offset的BIT值。
(integer) 1
redis 127.0.0.1:6379&gt; getbit mykey 10        #Offset已经超出了value的长度，因此返回0。
(integer) 0</pre>

#### 8、MSET、MGET、MSETNX

<pre class="lang:sh decode:true ">redis 127.0.0.1:6379&gt; mset key1 "hello" key2 "world"   #批量设置了key1和key2两个键。
OK
redis 127.0.0.1:6379&gt; mget key1 key2                        #批量获取了key1和key2两个键的值。
1) "hello"
2) "world"
#批量设置了key3和key4两个键，因为之前他们并不存在，所以该命令执行成功并返回1。
redis 127.0.0.1:6379&gt; msetnx key3 "stephen" key4 "liu" 
(integer) 1
redis 127.0.0.1:6379&gt; mget key3 key4                   
1) "stephen"
2) "liu"
#批量设置了key3和key5两个键，但是key3已经存在，所以该命令执行失败并返回0。
redis 127.0.0.1:6379&gt; msetnx key3 "hello" key5 "world" 
(integer) 0
#批量获取key3和key5，由于key5没有设置成功，所以返回nil。
redis 127.0.0.1:6379&gt; mget key3 key5                   
1) "stephen"
2) (nil)</pre>

## hash

[原文资料](http://www.cnblogs.com/stephen-liu74/archive/2012/03/19/2352932.html)

### 概述

我们可以将Redis中的Hashes类型看成具有String Key和String Value的map容器。所以该类型非常适合于存储值对象的信息。如Username、Password和Age等。如果Hash中包含很少的字段，那么该类型的数据也将仅占用很少的磁盘空间。每一个Hash可以存储4294967295个键值对。

### 相关命令列表

<table>
<thead>
<tr>
  <th align="left">命令原型</th>
  <th align="left">时间复杂度</th>
  <th align="left">命令描述</th>
  <th align="left">返回值</th>
</tr>
</thead>
<tbody>
<tr>
  <td align="left">**HSET** key field value</td>
  <td align="left">O(1)</td>
  <td align="left">为指定的Key设定Field/Value对，如果Key不存在，该命令将创建新Key以参数中的Field/Value对，如果参数中的Field在该Key中已经存在，则用新值覆盖其原有值。</td>
  <td align="left">1表示新的Field被设置了新值，0表示Field已经存在，用新值覆盖原有值。</td>
</tr>
<tr>
  <td align="left">**HGET** key field</td>
  <td align="left">O(1)</td>
  <td align="left">返回指定Key中指定Field的关联值。</td>
  <td align="left">返回参数中Field的关联值，如果参数中的Key或Field不存，返回nil。</td>
</tr>
<tr>
  <td align="left">**HEXISTS** key field</td>
  <td align="left">O(1)</td>
  <td align="left">判断指定Key中的指定Field是否存在。</td>
  <td align="left">1表示存在，0表示参数中的Field或Key不存在。</td>
</tr>
<tr>
  <td align="left">**HLEN** key</td>
  <td align="left">O(1)</td>
  <td align="left">获取该Key所包含的Field的数量。</td>
  <td align="left">返回Key包含的Field数量，如果Key不存在，返回0。</td>
</tr>
<tr>
  <td align="left">**HDEL** key field [field ...]</td>
  <td align="left">O(N)</td>
  <td align="left">时间复杂度中的N表示参数中待删除的字段数量。从指定Key的Hashes Value中删除参数中指定的多个字段，如果不存在的字段将被忽略。如果Key不存在，则将其视为空Hashes，并返回0.</td>
  <td align="left">实际删除的Field数量。</td>
</tr>
<tr>
  <td align="left">**HSETNX** key field value</td>
  <td align="left">O(1)</td>
  <td align="left">只有当参数中的Key或Field不存在的情况下，为指定的Key设定Field/Value对，否则该命令不会进行任何操作。</td>
  <td align="left">1表示新的Field被设置了新值，0表示Key或Field已经存在，该命令没有进行任何操作。</td>
</tr>
<tr>
  <td align="left">**HINCRBY** key field increment</td>
  <td align="left">O(1)</td>
  <td align="left">增加指定Key中指定Field关联的Value的值。如果Key或Field不存在，该命令将会创建一个新Key或新Field，并将其关联的Value初始化为0，之后再指定数字增加的操作。该命令支持的数字是64位有符号整型，即increment可以负数。</td>
  <td align="left">返回运算后的值。</td>
</tr>
<tr>
  <td align="left">**HGETALL** key</td>
  <td align="left">O(N)</td>
  <td align="left">时间复杂度中的N表示Key包含的Field数量。获取该键包含的所有Field/Value。其返回格式为一个Field、一个Value，并以此类推。</td>
  <td align="left">Field/Value的列表。</td>
</tr>
<tr>
  <td align="left">**HKEYS** key</td>
  <td align="left">O(N)</td>
  <td align="left">时间复杂度中的N表示Key包含的Field数量。返回指定Key的所有Fields名。</td>
  <td align="left">Field的列表。</td>
</tr>
<tr>
  <td align="left">**HVALS** key</td>
  <td align="left">O(N)</td>
  <td align="left">时间复杂度中的N表示Key包含的Field数量。返回指定Key的所有Values名。</td>
  <td align="left">Value的列表。</td>
</tr>
<tr>
  <td align="left">**HMGET** key field [field ...]</td>
  <td align="left">O(N)</td>
  <td align="left">时间复杂度中的N表示请求的Field数量。获取和参数中指定Fields关联的一组Values。如果请求的Field不存在，其值返回nil。如果Key不存在，该命令将其视为空Hash，因此返回一组nil。</td>
  <td align="left">返回和请求Fields关联的一组Values，其返回顺序等同于Fields的请求顺序。</td>
</tr>
<tr>
  <td align="left">**HMSET** key field value [field value ...]</td>
  <td align="left">O(N)</td>
  <td align="left">时间复杂度中的N表示被设置的Field数量。逐对依次设置参数中给出的Field/Value对。如果其中某个Field已经存在，则用新值覆盖原有值。如果Key不存在，则创建新Key，同时设定参数中的Field/Value。  </td>
  <td align="left">** **</td>
</tr>
</tbody>
</table>

### 命令示例

#### 1、HSET、HGET、HDEL、HEXISTS、HLEN、HSETNX

<pre class="lang:sh decode:true ">#在Shell命令行启动Redis客户端程序
/&gt; redis-cli
#给键值为myhash的键设置字段为field1，值为stephen。
redis 127.0.0.1:6379&gt; hset myhash field1 "stephen"
(integer) 1
#获取键值为myhash，字段为field1的值。
redis 127.0.0.1:6379&gt; hget myhash field1
"stephen"
#myhash键中不存在field2字段，因此返回nil。
redis 127.0.0.1:6379&gt; hget myhash field2
(nil)
#给myhash关联的Hashes值添加一个新的字段field2，其值为liu。
redis 127.0.0.1:6379&gt; hset myhash field2 "liu"
(integer) 1
#获取myhash键的字段数量。
redis 127.0.0.1:6379&gt; hlen myhash
(integer) 2
#判断myhash键中是否存在字段名为field1的字段，由于存在，返回值为1。
redis 127.0.0.1:6379&gt; hexists myhash field1
(integer) 1
#删除myhash键中字段名为field1的字段，删除成功返回1。
redis 127.0.0.1:6379&gt; hdel myhash field1
(integer) 1
#再次删除myhash键中字段名为field1的字段，由于上一条命令已经将其删除，因为没有删除，返回0。
redis 127.0.0.1:6379&gt; hdel myhash field1
(integer) 0
#判断myhash键中是否存在field1字段，由于上一条命令已经将其删除，因为返回0。
redis 127.0.0.1:6379&gt; hexists myhash field1
(integer) 0
#通过hsetnx命令给myhash添加新字段field1，其值为stephen，因为该字段已经被删除，所以该命令添加成功并返回1。
redis 127.0.0.1:6379&gt; hsetnx myhash field1 stephen
(integer) 1
#由于myhash的field1字段已经通过上一条命令添加成功，因为本条命令不做任何操作后返回0。
redis 127.0.0.1:6379&gt; hsetnx myhash field1 stephen
(integer) 0</pre>

#### 2、HINCRBY

<pre class="lang:sh decode:true ">#删除该键，便于后面示例的测试。
redis 127.0.0.1:6379&gt; del myhash
(integer) 1
#准备测试数据，该myhash的field字段设定值1。
redis 127.0.0.1:6379&gt; hset myhash field 5
(integer) 1
#给myhash的field字段的值加1，返回加后的结果。
redis 127.0.0.1:6379&gt; hincrby myhash field 1
(integer) 6
#给myhash的field字段的值加-1，返回加后的结果。
redis 127.0.0.1:6379&gt; hincrby myhash field -1
(integer) 5
#给myhash的field字段的值加-10，返回加后的结果。
redis 127.0.0.1:6379&gt; hincrby myhash field -10
(integer) -5</pre>

#### 3、HGETALL、HKEYS、HVALS、HMGET、HMSET

<pre class="lang:sh decode:true">#删除该键，便于后面示例测试。
redis 127.0.0.1:6379&gt; del myhash
(integer) 1
#为该键myhash，一次性设置多个字段，分别是field1 = "hello", field2 = "world"。
redis 127.0.0.1:6379&gt; hmset myhash field1 "hello" field2 "world"
OK
#获取myhash键的多个字段，其中field3并不存在，因为在返回结果中与该字段对应的值为nil。
redis 127.0.0.1:6379&gt; hmget myhash field1 field2 field3
1) "hello"
2) "world"
3) (nil)
#返回myhash键的所有字段及其值，从结果中可以看出，他们是逐对列出的。
redis 127.0.0.1:6379&gt; hgetall myhash
1) "field1"
2) "hello"
3) "field2"
4) "world"
#仅获取myhash键中所有字段的名字。
redis 127.0.0.1:6379&gt; hkeys myhash
1) "field1"
2) "field2"
#仅获取myhash键中所有字段的值。
redis 127.0.0.1:6379&gt; hvals myhash
1) "hello"
2) "world"</pre>

## list

[原文资料](http://www.cnblogs.com/stephen-liu74/archive/2012/03/16/2351859.html)

### 概述

在Redis中，List类型是按照插入顺序排序的字符串链表。和数据结构中的普通链表一样，我们可以在其头部(left)和尾部(right)添加新的元素。在插入时，如果该键并不存在，Redis将为该键创建一个新的链表。与此相反，如果链表中所有的元素均被移除，那么该键也将会被从数据库中删除。List中可以包含的最大元素数量是4294967295。
从元素插入和删除的效率视角来看，如果我们是在链表的两头插入或删除元素，这将会是非常高效的操作，即使链表中已经存储了百万条记录，该操作也可以在常量时间内完成。然而需要说明的是，如果元素插入或删除操作是作用于链表中间，那将会是非常低效的。相信对于有良好数据结构基础的开发者而言，这一点并不难理解。

### 相关命令列表

<table>
<thead>
<tr>
  <th align="left">命令原型</th>
  <th align="left">时间复杂度</th>
  <th align="left">命令描述</th>
  <th align="left">返回值</th>
</tr>
</thead>
<tbody>
<tr>
  <td align="left">**LPUSH** key value [value ...]</td>
  <td align="left">O(1)</td>
  <td align="left">在指定Key所关联的List Value的头部插入参数中给出的所有Values。如果该Key不存在，该命令将在插入之前创建一个与该Key关联的空链表，之后再将数据从链表的头部插入。如果该键的Value不是链表类型，该命令将返回相关的错误信息。</td>
  <td align="left">插入后链表中元素的数量。</td>
</tr>
<tr>
  <td align="left">**LPUSHX** key value</td>
  <td align="left">O(1)  </td>
  <td align="left">仅有当参数中指定的Key存在时，该命令才会在其所关联的List Value的头部插入参数中给出的Value，否则将不会有任何操作发生。</td>
  <td align="left">插入后链表中元素的数量。</td>
</tr>
<tr>
  <td align="left">**LRANGE** key start stop</td>
  <td align="left">O(S+N)</td>
  <td align="left">时间复杂度中的S为start参数表示的偏移量，N表示元素的数量。该命令的参数start和end都是0-based。即0表示链表头部(leftmost)的第一个元素。其中start的值也可以为负值，-1将表示链表中的最后一个元素，即尾部元素，-2表示倒数第二个并以此类推。该命令在获取元素时，start和end位置上的元素也会被取出。如果start的值大于链表中元素的数量，空链表将会被返回。如果end的值大于元素的数量，该命令则获取从start(包括start)开始，链表中剩余的所有元素。</td>
  <td align="left">返回指定范围内元素的列表。</td>
</tr>
<tr>
  <td align="left">**LPOP** key</td>
  <td align="left">O(1)</td>
  <td align="left">返回并弹出指定Key关联的链表中的第一个元素，即头部元素，。如果该Key不存，返回nil。</td>
  <td align="left">链表头部的元素。</td>
</tr>
<tr>
  <td align="left">**LLEN** key</td>
  <td align="left">O(1)</td>
  <td align="left">返回指定Key关联的链表中元素的数量，如果该Key不存在，则返回0。如果与该Key关联的Value的类型不是链表，则返回相关的错误信息。</td>
  <td align="left">链表中元素的数量。</td>
</tr>
<tr>
  <td align="left">**LREM** key count value</td>
  <td align="left">O(N)</td>
  <td align="left">时间复杂度中N表示链表中元素的数量。在指定Key关联的链表中，删除前count个值等于value的元素。如果count大于0，从头向尾遍历并删除，如果count小于0，则从尾向头遍历并删除。如果count等于0，则删除链表中所有等于value的元素。如果指定的Key不存在，则直接返回0。</td>
  <td align="left">返回被删除的元素数量。</td>
</tr>
<tr>
  <td align="left">**LSET** key index value</td>
  <td align="left">O(N)</td>
  <td align="left">时间复杂度中N表示链表中元素的数量。但是设定头部或尾部的元素时，其时间复杂度为O(1)。设定链表中指定位置的值为新值，其中0表示第一个元素，即头部元素，-1表示尾部元素。如果索引值Index超出了链表中元素的数量范围，该命令将返回相关的错误信息。</td>
  <td align="left">** **</td>
</tr>
<tr>
  <td align="left">**LINDEX** key index</td>
  <td align="left">O(N)</td>
  <td align="left">时间复杂度中N表示在找到该元素时需要遍历的元素数量。对于头部或尾部元素，其时间复杂度为O(1)。该命令将返回链表中指定位置(index)的元素，index是0-based，表示头部元素，如果index为-1，表示尾部元素。如果与该Key关联的不是链表，该命令将返回相关的错误信息。</td>
  <td align="left">返回请求的元素，如果index超出范围，则返回nil。</td>
</tr>
<tr>
  <td align="left">**LTRIM** key start stop</td>
  <td align="left">O(N)</td>
  <td align="left">N表示被删除的元素数量。该命令将仅保留指定范围内的元素，从而保证链接中的元素数量相对恒定。start和stop参数都是0-based，0表示头部元素。和其他命令一样，start和stop也可以为负值，-1表示尾部元素。如果start大于链表的尾部，或start大于stop，该命令不错报错，而是返回一个空的链表，与此同时该Key也将被删除。如果stop大于元素的数量，则保留从start开始剩余的所有元素。</td>
  <td align="left">** **</td>
</tr>
<tr>
  <td align="left">**LINSERT** key BEFORE\AFTER pivot value</td>
  <td align="left">O(N)</td>
  <td align="left">时间复杂度中N表示在找到该元素pivot之前需要遍历的元素数量。这样意味着如果pivot位于链表的头部或尾部时，该命令的时间复杂度为O(1)。该命令的功能是在pivot元素的前面或后面插入参数中的元素value。如果Key不存在，该命令将不执行任何操作。如果与Key关联的Value类型不是链表，相关的错误信息将被返回。</td>
  <td align="left">成功插入后链表中元素的数量，如果没有找到pivot，返回-1，如果key不存在，返回0。</td>
</tr>
<tr>
  <td align="left">**RPUSH** key value [value ...]</td>
  <td align="left">O(1)</td>
  <td align="left">在指定Key所关联的List Value的尾部插入参数中给出的所有Values。如果该Key不存在，该命令将在插入之前创建一个与该Key关联的空链表，之后再将数据从链表的尾部插入。如果该键的Value不是链表类型，该命令将返回相关的错误信息。</td>
  <td align="left">插入后链表中元素的数量。</td>
</tr>
<tr>
  <td align="left">**RPUSHX** key value</td>
  <td align="left">O(1)</td>
  <td align="left">仅有当参数中指定的Key存在时，该命令才会在其所关联的List Value的尾部插入参数中给出的Value，否则将不会有任何操作发生。</td>
  <td align="left">插入后链表中元素的数量。</td>
</tr>
<tr>
  <td align="left">**RPOP** key</td>
  <td align="left">O(1)</td>
  <td align="left">返回并弹出指定Key关联的链表中的最后一个元素，即尾部元素，。如果该Key不存，返回nil。</td>
  <td align="left">链表尾部的元素。</td>
</tr>
<tr>
  <td align="left">**RPOPLPUSH** source destination</td>
  <td align="left">O(1)</td>
  <td align="left">原子性的从与source键关联的链表尾部弹出一个元素，同时再将弹出的元素插入到与destination键关联的链表的头部。如果source键不存在，该命令将返回nil，同时不再做任何其它的操作了。如果source和destination是同一个键，则相当于原子性的将其关联链表中的尾部元素移到该链表的头部。</td>
  <td align="left">返回弹出和插入的元素。</td>
</tr>
</tbody>
</table>

### 命令示例

#### 1、LPUSH、LPUSHX、LRANGE

<pre class="lang:default decode:true ">/&gt; redis-cli    #在Shell提示符下启动redis客户端工具。
redis 127.0.0.1:6379&gt; del mykey
(integer) 1
#mykey键并不存在，该命令会创建该键及与其关联的List，之后在将参数中的values从左到右依次插入。
redis 127.0.0.1:6379&gt; lpush mykey a b c d
(integer) 4
#取从位置0开始到位置2结束的3个元素。
redis 127.0.0.1:6379&gt; lrange mykey 0 2
1) "d"
2) "c"
3) "b"
#取链表中的全部元素，其中0表示第一个元素，-1表示最后一个元素。
redis 127.0.0.1:6379&gt; lrange mykey 0 -1
1) "d"
2) "c"
3) "b"
4) "a"
#mykey2键此时并不存在，因此该命令将不会进行任何操作，其返回值为0。
redis 127.0.0.1:6379&gt; lpushx mykey2 e
(integer) 0
#可以看到mykey2没有关联任何List Value。
redis 127.0.0.1:6379&gt; lrange mykey2 0 -1
(empty list or set)
#mykey键此时已经存在，所以该命令插入成功，并返回链表中当前元素的数量。
redis 127.0.0.1:6379&gt; lpushx mykey e
(integer) 5
#获取该键的List Value的头部元素。
redis 127.0.0.1:6379&gt; lrange mykey 0 0
1) "e"</pre>

#### 2、LPOP、LLEN

<pre class="lang:sh decode:true ">redis 127.0.0.1:6379&gt; lpush mykey a b c d
(integer) 4
redis 127.0.0.1:6379&gt; lpop mykey
"d"
redis 127.0.0.1:6379&gt; lpop mykey
"c"
#在执行lpop命令两次后，链表头部的两个元素已经被弹出，此时链表中元素的数量是2
redis 127.0.0.1:6379&gt; llen mykey
(integer) 2</pre>

#### 3、LREM、LSET、LINDEX、LTRIM

<pre class="lang:sh decode:true ">#为后面的示例准备测试数据。
redis 127.0.0.1:6379&gt; lpush mykey a b c d a c
(integer) 6
#从头部(left)向尾部(right)变量链表，删除2个值等于a的元素，返回值为实际删除的数量。
redis 127.0.0.1:6379&gt; lrem mykey 2 a
(integer) 2
#看出删除后链表中的全部元素。
redis 127.0.0.1:6379&gt; lrange mykey 0 -1
1) "c"
2) "d"
3) "c"
4) "b"
#获取索引值为1(头部的第二个元素)的元素值。
redis 127.0.0.1:6379&gt; lindex mykey 1
"d"
#将索引值为1(头部的第二个元素)的元素值设置为新值e。
redis 127.0.0.1:6379&gt; lset mykey 1 e
OK
#查看是否设置成功。
redis 127.0.0.1:6379&gt; lindex mykey 1
"e"
#索引值6超过了链表中元素的数量，该命令返回nil。
redis 127.0.0.1:6379&gt; lindex mykey 6
(nil)
#设置的索引值6超过了链表中元素的数量，设置失败，该命令返回错误信息。
redis 127.0.0.1:6379&gt; lset mykey 6 hh
(error) ERR index out of range
#仅保留索引值0到2之间的3个元素，注意第0个和第2个元素均被保留。
redis 127.0.0.1:6379&gt; ltrim mykey 0 2
OK
#查看trim后的结果。
redis 127.0.0.1:6379&gt; lrange mykey 0 -1
1) "c"
2) "e"
3) "c"</pre>

#### 4、LINSERT

<pre class="lang:sh decode:true ">#删除该键便于后面的测试。
redis 127.0.0.1:6379&gt; del mykey
(integer) 1
#为后面的示例准备测试数据。
redis 127.0.0.1:6379&gt; lpush mykey a b c d e
(integer) 5
#在a的前面插入新元素a1。
redis 127.0.0.1:6379&gt; linsert mykey before a a1
(integer) 6
#查看是否插入成功，从结果看已经插入。注意lindex的index值是0-based。
redis 127.0.0.1:6379&gt; lindex mykey 0
"e"
#在e的后面插入新元素e2，从返回结果看已经插入成功。
redis 127.0.0.1:6379&gt; linsert mykey after e e2
(integer) 7
#再次查看是否插入成功。
redis 127.0.0.1:6379&gt; lindex mykey 1
"e2"
#在不存在的元素之前或之后插入新元素，该命令操作失败，并返回-1。
redis 127.0.0.1:6379&gt; linsert mykey after k a
(integer) -1
#为不存在的Key插入新元素，该命令操作失败，返回0。
redis 127.0.0.1:6379&gt; linsert mykey1 after a a2
(integer) 0</pre>

#### 5、RPUSH、RPUSHX、RPOP、RPOPLPUSH

<pre class="lang:sh decode:true ">#删除该键，以便于后面的测试。
redis 127.0.0.1:6379&gt; del mykey
(integer) 1
#从链表的尾部插入参数中给出的values，插入顺序是从左到右依次插入。
redis 127.0.0.1:6379&gt; rpush mykey a b c d
(integer) 4
#通过lrange的可以获悉rpush在插入多值时的插入顺序。
redis 127.0.0.1:6379&gt; lrange mykey 0 -1
1) "a"
2) "b"
3) "c"
4) "d"
#该键已经存在并且包含4个元素，rpushx命令将执行成功，并将元素e插入到链表的尾部。
redis 127.0.0.1:6379&gt; rpushx mykey e
(integer) 5
#通过lindex命令可以看出之前的rpushx命令确实执行成功，因为索引值为4的元素已经是新元素了。
redis 127.0.0.1:6379&gt; lindex mykey 4
"e"
#由于mykey2键并不存在，因此该命令不会插入数据，其返回值为0。
redis 127.0.0.1:6379&gt; rpushx mykey2 e
(integer) 0
#在执行rpoplpush命令前，先看一下mykey中链表的元素有哪些，注意他们的位置关系。
redis 127.0.0.1:6379&gt; lrange mykey 0 -1
1) "a"
2) "b"
3) "c"
4) "d"
5) "e"
#将mykey的尾部元素e弹出，同时再插入到mykey2的头部(原子性的完成这两步操作)。
redis 127.0.0.1:6379&gt; rpoplpush mykey mykey2
"e"
#通过lrange命令查看mykey在弹出尾部元素后的结果。
redis 127.0.0.1:6379&gt; lrange mykey 0 -1
1) "a"
2) "b"
3) "c"
4) "d"
#通过lrange命令查看mykey2在插入元素后的结果。
redis 127.0.0.1:6379&gt; lrange mykey2 0 -1
1) "e"
#将source和destination设为同一键，将mykey中的尾部元素移到其头部。
redis 127.0.0.1:6379&gt; rpoplpush mykey mykey
"d"
#查看移动结果。
redis 127.0.0.1:6379&gt; lrange mykey 0 -1
1) "d"
2) "a"
3) "b"
4) "c"#删除该键，以便于后面的测试。
redis 127.0.0.1:6379&gt; del mykey
(integer) 1
#从链表的尾部插入参数中给出的values，插入顺序是从左到右依次插入。
redis 127.0.0.1:6379&gt; rpush mykey a b c d
(integer) 4
#通过lrange的可以获悉rpush在插入多值时的插入顺序。
redis 127.0.0.1:6379&gt; lrange mykey 0 -1
1) "a"
2) "b"
3) "c"
4) "d"
#该键已经存在并且包含4个元素，rpushx命令将执行成功，并将元素e插入到链表的尾部。
redis 127.0.0.1:6379&gt; rpushx mykey e
(integer) 5
#通过lindex命令可以看出之前的rpushx命令确实执行成功，因为索引值为4的元素已经是新元素了。
redis 127.0.0.1:6379&gt; lindex mykey 4
"e"
#由于mykey2键并不存在，因此该命令不会插入数据，其返回值为0。
redis 127.0.0.1:6379&gt; rpushx mykey2 e
(integer) 0
#在执行rpoplpush命令前，先看一下mykey中链表的元素有哪些，注意他们的位置关系。
redis 127.0.0.1:6379&gt; lrange mykey 0 -1
1) "a"
2) "b"
3) "c"
4) "d"
5) "e"
#将mykey的尾部元素e弹出，同时再插入到mykey2的头部(原子性的完成这两步操作)。
redis 127.0.0.1:6379&gt; rpoplpush mykey mykey2
"e"
#通过lrange命令查看mykey在弹出尾部元素后的结果。
redis 127.0.0.1:6379&gt; lrange mykey 0 -1
1) "a"
2) "b"
3) "c"
4) "d"
#通过lrange命令查看mykey2在插入元素后的结果。
redis 127.0.0.1:6379&gt; lrange mykey2 0 -1
1) "e"
#将source和destination设为同一键，将mykey中的尾部元素移到其头部。
redis 127.0.0.1:6379&gt; rpoplpush mykey mykey
"d"
#查看移动结果。
redis 127.0.0.1:6379&gt; lrange mykey 0 -1
1) "d"
2) "a"
3) "b"
4) "c"</pre>

## set

[原文资料](http://www.cnblogs.com/stephen-liu74/archive/2012/03/21/2352512.html)

### 概述

在Redis中，我们可以将Set类型看作为没有排序的字符集合，和List类型一样，我们也可以在该类型的数据值上执行添加、删除或判断某一元素是否存在等操作。需要说明的是，这些操作的时间复杂度为O(1)，即常量时间内完成次操作。Set可包含的最大元素数量是4294967295。
和List类型不同的是，Set集合中不允许出现重复的元素，这一点和C++标准库中的set容器是完全相同的。换句话说，如果多次添加相同元素，Set中将仅保留该元素的一份拷贝。和List类型相比，Set类型在功能上还存在着一个非常重要的特性，即在服务器端完成多个Sets之间的聚合计算操作，如unions、intersections和differences。由于这些操作均在服务端完成，因此效率极高，而且也节省了大量的网络IO开销。

### 相关命令列表

<table>
<thead>
<tr>
  <th align="left">命令原型</th>
  <th align="left">时间复杂度</th>
  <th align="left">命令描述</th>
  <th align="left">返回值</th>
</tr>
</thead>
<tbody>
<tr>
  <td align="left">**SADD** key member [member ...]</td>
  <td align="left">O(N)</td>
  <td align="left">时间复杂度中的N表示操作的成员数量。如果在插入的过程用，参数中有的成员在Set中已经存在，该成员将被忽略，而其它成员仍将会被正常插入。如果执行该命令之前，该Key并不存在，该命令将会创建一个新的Set，此后再将参数中的成员陆续插入。如果该Key的Value不是Set类型，该命令将返回相关的错误信息。</td>
  <td align="left">本次操作实际插入的成员数量。</td>
</tr>
<tr>
  <td align="left">**SCARD** key</td>
  <td align="left">O(1)</td>
  <td align="left">获取Set中成员的数量。</td>
  <td align="left">返回Set中成员的数量，如果该Key并不存在，返回0。</td>
</tr>
<tr>
  <td align="left">**SISMEMBER** key member</td>
  <td align="left">O(1)</td>
  <td align="left">判断参数中指定成员是否已经存在于与Key相关联的Set集合中。</td>
  <td align="left">1表示已经存在，0表示不存在，或该Key本身并不存在。</td>
</tr>
<tr>
  <td align="left">**SMEMBERS** key</td>
  <td align="left">O(N)</td>
  <td align="left">时间复杂度中的N表示Set中已经存在的成员数量。获取与该Key关联的Set中所有的成员。</td>
  <td align="left">返回Set中所有的成员。</td>
</tr>
<tr>
  <td align="left">**SPOP** key</td>
  <td align="left">O(1)</td>
  <td align="left">随机的移除并返回Set中的某一成员。 由于Set中元素的布局不受外部控制，因此无法像List那样确定哪个元素位于Set的头部或者尾部。</td>
  <td align="left">返回移除的成员，如果该Key并不存在，则返回nil。</td>
</tr>
<tr>
  <td align="left">**SREM** key member [member ...]</td>
  <td align="left">O(N)</td>
  <td align="left">时间复杂度中的N表示被删除的成员数量。从与Key关联的Set中删除参数中指定的成员，不存在的参数成员将被忽略，如果该Key并不存在，将视为空Set处理。</td>
  <td align="left">从Set中实际移除的成员数量，如果没有则返回0。</td>
</tr>
<tr>
  <td align="left">**SRANDMEMBER** key</td>
  <td align="left">O(1)</td>
  <td align="left">和SPOP一样，随机的返回Set中的一个成员，不同的是该命令并不会删除返回的成员。</td>
  <td align="left">返回随机位置的成员，如果Key不存在则返回nil。</td>
</tr>
<tr>
  <td align="left">**SMOVE** source destination member</td>
  <td align="left">O(1)</td>
  <td align="left">原子性的将参数中的成员从source键移入到destination键所关联的Set中。因此在某一时刻，该成员或者出现在source中，或者出现在destination中。如果该成员在source中并不存在，该命令将不会再执行任何操作并返回0，否则，该成员将从source移入到destination。如果此时该成员已经在destination中存在，那么该命令仅是将该成员从source中移出。如果和Key关联的Value不是Set，将返回相关的错误信息。</td>
  <td align="left">1表示正常移动，0表示source中并不包含参数成员。</td>
</tr>
<tr>
  <td align="left">**SDIFF** key [key ...]</td>
  <td align="left">O(N)</td>
  <td align="left">时间复杂度中的N表示所有Sets中成员的总数量。返回参数中第一个Key所关联的Set和其后所有Keys所关联的Sets中成员的差异。如果Key不存在，则视为空Set。</td>
  <td align="left">差异结果成员的集合。</td>
</tr>
<tr>
  <td align="left">**SDIFFSTORE** destination key [key ...]</td>
  <td align="left">O(N)</td>
  <td align="left">该命令和SDIFF命令在功能上完全相同，两者之间唯一的差别是SDIFF返回差异的结果成员，而该命令将差异成员存储在destination关联的Set中。如果destination键已经存在，该操作将覆盖它的成员。</td>
  <td align="left">返回差异成员的数量。</td>
</tr>
<tr>
  <td align="left">**SINTER** key [key ...]</td>
  <td align="left">O(N*M)</td>
  <td align="left">时间复杂度中的N表示最小Set中元素的数量，M则表示参数中Sets的数量。该命令将返回参数中所有Keys关联的Sets中成员的交集。因此如果参数中任何一个Key关联的Set为空，或某一Key不存在，那么该命令的结果将为空集。</td>
  <td align="left">交集结果成员的集合。</td>
</tr>
<tr>
  <td align="left">**SINTERSTORE** destination key [key ...]</td>
  <td align="left">O(N*M)</td>
  <td align="left">该命令和SINTER命令在功能上完全相同，两者之间唯一的差别是SINTER返回交集的结果成员，而该命令将交集成员存储在destination关联的Set中。如果destination键已经存在，该操作将覆盖它的成员。</td>
  <td align="left">返回交集成员的数量。</td>
</tr>
<tr>
  <td align="left">**SUNION** key [key ...]</td>
  <td align="left">O(N)</td>
  <td align="left">时间复杂度中的N表示所有Sets中成员的总数量。该命令将返回参数中所有Keys关联的Sets中成员的并集。</td>
  <td align="left">并集结果成员的集合。</td>
</tr>
<tr>
  <td align="left">**SUNIONSTORE** destination key [key ...]</td>
  <td align="left">O(N)</td>
  <td align="left">该命令和SUNION命令在功能上完全相同，两者之间唯一的差别是SUNION返回并集的结果成员，而该命令将并集成员存储在destination关联的Set中。如果destination键已经存在，该操作将覆盖它的成员。</td>
  <td align="left">返回并集成员的数量。</td>
</tr>
</tbody>
</table>

### 命令示例

#### 1、SADD、SMEMBERS、SCARD、SISMEMBER

<pre class="lang:sh decode:true ">#在Shell命令行下启动Redis的客户端程序。
/&gt; redis-cli
#插入测试数据，由于该键myset之前并不存在，因此参数中的三个成员都被正常插入。
redis 127.0.0.1:6379&gt; sadd myset a b c
(integer) 3
#由于参数中的a在myset中已经存在，因此本次操作仅仅插入了d和e两个新成员。
redis 127.0.0.1:6379&gt; sadd myset a d e
(integer) 2
#判断a是否已经存在，返回值为1表示存在。
redis 127.0.0.1:6379&gt; sismember myset a
(integer) 1
#判断f是否已经存在，返回值为0表示不存在。
redis 127.0.0.1:6379&gt; sismember myset f
(integer) 0
#通过smembers命令查看插入的结果，从结果可以，输出的顺序和插入顺序无关。
redis 127.0.0.1:6379&gt; smembers myset
1) "c"
2) "d"
3) "a"
4) "b"
5) "e"
#获取Set集合中元素的数量。
redis 127.0.0.1:6379&gt; scard myset
(integer) 5</pre>

#### 2、SPOP、SREM、SRANDMEMBER、SMOVE

<pre class="lang:sh decode:true ">#删除该键，便于后面的测试。
redis 127.0.0.1:6379&gt; del myset
(integer) 1
#为后面的示例准备测试数据。
redis 127.0.0.1:6379&gt; sadd myset a b c d
(integer) 4
#查看Set中成员的位置。
redis 127.0.0.1:6379&gt; smembers myset
1) "c"
2) "d"
3) "a"
4) "b"
#从结果可以看出，该命令确实是随机的返回了某一成员。
redis 127.0.0.1:6379&gt; srandmember myset
"c"
#Set中尾部的成员b被移出并返回，事实上b并不是之前插入的第一个或最后一个成员。
redis 127.0.0.1:6379&gt; spop myset
"b"
#查看移出后Set的成员信息。
redis 127.0.0.1:6379&gt; smembers myset
1) "c"
2) "d"
3) "a"
#从Set中移出a、d和f三个成员，其中f并不存在，因此只有a和d两个成员被移出，返回为2。
redis 127.0.0.1:6379&gt; srem myset a d f
(integer) 2
#查看移出后的输出结果。
redis 127.0.0.1:6379&gt; smembers myset
1) "c"
#为后面的smove命令准备数据。
redis 127.0.0.1:6379&gt; sadd myset a b
(integer) 2
redis 127.0.0.1:6379&gt; sadd myset2 c d
(integer) 2
#将a从myset移到myset2，从结果可以看出移动成功。
redis 127.0.0.1:6379&gt; smove myset myset2 a
(integer) 1
#再次将a从myset移到myset2，由于此时a已经不是myset的成员了，因此移动失败并返回0。
redis 127.0.0.1:6379&gt; smove myset myset2 a
(integer) 0
#分别查看myset和myset2的成员，确认移动是否真的成功。
redis 127.0.0.1:6379&gt; smembers myset
1) "b"
redis 127.0.0.1:6379&gt; smembers myset2
1) "c"
2) "d"
3) "a"</pre>

#### 3、SDIFF、SDIFFSTORE、SINTER、SINTERSTORE

<pre class="lang:default decode:true ">#为后面的命令准备测试数据。
redis 127.0.0.1:6379&gt; sadd myset a b c d
(integer) 4
redis 127.0.0.1:6379&gt; sadd myset2 c
(integer) 1
redis 127.0.0.1:6379&gt; sadd myset3 a c e
(integer) 3
#myset和myset2相比，a、b和d三个成员是两者之间的差异成员。再用这个结果继续和myset3进行差异比较，b和d是myset3不存在的成员。
redis 127.0.0.1:6379&gt; sdiff myset myset2 myset3
1) "d"
2) "b"
#将3个集合的差异成员存在在diffkey关联的Set中，并返回插入的成员数量。
redis 127.0.0.1:6379&gt; sdiffstore diffkey myset myset2 myset3
(integer) 2
#查看一下sdiffstore的操作结果。
redis 127.0.0.1:6379&gt; smembers diffkey
1) "d"
2) "b"
#从之前准备的数据就可以看出，这三个Set的成员交集只有c。
redis 127.0.0.1:6379&gt; sinter myset myset2 myset3
1) "c"
#将3个集合中的交集成员存储到与interkey关联的Set中，并返回交集成员的数量。
redis 127.0.0.1:6379&gt; sinterstore interkey myset myset2 myset3
(integer) 1
#查看一下sinterstore的操作结果。
redis 127.0.0.1:6379&gt; smembers interkey
1) "c"
#获取3个集合中的成员的并集。    
redis 127.0.0.1:6379&gt; sunion myset myset2 myset3
1) "b"
2) "c"
3) "d"
4) "e"
5) "a"
#将3个集合中成员的并集存储到unionkey关联的set中，并返回并集成员的数量。
redis 127.0.0.1:6379&gt; sunionstore unionkey myset myset2 myset3
(integer) 5
#查看一下suiionstore的操作结果。
redis 127.0.0.1:6379&gt; smembers unionkey
1) "b"
2) "c"
3) "d"
4) "e"
5) "a"</pre>

## zset

[原文资料](http://www.cnblogs.com/stephen-liu74/archive/2012/03/23/2354994.html)

### 概述

Sorted-Sets和Sets类型极为相似，它们都是字符串的集合，都不允许重复的成员出现在一个Set中。它们之间的主要差别是Sorted-Sets中的每一个成员都会有一个分数(score)与之关联，Redis正是通过分数来为集合中的成员进行从小到大的排序。然而需要额外指出的是，尽管Sorted-Sets中的成员必须是唯一的，但是分数(score)却是可以重复的。
在Sorted-Set中添加、删除或更新一个成员都是非常快速的操作，其时间复杂度为集合中成员数量的对数。由于Sorted-Sets中的成员在集合中的位置是有序的，因此，即便是访问位于集合中部的成员也仍然是非常高效的。事实上，Redis所具有的这一特征在很多其它类型的数据库中是很难实现的，换句话说，在该点上要想达到和Redis同样的高效，在其它数据库中进行建模是非常困难的。

### 相关命令列表

<table>
<thead>
<tr>
  <th align="left">命令原型</th>
  <th align="left">时间复杂度</th>
  <th align="left">命令描述</th>
  <th align="left">返回值</th>
</tr>
</thead>
<tbody>
<tr>
  <td align="left">**ZADD** key score member [score] [member]</td>
  <td align="left">O(log(N))</td>
  <td align="left">时间复杂度中的N表示Sorted-Sets中成员的数量。添加参数中指定的所有成员及其分数到指定key的Sorted-Set中，在该命令中我们可以指定多组score/member作为参数。如果在添加时参数中的某一成员已经存在，该命令将更新此成员的分数为新值，同时再将该成员基于新值重新排序。如果键不存在，该命令将为该键创建一个新的Sorted-Sets Value，并将score/member对插入其中。如果该键已经存在，但是与其关联的Value不是Sorted-Sets类型，相关的错误信息将被返回。</td>
  <td align="left">本次操作实际插入的成员数量。</td>
</tr>
<tr>
  <td align="left">**ZCARD** key</td>
  <td align="left">O(1)</td>
  <td align="left">获取与该Key相关联的Sorted-Sets中包含的成员数量。</td>
  <td align="left">返回Sorted-Sets中的成员数量，如果该Key不存在，返回0。</td>
</tr>
<tr>
  <td align="left">**ZCOUNT** key min max</td>
  <td align="left">O(log(N)+M)</td>
  <td align="left">时间复杂度中的N表示Sorted-Sets中成员的数量，M则表示min和max之间元素的数量。该命令用于获取分数(score)在min和max之间的成员数量。**针对min和max参数需要额外说明的是，-inf和+inf分别表示Sorted-Sets中分数的最高值和最低值。缺省情况下，min和max表示的范围是闭区间范围，即min &lt;= score &lt;= max内的成员将被返回。然而我们可以通过在min和max的前面添加"("字符来表示开区间，如(min max表示min &lt; score &lt;= max，而(min (max表示min &lt; score &lt; max。**</td>
  <td align="left">分数指定范围内成员的数量。</td>
</tr>
<tr>
  <td align="left">**ZINCRBY** key increment member</td>
  <td align="left">O(log(N))</td>
  <td align="left">时间复杂度中的N表示Sorted-Sets中成员的数量。该命令将为指定Key中的指定成员增加指定的分数。如果成员不存在，该命令将添加该成员并假设其初始分数为0，此后再将其分数加上increment。如果Key不存，该命令将创建该Key及其关联的Sorted-Sets，并包含参数指定的成员，其分数为increment参数。如果与该Key关联的不是Sorted-Sets类型，相关的错误信息将被返回。</td>
  <td align="left">以字符串形式表示的新分数。</td>
</tr>
<tr>
  <td align="left">**ZRANGE** key start stop [WITHSCORES]</td>
  <td align="left">O(log(N)+M)</td>
  <td align="left">时间复杂度中的N表示Sorted-Set中成员的数量，M则表示返回的成员数量。该命令返回顺序在参数start和stop指定范围内的成员，这里start和stop参数都是0-based，即0表示第一个成员，-1表示最后一个成员。如果start大于该Sorted-Set中的最大索引值，或start &gt; stop，此时一个空集合将被返回。如果stop大于最大索引值，该命令将返回从start到集合的最后一个成员。**如果命令中带有可选参数WITHSCORES选项，该命令在返回的结果中将包含每个成员的分数值，如value1,score1,value2,score2...。**</td>
  <td align="left">返回索引在start和stop之间的成员列表。</td>
</tr>
<tr>
  <td align="left">**ZRANGEBYSCORE** key min max [WITHSCORES] [LIMIT offset count]</td>
  <td align="left">O(log(N)+M)</td>
  <td align="left">时间复杂度中的N表示Sorted-Set中成员的数量，M则表示返回的成员数量。该命令将返回分数在min和max之间的所有成员，即满足表达式min &lt;= score &lt;= max的成员，其中返回的成员是按照其分数从低到高的顺序返回，如果成员具有相同的分数，则按成员的字典顺序返回。可选参数LIMIT用于限制返回成员的数量范围。可选参数offset表示从符合条件的第offset个成员开始返回，同时返回count个成员。可选参数WITHSCORES的含义参照ZRANGE中该选项的说明。**最后需要说明的是参数中min和max的规则可参照命令ZCOUNT。**</td>
  <td align="left">返回分数在指定范围内的成员列表。</td>
</tr>
<tr>
  <td align="left">**ZRANK** key member</td>
  <td align="left">O(log(N))</td>
  <td align="left">时间复杂度中的N表示Sorted-Set中成员的数量。Sorted-Set中的成员都是按照分数从低到高的顺序存储，该命令将返回参数中指定成员的位置值，其中0表示第一个成员，它是Sorted-Set中分数最低的成员。</td>
  <td align="left">如果该成员存在，则返回它的位置索引值。否则返回nil。</td>
</tr>
<tr>
  <td align="left">**ZREM** key member [member ...]</td>
  <td align="left">O(M log(N))</td>
  <td align="left">时间复杂度中N表示Sorted-Set中成员的数量，M则表示被删除的成员数量。该命令将移除参数中指定的成员，其中不存在的成员将被忽略。如果与该Key关联的Value不是Sorted-Set，相应的错误信息将被返回。</td>
  <td align="left">实际被删除的成员数量。</td>
</tr>
<tr>
  <td align="left">**ZREVRANGE** key start stop [WITHSCORES]</td>
  <td align="left">O(log(N)+M)</td>
  <td align="left">时间复杂度中的N表示Sorted-Set中成员的数量，M则表示返回的成员数量。该命令的功能和`ZRANGE`基本相同，唯一的差别在于该命令是通过反向排序获取指定位置的成员，即从高到低的顺序。如果成员具有相同的分数，则按降序字典顺序排序。</td>
  <td align="left">返回指定的成员列表。</td>
</tr>
<tr>
  <td align="left">**ZREVRANK** key member</td>
  <td align="left">O(log(N))</td>
  <td align="left">时间复杂度中的N表示Sorted-Set中成员的数量。该命令的功能和`ZRANK`基本相同，唯一的差别在于该命令获取的索引是从高到低排序后的位置，同样0表示第一个元素，即分数最高的成员。</td>
  <td align="left">如果该成员存在，则返回它的位置索引值。否则返回nil。</td>
</tr>
<tr>
  <td align="left">**ZSCORE** key member</td>
  <td align="left">O(1)</td>
  <td align="left">获取指定Key的指定成员的分数。</td>
  <td align="left">如果该成员存在，以字符串的形式返回其分数，否则返回nil。</td>
</tr>
<tr>
  <td align="left">**ZREVRANGEBYSCORE** key max min [WITHSCORES] [LIMIT offset count]</td>
  <td align="left">O(log(N)+M)</td>
  <td align="left">时间复杂度中的N表示Sorted-Set中成员的数量，M则表示返回的成员数量。该命令除了排序方式是基于从高到低的分数排序之外，其它功能和参数含义均与`ZRANGEBYSCORE`相同。</td>
  <td align="left">返回分数在指定范围内的成员列表。</td>
</tr>
<tr>
  <td align="left">**ZREMRANGEBYRANK** key start stop</td>
  <td align="left">O(log(N)+M)</td>
  <td align="left">时间复杂度中的N表示Sorted-Set中成员的数量，M则表示被删除的成员数量。删除索引位置位于start和stop之间的成员，start和stop都是0-based，即0表示分数最低的成员，-1表示最后一个成员，即分数最高的成员。</td>
  <td align="left">被删除的成员数量。</td>
</tr>
<tr>
  <td align="left">**ZREMRANGEBYSCORE** key min max</td>
  <td align="left">O(log(N)+M)</td>
  <td align="left">时间复杂度中的N表示Sorted-Set中成员的数量，M则表示被删除的成员数量。删除分数在min和max之间的所有成员，即满足表达式min &lt;= score &lt;= max的所有成员。对于min和max参数，可以采用开区间的方式表示，具体规则参照`ZCOUNT`。</td>
  <td align="left">被删除的成员数量。</td>
</tr>
</tbody>
</table>

### 命令实例

#### 1、ZADD、ZCARD、ZCOUNT、ZREM、ZINCRBY、ZSCORE、ZRANGE、ZRANK

<pre class="lang:sh decode:true ">#在Shell的命令行下启动Redis客户端工具。
/&gt; redis-cli
#添加一个分数为1的成员。
redis 127.0.0.1:6379&gt; zadd myzset 1 "one"
(integer) 1
#添加两个分数分别是2和3的两个成员。
redis 127.0.0.1:6379&gt; zadd myzset 2 "two" 3 "three"
(integer) 2
#0表示第一个成员，-1表示最后一个成员。WITHSCORES选项表示返回的结果中包含每个成员及其分数，否则只返回成员。
redis 127.0.0.1:6379&gt; zrange myzset 0 -1 WITHSCORES
1) "one"
2) "1"
3) "two"
4) "2"
5) "three"
6) "3"
#获取成员one在Sorted-Set中的位置索引值。0表示第一个位置。
redis 127.0.0.1:6379&gt; zrank myzset one
(integer) 0
#成员four并不存在，因此返回nil。
redis 127.0.0.1:6379&gt; zrank myzset four
(nil)
#获取myzset键中成员的数量。    
redis 127.0.0.1:6379&gt; zcard myzset
(integer) 3
#返回与myzset关联的Sorted-Set中，分数满足表达式1 &lt;= score &lt;= 2的成员的数量。
redis 127.0.0.1:6379&gt; zcount myzset 1 2
(integer) 2
#删除成员one和two，返回实际删除成员的数量。
redis 127.0.0.1:6379&gt; zrem myzset one two
(integer) 2
#查看是否删除成功。
redis 127.0.0.1:6379&gt; zcard myzset
(integer) 1
#获取成员three的分数。返回值是字符串形式。
redis 127.0.0.1:6379&gt; zscore myzset three
"3"
#由于成员two已经被删除，所以该命令返回nil。
redis 127.0.0.1:6379&gt; zscore myzset two
(nil)
#将成员one的分数增加2，并返回该成员更新后的分数。
redis 127.0.0.1:6379&gt; zincrby myzset 2 one
"3"
#将成员one的分数增加-1，并返回该成员更新后的分数。
redis 127.0.0.1:6379&gt; zincrby myzset -1 one
"2"
#查看在更新了成员的分数后是否正确。
redis 127.0.0.1:6379&gt; zrange myzset 0 -1 WITHSCORES
1) "one"
2) "2"
3) "two"
4) "2"
5) "three"
6) "3"</pre>

#### 2、ZRANGEBYSCORE、ZREMRANGEBYRANK、ZREMRANGEBYSCORE

<pre class="lang:sh decode:true ">redis 127.0.0.1:6379&gt; del myzset
(integer) 1
redis 127.0.0.1:6379&gt; zadd myzset 1 one 2 two 3 three 4 four
(integer) 4
#获取分数满足表达式1 &lt;= score &lt;= 2的成员。
redis 127.0.0.1:6379&gt; zrangebyscore myzset 1 2
1) "one"
2) "two"
#获取分数满足表达式1 &lt; score &lt;= 2的成员。
redis 127.0.0.1:6379&gt; zrangebyscore myzset (1 2
1) "two"
#-inf表示第一个成员，+inf表示最后一个成员，limit后面的参数用于限制返回成员的自己，
#2表示从位置索引(0-based)等于2的成员开始，去后面3个成员。
redis 127.0.0.1:6379&gt; zrangebyscore myzset -inf +inf limit 2 3
1) "three"
2) "four"
#删除分数满足表达式1 &lt;= score &lt;= 2的成员，并返回实际删除的数量。
redis 127.0.0.1:6379&gt; zremrangebyscore myzset 1 2
(integer) 2
#看出一下上面的删除是否成功。
redis 127.0.0.1:6379&gt; zrange myzset 0 -1
1) "three"
2) "four"
#删除位置索引满足表达式0 &lt;= rank &lt;= 1的成员。
redis 127.0.0.1:6379&gt; zremrangebyrank myzset 0 1
(integer) 2
#查看上一条命令是否删除成功。
redis 127.0.0.1:6379&gt; zcard myzset
(integer) 0</pre>

#### 3、ZREVRANGE、ZREVRANGEBYSCORE、ZREVRANK:

<pre class="lang:sh decode:true ">#为后面的示例准备测试数据。
redis 127.0.0.1:6379&gt; del myzset
(integer) 0
redis 127.0.0.1:6379&gt; zadd myzset 1 one 2 two 3 three 4 four
(integer) 4
#以位置索引从高到低的方式获取并返回此区间内的成员。
redis 127.0.0.1:6379&gt; zrevrange myzset 0 -1 WITHSCORES
1) "four"
2) "4"
3) "three"
4) "3"
5) "two"
6) "2"
7) "one"
8) "1"
#由于是从高到低的排序，所以位置等于0的是four，1是three，并以此类推。
redis 127.0.0.1:6379&gt; zrevrange myzset 1 3
1) "three"
2) "two"
3) "one"
#由于是从高到低的排序，所以one的位置是3。
redis 127.0.0.1:6379&gt; zrevrank myzset one
(integer) 3
#由于是从高到低的排序，所以four的位置是0。
redis 127.0.0.1:6379&gt; zrevrank myzset four
(integer) 0
#获取分数满足表达式3 &gt;= score &gt;= 0的成员，并以相反的顺序输出，即从高到底的顺序。
redis 127.0.0.1:6379&gt; zrevrangebyscore myzset 3 0
1) "three"
2) "two"
3) "one"
#该命令支持limit选项，其含义等同于zrangebyscore中的该选项，只是在计算位置时按照相反的顺序计算和获取。
redis 127.0.0.1:6379&gt; zrevrangebyscore myzset 4 0 limit 1 2
1) "three"
2) "two"</pre>

## key

[原文资料](http://www.cnblogs.com/stephen-liu74/archive/2012/03/26/2356951.html)

### 概述

前面说到了如String、List、Set、Hashes和Sorted-Set这些数据类型的命令，这些命令都具有一个共同点，即所有的操作都是针对与Key关联的Value的。而本节讲述与Key相关的Redis命令是操作Key-Value中的Key这边的操作。

### 相关命令列表

<table>
<thead>
<tr>
  <th align="left">命令原型</th>
  <th align="left">时间复杂度</th>
  <th align="left">命令描述</th>
  <th align="left">返回值</th>
</tr>
</thead>
<tbody>
<tr>
  <td align="left">KEYS pattern</td>
  <td align="left">O(N)</td>
  <td align="left">时间复杂度中的N表示数据库中Key的数量。获取所有匹配pattern参数的Keys。需要说明的是，在我们的正常操作中应该尽量避免对该命令的调用，因为对于大型数据库而言，该命令是非常耗时的，对Redis服务器的性能打击也是比较大的。**pattern支持glob-style的通配符格式，如&#42;表示任意一个或多个字符，?表示任意字符，[abc]表示方括号中任意一个字母。**</td>
  <td align="left">匹配模式的键列表。</td>
</tr>
<tr>
  <td align="left">**DEL** key [key ...]</td>
  <td align="left">O(N)</td>
  <td align="left">时间复杂度中的N表示删除的Key数量。从数据库删除中参数中指定的keys，如果指定键不存在，则直接忽略。还需要另行指出的是，如果指定的Key关联的数据类型不是String类型，而是List、Set、Hashes和Sorted Set等容器类型，该命令删除每个键的时间复杂度为O(M)，其中M表示容器中元素的数量。而对于String类型的Key，其时间复杂度为O(1)。</td>
  <td align="left">实际被删除的Key数量。</td>
</tr>
<tr>
  <td align="left">**EXISTS** key</td>
  <td align="left">O(1)</td>
  <td align="left">判断指定键是否存在。</td>
  <td align="left">1表示存在，0表示不存在。</td>
</tr>
<tr>
  <td align="left">**MOVE** key db</td>
  <td align="left">O(1)</td>
  <td align="left">将当前数据库中指定的键Key移动到参数中指定的数据库中。如果该Key在目标数据库中已经存在，或者在当前数据库中并不存在，该命令将不做任何操作并返回0。  </td>
  <td align="left">移动成功返回1，否则0。</td>
</tr>
<tr>
  <td align="left">**RENAME** key newkey</td>
  <td align="left">O(1)</td>
  <td align="left">为指定指定的键重新命名，如果参数中的两个Keys的命令相同，或者是源Key不存在，该命令都会返回相关的错误信息。如果newKey已经存在，则直接覆盖。</td>
  <td align="left">** **</td>
</tr>
<tr>
  <td align="left">**RENAMENX** key newkey</td>
  <td align="left">O(1)</td>
  <td align="left">如果新值不存在，则将参数中的原值修改为新值。其它条件和RENAME一致。</td>
  <td align="left">1表示修改成功，否则0。</td>
</tr>
<tr>
  <td align="left">**PERSIST** key</td>
  <td align="left">O(1)</td>
  <td align="left">如果Key存在过期时间，该命令会将其过期时间消除，使该Key不再有超时，而是可以持久化存储。</td>
  <td align="left">1表示Key的过期时间被移出，0表示该Key不存在或没有过期时间。</td>
</tr>
<tr>
  <td align="left">**EXPIRE** key seconds</td>
  <td align="left">O(1)</td>
  <td align="left">该命令为参数中指定的Key设定超时的秒数，在超过该时间后，Key被自动的删除。如果该Key在超时之前被修改，与该键关联的超时将被移除。</td>
  <td align="left">1表示超时被设置，0则表示Key不存在，或不能被设置。</td>
</tr>
<tr>
  <td align="left">EXPIREAT key timestamp</td>
  <td align="left">O(1)</td>
  <td align="left">该命令的逻辑功能和EXPIRE完全相同，唯一的差别是该命令指定的超时时间是绝对时间，而不是相对时间。该时间参数是Unix timestamp格式的，即从1970年1月1日开始所流经的秒数。</td>
  <td align="left">1表示超时被设置，0则表示Key不存在，或不能被设置。</td>
</tr>
<tr>
  <td align="left">**TTL** key</td>
  <td align="left">O(1)</td>
  <td align="left">获取该键所剩的超时描述。</td>
  <td align="left">返回所剩描述，如果该键不存在或没有超时设置，则返回-1。</td>
</tr>
<tr>
  <td align="left">**ANDOMKEY**</td>
  <td align="left">O(1)  </td>
  <td align="left">从当前打开的数据库中随机的返回一个Key。</td>
  <td align="left">返回的随机键，如果该数据库是空的则返回nil。</td>
</tr>
<tr>
  <td align="left">**TYPE** key</td>
  <td align="left">O(1)</td>
  <td align="left">获取与参数中指定键关联值的类型，该命令将以字符串的格式返回。</td>
  <td align="left">返回的字符串为string、list、set、hash和zset，如果key不存在返回none。</td>
</tr>
<tr>
  <td align="left">**SORT** key [BY pattern] [LIMIT offset count] [GET pattern [GET pattern ...]] [ASC、DESC] [ALPHA] [STORE destination]</td>
  <td align="left">O(N+M*log(M))</td>
  <td align="left">这个命令相对来说是比较复杂的，因此我们这里只是给出最基本的用法，有兴趣的网友可以去参考redis的官方文档。</td>
  <td align="left">返回排序后的原始列表。</td>
</tr>
</tbody>
</table>

### 命令示例

#### 1、KEYS、RENAME、DEL、EXISTS、MOVE、RENAMENX

<pre class="lang:sh decode:true ">#在Shell命令行下启动Redis客户端工具。
/&gt; redis-cli
#清空当前选择的数据库，以便于对后面示例的理解。
redis 127.0.0.1:6379&gt; flushdb
OK
#添加String类型的模拟数据。
redis 127.0.0.1:6379&gt; set mykey 2
OK
redis 127.0.0.1:6379&gt; set mykey2 "hello"
OK
#添加Set类型的模拟数据。
redis 127.0.0.1:6379&gt; sadd mysetkey 1 2 3
(integer) 3
#添加Hash类型的模拟数据。
redis 127.0.0.1:6379&gt; hset mmtest username "stephen"
(integer) 1
#根据参数中的模式，获取当前数据库中符合该模式的所有key，从输出可以看出，该命令在执行时并不区分与Key关联的Value类型。
redis 127.0.0.1:6379&gt; keys my*
1) "mysetkey"
2) "mykey"
3) "mykey2"
#删除了两个Keys。
redis 127.0.0.1:6379&gt; del mykey mykey2
(integer) 2
#查看一下刚刚删除的Key是否还存在，从返回结果看，mykey确实已经删除了。
redis 127.0.0.1:6379&gt; exists mykey
(integer) 0
#查看一下没有删除的Key，以和上面的命令结果进行比较。
redis 127.0.0.1:6379&gt; exists mysetkey
(integer) 1
#将当前数据库中的mysetkey键移入到ID为1的数据库中，从结果可以看出已经移动成功。
redis 127.0.0.1:6379&gt; move mysetkey 1
(integer) 1
#打开ID为1的数据库。
redis 127.0.0.1:6379&gt; select 1
OK
#查看一下刚刚移动过来的Key是否存在，从返回结果看已经存在了。
redis 127.0.0.1:6379[1]&gt; exists mysetkey
(integer) 1
#在重新打开ID为0的缺省数据库。
redis 127.0.0.1:6379[1]&gt; select 0
OK
#查看一下刚刚移走的Key是否已经不存在，从返回结果看已经移走。
redis 127.0.0.1:6379&gt; exists mysetkey
(integer) 0
#准备新的测试数据。    
redis 127.0.0.1:6379&gt; set mykey "hello"
OK
#将mykey改名为mykey1
redis 127.0.0.1:6379&gt; rename mykey mykey1
OK
#由于mykey已经被重新命名，再次获取将返回nil。
redis 127.0.0.1:6379&gt; get mykey
(nil)
#通过新的键名获取。
redis 127.0.0.1:6379&gt; get mykey1
"hello"
#由于mykey已经不存在了，所以返回错误信息。
redis 127.0.0.1:6379&gt; rename mykey mykey1
(error) ERR no such key
#为renamenx准备测试key
redis 127.0.0.1:6379&gt; set oldkey "hello"
OK
redis 127.0.0.1:6379&gt; set newkey "world"
OK
#由于newkey已经存在，因此该命令未能成功执行。
redis 127.0.0.1:6379&gt; renamenx oldkey newkey
(integer) 0
#查看newkey的值，发现它也没有被renamenx覆盖。
redis 127.0.0.1:6379&gt; get newkey
"world"</pre>

#### 2、PERSIST、EXPIRE、EXPIREAT、TTL

<pre class="lang:sh decode:true ">#为后面的示例准备的测试数据。
redis 127.0.0.1:6379&gt; set mykey "hello"
OK
#将该键的超时设置为100秒。
redis 127.0.0.1:6379&gt; expire mykey 100
(integer) 1
#通过ttl命令查看一下还剩下多少秒。
redis 127.0.0.1:6379&gt; ttl mykey
(integer) 97
#立刻执行persist命令，该存在超时的键变成持久化的键，即将该Key的超时去掉。
redis 127.0.0.1:6379&gt; persist mykey
(integer) 1
#ttl的返回值告诉我们，该键已经没有超时了。
redis 127.0.0.1:6379&gt; ttl mykey
(integer) -1
#为后面的expire命令准备数据。
redis 127.0.0.1:6379&gt; del mykey
(integer) 1
redis 127.0.0.1:6379&gt; set mykey "hello"
OK
#设置该键的超时被100秒。
redis 127.0.0.1:6379&gt; expire mykey 100
(integer) 1
#用ttl命令看一下当前还剩下多少秒，从结果中可以看出还剩下96秒。
redis 127.0.0.1:6379&gt; ttl mykey
(integer) 96
#重新更新该键的超时时间为20秒，从返回值可以看出该命令执行成功。
redis 127.0.0.1:6379&gt; expire mykey 20
(integer) 1
#再用ttl确认一下，从结果中可以看出果然被更新了。
redis 127.0.0.1:6379&gt; ttl mykey
(integer) 17
#立刻更新该键的值，以使其超时无效。
redis 127.0.0.1:6379&gt; set mykey "world"
OK
#从ttl的结果可以看出，在上一条修改该键的命令执行后，该键的超时也无效了。
redis 127.0.0.1:6379&gt; ttl mykey
(integer) -1</pre>

#### 3、TYPE、RANDOMKEY、SORT

<pre class="lang:sh decode:true ">#由于mm键在数据库中不存在，因此该命令返回none。
redis 127.0.0.1:6379&gt; type mm
none
#mykey的值是字符串类型，因此返回string。
redis 127.0.0.1:6379&gt; type mykey
string
#准备一个值是set类型的键。
redis 127.0.0.1:6379&gt; sadd mysetkey 1 2
(integer) 2
#mysetkey的键是set，因此返回字符串set。
redis 127.0.0.1:6379&gt; type mysetkey
set
#返回数据库中的任意键。
redis 127.0.0.1:6379&gt; randomkey
"oldkey"
#清空当前打开的数据库。
redis 127.0.0.1:6379&gt; flushdb
OK
#由于没有数据了，因此返回nil。
redis 127.0.0.1:6379&gt; randomkey
(nil)</pre>

&nbsp;
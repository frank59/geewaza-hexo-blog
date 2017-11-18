---
title: Redis配置文件详解
tags:
  - Redis
id: 227
categories:
  - 笔记
date: 2015-01-14 20:25:39
---

![](http://ww1.sinaimg.cn/mw690/3d6ce2f1jw1eo6xmvijf7j206803f0sq.jpg)
<!--more-->

## 主要参数说明

<table>
<thead>
<tr>
  <th align="left">默认参数</th>
  <th align="left">参数详解</th>
</tr>
</thead>
<tbody>
<tr>
  <td align="left">`daemonize no`</td>
  <td align="left">Redis默认不是以守护进程的方式运行，可以通过该配置项修改，使用yes启用守护进程。</td>
</tr>
<tr>
  <td align="left">`pidfile /var/run/redis.pid`</td>
  <td align="left">当Redis以守护进程方式运行时，Redis默认会把pid写入/var/run/redis.pid文件，可以通过pidfile指定</td>
</tr>
<tr>
  <td align="left">`port 6379`</td>
  <td align="left">指定Redis监听端口，默认端口为6379，作者在自己的一篇博文中解释了为什么选用6379作为默认端口，因为6379在手机按键上MERZ对应的号码，而MERZ取自意大利歌女Alessia Merz的名字</td>
</tr>
<tr>
  <td align="left">`bind 127.0.0.1`</td>
  <td align="left">绑定的主机地址</td>
</tr>
<tr>
  <td align="left">`timeout 300`</td>
  <td align="left">当客户端闲置多长时间后关闭连接，如果指定为0，表示关闭该功能</td>
</tr>
<tr>
  <td align="left">`loglevel verbose`</td>
  <td align="left">指定日志记录级别，Redis总共支持四个级别：debug、verbose、notice、warning，默认为verbose</td>
</tr>
<tr>
  <td align="left">`logfile stdout`</td>
  <td align="left">日志记录方式，默认为标准输出，如果配置Redis为守护进程方式运行，而这里又配置为日志记录方式为标准输出，则日志将会发送给/dev/null</td>
</tr>
<tr>
  <td align="left">`databases 16`</td>
  <td align="left">设置数据库的数量，默认数据库为0，可以使用SELECT <dbid>命令在连接上指定数据库id</td>
</tr>
<tr>
  <td align="left">`save &lt;seconds&gt; &lt;changes&gt;`</td>
  <td align="left">分别表示900秒（15分钟）内有1个更改，300秒（5分钟）内有10个更改以及60秒内有10000个更改。</td>
</tr>
<tr>
  <td align="left">`rdbcompression yes`</td>
  <td align="left">指定存储至本地数据库时是否压缩数据，默认为yes，Redis采用LZF压缩，如果为了节省CPU时间，可以关闭该选项，但会导致数据库文件变的巨大</td>
</tr>
<tr>
  <td align="left">`dbfilename dump.rdb`</td>
  <td align="left">指定本地数据库文件名，默认值为dump.rdb</td>
</tr>
<tr>
  <td align="left">`dir ./`</td>
  <td align="left">指定本地数据库存放目录</td>
</tr>
<tr>
  <td align="left">`slaveof &lt;masterip&gt; &lt;masterport&gt;`</td>
  <td align="left">设置当本机为slav服务时，设置master服务的IP地址及端口，在Redis启动时，它会自动从master进行数据同步</td>
</tr>
<tr>
  <td align="left">`masterauth &lt;master-password&gt;`</td>
  <td align="left">当master服务设置了密码保护时，slav服务连接master的密码</td>
</tr>
<tr>
  <td align="left">`requirepass foobared`</td>
  <td align="left">设置Redis连接密码，如果配置了连接密码，客户端在连接Redis时需要通过AUTH <password>命令提供密码，默认关闭</td>
</tr>
<tr>
  <td align="left">`maxclients 128`</td>
  <td align="left">设置同一时间最大客户端连接数，默认无限制，Redis可以同时打开的客户端连接数为Redis进程可以打开的最大文件描述符数，如果设置 maxclients 0，表示不作限制。当客户端连接数到达限制时，Redis会关闭新的连接并向客户端返回max number of clients reached错误信息</td>
</tr>
<tr>
  <td align="left">`maxmemory &lt;bytes&gt;`</td>
  <td align="left">指定Redis最大内存限制，Redis在启动时会把数据加载到内存中，达到最大内存后，Redis会先尝试清除已到期或即将到期的Key，当此方法处理 后，仍然到达最大内存设置，将无法再进行写入操作，但仍然可以进行读取操作。Redis新的vm机制，会把Key存放内存，Value会存放在swap区</td>
</tr>
<tr>
  <td align="left">`appendonly no`</td>
  <td align="left">指定是否在每次更新操作后进行日志记录，Redis在默认情况下是异步的把数据写入磁盘，如果不开启，可能会在断电时导致一段时间内的数据丢失。因为 redis本身同步数据文件是按上面save条件来同步的，所以有的数据会在一段时间内只存在于内存中。默认为no</td>
</tr>
<tr>
  <td align="left">`appendfilename appendonly.aof`</td>
  <td align="left">指定更新日志文件名，默认为appendonly.aof</td>
</tr>
<tr>
  <td align="left">`appendfsync everysec`</td>
  <td align="left">指定更新日志条件，共有3个可选值：`no`表示等操作系统进行数据缓存同步到磁盘（快），`always`表示每次更新操作后手动调用fsync()将数据写到磁盘（慢，安全），`everysec`表示每秒同步一次（折衷，默认值）</td>
</tr>
<tr>
  <td align="left">`vm-enabled no`</td>
  <td align="left">指定是否启用虚拟内存机制，默认值为no，简单的介绍一下，VM机制将数据分页存放，由Redis将访问量较少的页即冷数据swap到磁盘上，访问多的页面由磁盘自动换出到内存中（在后面的文章我会仔细分析Redis的VM机制）</td>
</tr>
<tr>
  <td align="left">`vm-swap-file /tmp/redis.swap`</td>
  <td align="left">虚拟内存文件路径，默认值为/tmp/redis.swap，不可多个Redis实例共享</td>
</tr>
<tr>
  <td align="left">`vm-max-memory 0`</td>
  <td align="left">将所有大于vm-max-memory的数据存入虚拟内存,无论vm-max-memory设置多小,所有索引数据都是内存存储的(Redis的索引数据 就是keys),也就是说,当vm-max-memory设置为0的时候,其实是所有value都存在于磁盘。默认值为0</td>
</tr>
<tr>
  <td align="left">`vm-page-size 32`</td>
  <td align="left">设置swap文件中的page数量，由于页表（一种表示页面空闲或使用的bitmap）是在放在内存中的，，在磁盘上每8个pages将消耗1byte的内存。</td>
</tr>
<tr>
  <td align="left">`vm-pages 134217728`</td>
  <td align="left">设置swap文件中的page数量，由于页表（一种表示页面空闲或使用的bitmap）是在放在内存中的，，在磁盘上每8个pages将消耗1byte的内存。</td>
</tr>
<tr>
  <td align="left">`vm-max-threads 4`</td>
  <td align="left">设置访问swap文件的线程数,最好不要超过机器的核数,如果设置为0,那么所有对swap文件的操作都是串行的，可能会造成比较长时间的延迟。默认值为4</td>
</tr>
<tr>
  <td align="left">`glueoutputbuf yes`</td>
  <td align="left">设置在向客户端应答时，是否把较小的包合并为一个包发送，默认为开启</td>
</tr>
<tr>
  <td align="left">`hash-max-zipmap-entries 64`</td>
  <td align="left">指定在超过一定的数量超过某一临界值时，采用一种特殊的哈希算法</td>
</tr>
<tr>
  <td align="left">`hash-max-zipmap-value 512`</td>
  <td align="left">指定在最大的元素超过某一临界值时，采用一种特殊的哈希算法</td>
</tr>
<tr>
  <td align="left">`activerehashing yes`</td>
  <td align="left">指定是否激活重置哈希，默认为开启</td>
</tr>
<tr>
  <td align="left">`include /path/to/local.conf`</td>
  <td align="left">指定包含其它的配置文件，可以在同一主机上多个Redis实例之间使用同一份配置文件，而同时各个实例又拥有自己的特定配置文件</td>
</tr>
</tbody>
</table>

## 对应配置文件中文版

    # Redis示例配置文件

    # 注意单位问题：当需要设置内存大小的时候，可以使用类似1k、5GB、4M这样的常见格式：
    #
    # 1k =&gt; 1000 bytes
    # 1kb =&gt; 1024 bytes
    # 1m =&gt; 1000000 bytes
    # 1mb =&gt; 1024*1024 bytes
    # 1g =&gt; 1000000000 bytes
    # 1gb =&gt; 1024*1024*1024 bytes
    #
    # 单位是大小写不敏感的，所以1GB 1Gb 1gB的写法都是完全一样的。

    # Redis默认是不作为守护进程来运行的。你可以把这个设置为"yes"让它作为守护进程来运行。
    # 注意，当作为守护进程的时候，Redis会把进程ID写到 /var/run/redis.pid
    daemonize no

    # 当以守护进程方式运行的时候，Redis会把进程ID默认写到 /var/run/redis.pid。你可以在这里修改路径。
    pidfile /var/run/redis.pid

    # 接受连接的特定端口，默认是6379。
    # 如果端口设置为0，Redis就不会监听TCP套接字。
    port 6379

    # 如果你想的话，你可以绑定单一接口；如果这里没单独设置，那么所有接口的连接都会被监听。
    #
    # bind 127.0.0.1

    # 指定用来监听连接的unxi套接字的路径。这个没有默认值，所以如果你不指定的话，Redis就不会通过unix套接字来监听。
    #
    # unixsocket /tmp/redis.sock
    # unixsocketperm 755

    #一个客户端空闲多少秒后关闭连接。(0代表禁用，永不关闭)
    timeout 0

    # 设置服务器调试等级。
    # 可能值：
    # debug （很多信息，对开发/测试有用）
    # verbose （很多精简的有用信息，但是不像debug等级那么多）
    # notice （适量的信息，基本上是你生产环境中需要的程度）
    # warning （只有很重要/严重的信息会记录下来）
    loglevel verbose

    # 指明日志文件名。也可以使用"stdout"来强制让Redis把日志信息写到标准输出上。
    # 注意：如果Redis以守护进程方式运行，而你设置日志显示到标准输出的话，那么日志会发送到 /dev/null
    logfile stdout

    # 要使用系统日志记录器很简单，只要设置 "syslog-enabled" 为 "yes" 就可以了。
    # 然后根据需要设置其他一些syslog参数就可以了。
    # syslog-enabled no

    # 指明syslog身份
    # syslog-ident redis

    # 指明syslog的设备。必须是一个用户或者是 LOCAL0 ~ LOCAL7 之一。
    # syslog-facility local0

    # 设置数据库个数。默认数据库是 DB 0，你可以通过SELECT &lt;dbid&gt; WHERE dbid（0～'databases' - 1）来为每个连接使用不同的数据库。
    databases 16

    ################################ 快照 #################################

    #
    # 把数据库存到磁盘上:
    #
    # save &lt;seconds&gt; &lt;changes&gt;
    # 
    # 会在指定秒数和数据变化次数之后把数据库写到磁盘上。
    #
    # 下面的例子将会进行把数据写入磁盘的操作:
    # 900秒（15分钟）之后，且至少1次变更
    # 300秒（5分钟）之后，且至少10次变更
    # 60秒之后，且至少10000次变更
    #
    # 注意：你要想不写磁盘的话就把所有 "save" 设置注释掉就行了。

    save 900 1
    save 300 10
    save 60 10000

    # 当导出到 .rdb 数据库时是否用LZF压缩字符串对象。
    # 默认设置为 "yes"，所以几乎总是生效的。
    # 如果你想节省CPU的话你可以把这个设置为 "no"，但是如果你有可压缩的key的话，那数据文件就会更大了。
    rdbcompression yes

    # 数据库的文件名
    dbfilename dump.rdb

    # 工作目录
    #
    # 数据库会写到这个目录下，文件名就是上面的 "dbfilename" 的值。
    # 
    # 累加文件也放这里。
    # 
    # 注意你这里指定的必须是目录，不是文件名。
    dir ./

    ################################# 同步 #################################

    #
    # 主从同步。通过 slaveof 配置来实现Redis实例的备份。
    # 注意，这里是本地从远端复制数据。也就是说，本地可以有不同的数据库文件、绑定不同的IP、监听不同的端口。
    #
    # slaveof &lt;masterip&gt; &lt;masterport&gt;

    # 如果master设置了密码（通过下面的 "requirepass" 选项来配置），那么slave在开始同步之前必须进行身份验证，否则它的同步请求会被拒绝。
    #
    # masterauth &lt;master-password&gt;

    # 当一个slave失去和master的连接，或者同步正在进行中，slave的行为有两种可能：
    #
    # 1) 如果 slave-serve-stale-data 设置为 "yes" (默认值)，slave会继续响应客户端请求，可能是正常数据，也可能是还没获得值的空数据。
    # 2) 如果 slave-serve-stale-data 设置为 "no"，slave会回复"正在从master同步（SYNC with master in progress）"来处理各种请求，除了 INFO 和 SLAVEOF 命令。
    #
    slave-serve-stale-data yes

    # slave根据指定的时间间隔向服务器发送ping请求。
    # 时间间隔可以通过 repl_ping_slave_period 来设置。
    # 默认10秒。
    #
    # repl-ping-slave-period 10

    # 下面的选项设置了大块数据I/O、向master请求数据和ping响应的过期时间。
    # 默认值60秒。
    #
    # 一个很重要的事情是：确保这个值比 repl-ping-slave-period 大，否则master和slave之间的传输过期时间比预想的要短。
    #
    # repl-timeout 60

    ################################## 安全 ###################################

    # 要求客户端在处理任何命令时都要验证身份和密码。
    # 这在你信不过来访者时很有用。
    #
    # 为了向后兼容的话，这段应该注释掉。而且大多数人不需要身份验证（例如：它们运行在自己的服务器上。）
    # 
    # 警告：因为Redis太快了，所以居心不良的人可以每秒尝试150k的密码来试图破解密码。
    # 这意味着你需要一个高强度的密码，否则破解太容易了。
    #
    # requirepass foobared

    # 命令重命名
    #
    # 在共享环境下，可以为危险命令改变名字。比如，你可以为 CONFIG 改个其他不太容易猜到的名字，这样你自己仍然可以使用，而别人却没法做坏事了。
    #
    # 例如:
    #
    # rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c52
    #
    # 甚至也可以通过给命令赋值一个空字符串来完全禁用这条命令：
    #
    # rename-command CONFIG ""

    ################################### 限制 ####################################

    #
    # 设置最多同时连接客户端数量。
    # 默认没有限制，这个关系到Redis进程能够打开的文件描述符数量。
    # 特殊值"0"表示没有限制。
    # 一旦达到这个限制，Redis会关闭所有新连接并发送错误"达到最大用户数上限（max number of clients reached）"
    #
    # maxclients 128

    # 不要用比设置的上限更多的内存。一旦内存使用达到上限，Redis会根据选定的回收策略（参见：maxmemmory-policy）删除key。
    #
    # 如果因为删除策略问题Redis无法删除key，或者策略设置为 "noeviction"，Redis会回复需要更多内存的错误信息给命令。
    # 例如，SET,LPUSH等等。但是会继续合理响应只读命令，比如：GET。
    #
    # 在使用Redis作为LRU缓存，或者为实例设置了硬性内存限制的时候（使用 "noeviction" 策略）的时候，这个选项还是满有用的。
    #
    # 警告：当一堆slave连上达到内存上限的实例的时候，响应slave需要的输出缓存所需内存不计算在使用内存当中。
    # 这样当请求一个删除掉的key的时候就不会触发网络问题／重新同步的事件，然后slave就会收到一堆删除指令，直到数据库空了为止。
    #
    # 简而言之，如果你有slave连上一个master的话，那建议你把master内存限制设小点儿，确保有足够的系统内存用作输出缓存。
    # （如果策略设置为"noeviction"的话就不无所谓了）
    #
    # maxmemory &lt;bytes&gt;

    # 内存策略：如果达到内存限制了，Redis如何删除key。你可以在下面五个策略里面选：
    # 
    # volatile-lru -&gt; 根据LRU算法生成的过期时间来删除。
    # allkeys-lru -&gt; 根据LRU算法删除任何key。
    # volatile-random -&gt; 根据过期设置来随机删除key。
    # allkeys-&gt;random -&gt; 无差别随机删。
    # volatile-ttl -&gt; 根据最近过期时间来删除（辅以TTL）
    # noeviction -&gt; 谁也不删，直接在写操作时返回错误。
    # 
    # 注意：对所有策略来说，如果Redis找不到合适的可以删除的key都会在写操作时返回一个错误。
    #
    # 这里涉及的命令：set setnx setex append
    # incr decr rpush lpush rpushx lpushx linsert lset rpoplpush sadd
    # sinter sinterstore sunion sunionstore sdiff sdiffstore zadd zincrby
    # zunionstore zinterstore hset hsetnx hmset hincrby incrby decrby
    # getset mset msetnx exec sort
    #
    # 默认值如下：
    #
    # maxmemory-policy volatile-lru

    # LRU和最小TTL算法的实现都不是很精确，但是很接近（为了省内存），所以你可以用样例做测试。
    # 例如：默认Redis会检查三个key然后取最旧的那个，你可以通过下面的配置项来设置样本的个数。
    #
    # maxmemory-samples 3

    ############################## 纯累加模式 ###############################

    # 默认情况下，Redis是异步的把数据导出到磁盘上。这种情况下，当Redis挂掉的时候，最新的数据就丢了。
    # 如果不希望丢掉任何一条数据的话就该用纯累加模式：一旦开启这个模式，Redis会把每次写入的数据在接收后都写入 appendonly.aof 文件。
    # 每次启动时Redis都会把这个文件的数据读入内存里。
    #
    # 注意，异步导出的数据库文件和纯累加文件可以并存（你得把上面所有"save"设置都注释掉，关掉导出机制）。
    # 如果纯累加模式开启了，那么Redis会在启动时载入日志文件而忽略导出的 dump.rdb 文件。
    #
    # 重要：查看 BGREWRITEAOF 来了解当累加日志文件太大了之后，怎么在后台重新处理这个日志文件。

    appendonly no

    # 纯累加文件名字（默认："appendonly.aof"）
    # appendfilename appendonly.aof

    # fsync() 请求操作系统马上把数据写到磁盘上，不要再等了。
    # 有些操作系统会真的把数据马上刷到磁盘上；有些则要磨蹭一下，但是会尽快去做。
    #
    # Redis支持三种不同的模式：
    #
    # no：不要立刻刷，只有在操作系统需要刷的时候再刷。比较快。
    # always：每次写操作都立刻写入到aof文件。慢，但是最安全。
    # everysec：每秒写一次。折衷方案。
    #
    # 默认的 "everysec" 通常来说能在速度和数据安全性之间取得比较好的平衡。
    # 如果你真的理解了这个意味着什么，那么设置"no"可以获得更好的性能表现（如果丢数据的话，则只能拿到一个不是很新的快照）；
    # 或者相反的，你选择 "always" 来牺牲速度确保数据安全、完整。
    #
    # 如果拿不准，就用 "everysec"

    # appendfsync always
    appendfsync everysec
    # appendfsync no

    # 如果AOF的同步策略设置成 "always" 或者 "everysec"，那么后台的存储进程（后台存储或写入AOF日志）会产生很多磁盘I/O开销。
    # 某些Linux的配置下会使Redis因为 fsync() 而阻塞很久。
    # 注意，目前对这个情况还没有完美修正，甚至不同线程的 fsync() 会阻塞我们的 write(2) 请求。
    #
    # 为了缓解这个问题，可以用下面这个选项。它可以在 BGSAVE 或 BGREWRITEAOF 处理时阻止 fsync()。
    # 
    # 这就意味着如果有子进程在进行保存操作，那么Redis就处于"不可同步"的状态。
    # 这实际上是说，在最差的情况下可能会丢掉30秒钟的日志数据。（默认Linux设定）
    # 
    # 如果你有延迟的问题那就把这个设为 "yes"，否则就保持 "no"，这是保存持久数据的最安全的方式。
    no-appendfsync-on-rewrite no

    # 自动重写AOF文件
    #
    # 如果AOF日志文件大到指定百分比，Redis能够通过 BGREWRITEAOF 自动重写AOF日志文件。
    # 
    # 工作原理：Redis记住上次重写时AOF日志的大小（或者重启后没有写操作的话，那就直接用此时的AOF文件），
    # 基准尺寸和当前尺寸做比较。如果当前尺寸超过指定比例，就会触发重写操作。
    #
    # 你还需要指定被重写日志的最小尺寸，这样避免了达到约定百分比但尺寸仍然很小的情况还要重写。
    #
    # 指定百分比为0会禁用AOF自动重写特性。

    auto-aof-rewrite-percentage 100
    auto-aof-rewrite-min-size 64mb

    ################################## 慢查询日志 ###################################

    # Redis慢查询日志可以记录超过指定时间的查询。运行时间不包括各种I/O时间。
    # 例如：连接客户端，发送响应数据等。只计算命令运行的实际时间（这是唯一一种命令运行线程阻塞而无法同时为其他请求服务的场景）
    # 
    # 你可以为慢查询日志配置两个参数：一个是超标时间，单位为微妙，记录超过个时间的命令。
    # 另一个是慢查询日志长度。当一个新的命令被写进日志的时候，最老的那个记录会被删掉。
    #
    # 下面的时间单位是微秒，所以1000000就是1秒。注意，负数时间会禁用慢查询日志，而0则会强制记录所有命令。
    slowlog-log-slower-than 10000

    # 这个长度没有限制。只要有足够的内存就行。你可以通过 SLOWLOG RESET 来释放内存。（译者注：日志居然是在内存里的Orz）
    slowlog-max-len 128

    ################################ 虚拟内存 ###############################

    ### 警告！虚拟内存在Redis 2.4是反对的。
    ### 非常不鼓励使用虚拟内存！！

    # 虚拟内存可以使Redis在内存不够的情况下仍然可以将所有数据序列保存在内存里。
    # 为了做到这一点，高频key会调到内存里，而低频key会转到交换文件里，就像操作系统使用内存页一样。
    #
    # 要使用虚拟内存，只要把 "vm-enabled" 设置为 "yes"，并根据需要设置下面三个虚拟内存参数就可以了。

    vm-enabled no
    # vm-enabled yes

    # 这是交换文件的路径。估计你猜到了，交换文件不能在多个Redis实例之间共享，所以确保每个Redis实例使用一个独立交换文件。
    #
    # 最好的保存交换文件（被随机访问）的介质是固态硬盘（SSD）。
    #
    # *** 警告 *** 如果你使用共享主机，那么默认的交换文件放到 /tmp 下是不安全的。
    # 创建一个Redis用户可写的目录，并配置Redis在这里创建交换文件。
    vm-swap-file /tmp/redis.swap

    # "vm-max-memory" 配置虚拟内存可用的最大内存容量。
    # 如果交换文件还有空间的话，所有超标部分都会放到交换文件里。
    #
    # "vm-max-memory" 设置为0表示系统会用掉所有可用内存。
    # 这默认值不咋地，只是把你能用的内存全用掉了，留点余量会更好。
    # 例如，设置为剩余内存的60%-80%。
    vm-max-memory 0

    # Redis交换文件是分成多个数据页的。
    # 一个可存储对象可以被保存在多个连续页里，但是一个数据页无法被多个对象共享。
    # 所以，如果你的数据页太大，那么小对象就会浪费掉很多空间。
    # 如果数据页太小，那用于存储的交换空间就会更少（假定你设置相同的数据页数量）
    #
    # 如果你使用很多小对象，建议分页尺寸为64或32个字节。
    # 如果你使用很多大对象，那就用大一些的尺寸。
    # 如果不确定，那就用默认值 :)
    vm-page-size 32

    # 交换文件里数据页总数。
    # 根据内存中分页表（已用/未用的数据页分布情况），磁盘上每8个数据页会消耗内存里1个字节。
    #
    # 交换区容量 = vm-page-size * vm-pages
    #
    # 根据默认的32字节的数据页尺寸和134217728的数据页数来算，Redis的数据页文件会占4GB，而内存里的分页表会消耗16MB内存。
    #
    # 为你的应验程序设置最小且够用的数字比较好，下面这个默认值在大多数情况下都是偏大的。
    vm-pages 134217728

    # 同时可运行的虚拟内存I/O线程数。
    # 这些线程可以完成从交换文件进行数据读写的操作，也可以处理数据在内存与磁盘间的交互和编码/解码处理。
    # 多一些线程可以一定程度上提高处理效率，虽然I/O操作本身依赖于物理设备的限制，不会因为更多的线程而提高单次读写操作的效率。
    #
    # 特殊值0会关闭线程级I/O，并会开启阻塞虚拟内存机制。
    vm-max-threads 4

    ############################### 高级配置 ###############################

    # 当有大量数据时，适合用哈希编码（需要更多的内存），元素数量上限不能超过给定限制。
    # 你可以通过下面的选项来设定这些限制：
    hash-max-zipmap-entries 512
    hash-max-zipmap-value 64

    # 与哈希相类似，数据元素较少的情况下，可以用另一种方式来编码从而节省大量空间。
    # 这种方式只有在符合下面限制的时候才可以用：
    list-max-ziplist-entries 512
    list-max-ziplist-value 64

    # 还有这样一种特殊编码的情况：数据全是64位无符号整型数字构成的字符串。
    # 下面这个配置项就是用来限制这种情况下使用这种编码的最大上限的。
    set-max-intset-entries 512

    # 与第一、第二种情况相似，有序序列也可以用一种特别的编码方式来处理，可节省大量空间。
    # 这种编码只适合长度和元素都符合下面限制的有序序列：
    zset-max-ziplist-entries 128
    zset-max-ziplist-value 64

    # 哈希刷新，每100个CPU毫秒会拿出1个毫秒来刷新Redis的主哈希表（顶级键值映射表）。
    # redis所用的哈希表实现（见dict.c）采用延迟哈希刷新机制：你对一个哈希表操作越多，哈希刷新操作就越频繁；
    # 反之，如果服务器非常不活跃那么也就是用点内存保存哈希表而已。
    # 
    # 默认是每秒钟进行10次哈希表刷新，用来刷新字典，然后尽快释放内存。
    #
    # 建议：
    # 如果你对延迟比较在意的话就用 "activerehashing no"，每个请求延迟2毫秒不太好嘛。
    # 如果你不太在意延迟而希望尽快释放内存的话就设置 "activerehashing yes"。
    activerehashing yes

    ################################## 包含 ###################################

    # 包含一个或多个其他配置文件。
    # 这在你有标准配置模板但是每个redis服务器又需要个性设置的时候很有用。
    # 包含文件特性允许你引人其他配置文件，所以好好利用吧。
    #
    # include /path/to/local.conf
    # include /path/to/other.conf
    
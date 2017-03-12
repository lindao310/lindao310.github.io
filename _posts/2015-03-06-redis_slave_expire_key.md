---
layout: post
title:  "redis 从库 过期key问题"
date:   2015-03-26 15:14:54
categories: Redis
tags: Redis 
---

* content
{:toc}

Redis Slave ttl 为0的问题  http://get.ftqq.com/7297.get

### 问题描述
有个key，设置为5分钟过期； master写， slave读； 然后发现slave获取这个key始终存在， 从库上ttl这个key，返回0；

### redis过期删除策略

- 主动方式， 过期的key被访问时， redis会判断key是否已经过期，过期则删除
- 被动方式， redis每秒10次随机抽取20个带过期设置的key，检测是否过期，过期则删除； 如果过期key占比超过25%，立刻再抽取20个key进行检查
- 所以了只能保证过期key的存在占比在25%以下。

### 从库过期key处理

while the slaves connected to a master will not expire keys independently (but will wait for the DEL coming from the master), they'll still take the full state of the expires existing in the dataset, so when a slave is elected to a master it will be able to expire the keys independently, fully acting as a master.

（从库们不会独立的进行过期删除操作，必须等待主库判断过期删除时候通知过来的DEL来删除，但是key的ttl还是会记录的，然后了当一个从库被选举成主库的时候，就可以像上面那样说的进行删除了）

### 结论
那么问题也就很清楚了， 如果你redis的架构是一主一从，然后你在主上写带过期key， 在从库上读这个key， 那么除非你的过期key数量很少，通过master的被动方式能很快清理掉，不然肯定会出现bug的。 


### 后续
我们之前完全没有发现这个问题， 采用读写分离的方式使用带过期设置的key； 然后我看6下这个节点，过期key占95%, 反正总共就640万个key， 我决定用scan把key全部导出来研究研究；

处理前

```

used_memory:3728446872
used_memory_human:3.47G
used_memory_rss:4219183104
used_memory_peak:4027994552
used_memory_peak_human:3.75G
used_memory_lua:35840

db0:keys=6388689,expires=6078233,avg_ttl=1263844535

```

scan 后

```

used_memory:3620033736
used_memory_human:3.37G
used_memory_rss:4214988800
used_memory_peak:4027994552
used_memory_peak_human:3.75G
used_memory_lua:35840

db0:keys=6143078,expires=5832622,avg_ttl=3708725069

```
显然，由于scan会试图访问key， 触发了过期key的主动删除，清理掉24万个key; 然后没发现别的异常

### 误写入从库的过期key如何处理
发现这个情况后我检查了下其余的redis，发现有发生过把过期key直接写到从库的情况...
从库比主库多了差不多1000万个key

```

Slave  Keyspace
db0:keys=29586731,expires=15048434,avg_ttl=0

Master Keyspace
db0:keys=19746281,expires=5207805,avg_ttl=48908945

```
这个怎么弄了

- 一开始想的是scan主库，触发过期key的删除，主库发现key过期会同步给从库删除这个key，但实际是不行的，因为你直接写入到从库，主库上根本没有这个key

- 然后了我就打算把从库的key整个scan一边，如果是ttl是0，再删掉， 结果发现找不到ttl为0的key，万分不解

- 我决定把这近3000万key全scan导出来看看，导完发现只有大概1800万，无语了。。。又跑去看了下rdb，发现rdb里面也没有。。。所以只能下结论，从库里已经过了期的key，存在，keyspace可以看到， 内存也占着，但是rdb文件里是没有的，scan命令也是检索不出来的

- 最后方案：让dba把从库域名切换到master的节点， down掉从库重新从master同步数据， 同步完成，把从库域名切回到slave所在的节点


### 总结

- 过期key的读写都一定要在主库上。

- 如果读写分离的话，过期key和不过期的key最好不要混用在同一个节点，尤其是在数量比较大的时候




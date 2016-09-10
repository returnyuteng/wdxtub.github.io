title: 云计算 Twitter 语料分析 8 处理复杂读请求
categories:
- Technique
toc: true
date: 2016-03-17 12:02:40
tags:
- CMU
- 云计算
- 服务
---

从这节课开始，就进入了第二阶段，熟悉了基本的架构和操作之后，需要处理更加接近现实的稍微复杂一些的请求（不能像之前那么简单粗暴）。

<!-- more -->

---

## 学习目标

1. 设计、开发、部署一个全功能高可用的 web 服务
2. 实现 ETL（约 1TB 数据），并导入到数据库中
3. 评估存储架构堆性能的影响
4. 优化 MySQL 和 HBase 来提高吞吐量
5. 探索和使用不同的方法和工具来提高 web 服务的性能
6. 拓展服务器使其可以处理不同的请求
7. 开发后端服务器处理范围请求

## 任务目标

前面我们创建了基本的 web 服务用于处理 HTTP GET 请求。

Q1 中我们学会了如何处理简单的并发请求。Q2 中我们结合后端（MySQL 和 HBase） 完成了简单的键值对存储和查询工作。

Q3 中请求不会这么简单，需要对 MySQL 和 HBase 有深入了解，并利用它们的特性来提高性能。

+ 只能使用 M 系列不大于 large 型号的机器
+ 打标签 `15619project:phase2`, `15619backend:hbase`, `15619backend:mysql`
+ 预算是 `$0.85/h`，按照标注计费
+ 队伍的预算是 `$60`

## Query 3 (Word Count and Range Query)

目标 6000 rps

给定一个范围的 UserID，一个范围的日期和 3 个单词，需要返回给定用户在给定的日期范围内的 tweet 中每个单词出现的次数。具体的方法是：

+ 只保存 `lang` 部分为 `en` 的 tweet，如果没有这个域，可以直接扔掉，在 `text` 部分的内容才会被计算在内。
+ 内容中会有很多短链接，在切分单词之前也需要过滤掉，对应的这正表达式为：
    + Python: `(https?|ftp):\/\/[^\s/$.?#][^\s]*`
    + Java: `(https?|ftp):\\/\\/[^\\s/$.?#][^\\s]*`
+ 单词(`[a-zA-Z0-9]+`) 间由非字母字符(`[^a-zA-Z0-9]+`)间隔
+ 去掉 stopwords（可以在 ETL 中完成）
+ 大小写不敏感
+ 和 Q2 一样需要移除重复和格式不对的 tweet

这里提供了一个参考文件 `s3://cmucc-datasets/twitter/ref/q3-part-00000-reference` 对应于 `s3://cmucc-datasets/twitter/s16/part-00000` 的 ETL 结果，逐行对应，删除了重复 tweet 以及不包含有效单词的 tweet，格式如下，以 `\t` 分隔

1. tweet id.
2. user id.
3. tweet date.
4. valid_word:word_count 小写，按照升序排列

### 具体请求

范围是包括当天的，例如给出的范围是从 2013-10-10 到 2013-11-11，那么这两天的 tweet 也需要计算在内

> 记录下 userid 和 dates 的范围，有用！

假设原始数据是：

```
tweet1: {‘time’ = ‘2012-12-25’, ‘userid’ = 1, ‘text’ = “RT @MyBaeBeLike: When your bae says we're just friends... http://t.co/iZbYagMN6e”

tweet2: {‘time’ = ‘2012-12-26’, ‘userid’ = 2, ‘text’ = “Hello, my friend!!”}

tweet3: {‘time’ = ‘2012-12-27’, ‘userid’ = 3, ‘text’ = “This is awesome!”}
```

> 请求

`GET/q3?start_date=yyyy-mm-dd&end_date=yyyy-mm-dd&start_userid=uid&end_userid=uid&words=w1,w2,w3`

具体例子

`GET/q3?start_date=2014-04-01&end_date=2014-05-28&start_userid=51538630&end_userid=51539182&words=u,petition,loving`

> 响应

```
TEAMID,TEAM_AWS_ACCOUNT_ID\n
w1:count1\n
w2:count2\n
w3:count3\n
```

具体例子

```
Team,1234-5678-1234
u:7\n
petition:2\n
loving:5\n
```

> Reference Server

可以利用提供的 [Reference Server](https://Q1-1848733628.us-east-1.elb.amazonaws.com) 来检测结果的正确性（以及编码问题）

## 工作日志

+ Get 比 Scan 要快
+ 用 Cloudera 自己部署 HBase
+ 注意集群中的负载均衡
+ Live test 中考虑使用 ELB

undertow 的路径处理

```java
HttpHandler q1 = new Q1();
HttpHandler q2 = new Q2();
HttpHandler q3 = new Q3();

PathHandler ph = Handlers.path(Handlers.redirect(ROOTPATH))
                        .addPrefixPath("/q1", q1)
                        .addPrefixPath("/q2", q2)
                        .addPrefixPath("/q3", q3);
server = Undertow.builder().setIoThreads(64)
                .addHttpListener(80, "0.0.0.0").setHandler(ph).build();
```

HBase 创建表格及导入数据

```bash
hadoop fs -mkdir /housailei
hadoop fs -mkdir /housailei/csv
hbase(main):001:0> create 'q3db', {NAME => 'data', DATA_BLOCK_ENCODING => 'FAST_DIFF', COMPRESSION => 'SNAPPY', BLOCKSIZE => '131072', IN_MEMORY => 'true', BLOCKCACHE => 'true', BLOOMFILTER => 'ROWCOL'}

cd /mnt
wget http://s3.amazonaws.com/housailei15619/Query3-Result-All-Postprocessed/q3-result-data
scp -i ../group.pem ./washq3.py hadoop@ec2-54-172-101-254.compute-1.amazonaws.com:/mnt/

aws s3 cp ./q3-hbase-data s3://housailei15619/Query3-Result-All-Postprocessed/

hadoop fs -put ./q3-hbase-data /housailei/csv/


# 这里是处理完全部数据导入的时候的命令
# rowkey 直接是 userid+hashtag：
# 以 \t 为分隔符的话就不需要设定，这是默认的
hbase org.apache.hadoop.hbase.mapreduce.ImportTsv -Dimporttsv.bulk.output=/hfile_groupt3 -Dimporttsv.columns=HBASE_ROW_KEY,data:t,data:w q3db /housailei/csv/q3-hbase-data

hbase org.apache.hadoop.hbase.mapreduce.LoadIncrementalHFiles /hfile_groupt3 q3db
```

测试网址

`http://q1-1848733628.us-east-1.elb.amazonaws.com/q3?start_date=2014-04-01&end_date=2014-05-28&start_userid=51538630&end_userid=51539182&words=u,petition,loving`

其他一些思路

+ HBase 考虑使用多线程 + HTable 来手动处理大量请求（HConnection 感觉有点不靠谱？）
+ 使用 ELB 一定需要热机，然后注意 health check 参数的设定
+ MySQL 可能需要多台 front end，HBase 一般 1 台就可以
+ 用 userid 的最后两位 + userid 做 rowkey，保证 shuffle 的效率，可以有效提高性能
+ HBase：设定 block cache，调整 memstore 的上下限，修改 max hfile sieze（修改 hbase-site.xml）
+ HBase：presplitting table 以及 blook filter 'ROW'（hbase shell 创建 table 的时候）`bloom filter=>’rowcol’`
+ HBase：使用 lzo 压缩（先安装 lzo，然后在创建表的时候设定压缩方法）
+ MySQL：设定 cache 大小（MySQL 的配置文件）
+ HBase：会发现数据都集中在一个 region，需要进行拆分（或者把每块的最大容量缩小）
+ HBase：疯狂 split 然后让 region 平均分配效果会比较好

### 常用命令

开启 HBase 并进行配置，注意 SubnetId 要和前端一致，KeyName 需要是 pem 的名字

```bash
aws emr create-cluster --name "HBase_p21" --ami-version 3.11.0 \
--applications Name=HBase \
--tags "15619project=phase2" \
--use-default-roles --ec2-attributes KeyName=group,SubnetId=subnet-28a0345e \
--instance-groups InstanceGroupType=MASTER,InstanceType=m4.large,InstanceCount=1,BidPrice=0.05 \
InstanceGroupType=CORE,BidPrice=0.05,InstanceType=m4.large,InstanceCount=3 \
--bootstrap-action Path=s3://us-west-1.elasticmapreduce/bootstrap-actions/configure-hadoop,Args=["-m,mapred.tasktracker.map.tasks.maximum=2,-m,mapred.tasktracker.reduce.tasks.maximum=1,-h,dfs.namenode.handler.count=200,-h,dfs.datanode.handler.count=200,-h,dfs.replication.interval=999999"] \
--bootstrap-action Path=s3://elasticmapreduce/bootstrap-actions/configure-daemons,Args["--namenode-heap-size=4096,--namenode-opts=-XX:GCTimeRatio=19,--datanode-heap-size=4096,--jobtracker-heap-size=2048,--tasktracker-heap-size=256"]\
--bootstrap-action Path=s3://us-west-1.elasticmapreduce/bootstrap-actions/configure-hbase,Args["zookeeper.session.timeout=60000,-s,hbase.regionserver.handler.count=200,-s,hbase.hregion.max.filesize=16252928,-s,hbase.hregion.memstore.flush.size=16252928,-s,hbase.hfile.block.cache.size=0.5"] \
--bootstrap-action Path=s3://us-west-1.elasticmapreduce/bootstrap-actions/configure-hbase-daemons,Args=["--hbase-zookeeper-opts=-Xmx1024m -XX:GCTimeRatio=19,--hbase-master-opts=-Xmx4096m,--hbase-regionserver-opts=-Xmx4096m"]
```

HBase 创建表格时候的命令

```
Configurable block size
hbase(main):002:0> create 'mytable',{NAME => 'colfam1', BLOCKSIZE => '65536'}

Block cache
workloads don’t benefit from putting data into a read cache--for instance, if a certain table or column family in a table is only accessed for sequential scans or isn’t
accessed a lot and you don’t care if Gets or Scans take a little longer.By default, the block cache is enabled. You can disable it at the time of table creation
or by altering the table:
hbase(main):002:0> create 'mytable',{NAME => 'colfam1', BLOCKCACHE => 'false’}

Aggressive caching
You can choose some column families to have a higher priority in the block cache (LRU cache). This comes in handy if you expect more random reads on one column family compared to another. This configuration is also done at table-instantiation time:
hbase(main):002:0> create 'mytable',
{NAME => 'colfam1', IN_MEMORY => 'true'}
The default value for the IN_MEMORY parameter is false.

Bloom filters
hbase(main):007:0> create 'mytable',{NAME => 'colfam1', BLOOMFILTER => 'ROWCOL'}
The default value for the BLOOMFILTER parameter is NONE. A row-level bloom filter is enabled with ROW, and a qualifier-level bloom filter is enabled with ROWCOL. The rowlevel bloom filter checks for the non-existence of the particular rowkey in the block,and the qualifier-level bloom filter checks for the non-existence of the row and column qualifier combination. The overhead of the ROWCOL bloom filter is higher than that of the ROW bloom filter.

TTL (Time To Live)
can set the TTL while creating the table like this:
hbase(main):002:0> create 'mytable', {NAME => 'colfam1', TTL => '18000'}
This command sets the TTL on the column family colfam1 as 18,000 seconds = 5 hours. Data in colfam1 that is older than 5 hours is deleted during the next major compaction.

Compression
can enable compression on a column family when creating tables like this:
hbase(main):002:0> create 'mytable',
{NAME => 'colfam1', COMPRESSION => 'SNAPPY'}
Note that data is compressed only on disk. It’s kept uncompressed in memory (Mem-Store or block cache) or while transferring over the network.

Cell versioning
Versions are also configurable at a column family level and can be specified at
the time of table instantiation:
hbase(main):002:0> create 'mytable', {NAME => 'colfam1', VERSIONS => 1}
hbase(main):002:0> create 'mytable',
{NAME => 'colfam1', VERSIONS => 1, TTL => '18000'}
hbase(main):002:0> create 'mytable', {NAME => 'colfam1', VERSIONS => 5,
MIN_VERSIONS => '1'}

Description of a table
hbase(main):004:0> describe 'follows'
DESCRIPTION ENABLED
{NAME => 'follows', coprocessor$1 => 'file:///U true users/ndimiduk/repos/hbaseia twitbase/target/twitbase-1.0.0.jar|HBaseIA.TwitBase.coprocessors.FollowsObserver|1001|', FAMILIES => [{NAME => 'f', BLOOMFILTER => 'NONE', REPLICATION_SCOPE =>'0', VERSIONS => '1', COMPRESSION => 'NONE', MIN_VERSIONS => '0', TTL => '2147483647', BLOCKSIZE => '65536', IN_MEMORY => 'false', BLOCKCACHE => 'true'}]}
1 row(s) in 0.0330 seconds
```

## 参考链接

+ [HBase优化相关](http://www.cnblogs.com/skyl/p/4814347.html)
+ [Hbase shell and Commands](http://hadoopbigdatas.blogspot.com/2013/03/hbase-shell-and-commands.html)
+ [配置 HBase](http://docs.amazonaws.cn/ElasticMapReduce/latest/DeveloperGuide/emr-hbase-configure.html)


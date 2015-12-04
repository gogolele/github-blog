title: Cassandra使用SSTableLoader导入数据
date: 2013-10-17 14:11:35
categories: dev
tags: Cassandra
---

Step 1： 从远程Cassandra集群拷贝sstable数据，只需Data / Index数据即可
Step 2： 在本地集群创建对应的Keyspace和Column Family
Step 3： 运行SSTableLoader

```
 $ sstableloader --debug -d localhost Keyspace1/ColumnFamily1
```

 最好在非Cassandra node的server上运行sstableloader. 根据[Cassandra Bulkloader]的官方文档，在一个Cassandra节点运行sstableloader，会无法使用其network interface，也可能会因为这个节点的网络流量较大降低效率。
- SSTable的文件路径，需安装 <Keyspace> / <CF> 这样的路径存放， 并输入给sstableloader
- --debug 参数， 可以在控制台打出debug信息
- SSTable下有子目录如snapshot，会导致sstableloader异常

运行成功示例

```
progress: [/10.16.83.86 3/3 (100)] [/10.16.83.87 3/3 (100)] [/10.16.83.84 3/3 (100)]
[total: 100 - 0MB/s (avg: 6MB/s)]
Waiting for targets to rebuild indexes ...
```

很好用，速度也很快。

END-------
伍涛@成都
2013-10-18

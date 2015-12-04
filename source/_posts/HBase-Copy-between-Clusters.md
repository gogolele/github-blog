title: HBase CopyTable between Clusters
date: 2015-05-21 14:58:03
categories: dev
tags: [HBase, CopyTable]
---

Copy tables from one hbase cluster to another.

Note1, this copytable shoud be ran on the source cluster, it is a push action, which push data from source to destination.
Note2, create table in destination cluster first.

General usage

```shell
[tw79@sourcecluster ~]$ hbase org.apache.hadoop.hbase.mapreduce.CopyTable --peer.adr=zk-destination:2181:/hbase backend:TSDB-TREE
```


Copy Table for a specific time range.
```shell
[tw79@sourcecluster ~]$ hbase org.apache.hadoop.hbase.mapreduce.CopyTable --peer.adr=zk-destination:2181:/hbase --starttime=1432156100000 --endtime=1432242500000 backend:TSDB

```

-EOF-

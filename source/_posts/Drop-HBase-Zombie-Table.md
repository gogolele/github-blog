title: Drop HBase Zombie Table
date: 2015-06-26 10:53:34
categories: dev
tags: HBase
---

when i want to drop / disable / truncate / create a hbase table in hbase shell, it hang a long time. propably we meet the zombie table in hbase. here is the action to delete it without restart hbase daemon.

## check the meta table
```shell
[root@server]# hbase shell
scan 'hbase:meta', {LIMIT=>10, FILTER=>"PrefixFilter('ecitem:Table1')"}
```
no result, seems fine in hbase meta table.

## delete data in zookeeper
```shell
[root@server]#bin/zkCli.sh -server zk1,zk2,zk3
> rmr /hbase/table/ecitem:Table1
```

## delete hdfs data
```shell
[root@server]# bin/hadoop fs -rm -r /hbase/data/ecitem/Table1
[root@server]# bin/hadoop fs -rm -rf /hbase/.tmp/data/ecitem/Table1
```

note, if there are data in hbase/.tmp of this table, you should delete it first. or it will automatically load this tmp data when you create table. And will hang until load tmp data completed.

go to hbase shell, then create the hbase table again. should be ok.


-EOF-

TAO WU@CA, USA

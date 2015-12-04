title: DistCP使用注意
date: 2012-04-22 14:45:50
categories: dev
tags: HDFS
---


distcp主要用于在hadoop集群之间拷贝数据。
### 1. 如果haboop版本相同，可以使用如下格式
```
hadoop distcp hdfs://<hdfs_address:hdfs_port>/src hdfs://<hdfs address:port>/des
```

### 2. 如果在不同版本的hadoop集群之间拷贝数据，可以使用如下格式
```
hadoop distcp -i hftp://<hdfs_address:http_port>>/src hdfs://<hdfs address:port>/des
```
注意，这个时候，需要在目标集群上运行distcp， -i是忽略错误。
注意hftp和ftp没有什么关系，它是通过http访问hdfs文件系统的协议包装，以支持不同版本之间拷贝数据。它的端口，不是dfs端口，而是http端口。
### 3. 实际应用

在我的应用中，一个是hadoop1.0.0集群，一个是cloudera cdh3u0集群，此时需要将hadoop1.0.0里面的数据拷贝到cloudera cdh3u0的hdfs中。因此采用hftp的distcp。
更进一步，如果只是造无逻辑关系的数据，distcp没有只写的teragen或slive快。在我的测试中，teragen和slive的混合写入，磁盘写入速度可以达到300MB/s，网络io可以达到100+MB/s。而distcp，磁盘写入为100MB/s，网络io也达到100+MB/s。
补充一下，如果是升级hdfs的hadoop版本，可以在启动时start-dfs -upgrade，这样即可以将文件系统升级至新的hadoop版本。如从hadoop-0.19 至hadoop-0.20，但是如果不是一脉相承的版本，升级也有问题。如我这边不能将hadoop-1.0.0与cloudera版本之间进行升级。

Reference
http://hadoop.apache.org/hdfs/docs/current/hftp.html
http://www.linezing.com/blog/?p=452

-EOF-

伍涛@成都 重整
2013-07-28

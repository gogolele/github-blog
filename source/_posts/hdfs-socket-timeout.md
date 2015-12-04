title: 大并发写时，HBase的HDFS DFSClient端报SocketTimeoutException的问题分析和解决
date: 2012-03-08 15:13:56
categories: dev
tags: [HDFS, HBase]
---


## 问题现象

HBase大并发写，出现异常日志：
```
2012-03-02 12:11:21,983 WARN org.apache.hadoop.hdfs.DFSClient: DFSOutputStream ResponseProcessor exception  for block blk_1439491087566691588_6207java.net.SocketTimeoutException: 69000 millis timeout while waiting for channel to be ready for read. ch : java.nio.channels.SocketChannel[connected local=/dw1:54889 remote=/dw2:62010]
        at org.apache.hadoop.net.SocketIOWithTimeout.doIO(SocketIOWithTimeout.java:164)
        at org.apache.hadoop.net.SocketInputStream.read(SocketInputStream.java:150)
        at org.apache.hadoop.net.SocketInputStream.read(SocketInputStream.java:123)
        at java.io.DataInputStream.readFully(DataInputStream.java:178)
        at java.io.DataInputStream.readLong(DataInputStream.java:399)
        at org.apache.hadoop.hdfs.protocol.DataTransferProtocol$PipelineAck.readFields(DataTransferProtocol.java:130)
        at org.apache.hadoop.hdfs.DFSClient$DFSOutputStream$ResponseProcessor.run(DFSClient.java:2459)

2012-03-02 12:11:21,984 WARN org.apache.hadoop.hdfs.DFSClient: Error Recovery for block blk_1439491087566691588_6207 bad datanode[0]dw2:62010
2012-03-02 12:11:21,984 WARN org.apache.hadoop.hdfs.DFSClient: Error Recovery for block blk_1439491087566691588_6207 in pipelinedw2:62010,dw1:62010,dw3:62010: bad datanode dw2:62010
2012-03-02 12:11:22,128 WARN org.apache.hadoop.hbase.regionserver.wal.HLog: HDFS pipeline error detected. Found 2 replicas but expecting 3 replicas.  Requesting close of hlog.
```

## 代码分析

正常情况下DFSClient写block数据的过程是：
### 1. DFSClient端

DFSOutputStream负责数据的接收和写入，即通过DFSOutputSummer中的write方法（synchronized）获得数据，而sync（主要代码 synchronized(this)）通过FlushBuffer建立packet后，通过enqueuePacket向dataQueue中写入数据。
DFSOutputStream中的DataStreamer（Daemon线程），负责向DataNode发送数据，每次发送前会检查dataQueue中是否有数据，没有就等待。
DataStreamer建立pipeline传输数据时，对这个pipeline会起一个ResponseProcessor（Thread）去获得DataNode的反馈ack，并判断是否有错误、进行recoverBlock等
### 2. DataNode端

在每个packet传输过程中，根据建立数据传输的pipleLine，上游依次向下游发送数据，下游依次向上游发送ack。
pipeline的最后一个节点（numTarget=0），PacketResponder 会一直运行lastDatanodeRun?方法，这个方法会在ack发送完毕(ackQueue.size()=0)后约1/2个dfs.socket.timeout?时刻发送心跳包，沿着pipeline发送给client。
### 3. HBase端

HBase端通过hlog中的writer向hdfs写数据，每次有数据写入，都会sync。同时，HLog中有个logSyncer，默认配置是每秒钟调用一次sync，不管有没有数据写入。

## 问题分析

这个问题首先是由于超时引起的，我们先分析一下超时前后DFSClient和DataNode上发生了什么。
### 1. 问题重现

客户端ResponseProcessor报69秒socket超时，出错点在PipelineAck.readFields()。出错后直接catch，标记hasError=true，closed=true。这个线程不会停止。
DataStreamer在轮询中调用processDatanodeError对hasError=true进行处理。此时errorIndex=0（默认值），首先会抛出Recovery for Block的异常. 然后关闭blockstream，重新基于两个节点的pipeline进行recoverBlock。
在DataNode上，processDatanodeError()关闭blockstream。这将导致pipeline中的packetResponder被interrupted和terminated。
在DataNode上，processDatanodeError()关闭blockstream，导致BlockReceiver的readNextPacket()中的readToBuf读取不到数据，throw EOFException的异常。这个异常一直向上抛，直到DataXceiver的run中，它将导致DataXceiver中止运行，提示DataNode.dnRegistration Error。
recoverBlock会正常进行，并先在两个节点上完成（第二个和第三个）。随后Namenode会发现replicas数量不足，向DataNode发起transfer block的命令，这是一个异步的过程。但是在hlog检查时，transfer很有可能未完成，这时会报 pipeline error detected. Found 2 replicas but expecting 3 replicas。并关闭hlog。
以上就是根据日志可以看到的错误过程。
### 2. 问题分析

为什么会超时，为什么心跳包没有发？ 根据以上的分析，ResponseProcessor socket 69秒超时是导致后续一系列异常和hlog关闭的原因。那么为何会发生socket超时？ResponseProcessor应该会在dfs.socket.timeout的1/2时间内收到HeartBeat包。 经过打印日志，我们发现，DataNode上配置的dfs.socket.timeout为180秒，而HBase调用DFSClient时采用默认配置，即60秒。因此，DFSClient认为超时时间为3×nodes.length+60=69秒，而DataNode端发送心跳包的timeout=1/2×180=90秒！因此，如果在没有数据写入的情况下，DataNode将在90秒后发送心跳包，此时DFSClient已经socketTimeout了，并导致后续的一系列现象。
为什么会在69秒内没有新的packet发送过去呢？ 我们先分析一下DFSOutputStream写数据和sync的同步关系。DFSOutputStream继承自FSOutputSummer，DFSOutputStream接收数据是通过FSOutputSummer的write方法，这个方法是synchronized。而sync方法的flushBuffer()和enqueuePacket()，也在synchronized(this)代码块中。也就是说，对一个DFSOutputStream线程，如果sync和write同时调用，将发生同步等待。在hbase的场景下，sync发生的频率非常高，sync抢到锁的可能性很大。这样，就很有可能在不断的sync，不断的flushBuffer，但是却没能通过write写入数据（被blocked了）。这就是导致超时时间内一直没有packet发送的原因。
综上，HBase业务调用的特点和DFSOutputStream的synchronized代码块，很有可能69秒中没有packet写入。但这个时候，不应该socket超时，socket超时是这个问题的根本原因，而socket超时的原因是配置不一致。
### 3. 问题解决

在hdfs端和hbase端，配置同样的dfs.socket.timeout值。在云梯环境下配置的是180000（180秒）。

------END-------

伍涛@成都
2013-07-27

title: HDFS RecoverLease RecoverBlock源码分析
date: 2012-03-22 14:52:44
categories: dev
tags: HDFS
---


## 起因

最近需要搞一下Lease，分析一下recoverLease的过程，顺带把recoverBlock的过程分析一下。

## 1. RecoverLease

recoverLease是恢复租约，我理解为释放文件之前的租约，close文件，报告namenode。
recoverLease有两条路径去调用
```
DistributedFileSystem.create
-> DFSClient.create
-> Namenode.create
-> FSNamesystem.startFile
-> FSNamesystem.startFileInternal
-> recoverLeaseInternal(myFile, src, holder, clientMachine, false)
```

这条路径是在客户端文件create时调用的，此时它不需要close文件。
```
DistributedFileSystem.recoverLease
-> DFSClient.recoverLease
-> Namenode.recoverLease(src,clientname)
-> FSNamesystem.recoverLease(src,holder,clientMachine)
-> recoverLeaseInternal(inode, src, holder, clientMachine, true)
```

这条路径是在客户端显式调用一个path的recoverLease，此时它需要close文件。 参看 HDFS-1554
以上两个路径，最终会调用FSNamesystem.recoverLeaseInternal，下面我们着重看一下recoverLeaseInternal，它主要做以下事情：
```
1. 获得pendingfile，获得当前holder的lease
2. 如果lease不为空，且不是force，则AlreadyBeingCreatedException
3. 获得当前client的lease，如果lease为空，则AlreadyBeingCreatedException
4. 如果force则强制internalReleaseLeaseOne；否则，如果超过softlimit，也强制internalReleaseLease，但也抛出AlreadyBeingCreatedException。
```

它会通过调用internalReleaseLeaseOne实现对recoverBlock的执行。
```
recoverLeaseInternal
-> internalReleaseLease
-> internalReleaseLeaseOne
internalReleaseLeaseOne的过程：
```

```
1.找这个file的最后一个block的targets

2. pendingFile.assignPrimaryDatanode()  
   -> DatanodeDescriptor.addBlockToBeRecovered()
   -> recoverBlocks.offer()
    recoverBlocks会在datanode向namenode发送心跳包时，将recoverBlock的命令以   NN_RECOVERY的holder身份返回给datanode，datanode执行命令。

3.reassignLease
```

## 2. RecoverBlock过程

recoverBlock是在Datanode上执行的，有两条路径调用，一个由Namenode发起的，一个DFSClient发起的。

### 2.1. Namenode发起recoverBlock

Namenode上的过程：
```
  Namenode.sendHeartbeat(datanode调用，向namenode发送心跳包，namenode返回i要执行的cmd
  -> FSNamesystem.handleHeartbeat
  -> DatanodeDescritor.getLeaseRecoveryCommand
  -> recoverBlocks.poll()
```  

按照上面recoverLease的分析，如果有recoverLease的请求，BlockQueue类型的recoverBlocks就有待处理的recoverBlock
Datanode上的过程
```
Datanode.run
-> offerService()(一直运行，和namenode交互)
-> processCommand(cmds [])
-> processCommand(cmd)
-> DNA_RECOVERBLOCK
-> recoverBlocks(bcmd.getBlocks(), bcmd.getTargets())
-> recoverBlock(blocks[i], false, targets[i], true)
   [closeFile = true]
```

即datanode上有个心跳线程，默认每个3秒钟，向namenode汇报心跳包，并获得要在datanode上执行的cmd。如果有namenode发来的recoverBlock请求，
datanode上最终会调用recoverBlock方法，此时closeFile=true。它是recoverLease发起的，此时要关闭文件，并使得这个文件的block在datanode上的信息一致。

### 2.2. DFSClient发起的recoverBlock

```
DFSClient.processDatanodeError
-> DataNode.recoverBlock(Block block, boolean keepLength, DatanodeInfo[] targets)
-> DataNode.recoverBlock(Block block, boolean keepLength, DatanodeInfo[] targets, false) [closeFile = false]
```

DFSClient通过dataStreamer发送block的packet数据，如果这个过程出现异常，会由processDatanodeError进行recover处理，即获得pipeline中错误的datanode，在剩余的两个datanode选取一个datanode发起recoverBlock，从这个datanode开始重建pipeline。
此时调用datanode的recoverBlock传递的closeFile=false，因为在DFSClient写入Block出现异常时，需要的是recover，不是关闭文件。
关于recoverBlock
```
recoverBlock(Block block, boolean keepLength, DatanodeInfo[] targets, boolean closeFile)

1.检查ongoingRecovery中是否有这个block正在recover，如果有，则抛出IOException，Block is already being recovered, ignoring this request to recover it。如果没有，则add进入ongoingRecovery

2.根据targets建立syncList，即明确向哪些节点sync哪个block

---> syncBlock
1.首先为这次syncBlock从namenode处获得generationStamp
2.以新的generationStamp创建新的newBlock
3.对syncList中的每个datanode，执行updateBlock操作，将旧的block更新为newBlock
4.如果执行成功，向namenode报告commitBlockSynchronization，包含新的block和generationStamp
5.返回locatedBlock

---> updateBlock
1.FSDataset.updateBlock
2.如果finalize为true，FSDataset.finalizeBlockIfNeeded；通知namenode已经received block
```

以上，记录以备忘。

END-------
伍涛@成都 重整
2013-07-28

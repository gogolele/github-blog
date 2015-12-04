title: Spark Training
date: 2015-02-26 15:52:51
categories: dev
tags: Spark
---

### Problem 1
Problem: java.lang.UnsatisfiedLinkError: no snappyjava in java.library.path
Solution:
 open the org-xerial-snappy.properties in Hadoop Conf folder, and set this configuration to false.
 ```
 org.xerial.snappy.disable.bundled.libs=false
 ```
Stack output:
```
15/02/26 11:23:09 ERROR ActorSystemImpl: Uncaught fatal error from thread [sparkDriver-akka.actor.default-dispatcher-4] shutting down ActorSystem [sparkDriver]
java.lang.UnsatisfiedLinkError: no snappyjava in java.library.path
at java.lang.ClassLoader.loadLibrary(ClassLoader.java:1738)
at java.lang.Runtime.loadLibrary0(Runtime.java:823)
at java.lang.System.loadLibrary(System.java:1028)
at org.xerial.snappy.SnappyLoader.loadNativeLibrary(SnappyLoader.java:170)
at org.xerial.snappy.SnappyLoader.load(SnappyLoader.java:145)
at org.xerial.snappy.Snappy.<clinit>(Snappy.java:47)
at org.xerial.snappy.SnappyOutputStream.<init>(SnappyOutputStream.java:90)
at org.xerial.snappy.SnappyOutputStream.<init>(SnappyOutputStream.java:83)
at org.apache.spark.io.SnappyCompressionCodec.compressedOutputStream(CompressionCodec.scala:125)
at org.apache.spark.broadcast.TorrentBroadcast$$anonfun$4.apply(TorrentBroadcast.scala:199)
at org.apache.spark.broadcast.TorrentBroadcast$$anonfun$4.apply(TorrentBroadcast.scala:199)
at scala.Option.map(Option.scala:145)
at org.apache.spark.broadcast.TorrentBroadcast$.blockifyObject(TorrentBroadcast.scala:199)
at org.apache.spark.broadcast.TorrentBroadcast.writeBlocks(TorrentBroadcast.scala:101)
at org.apache.spark.broadcast.TorrentBroadcast.<init>(TorrentBroadcast.scala:84)
at org.apache.spark.broadcast.TorrentBroadcastFactory.newBroadcast(TorrentBroadcastFactory.scala:34)
at org.apache.spark.broadcast.TorrentBroadcastFactory.newBroadcast(TorrentBroadcastFactory.scala:29)
at org.apache.spark.broadcast.BroadcastManager.newBroadcast(BroadcastManager.scala:62)
at org.apache.spark.SparkContext.broadcast(SparkContext.scala:945)
at org.apache.spark.scheduler.DAGScheduler.org$apache$spark$scheduler$DAGScheduler$$submitMissingTasks(DAGScheduler.scala:838)
at org.apache.spark.scheduler.DAGScheduler.org$apache$spark$scheduler$DAGScheduler$$submitStage(DAGScheduler.scala:778)
at org.apache.spark.scheduler.DAGScheduler.handleJobSubmitted(DAGScheduler.scala:762)
at org.apache.spark.scheduler.DAGSchedulerEventProcessActor$$anonfun$receive$2.applyOrElse(DAGScheduler.scala:1389)
at akka.actor.Actor$class.aroundReceive(Actor.scala:465)
at org.apache.spark.scheduler.DAGSchedulerEventProcessActor.aroundReceive(DAGScheduler.scala:1375)
at akka.actor.ActorCell.receiveMessage(ActorCell.scala:516)
at akka.actor.ActorCell.invoke(ActorCell.scala:487)
at akka.dispatch.Mailbox.processMailbox(Mailbox.scala:238)
at akka.dispatch.Mailbox.run(Mailbox.scala:220)
at akka.dispatch.ForkJoinExecutorConfigurator$AkkaForkJoinTask.exec(AbstractDispatcher.scala:393)
at scala.concurrent.forkjoin.ForkJoinTask.doExec(ForkJoinTask.java:260)
at scala.concurrent.forkjoin.ForkJoinPool$WorkQueue.runTask(ForkJoinPool.java:1339)
at scala.concurrent.forkjoin.ForkJoinPool.runWorker(ForkJoinPool.java:1979)
at scala.concurrent.forkjoin.ForkJoinWorkerThread.run(ForkJoinWorkerThread.java:107)

```
-EOF-

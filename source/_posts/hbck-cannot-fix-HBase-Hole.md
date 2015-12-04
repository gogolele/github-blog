title: hbck cannot fix HBase Hole
date: 2015-10-28 17:01:09
categories: dev
tags: [hbase, hbck]
---

today we encountered a problem, a OpenTSDB table got one region stucked in region in transition.
![region stuck](/image/region-stuck.png)

first, it looks region missing / not assigned problem, we use hbck to fix, but it doesn't work.
```shell
$> sudo -u hbase hbase hbck -fixMeta -fixAssignments
ERROR: Region { meta => backend:EC-TSDB,\x00\x0B\x0FU\xD8rP\x00\x00\x01\x00\x17i,1441495243944.ebbf59429ade45b0d6351dbad831c640., hdfs => hdfs://nameservice1/hbase/data/backend/EC-TSDB/ebbf59429ade45b0d6351dbad831c640, deployed => , replicaId => 0 } not deployed on any region server.
ERROR: Last region should end with an empty key. You need to create a new region and regioninfo in HDFS to plug the hole.
ERROR: Found inconsistency in table backend:EC-TSDB

```

second, go to the region server and read the hbase log.
```txt
2015-10-28 16:09:17,040 INFO org.apache.hadoop.hbase.regionserver.HRegion: Started memstore flush for backend:EC-TSDB,\x00\x0B\x0FU\xD8rP\x00\x00\x01\x00\x17i,1441495243944.ebbf59429ade45b0d6351dbad831c640., current region memstore size 77.71 MB, and 1/1 column families' memstores are being flushed.; wal is null, using passed sequenceid=46441915
2015-10-28 16:09:17,726 ERROR org.apache.hadoop.hbase.regionserver.handler.OpenRegionHandler: Failed open of region=backend:EC-TSDB,\x00\x0B\x0FU\xD8rP\x00\x00\x01\x00\x17i,1441495243944.ebbf59429ade45b0d6351dbad831c640., starting to roll back the global memstore size.
org.apache.hadoop.hbase.DroppedSnapshotException: region: backend:EC-TSDB,\x00\x0B\x0FU\xD8rP\x00\x00\x01\x00\x17i,1441495243944.ebbf59429ade45b0d6351dbad831c640.
        at org.apache.hadoop.hbase.regionserver.HRegion.internalFlushCacheAndCommit(HRegion.java:2243)
        at org.apache.hadoop.hbase.regionserver.HRegion.internalFlushcache(HRegion.java:1972)
        at org.apache.hadoop.hbase.regionserver.HRegion.replayRecoveredEditsIfAny(HRegion.java:3826)
        at org.apache.hadoop.hbase.regionserver.HRegion.initializeRegionStores(HRegion.java:969)
        at org.apache.hadoop.hbase.regionserver.HRegion.initializeRegionInternals(HRegion.java:841)
        at org.apache.hadoop.hbase.regionserver.HRegion.initialize(HRegion.java:814)
        at org.apache.hadoop.hbase.regionserver.HRegion.openHRegion(HRegion.java:5828)
        at org.apache.hadoop.hbase.regionserver.HRegion.openHRegion(HRegion.java:5794)
        at org.apache.hadoop.hbase.regionserver.HRegion.openHRegion(HRegion.java:5765)
        at org.apache.hadoop.hbase.regionserver.HRegion.openHRegion(HRegion.java:5721)
        at org.apache.hadoop.hbase.regionserver.HRegion.openHRegion(HRegion.java:5672)
        at org.apache.hadoop.hbase.regionserver.handler.OpenRegionHandler.openRegion(OpenRegionHandler.java:356)
        at org.apache.hadoop.hbase.regionserver.handler.OpenRegionHandler.process(OpenRegionHandler.java:126)
        at org.apache.hadoop.hbase.executor.EventHandler.run(EventHandler.java:128)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1145)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
        at java.lang.Thread.run(Thread.java:745)
Caused by: java.lang.NegativeArraySizeException
        at org.apache.hadoop.hbase.CellComparator.getMinimumMidpointArray(CellComparator.java:494)
        at org.apache.hadoop.hbase.CellComparator.getMidpoint(CellComparator.java:448)
        at org.apache.hadoop.hbase.io.hfile.HFileWriterV2.finishBlock(HFileWriterV2.java:165)
        at org.apache.hadoop.hbase.io.hfile.HFileWriterV2.checkBlockBoundary(HFileWriterV2.java:146)
        at org.apache.hadoop.hbase.io.hfile.HFileWriterV2.append(HFileWriterV2.java:263)
        at org.apache.hadoop.hbase.regionserver.StoreFile$Writer.append(StoreFile.java:949)
        at org.apache.hadoop.hbase.regionserver.StoreFlusher.performFlush(StoreFlusher.java:121)
        at org.apache.hadoop.hbase.regionserver.DefaultStoreFlusher.flushSnapshot(DefaultStoreFlusher.java:71)
        at org.apache.hadoop.hbase.regionserver.HStore.flushCache(HStore.java:940)
        at org.apache.hadoop.hbase.regionserver.HStore$StoreFlusherImpl.flushCache(HStore.java:2217)
        at org.apache.hadoop.hbase.regionserver.HRegion.internalFlushCacheAndCommit(HRegion.java:2197)
        ... 16 more
```

it looks the recoveredits caused the problem. so move away the recoveredits data.
```shell
$> sudo -u hbase hadoop fs -mv /hbase/data/backend/EC-TSDB/ebbf59429ade45b0d6351dbad831c640/recovered.edits/* /tmp/hbase/
```

third, re-run the hbck, then it works.
```shell
$> sudo -u hbase hbase hbck -fixMeta -fixAssignments
```

After this action, we found this issue happened again on this same region.

we remove the recoveredits again, and do hbase hbck fix.
```shell
$> sudo -u hbase hadoop fs -mv /hbase/data/backend/EC-TSDB/ebbf59429ade45b0d6351dbad831c640/recovered.edits/* /tmp/hbase/
$> sudo -u hbase hbase hbck -fixMeta -fixAssignments
```

the log for compacting this region.
```txt
2015-10-30 10:42:22,653 INFO org.apache.hadoop.hbase.regionserver.HStore: Starting compaction of 3 file(s) in t of backend:EC-TSDB,\x00\x0B\x0FU\xD8rP\x00\x00\x01\x00\x17i,1441495243944.ebbf59429ade45b0d6351dbad831c640. into tmpdir=hdfs://nameservice1/hbase/data/backend/EC-TSDB/ebbf59429ade45b0d6351dbad831c640/.tmp, totalSize=17.4 M
2015-10-30 10:42:22,712 INFO org.apache.hadoop.hbase.io.hfile.CacheConfig: blockCache=LruBlockCache{blockCount=70992, currentSize=4753916128, freeSize=249993504, maxSize=5003909632, heapSize=4753916128, minSize=4753714176, minFactor=0.95, multiSize=2376857088, multiFactor=0.5, singleSize=1188428544, singleFactor=0.25}, cacheDataOnRead=true, cacheDataOnWrite=false, cacheIndexesOnWrite=false, cacheBloomsOnWrite=false, cacheEvictOnClose=false, cacheDataCompressed=false, prefetchOnOpen=false
2015-10-30 10:42:23,828 ERROR org.apache.hadoop.hbase.regionserver.CompactSplitThread: Compaction failed Request = regionName=backend:EC-TSDB,\x00\x0B\x0FU\xD8rP\x00\x00\x01\x00\x17i,1441495243944.ebbf59429ade45b0d6351dbad831c640., storeName=t, fileCount=3, fileSize=17.4 M (4.8 M, 6.8 M, 5.8 M), priority=-170, time=1185433419673966
java.lang.ArrayIndexOutOfBoundsException: -32710
        at org.apache.hadoop.hbase.CellComparator.getMinimumMidpointArray(CellComparator.java:478)
        at org.apache.hadoop.hbase.CellComparator.getMidpoint(CellComparator.java:448)
        at org.apache.hadoop.hbase.io.hfile.HFileWriterV2.finishBlock(HFileWriterV2.java:165)
        at org.apache.hadoop.hbase.io.hfile.HFileWriterV2.checkBlockBoundary(HFileWriterV2.java:146)
        at org.apache.hadoop.hbase.io.hfile.HFileWriterV2.append(HFileWriterV2.java:263)
        at org.apache.hadoop.hbase.regionserver.StoreFile$Writer.append(StoreFile.java:949)
        at org.apache.hadoop.hbase.regionserver.compactions.Compactor.performCompaction(Compactor.java:270)
        at org.apache.hadoop.hbase.regionserver.compactions.DefaultCompactor.compact(DefaultCompactor.java:101)
        at org.apache.hadoop.hbase.regionserver.DefaultStoreEngine$DefaultCompactionContext.compact(DefaultStoreEngine.java:122)
        at org.apache.hadoop.hbase.regionserver.HStore.compact(HStore.java:1230)
        at org.apache.hadoop.hbase.regionserver.HRegion.compact(HRegion.java:1724)
        at org.apache.hadoop.hbase.regionserver.CompactSplitThread$CompactionRunner.run(CompactSplitThread.java:511)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1145)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
        at java.lang.Thread.run(Thread.java:745)
```

we checked the store files for this region. only this region have 182 store files, it means compaction failed.
```txt
3
-----/hbase/data/backend/EC-TSDB/ebad1601624c7f4c376a44ae95b09eb4/t--------
2
-----/hbase/data/backend/EC-TSDB/ebbf59429ade45b0d6351dbad831c640/t--------
182
-----/hbase/data/backend/EC-TSDB/ebd08a2fb7742467aa613ac57968d0b2/t--------
3
-----/hbase/data/backend/EC-TSDB/ebd389283198d86f3642408bc27efbc8/t--------
4
-----/hbase/data/backend/EC-TSDB/eca5aef79f04361906f691353f5ac2e9/t--------
5
-----/hbase/data/backend/EC-TSDB/ecab3355b56a074c3c031fc34492f300/t--------
2
-----/hbase/data/backend/EC-TSDB/f12e77ee52b82476275c0475334ec0be/t--------
3
-----/hbase/data/backend/EC-TSDB/f14fc68762b20bd20c23abd33c941a09/t--------
2
-----/hbase/data/backend/EC-TSDB/f69e6d2fe8091105c02aeddfaa0563d8/t--------
2
-----/hbase/data/backend/EC-TSDB/f82710e01bc9a1da2b3a3e5b855f0fde/t--------
3
-----/hbase/data/backend/EC-TSDB/ff0add7623183a07fe10304343841426/t--------
2
```

they we do manually major_compact on this region.
```shell
$> major_compact 'ebbf59429ade45b0d6351dbad831c640'
```

the compaction log:
```txt
2015-10-30 11:51:22,427 INFO org.apache.hadoop.hbase.regionserver.RSRpcServices: Compacting backend:EC-TSDB,\x00\x0B\x0FU\xD8rP\x00\x00\x01\x00\x17i,1441495243944.ebbf59429ade45b0d6351dbad831c640.
2015-10-30 11:51:22,434 INFO org.apache.hadoop.hbase.regionserver.HRegion: Starting compaction on t in region backend:EC-TSDB,\x00\x0B\x0FU\xD8rP\x00\x00\x01\x00\x17i,1441495243944.ebbf59429ade45b0d6351dbad831c640.
2015-10-30 11:51:22,436 INFO org.apache.hadoop.hbase.regionserver.HStore: Starting compaction of 181 file(s) in t of backend:EC-TSDB,\x00\x0B\x0FU\xD8rP\x00\x00\x01\x00\x17i,1441495243944.ebbf59429ade45b0d6351dbad831c640. into tmpdir=hdfs://nameservice1/hbase/data/backend/EC-TSDB/ebbf59429ade45b0d6351dbad831c640/.tmp, totalSize=2.3 G
2015-10-30 11:58:24,044 INFO org.apache.hadoop.hbase.regionserver.HStore: Completed major compaction of 181 (all) file(s) in t of backend:EC-TSDB,\x00\x0B\x0FU\xD8rP\x00\x00\x01\x00\x17i,1441495243944.ebbf59429ade45b0d6351dbad831c640. into 729d84ac95e6462da47917975668c67f(size=2.1 G), total size for store is 2.1 G. This selection was in queue for 0sec, and took 7mins, 1sec to execute.
2015-10-30 11:58:24,044 INFO org.apache.hadoop.hbase.regionserver.CompactSplitThread: Completed compaction: Request = regionName=backend:EC-TSDB,\x00\x0B\x0FU\xD8rP\x00\x00\x01\x00\x17i,1441495243944.ebbf59429ade45b0d6351dbad831c640., storeName=t, fileCount=181, fileSize=2.3 G, priority=1, time=1189573199716718; duration=7mins, 1sec
2015-10-30 11:58:24,050 INFO org.apache.hadoop.hbase.regionserver.HRegion: Starting compaction on i in region ecitem:EC_Inventory_country,ec040ea57c71eb008b287809e4919f5c4aa09e7bb0a56fe1d95f04b1216fcede,1442351262912.ff867a2b2b1b66e194d2afa22584118e.
```

after this, i think we fixed this problem.

-EOF-

Tao Wu@CA, USA

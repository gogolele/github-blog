title: Redis PipeLine
date: 2015-08-19 07:36:08
tags: [redis, pipeLine]
categories: dev
---

## Redis PipeLine Wirter
```java
package com.tonywutao.redis;

import java.util.ArrayList;
import java.util.List;
import com.tonywutao.config.CommonValue;
import com.tonywutao.util.Util;
import redis.clients.jedis.Jedis;
import redis.clients.jedis.Pipeline;

/**
 * redis concurrent writer.
 *
 * @author tao wu
 *
 */
public class RedisPipelineWriter {
	public void start() {
		for (int i = 0; i < CommonValue.REDIS_CONCURRENT_WRITER; i++) {
			RedisPipelineWriterThread runnable = new RedisPipelineWriterThread();
			Thread thread = new Thread(runnable);
			thread.setName("REDIS-PIPELINE-WRITER-" + i);
			thread.start();
		}
	}
}

class RedisPipelineWriterThread implements Runnable {
	Jedis jedis = JedisPoolManager.getJedisObject();
	int tcount = 0;

	public RedisPipelineWriterThread() {
	}

	public void run() {
		boolean bPipeline = false;
		List<RedisSetObject> lstZadd = new ArrayList<RedisSetObject>();
		RedisSetObject setObj = null;
		long startWhile = 0L, endWhile = 0L;
		int count = 0;
		while (true) {
			startWhile = System.currentTimeMillis();
			while (!bPipeline) {
				try {
					setObj = MktRedisReader.REDIS_PUT_QUEUE.take();
					count++;
					tcount++;
					if (setObj != null) {
						lstZadd.add(setObj);
					}
				} catch (InterruptedException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}

				if (lstZadd.size() >= 1000) {
					bPipeline = true;
					break;
				}

				if (count % 100 == 0) {
					endWhile = System.currentTimeMillis();
					if (endWhile - startWhile > 100) {
						bPipeline = true;
						break;
					}
				}
			}

			// bPipeline = true
			if (lstZadd.size() > 0){
				submitSet(lstZadd);
				Util.printLog(" tcount = " + tcount);
			}
			bPipeline = false;
			lstZadd.clear();
			count = 0;

			try {
				Thread.sleep(100);
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
	}

	private void submitSet(List<RedisSetObject> lstZadd) {
		int scount = 0;
		try {
			Pipeline pipeline = jedis.pipelined();
			long start = System.currentTimeMillis();
			for (RedisSetObject obj : lstZadd) {
				pipeline.set(obj.getKey(), obj.getValue());
			}
			List<Object> results = pipeline.syncAndReturnAll();
			long end = System.currentTimeMillis();
			Util.printLog("Zadd submit: " + ((end - start) / 1000.0)
					+ " seconds, records " + lstZadd.size() + ",total count="
					+ tcount + ", result count" + results.size()+", 1 record =" + lstZadd.get(0).getKey() + "|"
					+ lstZadd.get(0).getKey() + "|"
					+ lstZadd.get(0).getValue());
		} catch (Exception e) {
			Util.printLog("count=" + scount + "," + e.getMessage());
		}
	}
}

```

## Redis PipeLine Reader
```java
      List<MktItemEntity> lstResult = new ArrayList<MktItemEntity>();
			HashMap<String, MktItemEntity> redisMap = new HashMap<String, MktItemEntity>(getList.size());
			Pipeline p = jedis.pipelined();
			for (String s : getList) {
				p.get(s);
			}
			List<Object> ret = p.syncAndReturnAll();
			for (Object s : ret) {
				String itemStr = (String) s;
				MktItemEntity redisItem = getMktItemFromStr(itemStr);
				if(redisItem == null) {
//					Util.printLog(" redisItem == null, itemStr = " + itemStr);
					continue;
				}
				redisMap.put(redisItem.getSpn(), redisItem);
			}
```

-EOF-

TAO WU@CA, USA

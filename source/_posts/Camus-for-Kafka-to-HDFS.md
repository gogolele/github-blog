title: "Camus for Kafka to HDFS "
date: 2015-02-23 16:30:32
categories: dev
tags: [Kafka, Camus]
---

Camus can dump messages in Kafka MQ to HDFS. It is based on Hadoop job, and Kafka simple consumer.

## which version of Camus?
As the many different versions of kafka and hadoop. I choosed the wikimedia / analytics-camus version for this experiement,
```
  branch: wmf
  reversion: c9f234a87...
  comments: Update Hadoop to CDH4.2.0
```

```shell
git clone https://github.com/wikimedia/analytics-camus.git
git checkout c9f234a87....
mvn clean package -DskipTests
```

After build, we expect to get this jar, which includes all the dependency jars.
```
camus-example-0.1.0-SNAPSHOT-shaded.jar
```

And you can execute Camus by hadoop jar command,
```
hadoop jar camus-example-0.1.0-SNAPSHOT-shaded.jar com.linkedin.camus.etl.kafka.CamusJob -P /home/tristan/camus.properties
```
As the messages are encoded in Json String in our systems, JsonStringMessageDecoder should be set as decoder. When we ran this job, it looks job is successfully completed, but no data was dumped to HDFS. We meet a problem like this:
```
Discarding topic (Decoder generation failed) : XXX Topic
```
After add the debug log in source code, we found that the JsonStringMessageDecoder is not existed in Wikimedia's Camus. So we add the latest version of this java class to project, then this problem was resloved.

Then we met another problem,
```
Exception in thread "main" java.nio.channels.UnresolvedAddressException
```

This problem is caused by the CamusJob is trying to make connections to Kafka Cluster by hostname. We need to move this app (CamusJob) to same network domain.

After that, the job can be ran successfully.

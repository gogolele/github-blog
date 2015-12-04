title: Solr Lucene High CPU by java.util.zip
date: 2015-10-21 11:39:48
categories: dev
tags: [lucene, solr]
---

We encounter a high cpu problem on Solr / Lucene server.

Scenario:
The cpu usage continously as high as 80+%, when we check the Disk / Memory, they all looks ok.

There are many threads are busy on CPU.
```shell
$> top -Hp 3238
3971 solr      20   0  449g  12g 3.9g R 100.1  4.9   2495:55 java
3990 solr      20   0  449g  12g 3.9g R 100.1  4.9   2495:13 java
3994 solr      20   0  449g  12g 3.9g R 100.1  4.9   2497:40 java
4017 solr      20   0  449g  12g 3.9g R 100.1  4.9   2494:37 java
4047 solr      20   0  449g  12g 3.9g R 100.1  4.9   2495:37 java
4049 solr      20   0  449g  12g 3.9g R 100.1  4.9   2494:01 java
6011 solr      20   0  449g  12g 3.9g R 100.1  4.9   2485:49 java
6020 solr      20   0  449g  12g 3.9g R 100.1  4.9   2485:57 java
6041 solr      20   0  449g  12g 3.9g R 100.1  4.9   2491:53 java
6047 solr      20   0  449g  12g 3.9g R 100.1  4.9   2493:57 java
3960 solr      20   0  449g  12g 3.9g R 99.8  4.9   2487:37 java
3962 solr      20   0  449g  12g 3.9g R 99.8  4.9   2484:43 java
3981 solr      20   0  449g  12g 3.9g R 99.8  4.9   2495:32 java
3993 solr      20   0  449g  12g 3.9g R 99.8  4.9   2494:56 java
4000 solr      20   0  449g  12g 3.9g R 99.8  4.9   2497:39 java
4003 solr      20   0  449g  12g 3.9g R 99.8  4.9   2494:33 java
4011 solr      20   0  449g  12g 3.9g R 99.8  4.9   2496:45 java
4020 solr      20   0  449g  12g 3.9g R 99.8  4.9   2490:52 java
4021 solr      20   0  449g  12g 3.9g R 99.8  4.9   2497:13 java
4025 solr      20   0  449g  12g 3.9g R 99.8  4.9   2497:00 java
4030 solr      20   0  449g  12g 3.9g R 99.8  4.9   2495:05 java
4036 solr      20   0  449g  12g 3.9g R 99.8  4.9   2494:20 java
4045 solr      20   0  449g  12g 3.9g R 99.8  4.9   2495:28 java
4048 solr      20   0  449g  12g 3.9g R 99.8  4.9   2493:22 java
6005 solr      20   0  449g  12g 3.9g R 99.8  4.9   2492:35 java
6017 solr      20   0  449g  12g 3.9g R 99.8  4.9   2484:36 java
6018 solr      20   0  449g  12g 3.9g R 99.8  4.9   2485:08 java
6021 solr      20   0  449g  12g 3.9g R 99.8  4.9   2489:00 java
6038 solr      20   0  449g  12g 3.9g R 99.8  4.9   2493:58 java
6083 solr      20   0  449g  12g 3.9g R 99.8  4.9   2493:58 java
```

Use jstack to check which class / method are using cpu.

```txt
"qtp1061200218-230" prio=10 tid=0x00007ed2a0001000 nid=0x17c3 runnable [0x00007ed3162e0000]
   java.lang.Thread.State: RUNNABLE
	at java.util.zip.Inflater.inflateBytes(Native Method)
	at java.util.zip.Inflater.inflate(Inflater.java:259)
	- locked <0x000000038b377730> (a java.util.zip.ZStreamRef)
	at java.util.zip.Inflater.inflate(Inflater.java:280)
	at org.apache.lucene.document.CompressionTools.decompress(CompressionTools.java:106)
	at org.apache.lucene.index.FieldsReader.uncompress(FieldsReader.java:639)
	at org.apache.lucene.index.FieldsReader.addField(FieldsReader.java:402)
	at org.apache.lucene.index.FieldsReader.doc(FieldsReader.java:253)
	at org.apache.lucene.index.SegmentReader.document(SegmentReader.java:477)
	at org.apache.lucene.index.DirectoryReader.document(DirectoryReader.java:583)
	at org.apache.lucene.index.IndexReader.document(IndexReader.java:1115)
	at org.apache.solr.search.SolrIndexReader.document(SolrIndexReader.java:445)
	at org.apache.solr.search.SolrIndexSearcher.doc(SolrIndexSearcher.java:450)
	at org.apache.solr.util.SolrPluginUtils.optimizePreFetchDocs(SolrPluginUtils.java:271)
	at org.apache.solr.handler.component.QueryComponent.doPrefetch(QueryComponent.java:478)
	at org.apache.solr.handler.component.QueryComponent.process(QueryComponent.java:385)
	at org.apache.solr.handler.component.SearchHandler.handleRequestBody(SearchHandler.java:194)
	at org.apache.solr.handler.RequestHandlerBase.handleRequest(RequestHandlerBase.java:140)
	at org.apache.solr.core.SolrCore.execute(SolrCore.java:1372)
	at org.apache.solr.servlet.SolrDispatchFilter.execute(SolrDispatchFilter.java:390)
	at org.apache.solr.servlet.SolrDispatchFilter.doFilter(SolrDispatchFilter.java:253)
	at org.eclipse.jetty.servlet.ServletHandler$CachedChain.doFilter(ServletHandler.java:1650)
	at net.bull.javamelody.MonitoringFilter.doFilter(MonitoringFilter.java:206)
	at net.bull.javamelody.MonitoringFilter.doFilter(MonitoringFilter.java:179)
	at org.eclipse.jetty.servlet.ServletHandler$CachedChain.doFilter(ServletHandler.java:1650)
	at org.eclipse.jetty.servlet.ServletHandler.doHandle(ServletHandler.java:583)
	at org.eclipse.jetty.server.handler.ScopedHandler.handle(ScopedHandler.java:143)
	at org.eclipse.jetty.security.SecurityHandler.handle(SecurityHandler.java:577)
	at org.eclipse.jetty.server.session.SessionHandler.doHandle(SessionHandler.java:223)
	at org.eclipse.jetty.server.handler.ContextHandler.doHandle(ContextHandler.java:1125)
	at org.eclipse.jetty.servlet.ServletHandler.doScope(ServletHandler.java:515)
	at org.eclipse.jetty.server.session.SessionHandler.doScope(SessionHandler.java:185)
	at org.eclipse.jetty.server.handler.ContextHandler.doScope(ContextHandler.java:1059)
	at org.eclipse.jetty.server.handler.ScopedHandler.handle(ScopedHandler.java:141)
	at org.eclipse.jetty.server.handler.ContextHandlerCollection.handle(ContextHandlerCollection.java:215)
	at org.eclipse.jetty.server.handler.HandlerCollection.handle(HandlerCollection.java:110)
	at org.eclipse.jetty.server.handler.HandlerWrapper.handle(HandlerWrapper.java:97)
	at org.eclipse.jetty.server.Server.handle(Server.java:497)
	at org.eclipse.jetty.server.HttpChannel.handle(HttpChannel.java:311)
	at org.eclipse.jetty.server.HttpConnection.onFillable(HttpConnection.java:248)
	at org.eclipse.jetty.io.AbstractConnection$2.run(AbstractConnection.java:540)
	at org.eclipse.jetty.util.thread.QueuedThreadPool.runJob(QueuedThreadPool.java:610)
	at org.eclipse.jetty.util.thread.QueuedThreadPool$3.run(QueuedThreadPool.java:539)
	at java.lang.Thread.run(Thread.java:744)
```

Find out the source code
```java
/** Decompress the byte array previously returned by
   *  compress */
  public static byte[] decompress(byte[] value) throws DataFormatException {
    // Create an expandable byte array to hold the decompressed data
    ByteArrayOutputStream bos = new ByteArrayOutputStream(value.length);

    Inflater decompressor = new Inflater();

    try {
      decompressor.setInput(value);

      // Decompress the data
      final byte[] buf = new byte[1024];
      while (!decompressor.finished()) {
        int count = decompressor.inflate(buf);
        bos.write(buf, 0, count);
      }
    } finally {  
      decompressor.end();
    }

    return bos.toByteArray();
  }

```

So these threads are while loop for decompress byte array.

the compress field is already deprecated in 2.9.0+ version.
```java
int bits = fieldsStream.readByte() & 0xFF;
      assert bits <= (FieldsWriter.FIELD_IS_NUMERIC_MASK | FieldsWriter.FIELD_IS_COMPRESSED | FieldsWriter.FIELD_IS_TOKENIZED | FieldsWriter.FIELD_IS_BINARY): "bits=" + Integer.toHexString(bits);

boolean compressed = (bits & FieldsWriter.FIELD_IS_COMPRESSED) != 0;
   assert (compressed ? (format < FieldsWriter.FORMAT_LUCENE_3_0_NO_COMPRESSED_FIELDS) : true)
     : "compressed fields are only allowed in indexes of version <= 2.9";
```

Then I found a potential Scenario to re-produce this issue. if pass empty byte [] to Inflater.decompress, decompressor.finished() will be false, the go to infinite loop;

test code
```java
package com.newegg.ec.bigdata.hbasetool;
import java.util.zip.Inflater;

public class CompressTest {
	public static void main(String[] args) {
		     final Inflater decompressor = new Inflater();
		     final byte[] noBytes = new byte[0];
		     decompressor.setInput(noBytes);
		     final boolean notFinished = false == decompressor.finished();
		     System.err.println("notFinished?: "+notFinished);
		     assert notFinished;
	}
}
```

This server use the OpenJDK
```shell
$> java -version
java version "1.7.0_45"
OpenJDK Runtime Environment (rhel-2.4.3.3.el6-x86_64 u45-b15)
OpenJDK 64-Bit Server VM (build 24.45-b08, mixed mode)
```


Solutions (Need to watch result):
  * replace java.util.zip by a new compress provider
  * disable decompress field?
  * hack code to avoid this infinite loop

Simliar issues.
Index corruption can cause infinite spin loop when Lucene attempts to incorrectly uncompress fields
https://issues.apache.org/jira/browse/LUCENE-772
Remove deprecated Field.Store.COMPRESS
https://issues.apache.org/jira/browse/LUCENE-1960
empty input array causes infinite loop in o.a.l.document.CompressionTools#decompres
http://mail-archives.apache.org/mod_mbox/lucene-dev/201007.mbox/%3CAANLkTikzG8g4wNqxrgHvQ_42fCKXnBVzv1h9HptACZgM@mail.gmail.com%3E

-EOF-

Tao Wu@CA, USA

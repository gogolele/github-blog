title: Hive Table for Apache HTTP Log
date: 2015-10-14 09:25:00
categories: dev
tags: hive
---

We have these kind of apache http Log
```txt
179.214.59.110 - - [01/Aug/2015:21:48:45 +0000] "GET /images1.newegg.com/WebResource/Scripts/USA/TP_jQueryPlugin/jquery-1.10.2.min.js?purge=1 HTTP/1.1" 200 33164 "-" "Mozilla/5.0 (Windows NT 6.3; Win64; x64; Trident/7.0; rv:11.0) like Gecko" "-"
179.214.59.110 - - [01/Aug/2015:21:48:45 +0000] "GET /images1.newegg.com/WebResource/Themes/2005/CSS/USA/font-awesome.v1.w.10825.0.css HTTP/1.1" 200 5340 "-" "Mozilla/5.0 (Windows NT 6.3; Win64; x64; Trident/7.0; rv:11.0) like Gecko" "-"
179.214.59.110 - - [01/Aug/2015:21:48:45 +0000] "GET /images1.newegg.com/WebResource/Themes/2005/CSS/USA/googleAdsense.v1.w.8730.2.css HTTP/1.1" 200 1615 "-" "Mozilla/5.0 (Windows NT 6.3; Win64; x64; Trident/7.0; rv:11.0) like Gecko" "-"
179.214.59.110 - - [01/Aug/2015:21:48:45 +0000] "GET /images1.newegg.com/WebResource/Themes/2005/CSS/USA/newegg.v1.w.12235.0.css HTTP/1.1" 200 52642 "-" "Mozilla/5.0 (Windows NT 6.3; Win64; x64; Trident/7.0; rv:11.0) like Gecko" "-"
179.214.59.110 - - [01/Aug/2015:21:48:45 +0000] "GET /images1.newegg.com/WebResource/Scripts/USA/NeweggJS/NEG.0.2.2.js HTTP/1.1" 200 7878 "-" "Mozilla/5.0 (Windows NT 6.3; Win64; x64; Trident/7.0; rv:11.0) like Gecko" "-"
```

the schema should be

```shell
client_ip - - [utc_time] "method url protocol" status size "-" "client" "-"
```

To map this table into hive, we should use Regex Row Format for hive.

```sql
CREATE EXTERNAL TABLE `akamai_log`(
  `ip` string COMMENT '',
  `time_local` string COMMENT '',
  `method` string COMMENT '',
  `uri` string COMMENT '',
  `protocol` string COMMENT '',
  `status` string COMMENT '',
  `bytes_sent` string COMMENT '',
  `referer` string COMMENT '',
  `useragent` string COMMENT '')
PARTITIONED BY (
  `dt` string)
ROW FORMAT SERDE
  'org.apache.hadoop.hive.serde2.RegexSerDe'
WITH SERDEPROPERTIES (
  'input.regex'='^(\\S+) \\S+ \\S+ \\[([^\\[]+)\\] \"(\\w+) (\\S+) (\\S+)\" (\\d+) (\\d+) \"([^\"]+)\" \"([^\"]+)\".*')
STORED AS INPUTFORMAT
  'org.apache.hadoop.mapred.TextInputFormat'
OUTPUTFORMAT
  'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION
  'hdfs://nameservice1/user/hive/warehouse/dfis.db/akamai_log'
```

For date partition, we add a dt column to represent date. and store http log data in "dt=yyyymmdd" folder
```shell
[ttt@server01 ~]$ hadoop fs -ls -R /tmp/akamai_log/ | head
drwxr-xr-x   - ttt supergroup          0 2015-10-13 16:44 /tmp/akamai_log/dt=20150802
-rw-r--r--   3 ttt supergroup   47965319 2015-10-13 16:39 /tmp/akamai_log/dt=20150802/AkamaiNewegg_9241.esclf_S.201508020000-0100-1
-rw-r--r--   3 ttt supergroup   47963864 2015-10-13 16:39 /tmp/akamai_log/dt=20150802/AkamaiNewegg_9241.esclf_S.201508020000-0100-10
-rw-r--r--   3 ttt supergroup   47951161 2015-10-13 16:39 /tmp/akamai_log/dt=20150802/AkamaiNewegg_9241.esclf_S.201508020000-0100-11
-rw-r--r--   3 ttt supergroup   47952753 2015-10-13 16:39 /tmp/akamai_log/dt=20150802/AkamaiNewegg_9241.esclf_S.201508020000-0100-13
-rw-r--r--   3 ttt supergroup   47961626 2015-10-13 16:39 /tmp/akamai_log/dt=20150802/AkamaiNewegg_9241.esclf_S.201508020000-0100-20
-rw-r--r--   3 ttt supergroup   47957557 2015-10-13 16:39 /tmp/akamai_log/dt=20150802/AkamaiNewegg_9241.esclf_S.201508020000-0100-22
-rw-r--r--   3 ttt supergroup   47960924 2015-10-13 16:39 /tmp/akamai_log/dt=20150802/AkamaiNewegg_9241.esclf_S.201508020000-0100-24
-rw-r--r--   3 ttt supergroup   47958371 2015-10-13 16:39 /tmp/akamai_log/dt=20150802/AkamaiNewegg_9241.esclf_S.201508020000-0100-25
-rw-r--r--   3 ttt supergroup   47949473 2015-10-13 16:39 /tmp/akamai_log/dt=20150802/AkamaiNewegg_9241.esclf_S.201508020000-0100-26
```

then we can query this log by hive sql / presto sql.
```shell
hive> select * from default.akamai_log where dt='20150802' limit 10;
OK
179.214.59.110	01/Aug/2015:21:48:45 +0000	GET	/images1.newegg.com/WebResource/Scripts/USA/TP_jQueryPlugin/jquery-1.10.2.min.js?purge=1	HTTP/1.1	200	33164	-	Mozilla/5.0 (Windows NT 6.3; Win64; x64; Trident/7.0; rv:11.0) like Gecko	20150802
179.214.59.110	01/Aug/2015:21:48:45 +0000	GET	/images1.newegg.com/WebResource/Themes/2005/CSS/USA/font-awesome.v1.w.10825.0.cssHTTP/1.1	200	5340	-	Mozilla/5.0 (Windows NT 6.3; Win64; x64; Trident/7.0; rv:11.0) like Gecko	20150802
179.214.59.110	01/Aug/2015:21:48:45 +0000	GET	/images1.newegg.com/WebResource/Themes/2005/CSS/USA/googleAdsense.v1.w.8730.2.cssHTTP/1.1	200	1615	-	Mozilla/5.0 (Windows NT 6.3; Win64; x64; Trident/7.0; rv:11.0) like Gecko	20150802
179.214.59.110	01/Aug/2015:21:48:45 +0000	GET	/images1.newegg.com/WebResource/Themes/2005/CSS/USA/newegg.v1.w.12235.0.css	HTTP/1.1	200	52642	-	Mozilla/5.0 (Windows NT 6.3; Win64; x64; Trident/7.0; rv:11.0) like Gecko	20150802
179.214.59.110	01/Aug/2015:21:48:45 +0000	GET	/images1.newegg.com/WebResource/Scripts/USA/NeweggJS/NEG.0.2.2.js	HTTP/1.1200	7878	-	Mozilla/5.0 (Windows NT 6.3; Win64; x64; Trident/7.0; rv:11.0) like Gecko	20150802
179.214.59.110	01/Aug/2015:21:48:45 +0000	GET	/images1.newegg.com/WebResource/Scripts/USA/TP_jQueryPlugin/jquery-migrate-1.2.1.min.js	HTTP/1.1	200	3437	-	Mozilla/5.0 (Windows NT 6.3; Win64; x64; Trident/7.0; rv:11.0) like Gecko	20150802
179.214.59.110	01/Aug/2015:21:48:45 +0000	GET	/images1.newegg.com/WebResource/Scripts/USA/Common/GgewenFramework.v1.w.11862.0.js	HTTP/1.1	200	11628	-	Mozilla/5.0 (Windows NT 6.3; Win64; x64; Trident/7.0; rv:11.0) like Gecko	20150802
179.214.59.110	01/Aug/2015:21:48:45 +0000	GET	/images1.newegg.com/WebResource/Scripts/USA/Common/WebApplication.v1.w.11621.0.jsHTTP/1.1	200	6470	-	Mozilla/5.0 (Windows NT 6.3; Win64; x64; Trident/7.0; rv:11.0) like Gecko	20150802
189.61.24.121	01/Aug/2015:21:48:45 +0000	GET	/images1.newegg.com/brandimage/Brand1149.gif	HTTP/1.1	200	2289	-Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/44.0.2403.125 Safari/537.36	20150802
179.214.59.110	01/Aug/2015:21:48:45 +0000	GET	/images1.newegg.com/WebResource/Themes/2005/CSS/USA/opinionlab.v1.w.9165.0.css	HTTP/1.1	200	3806	-	Mozilla/5.0 (Windows NT 6.3; Win64; x64; Trident/7.0; rv:11.0) like Gecko	20150802
Time taken: 2.569 seconds, Fetched: 10 row(s)
```
-EOF-

Tao Wu@CA, USA

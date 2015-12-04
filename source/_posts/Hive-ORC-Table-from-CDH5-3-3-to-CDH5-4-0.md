title: 'Hive ORC Table from CDH5.3.3 to CDH5.4.0 '
date: 2015-08-21 16:45:09
tags: [orcfile, distcp]
categories: dev
---

# Hive ORC Table from CDH5.3.3 to CDH5.4.0

```shell
$ hive> desc formatted basetable;
OK
# col_name            	data_type           	comment

isdelete            	int
checked             	string
shipbynewegg        	boolean
detailspecification 	string
recertifiedflag     	boolean
oemflag             	boolean
imagename           	string
manufacturerpartnumber	string
manufacturer        	string
subcategorycode     	int
adsnumber           	string
humanrating         	int
rating              	int
description         	string
packageweight       	double
arrivingdate        	string
websitesubcategorycode	int
ntcitemmark         	boolean
sellerid            	string
parentitem          	string
itemgroupid         	int
upccode             	string
sellerpartnumber    	string
title               	string
webdescription      	string
viewdescription     	string
linedescription     	string
bulletdescription   	string
itemnumber          	string

# Detailed Table Information
Database:           	mars
Owner:              	test
CreateTime:         	Sat Aug 01 07:09:31 PDT 2015
LastAccessTime:     	UNKNOWN
Protect Mode:       	None
Retention:          	0
Location:           	hdfs://nameservice1/user/hive/warehouse/data.db/basetable
Table Type:         	EXTERNAL_TABLE
Table Parameters:
	COLUMN_STATS_ACCURATE	false
	EXTERNAL            	TRUE
	numFiles            	268
	numRows             	-1
	rawDataSize         	-1
	totalSize           	17453204160
	transient_lastDdlTime	1438438171

# Storage Information
SerDe Library:      	org.apache.hadoop.hive.ql.io.orc.OrcSerde
InputFormat:        	org.apache.hadoop.hive.ql.io.orc.OrcInputFormat
OutputFormat:       	org.apache.hadoop.hive.ql.io.orc.OrcOutputFormat
Compressed:         	No
Num Buckets:        	-1
Bucket Columns:     	[]
Sort Columns:       	[]
Storage Desc Params:
	serialization.format	1
Time taken: 0.117 seconds, Fetched: 60 row(s)


$ hive> show create table basetable
CREATE EXTERNAL TABLE `basetable`(
  `isdelete` int,
  `checked` string,
  `shipbynewegg` boolean,
  `detailspecification` string,
  `recertifiedflag` boolean,
  `oemflag` boolean,
  `imagename` string,
  `manufacturerpartnumber` string,
  `manufacturer` string,
  `subcategorycode` int,
  `adsnumber` string,
  `humanrating` int,
  `rating` int,
  `description` string,
  `packageweight` double,
  `arrivingdate` string,
  `websitesubcategorycode` int,
  `ntcitemmark` boolean,
  `sellerid` string,
  `parentitem` string,
  `itemgroupid` int,
  `upccode` string,
  `sellerpartnumber` string,
  `title` string,
  `webdescription` string,
  `viewdescription` string,
  `linedescription` string,
  `bulletdescription` string,
  `itemnumber` string)
ROW FORMAT SERDE
  'org.apache.hadoop.hive.ql.io.orc.OrcSerde'
STORED AS INPUTFORMAT
  'org.apache.hadoop.hive.ql.io.orc.OrcInputFormat'
OUTPUTFORMAT
  'org.apache.hadoop.hive.ql.io.orc.OrcOutputFormat'
LOCATION
  'hdfs://nameservice1/user/hive/warehouse/data.db/basetable'

$ hadoop distcp hdfs://nameservice1/user/hive/warehouse/data.db/basetable hdfs://10.0.0.1:8020/tmp/
$ sudo -u hdfs hadoop fs -mv /tmp/basetable /user/bi_coremetrics/basetable
$ sudo -u hdfs hadoop fs -chown -R hiveuser /user/bi_coremetrics/basetable
```

-EOF-

TAO WU@CA, USA

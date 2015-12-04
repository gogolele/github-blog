title: One Line Shell to Get te HBase Spilt Region Keys
date: 2015-06-27 10:50:42
categories: dev
tags: [Shell, HBase]
---

tr: join multiple lines to one line;
sed: to replace str
rev: to reverse a str
cut: to cut off some character in a str

```shell
[root@server]# vi scan-meta.sh
scan 'hbase:meta', { FILTER=>"PrefixFilter('ecitem:IM_ItemBase')"}
[root@server]# cat scan-meta.sql | hbase shell | tr "\n" "," | sed 's/,/\n/g' | grep -E 'START|END'| awk '{print $3}' | sed 's/}//g' | uniq | tr "\n" "," | rev | cut -c 5- | rev
```

-EOF-

title: 一个计算每秒钟请求次数和平均响应时间的shell脚本
date: 2014-10-08 11:29:19
categories: dev
tags: Shell
---

做一个shell脚本，用到了awk、sed和sort等进行每秒钟的请求次数和平均响应时间计算

```shell
cat hbase-request.log | awk '{printf "%-8s %-8s\n", $2, $19}'
| awk '$2> 1' | sed "s/,/ /g" | awk '{printf "%-8s %-8s\n",$1,$3}'
| awk '{a[$1]+=$2;b[$1]++} END {for (i in a) {printf "%-8s %-8s %-8s\n",
i,a[i]/b[i],b[i]};}' | sort -k 1 > total-minutes.log
```

解释

1. awk取出时间和包含响应时间的column
```shell
awk '{printf "%-8s %-8s\n", $2, $19}'
```

2. 取出时间大于1ms的数据
```shell
awk '$2>1'
```

3. 做一个替换
```shell
sed "s/,/ /g"
```

4. awk做sum和count
```shell
awk '{a[$1]+=$2;b[$1]++} END {for (i in a) {printf "%-8s %-8s %-8s\n",
i,a[i]/b[i],b[i]};}'
```

5. sort排序
```shell
sort -k 1
```

-EOF-

伍涛@USA

2014-10-08

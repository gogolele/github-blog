title: Linux Command for Ops
date: 2014-10-08 11:53:46
categories: dev
tags: [Shell, Linux]
---

运维工作中常见的linux shell命令

1.查看磁盘空间

	```shell
	df -h
	```

2.查看当前目录文件夹空间占用情况

	```shell
	du -h --max-depth=1
	```

3.查看线程的cpu使用情况

	```shell
	top -H -p pid
	```

  进一步可以通过jstack去查看，哪个线程占用的cpu高，可以定位到具体方法。
  需要注意，top中的线程编号是10进制，jstck中的线程编号是16进制，需要转换一下。

  10进制转换为16进制

	```shell
	printf "%x\n" 4095
	```

  16进制转换为10进制

  ```shell
  printf "%d\n" 0x10
  ```

4.生成sha-256 hashcode

  ```shell
  echo -n foobar | sha256sum
  ```

5.date转换为timestamp

  ```shell
  date +%s
  date -j -f "%Y-%m-%d" "2010-10-02" "+%s"
  ```

6.timestamp转换为date

  ```shell
  date -d @1234567890      # linux
  date -r 1234567890       # bsd
  ```

7.查看内存的使用情况

  ```shell
  free -m
  ```

 这个命令会提高明确的可用和已用内存，按照如下方法可以计算。
 可用内存为 free ＋ buffers ＋ cached
 已有内存为 used － buffers － cached

8.查看disk是否为ssd

  ```shell
  cat /sys/block/sda/queue/rotational
  ```
  SSD是非转动磁盘，0表示为SSD硬盘。

  ```shell
  sudo yum install smartmontools
  sudo smartctl -a /dev/sda
  ```
  这个命令可以查看硬盘具体的产商、型号，这样就可以查询是否为ssd硬盘。

9.查看centos的版本

  ```shell
  cat /proc/version        # kernel version
  uname -a                 # kernel version
  cat /etc/redhat-release  # centos version
  ```

10.硬盘速度测试
  简单的写速度测试

  ```shell
  ## wrting testing
  dd if=/dev/zero of=test bs=64k count=16k conv=fdatasync
  ## read testing
  dd if=/dev/sda of=/dev/null
  ```

11.TCP Dump for package analysis

	```shell
	sudo tcpdump -w /tmp/network55.log -i eth2 host 172.16.21.14 and port 60020
  ```

12.Java Remote Debug

	```shell
	java -Xdebug -Xrunjdwp:transport=dt_socket,address=8788,server=y,suspend=y
	```

13. grep from folder
```shell
grep -nr XXX *
```

14. downlod all the files from url
```shell
wget -r -np -k http://172.16.22.83/DataFeeds/A1F2/Inventory/2015-08-05/
```

15. delete svn info
```shell
find . -type d -name ".svn"|xargs rm -rf
```

16. git add all files
```shell
git add A .
```

17. tree  shell
```shell
#!/bin/bash
# only if you have bash 4 in your CentOS system
shopt -s globstar
for file in **/*
do
    slash=${file//[^\/]}
    case "${#slash}" in
        0) echo "|-- ${file}";;
        1) echo "|   |--  ${file}";;
        2) echo "|   |   |--  ${file}";;
    esac
done
```

18. hdfs batch chown
```shell
hadoop fs -ls -R /hbase/archive/data/default/IM_ItemPrice/ | grep -E 'rd29|bigdata' | awk '{print $8}' | xargs sudo -u hdfs hadoop fs -chown -R hbase:hbase
```


-EOF-


TAO WU@CA, USA

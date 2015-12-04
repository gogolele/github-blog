title: ssh deploy ulimit change
date: 2015-08-20 21:57:35
tags: [ssh, expect]
categories: dev
---
## perpare file to change ulimit
90-nproc.conf
```shell
# Default limit for number of user's processes to prevent
# accidental fork bombs.
# See rhbz #432903 for reasoning.

*          soft    nofile     65535
*          hard    nofile     65535
*          soft    nproc     65535
*          hard    nproc     65535
root       soft    nproc     unlimited
```

## execute sudo with automatically fill password
expect-cmd.sh

```shell
#!/usr/bin/expect
set timeout 60000
set password ""
set command [lindex $argv 0]
eval spawn $command
# This will match the ": " at the end. That is why $ symbol used here.
expect ": $"; # It will be more robust than giving as below
  #expect "\[sudo\] password for hduser:"
send "$password\r"
#This will match the literal '$' symbol, to match the '$' in terminal prompt
expect eof { puts "expect command completed successfully. :)" }
```

## break all the ssh pass without password

## ssh distributed deploy
ssh-deploy.sh
```shell
#!/bin/sh
#
# Wrote by Tao Wu
# 2015-04
set timeout 60000
set password ""
#hosts=('server21')
hosts=('server22' 'server23' 'server24' 'server25' 'server26' 'server27' 'server28' 'server29' 'server30' 'server31' 'server32' 'server33' 'server34' 'server35' 'server36' )
for m in ${hosts[@]};
do
scp ~/90-nproc.conf $m:~/
./expect-cmd.sh "ssh -t $m \"sudo mv /home/taowu/90-nproc.conf /etc/security/limits.d/\""
done
```

-EOF-

TAO WU@CA, USA

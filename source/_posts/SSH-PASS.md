title: SSH打通无密码登录脚本
date: 2014-11-07 09:54:18
categoiries: dev
tags: SSH
---

在我们管理大量机器时，我们通常会打通ssh无密码登录，以对sever统一的部署管理。

通常我们会才有ssh－keygen产生公钥，然后scp到目标机器上。ssh－copy－id实现了这个功能，
但是仍然需要交互式确认、输入密码。

如下的脚本，采用 ssh ＋ expect的方式，写好目标机器的用户名密码，即可自动打通ssh无密码登录。

```shell
#!/bin/bash
#
# Copyright (c) Finnbarr P. Murphy 2006
# modified by Tao Wu, 2014
#

# ---- start configurables ----
PATH=/usr/sbin:/usr/bin:/sbin:/bin


# ----- end configurables -----


function usage {
printf "Usage: setupkeys -U remote-username -P remote-password -H remote-host -u local-username\n"
exit 2
}


# --- script starts here
echo

(( $# == 0 )) && usage

username=""
password=""
host=""
localuser=""

while getopts "u:P:U:H:" OPTION
do
case $OPTION in
U) username="$OPTARG";;
P) password="$OPTARG";;
H) host="$OPTARG";;
u) localuser="$OPTARG";;
esac
done


# --- basic argument checking
if [[ -z "$username" ]]; then
echo "ERROR - No username entered."
exit 1
fi
if [[ -z "$password" ]]; then
echo "ERROR - No passed entered."
exit 1
fi
if [[ -z "$host" ]]; then
echo "ERROR - No host entered."
exit 1
fi
if [[ -z "$localuser" ]]; then
echo "ERROR - No localuser entered."
exit 1
fi

echo -n "Checking connectivity with $host. "
/bin/ping -q -c 2 $host > /dev/null 2>&1
RESULT=$?
if (( RESULT == 1 )); then
echo; echo "ERROR - could not ping $host."
exit 1
fi
echo "System is alive."

# --- check for $localuser public and private ssh keys
# --- we need to be $localuser here when using ssh-* utilities
echo -n "Checking for $localuser ssh key files. "
SSH_KEYS_FOUND=0
if [[ -d $HOME ]]; then
if [[ -s $HOME/.ssh/id_rsa && -s $HOME/.ssh/id_rsa.pub ]]; then
/usr/bin/ssh-keygen -e -f $HOME/.ssh/id_rsa.pub | grep "2048-bit RSA" > /dev/null 2>&1
RESULT=$?
if (( RESULT == 0 )); then
SSH_KEYS_FOUND=1
fi
fi
fi
if (( SSH_KEYS_FOUND == 1 )); then
echo "Found"
else
echo "Not found"
rm -rf $HOME/.ssh
mkdir $HOME/.ssh
chmod 700 $HOME/.ssh
chown -R $localuser:$localuser $HOME/.ssh
/usr/bin/ssh-keygen -q -t rsa -N "" -f $HOME/.ssh/id_rsa
echo "New ssh key files generated (RSA protocol)"
fi
# --- add $localname's public key to $username@$host authorized keys
TMPUSER=expectscript-user.$$

cat <<EOT > $TMPUSER
#!/usr/bin/expect

if {[llength \$argv] != 4} {
puts "usage: \$argv0 localuser username password host"
exit 1
}

log_file -a expectscript-user.log
log_user 0

set localuser [lindex \$argv 0]
set username [lindex \$argv 1]
set password [lindex \$argv 2]
set host [lindex \$argv 3]

set timeout 60

spawn /usr/bin/ssh-copy-id -i \$localuser/.ssh/id_rsa.pub \$username@\$host
expect {
"assword: " {
send "$password\n"
expect {
"again." { exit 1 }
"expecting." { }
timeout { exit 1 }
}
}
"(yes/no)? " {
send "yes\n"
expect {
"assword: " {
send "$password\n"
expect {
"again." { exit 1 }
"expecting." { }
timeout { exit 1 }
}
}
}
}
}

exit 0
EOT

echo -n "Copying $localuser's public key to $host. "
chmod 755 $TMPUSER
sleep 3
./$TMPUSER $HOME $username $password $host
RESULT=$?
rm -f $TMPUSER
if (( RESULT == 0 )); then
echo "Succeeded"
rm -f expectscript-user.log
else
echo "Failed"
echo "ERROR: Check expectscript-user.log"
exit 1
fi

# we are done
echo "Setup completed. Goodbye."
exit 0
```


-------EOF---------

TAO WU@USA

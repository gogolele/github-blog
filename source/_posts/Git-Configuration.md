title: Git Configuration
date: 2015-07-20 16:48:43
categories: dev
tags: Git
---

# git configuration for 2 git repository
## generate keys for each email
```shell
$ cd ~/.ssh
$ ssh  ssh-keygen -t ras -C "youremail@yourcompany.com" -f ~/.ssh/id_rsa_company
$ ssh  ssh-keygen -t ras -C "githubuser@gmail.com" -f ~/.ssh/id_rsa_github
```

## add config in ~/.ssh directory
```shell
$ vi ~/.ssh/config
```

```shell
# gitlab
Host gitlab
     HostName gitlab
     PreferredAuthentications publickey
     IdentityFile ~/.ssh/id_rsa_company

# github
Host github.com
     HostName github.com
     PreferredAuthentications publickey
     IdentityFile ~/.ssh/id_rsa_github
```
## add ssh keys into github and gitlab

## test connection
```shell
git -T git@github.com
git -T git@gitlab
```

# git config (proxy and gitignore)
```shell
$ vi ~/.gitconfig
```

```shell
[user]
	name = =
	email = =
[http]
	proxy = http://host:port
[https]
	proxy = http://host:port
[credential]
[core]
	excludesfile = /home/tw79/.gitignore
```

# git ignore for java and Eclipse

```shell
$ vi ~/.gitignore
```

```shell

*.class
.pmd

######################
# Eclipse
######################

*.pydevproject
.project
.metadata
bin/**
tmp/**
tmp/**/*
*.tmp
*.bak
*.swp
*~.nib
local.properties
.classpath
.settings/
.loadpath

# Directories #
/build/
/bin/
target/
lib/
logs/
log/
# Mobile Tools for Java (J2ME)
.mtj.tmp/

# Package Files #
*.jar
*.war
*.ear
```

-EOF-

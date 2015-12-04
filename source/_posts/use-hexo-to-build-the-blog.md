title: 使用Hexo搭建基于Github的静态博客
date: 2013-07-26 14:34:56
categories: dev
tags: Hexo
---

## 1. 缘起

最近试图在Github上搭建博客,目的其实是为了有个地方可以写团队的DevBlog,我想说我做了了若干尝试,但是不太会搞.汗^-^.
Octopress挺酷的,但是真的不是很方便,不知道大家是否同意.我看鬼厉的博客luoli523就是基于Octopress和Github的,看起来真不错,鬼哥是有Taste的人.
BeiYuu的博客基于Jekyll,我觉得也挺好看了.fork下来改了几下,觉得还是麻烦.
后来看到了hexo,尝试着用了一下,咦,还不错哦,清爽,轻快,我喜欢. 主要参考的是官方的指南,写得很清晰.这个文章use-hexo也给了我帮助,致谢.
## 2. 动手

### 2.1 我的环境
```
CentOS release 6.3 (Final)
2.6.32-279.el6.x86_64
```

### 2.2 本地安装

安装git

```
yum install git
```

git的ssh-key，我就不说了吧，把key拷贝进入github的个人设置中......

```
ssh-keygen -t rsa
cat ~/.ssh/id_rsa.pub
```

安装nodejs
```
wget http://nodejs.org/dist/v0.11.0/node-v0.11.0-linux-x64.tar.gz
tar zxvf node-v0.11.0-linux-x64.tar.gz
```

为node.js修改环境变量
```
vim ~/.bashrc
NODEJS_HOME=/home/bigdata/node-v0.11.0-linux-x64
PATH=$NODEJS_HOME/bin:$PATH
export NODEJS_HOME
export PATH
```

```
source ~/.bashrc
```

检查node.js是否安装成功
```
node --version
v0.11.0
```
安装npm
```
curl -s https://npmjs.org/install.sh > npm-install-$$.sh
sh npm-install-*.sh
```

安装hexo
```
npm install hexo -g
hexo init tonywutao.github.io
cd tonywutao.github.io
```

配置修改 Site部分的自定义配置
```
title: Blog of Tonywutao
subtitle:
description: Dev, Life, Talk
author: Tonywutao
email: tonywutao@gmail.com
language: zh-CN

# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: http://tonywutao.github.io
root: /
permalink: :year/:month/:day/:title/
tag_dir: tags
archive_dir: archives
category_dir: categories
code_dir: downloads/code
```

Git Deploy的配置
```
# Deployment
## Docs: http://zespia.tw/hexo/docs/deploy.html
deploy:
    type: github
    repository: git@github.com:tonywutao/tonywutao.github.io.git
    branch: master
```

Disqus的配置，disqus当然需要你先去disqus上注册，并为自己的博客地址生成key
```
# Disqus
disqus_shortname: blogoftonywutao
```

Theme的修改 我比较喜欢memoir这个主题，将这个主题git clone到theme目录下，并命名为memoir即可
```
git clone <repository> themes/<theme-name>
```

theme的地址为
```
https://github.com/tommy351/hexo/wiki/Themes/
```
我执行的操作是
```
git clone git@github.com:yhben/hexo-theme-memoir.git theme/memoir
```

至此，其实就装好了，后面就是写博客的事情了。
```
hexo new "new post"     #新写一个md的日志
hexo generate           #生成静态文件
hexo server             #启动本地服务，可以查看本地博客是否正常
```

部署到github
```
hexo deploy
```
或者
```
hexo deploy --generate
```
要删除部署的文件
```
rm -rf .deploy
```

这样就差不多搞定了。


### 2.3 建立github pages

回过头来再说一下github pages的构建，只需在github登录后，新建一个tonywutao.github.io的project。然后在setting中publish pages。大约等10多分钟，就可以通过 tonywutao.github.io看到网站了。在hexo中，配置的远程git仓库，应指向这个github上的仓库。

### 2.4 写MardDown的工具

我的OS是windows，我是通过在线工具写的，我也尝试过sublime这样的工具，我认为最重要的是实时预览功能，不然太折腾了。
```
http://benweet.github.io/stackedit/
```

-EOF-

伍涛@成都
2013-07-27

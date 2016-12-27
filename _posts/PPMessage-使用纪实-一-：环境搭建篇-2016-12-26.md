---
title: PPMessage 使用踩坑：环境搭建篇
tags:
  - PPMessage
  - Python
  - Mac
  - Redis
categories:
  - PPMessage
  - Python
date: 2016-12-26 10:19:07
---


>前言

最近项目想整一个类似`客服`系统的东西，网上找了找，整了一个叫 `PPMessage` 的开源项目叫研究，拿来看一看。今天讲一下使用的情况，首先今天说一下如何搭建这个东西（其实在[`GitHub`](https://github.com/PPMESSAGE/ppmessage)上有说明，但是具体上步骤还是有些问题，就来简单讲一讲）。
<!--- more --->

<br/>
# 下载项目

<br/>

在`GitHub`上你能找到在：

- *Linux/Ubuntu*
- *Mac*
- *Windows*

三个系统里的部署操作，我的是 *Mac*，*Linux/Ubuntu* 和 *Mac* 差不多，*Windows* 的略微麻烦。本质上差不多，因为这个项目是基于 `Python` 开发的，所以搭建的工作基本上就是下载项目的各种依赖。

首先，通过下载项目源码到本地，不多说，都知道的（当然官方上还有另外一个路子是使用 `Docker` 来进行搭建）：

```
git clone https://github.com/PPMESSAGE/ppmessage.git
```

<br/>

#  安装 Python && pip
<br/>
之后如果你按照官方的操作，*cd ppmessage* 到项目的目录下，执行 `bash` 的话，十有八九就会跪掉。。。因为项目需要的各种东西都需要你一步步来下载部署，不要看项目主页上就那么两三行命令，到后面你就知道了喝两三杯咖啡不一定能搞定。首先如果是 *Mac* 系统的话，我的是 `OS X EI Capitan 10.11`,装有 `Python` （版本2.7.10），但是并没有安装 `pip`，需要先按照这个，否则的话，你一上来就会报错的。

<br/>
## 替换镜像 

<br/>
在安装 `pip` 之前，我们要把镜像改为国内的，我相信现在很多的开发，无论你是 `Android` 还是 `Python` 的，基本上都要配置一把国内的镜像（原因你懂）。百度一下有很多文章，我用的是 `豆瓣` 的镜像。根据你自己的系统情况，在下面的目录中添加文件路径和文件：

- Mac  下： Users/xxx/.pip/pip.conf 
- Windows 下： Users/xxx\pip\pip.ini 

没有路径和文件的自行添加，然后在 `pip.conf/pip.ini` 中添加：


```
[global] 

index-url = http://pypi.douban.com/simple 

trusted-host = pypi.douban.com 
```

因为这个镜像不支持 *`HTTPS`*，在最后一定要添加 `trusted-host`，否则的话，执行后面的操作会出现一些文件，比如类似这样的：

```
 The repository located at pypi.douban.com is not a trusted or secure host and is being ignored. If this repository is available via HTTPS it is recommended to use HTTPS instead, otherwise you may silence this warning and allow it anyways with '--trusted-host pypi.douban.com'.
  Could not find a version that satisfies the requirement apns2 (from versions: )
No matching distribution found for apns2
```

<br/>
## 安装 pip 

<br/>

然后，在 `Terminal` 中执行：

```
sudo easy_install pip 
```
<br/>

# 安装依赖 

<br/>
OK，下面就来到这一步的操作了，执行：

```
bash ppmessage/scripts/set-up-ppmessage-on-mac.sh
```
然后，看到这样的：

```
XXXXMac:ppmessage-master xxx$ bash ppmessage/scripts/set-up-ppmessage-on-mac.sh 
==> This script will install:
/usr/local/bin/brew
/usr/local/share/doc/homebrew
/usr/local/share/man/man1/brew.1
/usr/local/share/zsh/site-functions/_brew
/usr/local/etc/bash_completion.d/brew
/usr/local/Homebrew

Press RETURN to continue or any other key to abort
==> Downloading and installing Homebrew...
remote: Counting objects: 4288, done.
remote: Compressing objects: 100% (2824/2824), done.
remote: Total 4288 (delta 2222), reused 2762 (delta 1316), pack-reused 0
Receiving objects: 100% (4288/4288), 2.46 MiB | 24.00 KiB/s, done.
Resolving deltas: 100% (2222/2222), done.
From https://github.com/Homebrew/brew
 * [new branch]      master     -> origin/master
 * [new tag]         0.1        -> 0.1
 * [new tag]         0.2        -> 0.2
 * [new tag]         0.3        -> 0.3
 ....
 ....
==> Next steps:
- Run `brew help` to get started
- Further documentation: 
    https://git.io/brew-docs
Updating Homebrew...
==> Downloading https://homebrew.bintray.com/bottles/autoconf-2.69.el_capitan.bottle.4.tar.gz
######################################################################## 100.0%
==> Pouring autoconf-2.69.el_capitan.bottle.4.tar.gz
==> Caveats
Emacs Lisp files have been installed to:
  /usr/local/share/emacs/site-lisp/autoconf
==> Summary
🍺  /usr/local/Cellar/autoconf/2.69: 70 files, 3.0M
==> Downloading https://homebrew.bintray.com/bottles/automake-1.15.el_capitan.bottle.2.tar.gz
######################################################################## 100.0%
==> Pouring automake-1.15.el_capitan.bottle.2.tar.gz
🍺  /usr/local/Cellar/automake/1.15: 130 files, 2.9M
==> Downloading https://homebrew.bintray.com/bottles/fdk-aac-0.1.4.el_capitan.bottle.1.tar.gz
######################################################################## 100.0%
==> Pouring fdk-aac-0.1.4.el_capitan.bottle.1.tar.gz
 
```

你要是打开刚才命令中的那个文件看看就知道了，里面内容大概就这样：

```

ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

brew install \
     autoconf \
     automake \
     fdk-aac \
     libtool \
     libmagic \
     libjpeg \
     libffi \
     lame \
     nodejs \
     redis

brew tap homebrew/services

# In Mac OS X EI Captain, if your encount below error when install green,

# OSError: [Errno 1] Operation not permitted: '/var/folders/nj/ky4gzkdn0db_wdxxyph3j93h0000gn/T/pip-xoS3tF-uninstall/System/Library/Frameworks/Python.framework/Versions/2.7/Extras/lib/python/six-1.4.1-py2.7.egg-info'

# try this command to install green:

# sudo pip install green --ignore-installed six

# Installing other package might have similar problem. In that case, use '--ignore-installed xxx' should do the trick.

# "pip install -i http://pypi.douban.com/simple xxx" might be faster
sudo pip install \
     apns2 \
     pillow \
     StringGenerator \
     beautifulsoup4 \
     filemagic \
     identicon \
     jieba \
     paramiko \
     paho-mqtt \
     ppmessage-mqtt \
     pypinyin \
     pyparsing \
     python-dateutil \
     python-gcm \
     python-magic \
     qrcode \
     readline \
     redis \
     sqlalchemy \
     tornado \
     xlrd


echo "Finish install the PPMessage requirements successfully, have fun with PPMessage"
```

恩，是的，就是安装一些依赖的命令的合集，并且里面也告诉你了使用 `豆瓣` 的镜像，并且可能的问题的解决方法。

<br/>

# 运行 PPMessage
<br/>

进行到这里了，最后一步了，执行这个：

```
./ppmessage.py 
```

如果成功的话，控制台应该会报出访问服务的地址信息，不过我就悲剧了，继续出错，报错这样：

```
Traceback (most recent call last):
  File "./ppmessage.py", line 5, in <module>
    import ppmessage
  File "/Proma/PPMessage/ppmessage/ppmessage-master/ppmessage/__init__.py", line 1, in <module>
    from . import backend
  File "/Proma/PPMessage/ppmessage/ppmessage-master/ppmessage/backend/__init__.py", line 1, in <module>
    from .main import _main
  File "/Proma/PPMessage/ppmessage/ppmessage-master/ppmessage/backend/main.py", line 22, in <module>
    from ppmessage.core.downloadhandler import DownloadHandler
  File "/Proma/PPMessage/ppmessage/ppmessage-master/ppmessage/core/downloadhandler.py", line 9, in <module>
    from ppmessage.db.models import FileInfo
  File "/Proma/PPMessage/ppmessage/ppmessage-master/ppmessage/db/models.py", line 16, in <module>
    from ppmessage.core.redis import row_to_redis_hash
  File "/Proma/PPMessage/ppmessage/ppmessage-master/ppmessage/core/redis.py", line 13, in <module>
    from sqlalchemy import String
ImportError: No module named sqlalchemy
```

定位最后一句这个报错，没有 `sqlalchemy`。OK，使用 `pip` 安装这个 *module* 就行了：

```
pip install sqlalchemy
```

但是，会出现新的 *module* 不存在的问题，比如我在接下来遇到了 `PIL` 的问题，关键是怎么都找不到这个 *module*，最后才发现这个玩意已经改名字了（新的名字 `pillow`，蛋疼啊 (⊙﹏⊙)b）。如果依赖一个个下载，想想头大啊。别怕，实际上，在 *ppmessage/scripts/requirements.txt* 这个目录下的文件里有需要的所有依赖，你直接执行：

```
pip install -r ppmessage/scripts/requirements.txt 
```

一次性就把所有的依赖都安装完啦。OK，再次执行，又报错了：


```
Traceback (most recent call last):
  File "./ppmessage.py", line 12, in <module>
    _main()
  File "./ppmessage.py", line 9, in _main
    ppmessage.backend._main()
  File "/Proma/PPMessage/ppmessage/ppmessage-master/ppmessage/backend/main.py", line 130, in _main
    _app.load_db_to_cache()
  File "/Proma/PPMessage/ppmessage/ppmessage-master/ppmessage/backend/main.py", line 101, in load_db_to_cache
    self.redis.flushdb()
  File "/Library/Python/2.7/site-packages/redis/client.py", line 652, in flushdb
    return self.execute_command('FLUSHDB')
  File "/Library/Python/2.7/site-packages/redis/client.py", line 578, in execute_command
    connection.send_command(*args)
  File "/Library/Python/2.7/site-packages/redis/connection.py", line 563, in send_command
    self.send_packed_command(self.pack_command(*args))
  File "/Library/Python/2.7/site-packages/redis/connection.py", line 538, in send_packed_command
    self.connect()
  File "/Library/Python/2.7/site-packages/redis/connection.py", line 442, in connect
    raise ConnectionError(self._error_message(e))
redis.exceptions.ConnectionError: Error 61 connecting to 127.0.0.1:6379. Connection refused. 
```


嗯？拒绝了，发了张好人卡。。。。不怕，排查下是不是 `Redis` 服务的问题。结果发现是服务没启，启动服务吧：

```
XXXXMac:~ xxx$ redis-server
526:C 26 Dec 14:13:17.587 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
526:M 26 Dec 14:13:17.589 * Increased maximum number of open files to 10032 (it was originally set to 256).
                _._                                                  
           _.-``__ ''-._                                             
      _.-``    `.  `_.  ''-._           Redis 3.2.6 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._                                   
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 526
  `-._    `-._  `-./  _.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |           http://redis.io        
  `-._    `-._`-.__.-'_.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |                                  
  `-._    `-._`-.__.-'_.-'    _.-'                                   
      `-._    `-.__.-'    _.-'                                       
          `-._        _.-'                                           
              `-.__.-'                                               

526:M 26 Dec 14:13:17.592 # Server started, Redis version 3.2.6
526:M 26 Dec 14:13:17.592 * The server is now ready to accept connections on port 6379
526:M 26 Dec 14:19:06.569 * 100 changes in 300 seconds. Saving...
526:M 26 Dec 14:19:06.569 * Background saving started by pid 566
566:C 26 Dec 14:19:06.570 * DB saved on disk
526:M 26 Dec 14:19:06.669 * Background saving terminated with success
```

蓝后，测试下连通性：

```
XXXXMac:~ xxx$ redis-cli
127.0.0.1:6379> set key "hellow world"
OK
127.0.0.1:6379> get key
"hellow world"
127.0.0.1:6379> 
```
成功了，运行 `PPMessage` 吧，然后通过你的 `Terminal` 显示的这个地址访问就行了，进到页面配置一些信息，重启下服务刷新页面就OK了。

```
[I 161226 14:16:58 main:139] Access http://192.168.191.3:8945/ to config PPMessage.
```

<br> 

# 最后
<br> 

## 关于镜像
<br> 

关于镜像这块，`豆瓣` 已有了支持  *`HTTPS`* 的：


```
[global] 

index-url = https://pypi.douban.com/simple 

```

另外，还有其他的可选，网上有很多，不过我自己试了一下，有的链接都打不开，这两个最快：

- 阿里云  http://mirrors.aliyun.com/pypi/simple/
- 中科大  https://mirrors.ustc.edu.cn/pypi/web/simple/






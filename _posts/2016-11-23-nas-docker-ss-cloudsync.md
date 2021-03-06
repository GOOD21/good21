---
layout:     post
title:      "Synology NAS同步Dropbox和GoogleDrive"
date:       2016-11-23
author:     "GOOD21"
header-img: "img/post-bg-js-module.jpg"
tags:
    - 生活
    - NAS
---

#### Cloud Sync
Synology DSM自带的**Cloud Sync**支持同步各种网盘到指定的文件夹。

其中**百度云**、**Onedrive**在国内同步没什么问题，但是**Dropbox**和**GoogleDrive**因为GFW的原因，只有**科学上网**才可以用。

#### Docker
Synolocy DSM里面的**Docker**简直屌爆了，有了它你能干的事情就多了。

首先，你要有一个VPS，搭一个Shadowsocks的Server。推荐搬瓦工的，这里就不细说了。

你可以在注册表里搜`shadowsocks-privoxy` 选择`gd41340811/shadowsocks-privoxy`。

![1](/img/in-post/nas-docker-ss-cloudsync/1.png)

下载之后在映像里点击`启动`-->`高级设置`。

![6](/img/in-post/nas-docker-ss-cloudsync/6.png)

在`端口设置`里，设置你的本地端口，默认是“自动”，你也可以指定固定的未被占用的端口。

![3](/img/in-post/nas-docker-ss-cloudsync/3.png)

在`环境`里，要加入你的Shadowsocks Server的`SERVER_ADDR`、`SERVER_PORT`、`PASSWORD`

![4](/img/in-post/nas-docker-ss-cloudsync/4.png)

设置完成之后，启动实例，在`控制面板`-->`网络`-->`代理服务器`-->`高级设置`，设置代理，其中http和https都对应容器端口8118的本地端口32771。(注：7070就是全局的ss代理，8118是provixy的pac代理，如果想要区分http和https的请求，在这里改就可以了)

![5](/img/in-post/nas-docker-ss-cloudsync/5.png)

之后，你的NAS就可以**科学上网**了。

#### privoxy

因为群晖NAS**只支持http代理**，所以必须要用**privoxy**。

`gd41340811/shadowsocks-privoxy`是根据bluebu这哥们改写的，他写的是代理全部，所有的请求都走shadowsocks代理，但是其实我们只需要Dropbox和GoogleDrive走代理。

>github地址：[https://github.com/GOOD21/shadowsocks-privoxy](https://github.com/GOOD21/shadowsocks-privoxy)
>dockerhub：[https://hub.docker.com/r/gd41340811/shadowsocks-privoxy/](https://hub.docker.com/r/gd41340811/shadowsocks-privoxy/)     
>欢迎 fork star

在privoxy里配置改为如下：

```
# forward-socks5  / 127.0.0.1:7070  .  # 打开就是代理全部请求
forward          /    .
forward-socks5  .dropbox*.com 127.0.0.1:7070  . # 代理dropbox的请求
forward-socks5  .*google*.* 127.0.0.1:7070  . # 代理googledrive相关请求
```

这里关于dropbox有个地方比较坑，几乎网上的文章写的配置都是这样的：

```
forward-socks5 .dropbox.com 127.0.0.1:7070 .
forward .dropbox.com:443 .
```

这样的话，在CloudSync里`暂停同步`之后再`恢复同步`是好用的，但是后续的10s一次检查就一直显示`连接中`，根据抓包的请求发现：

```
connecting cfl.dropboxstatic.com:443
connecting notify.dropboxapi.com:443
```

这些请求根本没走代理，改成`.dropbox*.com`之后就好使了。

在github上有个**gfwlist2privoxy**的repo，可以把所有gfwlist转换成privoxy的actionfiles，这样就实现了PAC。（然而感觉在NAS上并没有什么卵用...）

#### 按需同步

在CloudSync的设置里可以调整**轮询期**，默认是10s。

我是拿来做备份的，不需要实时性，所以改成了3600s。

![2](/img/in-post/nas-docker-ss-cloudsync/2.png)



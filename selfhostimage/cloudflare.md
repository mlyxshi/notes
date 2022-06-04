---
title: "苹果生态下自建图床(3): Cloudflare"
tags: 
disableToc: true
---

首先需要一个域名，推荐`Cloudflare`上直接购买，7位.com 结尾一年仅需要8.57美元, 非常良心。


[https://blog.meow.page/archives/free-personal-image-hosting-with-backblaze-b2-and-cloudflare-workers/](https://blog.meow.page/archives/free-personal-image-hosting-with-backblaze-b2-and-cloudflare-workers/)

设置`CNAME`, `Page Rule`缓存策略就不重复了

具体看下`Transform Rule`替代`Workers`


我的域名是`mlyxshi.com`
设置了CNAME `cdn.mlyxshi.com` 指向 `f002.backblazeb2.com`

一张图片的URL
`https://f002.backblazeb2.com/file/[BUCKET NAME]/test.png`

CNAME
`https://cdn.mlyxshi.com/file/[BUCKET NAME]/test.png`

我想要通过这样的链接不暴露BUCKET NAME访问
`https://cdn.mlyxshi.com/test.png`


![](/selfhostimage/media/2022-02-26-00-41-37.png)
配置一条`Transform Rule`
动态增加`/file/<BUCKET NAME>`到原始URL上。

这里条件要确保`URI PATH` `does not contain` `/file/<BUCKET NAME>`。

因为rclone的backblaze配置有个`download_url` 可以指定自己的域名。指定后rclone的下载连接是`https://cdn.mlyxshi.com/file/[BUCKET NAME]/test.png`

不然`https://cdn.mlyxshi.com/file/[BUCKET NAME]/test.png` 就会变成`https://cdn.mlyxshi.com/file/[BUCKET NAME]/file/[BUCKET NAME]/test.png`


现在就可以通过`rclone`，`backblaze`网页，第三方客户端，上传图片了。


但每种都有缺点

我需要的是在`Mac`上用[Snipaste](https://zh.snipaste.com)截图，copy到`clipboard`后一键直接上传

`iPhone`,`iPad` `Share Sheet`直接上传

`rclone`操作过于繁琐，移动端也没法使用

不想每次上传还要登录`backblaze`网页版

第三方开源免费客户端因为要兼顾跨平台，倾向于使用`Electron`，`Flutter`方案，实现了backblaze API的也很少。

`Apple`原生应用有很多，但基本都要订阅付费，吃像难看。


2021年`WWDC`上推出`MacOS`版本的`Shortcuts` [https://developer.apple.com/videos/play/wwdc2021/10232/](https://developer.apple.com/videos/play/wwdc2021/10232/) 功能实现后可以直接在`iPhone`, `iPad`, `Mac`上使用。非常适合实现上传图片这类场景。

[苹果生态下自建图床(4): Shortcuts](/selfhostimage/shortcuts)

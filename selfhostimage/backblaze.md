---
title: "苹果生态下自建图床(2): Backblaze"
tags: 
disableToc: true
---

[https://blog.meow.page/archives/free-personal-image-hosting-with-backblaze-b2-and-cloudflare-workers/](https://blog.meow.page/archives/free-personal-image-hosting-with-backblaze-b2-and-cloudflare-workers/)

[backup](https://web.archive.org/web/20210307151112/https://blog.meow.page/archives/free-personal-image-hosting-with-backblaze-b2-and-cloudflare-workers/)

`Backblaze`基本的创建账号，bucket，API KEY就不重复了。上文作者写于19年，使用`Cloudflare Worker`的方式避免暴露源站域名，2021年`Cloudflare`推出`Transform Rule`, 可以更优雅的解决源站域名暴露的问题。

关于`Cloudflare` `Transform Rule`: [https://blog.cloudflare.com/transform-http-response-headers/](https://blog.cloudflare.com/transform-http-response-headers/)


关于源站域名暴露的讨论: [https://v2ex.com/t/784561](https://v2ex.com/t/784561)




[苹果生态下自建图床(3): Cloudflare](/selfhostimage/cloudflare)
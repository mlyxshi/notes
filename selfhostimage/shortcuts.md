---
title: "苹果生态下自建图床(4): Shortcuts"
tags: 
disableToc: true
---

看下backblaze的文档[https://www.backblaze.com/b2/docs/calling.html](https://www.backblaze.com/b2/docs/calling.html)
是很常规的上传方式。

配合Surge MITM，很快就可以知道上传的流程。

Shortcuts GUI式编程很有趣，变量通过Select Magic Variable传递。熟悉了之后，方法调用非常的方便。



[https://cdn.mlyxshi.com/Upload+Image+%5BShare%5D.shortcut](https://cdn.mlyxshi.com/Upload+Image+%5BShare%5D.shortcut)

下载后，MacOS上双击安装，iPhone貌似会闪退。


需要配置下
![](/selfhostimage/media/2022-02-26-02-02-58.png)

`applicationKeyId`, `applicationKey`是`backblaze`的密钥

`bucketId`每个`Bucket`是一个固定的数值，因为`backblaze`每日`API`有数量限制，也不想每次上传还要额外多一次请求，浪费时间，就没写`bucketId`的获取逻辑。这里运行下`rclone`抓一下包就可以知道`bucketId`

`endpoint`为域名，比如我的就是`https://cdn.mlyxshi.com/`


MacOS可以Pin到Menu Bar上，Snipaste截图后, Copy一键上传
![](/selfhostimage/media/2022-02-26-02-22-17.png)

移动端Share Sheet一键上传。

最终URL自动保存到剪切板。


---
### Optional
最后有个问题是`uploadUrl`每次都不一样。`https://pod-000-[随机数字]-[随机数字].backblaze.com/b2api/v1/b2_upload_file/xxxxxxxxxxx` 

`Shortcuts`对`Privacy`管理很严格, 每次访问不同的域名都需要确认，非常麻烦。

我用了`Surge`的`$persistentStore.read([key<String>])`和`$persistentStore.write(data<String>, [key<String>])`读取和保存`uploadUrl`。在`Shortcuts`里固定`uploadUrl`为 `example.upload.com`, 再配合`Surge`脚本修改`URL`，这样Shortcuts就只访问4个域名了。第一次运行需要确认，之后就直接运行。

就不提供相关文件了，有兴趣可以自己尝试一下。

![](/selfhostimage/media/2022-02-26-21-01-05.png)

这样苹果生态下的图床工作流就很完美了。






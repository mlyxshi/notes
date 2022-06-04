---
title: "iCloud同步书签(3): Windows"
tags:
disableToc: true
---



一般的Windows系统直接去UUP下载Windows原版镜像 [https://zhuanlan.zhihu.com/p/104547677](https://zhuanlan.zhihu.com/p/104547677)

但UUP不提供LTSC镜像，LSTC只有订阅MSDN的服务才能下载。
找到了一位很专业的MSDN订阅者
[https://isofiles.bd581e55.workers.dev](https://isofiles.bd581e55.workers.dev)



按照给WinPE的加入驱动的流程，同样给Windows10加入驱动

注意WinPE只有`/sources/boot.wim` 需要加入驱动

Win10有`/sources/boot.wim` 还有`/sources/install.wim`

先后给`boot.wim` `install.wim` 打上驱动，替换掉原来的文件

`install.wim` 还可以自定义一些优化，去掉无用组件。

zip压缩Win10, 上传到服务器

寻找一个公网ip绑定在本机网卡上的设备，开启SMB server

注意，无法在VPS(Virtual Private Servers)上开启SMB server, 因为VPS网络大部分是VPC架构，本机网卡绑定的是IPV4内网IP，通过VPC的Internet Gateway SNAT出去。

![](/icloud/media/2022-02-04-14-18-20.png)
如果开SMB服务，虽然VPS有公网IPV4，但SMB IPV4只有内网可以访问，公网无法访问。

IPV6没有这样的问题，可以用IPV6访问，但WinPE环境不支持IPV6


解决方案

1.硬件
- 有公网IP的家用路由器
- 独立服务器(Dedicated Server)

2.软件
- [https://github.com/fatedier/frp](https://github.com/fatedier/frp)
- [https://github.com/rapiz1/rathole](https://github.com/rapiz1/rathole)

不懂为什么，Frp成功过一次就无法再成功了。对网络方面和SMB协议不是很清楚，最后解决方案：买个Kimsufi的KS1入门款独服，就没这么多网络上的潜在问题了......

[iCloud同步书签(4): Netboot](/icloud/install)


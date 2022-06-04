---
title: "iCloud同步书签(2): WinPE"
tags:
disableToc: true
---


整体流程 [https://netboot.xyz/docs/kb/pxe/windows](https://netboot.xyz/docs/kb/pxe/windows)

Windows PE
>Windows PE (WinPE) is a small operating system used to install, deploy, and repair Windows desktop editions, Windows Server, and other Windows operating systems. 

Windows PE大概类似Linux里的LiveCD的概念，用于安装和修复系统


安装SDK 和 Add-on
![](/icloud/media/2022-02-04-12-10-45.png)

Run as Admin
![](/icloud/media/2022-02-04-12-12-50.png)

Create working files
```
copype amd64 C:\WinPE_amd64
```
![](/icloud/media/2022-02-04-12-14-34.png)


Create a WinPE ISO from working files
```
MakeWinPEMedia /ISO C:\WinPE_amd64 C:\WinPE_amd64\WinPE_amd64.iso
```

![](/icloud/media/2022-02-04-12-18-14.png)

得到普通情况下可以运行在amd64实体机器 WinPE.iso

VPS的运行环境为虚拟化(QEMU+KVM)，为了在虚拟化下启动WinPE，还要用Dism命令预先加入Virtio的驱动

Dism和Windows文档 [https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop](https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop)

推荐GUI的Dism++ [https://github.com/Chuyu-Team/Dism-Multi-language](https://github.com/Chuyu-Team/Dism-Multi-language)

关于Virtio Driver的详细解释 [https://wiki.archlinux.org/title/QEMU#Installing_virtio_drivers](https://wiki.archlinux.org/title/QEMU#Installing_virtio_drivers)





下载Virtio驱动 [https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/)


解压 WinPE_amd64.iso 到Desktop\WinPE_amd64
建立临时挂载文件夹 TmpMount


File->Mount Image
![](/icloud/media/2022-02-04-12-36-34.png)

Mount Virtio Driver
![](/icloud/media/2022-02-04-14-45-49.png)


Drivers->Add->Select Folder
![](/icloud/media/2022-02-04-14-47-51.png)

Install Driver
![](/icloud/media/2022-02-04-12-39-51.png)

虽然有个Error标题，但是正常情况，成功加入196个驱动
![](/icloud/media/2022-02-04-12-40-32.png)

File->Save Image As  保存到Desktop\boot.wim
![](/icloud/media/2022-02-04-12-41-52.png)


File->Unmout image

把桌面上新生成的包含Virtio驱动的boot.wim 移动到WinPE_amd64\sources里，替换旧的没有Virtio驱动的boot.wim

zip压缩WinPE_amd64, 上传到服务器

服务器上开启Nginx或Python简易文件服务
注意对应netboot.xyz需要的URL格式


[iCloud同步书签(3): Windows](/icloud/win10)



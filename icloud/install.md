---
title: "iCloud同步书签(4): Netboot"
tags:
disableToc: true
---



登录用于安装Windows的amd64服务器, 添加一个netboot.xyz [https://github.com/netbootxyz/netboot.xyz](https://github.com/netbootxyz/netboot.xyz) EFI 启动文件
```
sudo -i
mkdir -p /boot/efi/EFI/rescue && cd /boot/efi/EFI/rescue

//x64
wget https://boot.netboot.xyz/ipxe/netboot.xyz.efi

//Arm64
wget https://boot.netboot.xyz/ipxe/netboot.xyz-arm64.efi
```
Oracle Portal开启console连接，Console connection->Create local connection->Upload your public key(id_rsa.pub)
![](/icloud/media/2022-02-04-02-19-27.png)

![](/icloud/media/2022-02-04-02-19-55.png)
因为要用到GUI安装windows，这里选择Copy VNC connection for Linux/Mac
![](/icloud/media/2022-02-04-02-20-39.png)

本地打开Terminal，开启VNC端口转发

![](/icloud/media/2022-02-04-11-33-44.png)


一开始用免费的跨平台VNC Viewer，但很多按键无法Mapping，最终选择付费的Mac原生应用Screens 4.
![](/icloud/media/2022-02-04-02-22-25.png)

Oracle Portal Reboot Server, 狂按ESC进入UEFI


![](/icloud/media/2022-02-04-02-46-14.png)


![](/icloud/media/2022-02-04-02-46-38.png)

![](/icloud/media/2022-02-04-02-46-52.png)

![](/icloud/media/2022-02-04-02-47-15.png)

![](/icloud/media/2022-02-04-02-47-29.png)

![](/icloud/media/2022-02-04-02-47-41.png)

![](/icloud/media/2022-02-04-02-48-27.png)

![](/icloud/media/2022-02-04-02-48-39.png)

设定Windows PE Server URL

自用amd64 WinPE
```
http://ovh.mlyxshi.com/PE
```
![](/icloud/media/2022-02-11-21-53-46.png)

等待下载完成


进入WinPE环境




```
# Mount a shared resource to DeviceName

net use <DeviceName>: \\<server-ip-address>\<share-name> /user:<username> <password>
```
![](/icloud/media/2022-02-06-11-22-45.png)
自用SMB(只读权限)
```
net use F: \\ovh.mlyxshi.com\SMB /user:root share
```

![](/icloud/media/2022-02-11-23-14-01.png)

Enter `F: `
```
F:
dir
```

选择Windows10LTSC2021(**图为测试2019，测试后才发现2019版本号不满足iCloud要求。如果有iCloud需求，请勿使用2019版！**)

`cd win10ltsc2021x64`


先`setup.exe`一次，安装界面不会立刻显示，需要准备。等1-2分钟，再`setup.exe`



接下来就是普通的Windows安装了
```
#Product Key 
M7XTQ-FN8P6-TTKYV-9D4CC-J462D
```
![](/icloud/media/2022-02-04-20-48-45.png)
Windows安装很简单，Delete Parition 3, 选中Unallocated Space ，点Next就完事了。


![](/icloud/media/2022-02-04-20-51-38.png)

速度非常缓慢。Windows系统大概有1000个文件，大多数是小文件，关于SMB传输很多小文件很慢的原因 [https://docs.microsoft.com/en-us/windows-server/storage/file-server/troubleshoot/slow-file-transfer](https://docs.microsoft.com/en-us/windows-server/storage/file-server/troubleshoot/slow-file-transfer)

![](/icloud/media/2022-02-11-21-23-25.png)
结束后会重启

装Windows前
![](/icloud/media/2022-02-11-21-41-44.png)
虽然`EFI\ubuntu\grubx64.efi` `EFI\BOOT\BOOTx64.efi` 有点区别，本质都是Grub引导ubuntu系统

[https://unix.stackexchange.com/questions/565615/efi-boot-bootx64-efi-vs-efi-ubuntu-grubx64-efi-vs-boot-grub-x86-64-efi-gru](https://unix.stackexchange.com/questions/565615/efi-boot-bootx64-efi-vs-efi-ubuntu-grubx64-efi-vs-boot-grub-x86-64-efi-gru)

重启后，这时ESP:/EFI/BOOT里还是原来的Ubuntu的Grub EFI文件，想启动原来Partition3里的Ubuntu(现在变成Windows了)，原来根目录下的`/usr/sbin/init`(Systemd)没了，启动失败，进入Grub命令行模式

![](/icloud/media/2022-02-12-00-41-16.png)

`reboot`后，还是狂按ESC，进入UEFI，手动选择Boot From File, 启动Windows，目录 `ESP:/EFI/Microsoft/Boot/bootmgfw.efi`


`bootmgfw.efi`: UEFI启动

`bootmgr.efi`: BIOS启动

![](/icloud/media/2022-02-12-00-45-47.png)

接下继续Windows安装，等待转圈后，开始配置系统   
![](/icloud/media/2022-02-12-00-57-46.png)


[iCloud同步书签(5): Windows Setting](/icloud/final)
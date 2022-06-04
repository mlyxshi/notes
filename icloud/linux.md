---
title: "Linux 重装备份"
tags: 
disableToc: false
---
## 重装系统
### Debian(x86_64, AArch64)
```shell
bash <(wget -qO- https://raw.githubusercontent.com/bohanyang/debi/master/debi.sh) \
--user root \
--password <ROOTPASSWD> \
--authorized-keys-url https://github.com/<GITHUBNAME>.keys \
--version stable \
--filesystem ext4 

reboot
```

### ArchLinux (x86_64)
```shell
bash <(wget -qO- https://raw.githubusercontent.com/felixonmars/vps2arch/master/vps2arch)
```

最初接触的Linux是Ubuntu,非常简单的GUI安装。后来又接触Archlinux, Gentoo, Debian。于是就放弃了Ubuntu这个基于Debian的商业魔改版。但很多VPS厂商初始镜像只有Ubuntu没Debian。

Debian的Testing也是比较稳定的滚动发行版，并且offical支持x86_64,AArch64,RISC-V。于是Archlinux就吸引力不那么大了(Archlinux只支持x86_64,unoffical支持AArch64,RISC-V)。iOS越狱后的源也是dpkg+apt这一套相同的包管理系统，于是Debian成为了我大部分情况下的系统选择。只在本地完全掌控的机器上选择Gentoo。




## 备份原系统
source: [https://github.com/TheChymera/mkstage4](https://github.com/TheChymera/mkstage4)
### tar

```shell
## make sure xz-utils package is installed
apt -y install xz-utils
emerge -a xz-utils
```

```shell
#-J : xz
#-c : create tar
#-v : verbose
#-f : output file 
#-P : absolute names(don't strip leading '/'s from file names)
#-p : preserve permissions(default for superuser)
#--ignore-failed-read : fix tar: /: file changed as we read it

tar --ignore-failed-read -JPpcvf /backup.tar.xz --xattrs-include='*.*' --numeric-owner \
--exclude='/dev/*' \
--exclude='/mnt/*' \
--exclude='/proc/*' \
--exclude='/run/*' \
--exclude='/sys/*' \
--exclude='/tmp/*' \
--exclude='/boot/*' \
--exclude='/media/*' \
--exclude='/var/tmp/*' \
--exclude='/var/lock/*' \
--exclude='/var/log/*' \
--exclude='/var/run/*' \
--exclude='/var/lib/docker/*' \
--exclude='/lost+found' \
--exclude='/backup.tar.xz' \
/ 

```
## 恢复备份
为了恢复备份，需要一个ramfs environment

### 本地直接使用livecd chroot

### 远程环境中推荐使用IPXE

以Oracle Linux举例（2022/04/24适用）
![](/icloud/media/2022-04-24-10-38-18.png)
Oracle x86_64 推荐使用`Create local connection`的方式，如果使用`Launch Cloud Shell connection`, 网页大部分情况可以正常输出信号，但ipxe boot时候就会卡住。

![](/icloud/media/2022-04-24-10-47-14.png)
Oracle Arm64 只能使用`Launch Cloud Shell connection`的方式。网页都正常显示，但`Create local connection`完全无法连接。

netboot.xyz 已经包装好各种distribution的installer了，推荐使用Gentoo的installer(ramfs内存限制，Gentoo < 1G RAM)

```shell
//x86_64 UEFI
mkdir -p /boot/efi/EFI/rescue && wget https://boot.netboot.xyz/ipxe/netboot.xyz.efi -P /boot/efi/EFI/rescue 

//AArch64 UEFI
mkdir -p /boot/efi/EFI/rescue && wget https://boot.netboot.xyz/ipxe/netboot.xyz-arm64.efi -P /boot/efi/EFI/rescue 
```

#### VNC环境下不太方便输入，开启ssh
```shell
rc-service sshd start
passwd
```

#### Repartition and Delete Old rootfs
```shell
fdisk -l
fdisk /dev/sda
o MBR
g GTP 
p print
n new  +512M(sda1:ESP)  +500G(sda2:/)
t type L(list) 1(choose efi)
w save
q quit
```

```shell
mkfs.fat -F32 /dev/sda1
mkfs.ext4 /dev/sda2  
```

```shell
#mount device to ramfs
mkdir /mnt/newfs
mount /dev/sda2 /mnt/newfs

mkdir /mnt/newfs/boot
mount /dev/sda1 /mnt/newfs/boot
```

```shell
cd /mnt/newfs
wget <LINK TO BACKUP.TAR.XZ>
```

#### decompress to /mnt/newfs
```shell
tar -Jpxvf backup.tar.xz --xattrs-include='*.*' --numeric-owner -C /mnt/newfs
```

#### netboot.xyz
```shell
# 原来的ESP已被格式化。为了后续修改, 继续下载netboot.xyz.efi
# x64
mkdir -p /mnt/newfs/boot/efi/EFI/rescue && wget https://boot.netboot.xyz/ipxe/netboot.xyz.efi -P /mnt/newfs/boot/efi/EFI/rescue

# AArch64
mkdir -p /mnt/newfs/boot/efi/EFI/rescue && wget https://boot.netboot.xyz/ipxe/netboot.xyz-arm64.efi -P /mnt/newfs/boot/efi/EFI/rescue
```
#### chroot
```shell
mount --types proc /proc /mnt/newfs/proc
mount --rbind /sys /mnt/newfs/sys
mount --make-rslave /mnt/newfs/sys
mount --rbind /dev /mnt/newfs/dev
mount --make-rslave /mnt/newfs/dev
mount --bind /run /mnt/newfs/run
mount --make-slave /mnt/newfs/run

chroot /mnt/newfs/  /bin/bash 
export PS1="(chroot) ${PS1}"
```

#### repair bootloader
```shell
# 1. systemd-boot(Preferred) 
# 2. EFISTUB+efibootmgr(Recompile kernel, UEFI directly launch kernel.efi) 
# 3. Grub(Not KISS)
# 推荐 systemd-boot
bootctl install

cat > /boot/loader/loader.conf<<EOF
default Debian
timeout 3
EOF

#Please change to actual path
cat > /boot/loader/entries/Debian.conf <<EOF
title    Debian
linux    /vmlinuz
initrd   /initrd.img
options root=/dev/sda2  rw 
EOF
```

#### Configure
`fstab`配置, `root`只有一个分区下可以省略

debian自带`ifupdown`管理网络

习惯了能用`systemd`就用`systemd`的方式

替换成`systemd-networkd` `systemd-resolved`管理

我的使用习惯
https://github.com/mlyxshi/shell/blob/main/debian


## dd

### 制作livecd
```shell
# MacOS
diskutil list 
diskutil unmountDisk /dev/disk4
# 只有当U盘unmount后，才可以使用 dd 命令将镜像文件写入U盘（否则会报 Resource busy 错误） 
sudo dd if=live-server-amd64.iso of=/dev/disk4 bs=1m

```

### dd windows
dd常用于安装无人值守的windows

用IPXE(Wimboot)+WinPE可以实现x86_64的手动安装，

IPXE(Wimboot)目前只支持x86_64, 但dd更底层，可以支持AArch64

我用windows主要为了iCloud的书签同步，就不折腾无人值守的安装方式了 



## 黑科技
[https://github.com/felixonmars/vps2arch/](https://github.com/felixonmars/vps2arch/)

[https://github.com/Jamesits/menhera.sh](https://github.com/Jamesits/menhera.sh)





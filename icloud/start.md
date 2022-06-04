---
title: "iCloud同步书签(1): 序"
tags: 
disableToc: true
---



一直同时使用Safari, Firefox, Chrome.

日常苹果生态下，Safari是绝对主力。

Firefox在Linux上是主力，同时不想Chrome一家垄断。

Chrome优点是插件很多，于是用来装一些平时使用频率很低，“流氓但又不得不用”的插件，不想污染我的Safari和Firefox环境

多浏览器同步主要就是密码和书签的同步。密码同步用自建Bitwarden的方案非常完美。书签同步就很麻烦，尝试了一些第三方书签插件后，依旧觉得浏览器原生书签才是最舒服的使用方式。有Mac以后就没用过Windows的iCloud，无意看到Windows iCloud居然有Safari，Firefox, Chrome同步功能！但我平时很少开机Windows，实在不想专门为同步书签一直开机。于是开始折腾零成本的iCloud同步书签方案。


Orcale Always Free的`VM.Standard.E2.1.Micro`就是最好的选择了，1核心1G内存配上虚拟内存应该能应付这种挂机任务。


iCloud [https://support.apple.com/en-us/HT204283](https://support.apple.com/en-us/HT204283) 是很特殊的软件，只能安装在Windows Home, Windows Pro, Windows Enterprise中，不能安装在Windows Server。

如果系统是Win10/Win11要求系统版本号必须高于 Windows 10 version 1903, Build 18362.145(截止2022/02/07), 从Windows Store安装。

如果是Win7/Win8则官网下载安装包。


因为Windows 11加入TPM 2.0，Oracle的QEMU应该没有配置TPM，很大可能无法安装，所以Windows 11直接排除。LTSC适合长时间挂机需求，也是系统洁癖症的首选。Windows 10 Enterprise LTSC 2021就是理想版本了（LTSC 2019问题：LTSC更新但版本号一直是version 1809，不满足iCloud的最低版本要求）

[iCloud同步书签(2): WinPE](/icloud/winpe)



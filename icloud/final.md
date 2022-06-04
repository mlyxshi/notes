---
title: "iCloud同步书签(5): Windows Setting"
tags:
disableToc: true
---



一堆配置后，进到系统。首先开启远程，使用Microsoft Remote Desktop App连接，RDP比VNC更流畅。
Settings->System->Remote Desktop
![](/icloud/media/2022-02-05-13-38-35.png)



安装Chrome，Firefox和iCloud Bookmark Browser Extension后，再在Windows Store里安装iCloud。

LTSC不自带Windows Store, 但可以手动安装。[https://github.com/kkkgo/LTSC-Add-MicrosoftStore](https://github.com/kkkgo/LTSC-Add-MicrosoftStore) (支持2019/2021的LTSC，脚本会安装旧版Store，运行Store自己更新自己到最新版)。

如果内存只有1G，Windows+两个浏览器+后台iCloud同步完全不够，可能导致应用闪退，需要优化细节。

Chrome选则Enterprise版本 [https://chromeenterprise.google/browser/download/](https://chromeenterprise.google/browser/download/)，Firfox选择ESR版本 [https://www.mozilla.org/en-US/firefox/enterprise/](https://www.mozilla.org/en-US/firefox/enterprise/)，稳定优先

全部打包一个zip，偷懒可以直接下 [https://cdn.mlyxshi.com/winsetup.zip](https://cdn.mlyxshi.com/winsetup.zip)


性能优先
![](/icloud/media/2022-02-12-02-05-59.png)
给4G~10G虚拟内存，挂机iCloud够用了
![](/icloud/media/2022-02-12-02-05-50.png)



关闭Firefox更新
![](/icloud/media/2022-02-05-14-27-27.png)

关闭Windows Store自动更新
![](/icloud/media/2022-02-05-13-58-40.png)

关闭Windows自动更新重启
[给B站引流](https://www.zhihu.com/question/65332770/answer/701978460)
![](/icloud/media/2022-02-05-13-53-16.png)

![](/icloud/media/2022-02-05-13-55-45.png)



浏览器仅同步书签，除iCloud Bookmark以外的Extension全部删除
![](/icloud/media/2022-02-05-14-17-30.png)
![](/icloud/media/2022-02-05-14-17-38.png)

KMS激活
```
# Windows 10 LTSC 2019/2021

# Install product key
slmgr.vbs /ipk M7XTQ-FN8P6-TTKYV-9D4CC-J462D

# Specify KMS host 
slmgr.vbs /skms ovh.mlyxshi.com

# Prompt KMS activation attempt.
slmgr.vbs /ato

# Display detailed license information.
slmgr.vbs -dlv
```

最终~~零成本~~同步书签。


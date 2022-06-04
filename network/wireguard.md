---
title: "Modern Network: Wireguard"
tags: 
disableToc: true
---
## Prerequisite
[Wireguard Whitepaper](https://www.wireguard.com/papers/wireguard.pdf)

[https://ro-che.info/articles/2021-02-27-linux-routing](https://ro-che.info/articles/2021-02-27-linux-routing) [backup](https://web.archive.org/web/20210621000331/https://ro-che.info/articles/2021-02-27-linux-routing)


---


>Wireguard is written and maintained by Jason A. Donenfeld (zx2c4) , a Gentoo developer.

内核编译选项看`Gentoo`的[Wiki](https://wiki.gentoo.org/wiki/Wireguard)就可以了

`Wireguard`有2种运行方式，`wg`和`wg-quick`

wg: 一行一行输入配置命令

wg-quick: 读取配置文件，直接运行。一般使用wg-quick命令

`Wireguard`没有`Server`和`Client`的概念, 只有`Peer`。

为了区分`Peer`的定位，下文还是使用`Server`和`Client`。


## config
### Client
`Client`为主动发起连接的一方，对比`Server`无需`ListenPort`, 但要指定`DNS`, `Endpoint`
```
vim /etc/wireguard/wg0.conf
```
```
[Interface]
Address = 172.16.0.2/24
MTU = 1420
PrivateKey = <Client1 PrivateKey>
DNS = 1.1.1.1

[Peer]
PublicKey = <Server PublicKey>
AllowedIPs = 0.0.0.0/0
Endpoint = <Server Public IP>:<Server Port>
```
### Server
```
vim /etc/wireguard/wg0.conf
```
```
[Interface]
Address = 172.16.0.1/24
MTU = 1420
ListenPort = <Server Port>
PrivateKey = <Server PrivateKey>

[Peer]
PublicKey = <Client1 PublicKey>
AllowedIPs = 172.16.0.2/32

[Peer]
PublicKey = <Client2 PublicKey>
AllowedIPs = 172.16.0.3/32
```
## command
`wg-quick` 只能临时运行，推荐使用`systemctl`管理
### wg-quick
```
wg-quick up wg0 
wg-quick down wg0 
```
### systemctl
```
systemctl start wg-quick@wg0 
systemctl stop wg-quick@wg0 
systemctl enable wg-quick@wg0 
systemctl disable wg-quick@wg0 
```

## 额外配置
假设情况为本地`ClientA` 使用`Server`的`Wireguard VPN`


`ClientA` 是终端用户

`ClinetA` 填写完成wg0.conf后

`systemctl start wg-quick@wg0` 就可以了

---
Server 配置

`filter table`先全部禁用(不推荐~~但很简单~~)，如果无法连接，排除是filter table的干扰
```
//Disable the firewall temporarily and flush all the rules

//不指定table时默认filter table
iptables -P INPUT ACCEPT
iptables -P FORWARD ACCEPT
iptables -P OUTPUT ACCEPT
iptables -F
```

开启Packet Forward
```
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
echo "net.ipv6.conf.all.forwarding = 1" >> /etc/sysctl.conf
sysctl -p
```


配置`SNAT`
```
//-o(--out-interface) 和 --to-source 请自行更换
iptables -t nat -A POSTROUTING -s 172.16.0.2 -o enp0s3 -j SNAT --to-source 10.0.0.220
```

![](/network/media/2022-03-03-00-56-24.png)
为什么要开启Packet Forward和配置`SNAT`？

从Server的角度，wg0里的Packet SourceIP=172.16.0.2(从ClientA而来), DestinationIP=ClientA想要访问网址的IP，不等于Wireguard Server本机IP，如果不开启Packet Forward，在Routing阶段直接Drop

开启后Packet Forward后

wg0-->PREROUTING-->Routing

Routing 时候匹配到main table里的`default via 10.0.0.1 dev enp0s3 proto dhcp src 10.0.0.220`

-->Forward-->POSTROUTING

POSTROUTING时候必须SNAT，因为现在SourceIP=172.16.0.2 是内网IP，没法在公网传输


注意，大部分VPS厂商网络是VPC架构，Server eth0的IP是内网地址，比如这里的Server IPv4 10.0.0.220，经过VPC 里的Internet Gateway SNAT到Public IP。

Dedicated Server本机网卡直接绑定Public IP。

不管是VPS还是Dedicated Server，原理都是一样，区别在于VPS这样实际上会SNAT 2次，Dedicated Server 1次。

Server 填写完成wg0.conf
```
PostUp = iptables -t nat -A POSTROUTING -s 172.16.0.2 -o enp0s3 -j SNAT --to-source 10.0.0.220
PostDown = iptables -t nat -D POSTROUTING -s 172.16.0.2 -o enp0s3 -j SNAT --to-source 10.0.0.220
```
`SNAT`规则可以直接配置到wg0.conf里，方便使用

systemctl start wg-quick@wg0


这里只是说明Wireguard Server端的配置原理，下篇说明Wireguard Client端的详细配置




[Modern Network: Cloudflare Warp](/network/warp)
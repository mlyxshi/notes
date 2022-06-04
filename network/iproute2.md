---
title: "Modern Network: iproute2 and iptables"
tags: 
disableToc: true
---

`Shadowsocks` 工作在Layer 5, 只需要打开`Firewall`的端口就可以连接

`Wireguard` 工作在Layer 3，会涉及到配置`Route`的问题，增加了很多难度


![](/network/media/2022-03-01-15-12-04.png)

上图是`Netfilter`的简化版本

### iptables
`iptables` 主要是配置`PREROUTING`，`INPUT`，`OUTPUT`，`FORWARD`, `POSTROUTING` 这些`chain` 里`table`里的规则, 也就是蓝色方框部分。

主要起`Filter`，`NAT`，`Mark`的作用

#### Filter
```
iptables -A INPUT -i eth0 -p tcp --dport 21 -j DROP
```

#### NAT
```
//SNAT
iptables -t nat -A POSTROUTING -o eth0 -j SNAT --to-source 192.168.1.100
```

```
//DNAT
iptables -t nat -A PREROUTING -p tcp  --dport 80 -j REDIRECT --to-ports 8080
```
#### Mark

```
iptables -A PREROUTING -i eth0 -t mangle -p tcp --dport 25 -j MARK --set-mark 1
```

### iproute2
`ip rule` 和 `ip route` 用于配置Route规则，也就是图中的2个`Routing`部分

#### ip rule
默认状态下的Routing policies
```
~ ip rule
0:	from all lookup local
32766:	from all lookup main
32767:	from all lookup default
```

#### ip route
默认状态下有`local` `main` `default` table

`ip route`默认显示`main table`也就是平时经常查看的table
```
~ ip route
default via 10.0.0.1 dev enp0s3 proto dhcp src 10.0.0.69 metric 100 
10.0.0.0/24 dev enp0s3 proto kernel scope link src 10.0.0.69 
169.254.0.0/16 dev enp0s3 proto dhcp scope link src 10.0.0.69 metric 100 
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 
```

注意区分不同table的概念
- `iptables`：
命令行工具
 
- `Chain`里的`table`: 
(PREROUTING Chain里的Mangle table, NAT table)

- `Routing tables`: 
`local` `main` `default`

因为Network里涉及的东西很多，本章只是简单梳理下基本的流程和概念，具体请看`Reference`。`Wireguard` 使用了`Improved Rule-based Routing`, 会新建一个`Routing tables`，并使用很多高级技巧配置路由。所以阅读理解相关文档非常重要。

### Reference
[Guide to IP Layer Network Administration with Linux](http://linux-ip.net/html/index.html)

[Linux Advanced Routing & Traffic Control](https://lartc.org)

[Iptables Tutorial 1.2.2](https://www.frozentux.net/iptables-tutorial/iptables-tutorial.html)

Computer Networking: A Top-down Approach

---
[Modern Network: Wireguard](/network/wireguard)
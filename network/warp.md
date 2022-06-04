---
title: "Modern Network: Cloudflare Warp"
tags: 
disableToc: true
---
上一篇已经说明了Wireguard的基本使用，足以应对大部分情况了。这一章详细分析下VPS中转Cloudflare Warp。

原文 [Cloudflare WARP 给 VPS 服务器额外添加 IPv4 或 IPv6 网络获得“原生”IP](https://p3terx.com/archives/use-cloudflare-warp-to-add-extra-ipv4-or-ipv6-network-support-to-vps-servers-for-free.html) [backup](https://web.archive.org/web/20220223135521/https://p3terx.com/archives/use-cloudflare-warp-to-add-extra-ipv4-or-ipv6-network-support-to-vps-servers-for-free.html)

TLDR: [https://github.com/P3TERX/warp.sh](https://github.com/P3TERX/warp.sh)

首先为什么要在VPS上套Warp？

[关于 Cloudflare Warp 的一些细节以及是否暴露访客真实IP的测试](https://blog.skk.moe/post/something-about-cf-warp/) [backup](https://web.archive.org/web/20200718161744/https://blog.skk.moe/post/something-about-cf-warp/)

[搭建 Cloudflare 背后的 IPv6 AnyCast 网络](https://nova.moe/simulate-argo-using-anycast-network-behind-cloudflare/) [backup](https://web.archive.org/web/20210414161004/https://nova.moe/simulate-argo-using-anycast-network-behind-cloudflare/)

Cloudflare Warp客户端和服务器用的是Wireguard协议通信，那就没必要用Cloudflare提供的客户端, 本质就是Wireguard节点信息。

如何获得Cloudflare Warp的Client配置？
1. TLDR: [https://github.com/P3TERX/warp.sh](https://github.com/P3TERX/warp.sh)

2. MITM Cloudflare客户端网络请求，替换key

`Surge` `MITM` `api.cloudflareclient.com`

`1.1.1.1` 客户端 Settings-->Advanced-->Connection options-->Reset encryption keys

会发现一条这样的`PUT`请求
https://api.cloudflareclient.com/vxxxxxxxxxxxx/reg/t.xxxxxxxxxxx

这里的逻辑是当`Reset encryption keys`时，`1.1.1.1` APP 会重新生成`PublicKey`和`PrivateKey`，然后上传`PublicKey`到`Cloudflare`数据库

会逆向来提取`PrivateKey`当然是最好的~~但不会~~

这里简单的通过自己生成一对`PublicKey`和`PrivateKey`后，用`Surge`脚本替换掉这条`PUT`请求`request body`里的`PublicKey`就行了。

```
if($request.method=="PUT"){

    let body = $request.body;
    body=JSON.parse(body)
    if(body['key']){
        console.log(body['key'])
        body['key'] = "<YOUR PUBLIC KEY>";
    }

    body=JSON.stringify(body)
    $done({body});
}

else{
    $done();  
}
```

`Cloudflare Warp`是分个人账号和Team账号，Team账号`Warp+` 流量是无限的

不用一键脚本的原因是，脚本使用[wgcf](https://github.com/ViRb3/wgcf)注册账号是个人账号，`Warp+` 流量有限。

`Cloudflare Warp`配置除了`PrivateKey`以外，都是一样的`PublicKey`和`Endpoint`。[详情](https://github.com/VirgilClyne/GetSomeFries#-cloudflare-warp)

```
[Interface]
Address = 172.16.0.2/32
MTU = 1420
PrivateKey = <YOUR PRIVATE KEY>

[Peer]
PublicKey = bmXOC+F1FxEMF9dyiK2H5/1SUtzH0JuVo51h2wPfgyo=
AllowedIPs = 0.0.0.0/0
Endpoint = engage.cloudflareclient.com:2408
```

得到了最基本的客户端配置后。

设计下最终网络:

Local<---Shadowsocks--->VPS<---Wireguard--->Cloudflare Warp

Local到VPS这段还是使用Layer 5的协议是因为不同地区对udp流量的策略问题(~~才不是因为这样最简单，如果这段也是Wireguard, VPS路由表可能会更难理解配置~~)


上篇讲过本地终端用户配置，没有什么额外需要配置的地方。但此时远端VPS将成为Wireguard客户端，如果运行
`systemctl start wg-quick@warp`, 会发现Local到VPS的SSH直接断开，VPS直接“失联”，

>双栈 WARP 全局网络是指 IPv4 和 IPv6 都通过 WARP 网络对外进行网络访问，实际上默认生成的 Wire­Guard 配置文件就是这个效果。由于默认的配置文件没有外部对 VPS 本机 IP 网络访问的相关路由规则，一旦直接使用 VPS 就会直接失联，所以我们还需要对配置文件进行修改。路由规则需要添加在配置文件的 [Interface] 和 [Peer] 之间的位置，以下是路由规则示例：
```
[Interface]
...
PostUp = ip -4 rule add from <替换IPv4地址> lookup main pref 18
PostDown = ip -4 rule delete from <替换IPv4地址> lookup main pref 18
PostUp = ip -6 rule add from <替换IPv6地址> lookup main pref 18
PostDown = ip -6 rule delete from <替换IPv6地址> lookup main pref 18

[Peer]
```
按照原文的方式，确实增加`ip -4 rule add from <替换IPv4地址> lookup main pref 18`后，VPS就可以正常访问了。但是为什么？

首先得弄明白`Wireguard`命令到底干了什么

使用`wg-quick up wg0` 会输出运行的命令。

关于Wireguard的分析，[https://ro-che.info/articles/2021-02-27-linux-routing](https://ro-che.info/articles/2021-02-27-linux-routing) [backup](https://web.archive.org/web/20210621000331/https://ro-che.info/articles/2021-02-27-linux-routing)很详细分析了，是非常完整的解释

当理解了Wireguard对路由做了什么后，看一下经过`ip -4 rule add from <替换IPv4地址> lookup main pref 18` 后的`ip rule` 输出
![](/network/media/2022-03-01-19-43-09.png)

我原来的理解是
既然增加了`ip -4 rule add from <替换IPv4地址> lookup main pref 18` 优先级第二，那么所有`Outbound Packet`不都是匹配这条Policy，走main table吗，就没`wireguard`什么事了？

做了下`Wireshark`的抓包实验发现

第一类出站请求：远程SSH的response packet就直接走了eth0，理想状态下的路由状况。

第二类出站请求：本机Local Process主动产生的请求(比如 curl ip.sb)确实走了wg0，然后再从eth0出去


这就很奇怪了，按照我的理解，第一类请求和第二类请求的SourceIP都是10.0.0.44，都匹配`18: from 10.0.0.44 lookup main`,为什么第一类请求是我设想中的情况，第二种不是？

看iproute2文档的时候[Source Address Selection](http://linux-ip.net/html/routing-saddr-selection.html)这个章节引起我的注意，我以前都是忽视路由表的src地址，认为本机发出的packet的SourceIP不就是本机IP吗(take it for granted)。 但是实际上
>The initial source address for an outbound packet is chosen in according to the following series of rules. The application can request a particular IP, the kernel will use the src hint from the chosen route path, or, lacking this hint, the kernel will choose the first address configured on the interface which falls in the same network as the destination address or the nexthop router.

也就是说从`Local Procerss-->Routing`，Routing前，Outbound Packet可能并没有Source IP

更具体的解释在[https://unix.stackexchange.com/a/282575](https://unix.stackexchange.com/a/282575) [backup](https://web.archive.org/web/20220304103048/https://unix.stackexchange.com/questions/123084/what-is-the-interface-scope-global-vs-link-used-for/)找到了，完全契合我的问题

>The scope influences source address selection.
>
>For connections/associations where the source address is not yet fixed (e.g. initiating a TCP connection, but not when reacting to an incoming packet), the source address will be selected depending on the scope of the route the packet is about to hit.
>
>This is why addresses also have a scope attribute.
>
>Example where no source address selection occurs: an incoming TCP connection initiation or ping packet will be answered with the IP addresses reversed (source → destination, destination → source), otherwise the other host would not recognize the packet as answer.
>
>Example where source address selection occurs: ping xyz or telnet xyz. Common programs do not tell the operating system which source address to use (and that is a good habit). The OS needs to pick one and is prepared to do so: it tests the potential outgoing packet for the route it would hit (normal routing uses the destination address only, if you use advanced routing, the packet will not have a source address yet!). The resulting scope reduces the selection to addresses from the corresponding scope on the outgoing interface if any are available.


`SSH`或者`Local<--Shadowsocks-->VPS`这样的source → destination, destination → source类Outbound packet在Routing前的Source IP就已经固定为`10.0.0.44`, 匹配`18: from 10.0.0.44 lookup main`


`VPS`上直接`curl ip.sb`的`Outbound Packet`在`Routing`前的没有`Source IP`，不匹配`18: from 10.0.0.44 lookup main`，在`Routing`阶段匹配到`default dev wg0(warp) scope link` Source IP就变为了`wireguard` 配置里的`Address`, 经过`Wireguard`处理成加密的udp最终从eth0出去


以上说明了为什么配置`ip -4 rule add from <替换IPv4地址> lookup main pref 18` 网络就正常了

回到一开始的问题，不配置`ip -4 rule add from <替换IPv4地址> lookup main pref 18` SSH Response Packet 经历了什么？
注意这时VPS因为Route问题，是无法访问的。
但是VPS是QEMU+KVM，有些云服务公司提供了VNC连接，这里的VNC不是运行在VPS里的VNC，而是QEMU的[VNC](https://wiki.archlinux.org/title/QEMU#VNC)，所以可以直接访问
>One can add the -vnc :X option to have QEMU redirect the VGA display to the VNC session. Substitute X for the number of the display (0 will then listen on 5900, 1 on 5901...).

首先SSH inbound流量可以正常送达到Local Process的sshd里吗？

假设我的Public IP为1.1.166.29, VPS 公网IP 2.2.2.2 内网IP 10.0.0.26

到达VPC的Internet Gateway
```
SRCIP         DESTIP
1.1.166.29    2.2.2.2

Internet Gateway DNAT
1.1.166.29    10.0.0.26
```
到达VPS的eth0, PREROUTING-->Routing

因为DESTIP=10.0.0.26=本机IP，无需Forward

-->INPUT-->sshd

所以SSH inbound流量可以正常送达到Local Process的sshd里

sshd 生成Response Packet，因为是上文提到的第一类Outbound Packet, 已经自带SourceIP
```
Local Process --> Routing

SRCIP         DESTIP
10.0.0.26    1.1.166.29
```

Routing判断就是[https://ro-che.info/articles/2021-02-27-linux-routing](https://ro-che.info/articles/2021-02-27-linux-routing) [backup](https://web.archive.org/web/20210621000331/https://ro-che.info/articles/2021-02-27-linux-routing)这里的流程了，进入wg0

Wireshark 查看wg0, 确实是SRCIP: 10.0.0.26, DESTIP: 1.1.166.29
![](/network/media/2022-03-02-00-19-53.png)
可以看到SSH的Reponse一直Retransmission, 也就是VPS无法访问的表面现象，这里为什么会一直Retransmission呢？Wireguard的[paper](https://www.wireguard.com/papers/wireguard.pdf)

>Once the packet payload is decrypted, the interface has a plaintext packet. If this is not an IP packet, it is dropped. Otherwise, WireGuard checks to see if the source IP address of the plaintext inner-packet routes correspondingly in the cryptokey routing table. For example, if the source IP of the decrypted plaintext packet is 192.168.31.28, the packet correspondingly routes. But if the source IP is 10.192.122.3, the packet does not route correspondingly for this peer, and is dropped.

我并不知道Cloudflare Warp的服务器端具体配置，但Client端的Address(self-ip)可以为任意值。一个推测是假如Client端的Address设置为172.16.0.2/32后，只有SourceIP=172.16.0.2的Packet(也就是第二类Outbound Packet)，server才会accept。 SSH response packet 的SourceIP= 10.0.0.26, server drop。


虽然依旧不是很清楚原因，但就如解决方案一样，设置Route，把第一类Outbound Packet不经过wg0就可以规避这样的问题。


这样`Wireguard`算是搞清楚了一点了，但还有很多问题，以后再研究吧。


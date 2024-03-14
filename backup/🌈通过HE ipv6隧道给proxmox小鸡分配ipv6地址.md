# 通过HE ipv6隧道给proxmox小鸡分配ipv6地址

## 前言

希望可以帮助到更多的灵车。

放开一下思维。是个vps都能安装proxmox开ipv6 LXC小鸡。

原文在[[我的博客](https://www.cfeng.com.cn/article/pg7)](https://www.cfeng.com.cn/article/pg7)

本文假设你已经拥有或者完成了

- 正常工作并且有独立ipv4的proxmox ve服务器
- 已经注册并开通了隧道的HE账号
- 除了HE以外的其他ipv6隧道也可以

完成设置后可以得到

- 主机可以通过he ipv6隧道获取ipv6连接
- 主机可以通过ndppd来设置内部ipv6邻居地址信息（可选）
- 主机可以通过radvd来给虚拟机通过SLAAC方式自动分配ipv6地址（本文不使用DHCPv6）

## 设置ipv6隧道

首先，需要设置ipv6隧道并使主机能通过隧道使用ipv6

在Proxmox ve主机里进行ipv6需要的系统参数设置

修改/etc/sysctl.conf 加入以下段落来开启ipv6转发。

```
net.ipv6.conf.all.forwarding = 1

```

=

然后使其生效

```
sysctl -p

```

proxmox默认应该是允许所有ipv6
icmp的。这里就略过。如果默认没有放行的话ip6tables加一下就行。

打开Tunnelbroker网站的已经创建好的Tunnel Details

!https://pic.cosmiccat.net/uploads/2024/01/08/he-0.png

记录下这些，然后在proxmox ve主机里添加隧道

修改/etc/network/interface 添加

```
auto he-ipv6                         #隧道网卡名，可以换成其他你喜欢的名字
iface he-ipv6 inet6 tunnel           #设置网卡ipv6部分为隧道
        mode sit                     #使用sit模式隧道
        address 你的Client IPv6 Address   # 比如2001:470:1234:1234::2  (仅举例，请不要复制)
        netmask 64
        endpoint 你的Server IPv4 Address
        local 你的Client IPv4 Address
        ttl 255
        gateway 你的Server IPv6 Address

```

然后启动隧道

```
ifup he-ipv6

```

检查是否成功使用ipv6

```
ping6 google.com

#应该有类似以下输出
PING google.com(lga25s73-in-x0e.1e100.net (2607:f8b0:4006:816::200e)) 56 data bytes
64 bytes from lga25s73-in-x0e.1e100.net (2607:f8b0:4006:816::200e): icmp_seq=1 ttl=120 time=10.7 ms
64 bytes from lga25s73-in-x0e.1e100.net (2607:f8b0:4006:816::200e): icmp_seq=2 ttl=120 time=13.7 ms
64 bytes from lga25s73-in-x0e.1e100.net (2607:f8b0:4006:816::200e): icmp_seq=3 ttl=120 time=10.7 ms
64 bytes from lga25s73-in-x0e.1e100.net (2607:f8b0:4006:816::200e): icmp_seq=4 ttl=120 time=10.8 ms

```

## 为虚拟机网桥添加ipv6支持

### 设置vm网桥ipv6设置

💡
在继续前请关闭所有虚拟机。修改网桥设置会使所有使用它的正在运行的虚拟机断网（只能通过重启虚拟机来恢复）

假设我们虚拟机使用vmbr1作为网桥（不是默认的vmbr0）（通常NAT机会这样配置）

在/etc/network/interface里添加

```
iface vmbr1 inet6 static
        # 请填入和he-ipv6同段的地址比如2001:470:1234:1234::3/64  (仅举例，请不要完全复制)
        # 不要使用和he-ipv6或其他网卡相同的地址
        address 你的Client Ipv6地址同段的另一个地址/64

```

使vmbr1使用新设置

```
#关闭vmbr1
ifdown vmbr1
#打开vmbr1
ifup vmbr1

```

### 获取HE
TunnelBroker分配的已路由的网段

隧道使用的地址是/64。并不能继续在划分网段了。但是HE
TunnelBroker一共给我们了两个/64网段。一个是给我们隧道用的。另一个才是给我们虚拟机用的。

💡 目前HE
TunnelBroker热门地区默认只有/64网段可以使用。部分地区才可以分配到/48网段。本文使用/64网段作为例子。

!https://pic.cosmiccat.net/uploads/2024/01/08/he-1.png

其中Routed
/64的网段就是我们将要给虚拟机使用的网段。请注意它和之前的隧道用的Server
IPv6 Address 不是一个网段。请记录下来Routed /64的值。稍后会用到。

本文假设分配到的Routed /64的值为”2001:1111:2222:3333::/64”

### 安装ndppd(可选）和radvd

接下来需要安装ndppd和radvd。

ndppd用来管理邻居发现信息。感谢评论区，he隧道已经有路由了，不需要ndppd。因此使用he
ipv6隧道的话不需要安装和配置ndppd。

radvd用来给小鸡网桥通告路由和自动分配地址。

```
apt update && apt -y install ndppd radvd

```

### 设置ndppd（可选）

创建 /etc/ndppd.conf 为以下内容

```
#请填写隧道的interface名称。比如he-ipv6
proxy he-ipv6 {
  router yes
  #填写Routed /64的值。 如果你有/48的话也可以填在这里
  rule 2001:1111:2222:3333::/64 {
    static
  }
}

```

然后启动并设置为开机自动启动

```
systemctl start ndppd.service
systemctl enable ndppd.service

```

### 设置radvd

创建 /etc/radvd.conf 为以下内容

```
#这里填写虚拟机用的网桥名称。比如vmbr1
interface vmbr1 {
  AdvSendAdvert on;
  MinRtrAdvInterval 3;
  MaxRtrAdvInterval 10;
#这里填写你的Routed/64前缀的值。如果有/48前缀。也必须手动分成/64才能自动分配地址。要不然就得用带状态的dhcpv6或者手动在虚拟机里配置网段。
  prefix 2001:1111:2222:3333::/64 {
    AdvOnLink on;
    AdvAutonomous on;
    AdvRouterAddr on;
  };
};

#如果有多个虚拟机用的网桥，可以仿照上面继续添加

```

然后启动并设置为开机自动启动

```
systemctl start radvd.service
systemctl enable radvd.service

```

### 启动虚拟机

最后，我们把虚拟机重新启动。现在虚拟机应该可以通过SLAAC自动获取到ipv6地址并使用了.
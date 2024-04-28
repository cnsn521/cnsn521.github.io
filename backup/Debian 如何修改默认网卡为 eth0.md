[Debian](https://www.debian.cn/tag/debian/) 系统安装以后，可能会遇到网卡设备名不是常见的 eth0 的情况。我们有没有办法统一网卡设备名称呢？

在服务器环境中，统一网卡设备名是很有必要的。标准化的配置会节省我们大量的时间，这些时间可能会花在排障、监控的配置、状态收集脚本的调整等。

这里我们介绍如何把 [Debian](https://www.debian.cn/tag/debian/) 系统中的网卡从非 eth0，调整为 eth0，这个设备名是各 Linux 系统中比较通用的网卡设备名。下面我们以设备名 ens3 为例，介绍在Debian 系统中，如何修改网卡设备名为 eth0 的具体步骤。

首选，我们需要编辑 grub 的配置文件，修改启动参数。使用编辑器打开 /etc/default/grub， 查找：
```
GRUB_CMDLINE_LINUX=""
```

找到这行，并修改为：

```
GRUB_CMDLINE_LINUX="net.ifnames=0 biosdevname=0"
```

修改后记得保存。随后修改网络的配置文件，调整网卡设备名：

```
 sed -i 's/ens3/eth0/g'  /etc/network/interfaces
 sed -i 's/enp3s0/eth0/g'  /etc/network/interfaces
```

最后重新生成 grub 引导配置文件，并重启系统。

```
`grub-mkconfig -o /boot/grub/grub.cfg`
```

Grub 的配置更新后，需要重启系统生效。系统重启后，网卡的名字便更新为 eth0 了。如果遇到问题，欢迎留言。
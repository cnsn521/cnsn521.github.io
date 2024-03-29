### **一、前提**

检查是否满足虚拟化要求, 可以根据如下步骤验证:

检查是否开启虚拟化,如果值大于0则支持虚拟化，否则需要在BIOS中开启虚拟化, 重启重新执行如下命令
`egrep -c '(vmx|svm)' /proc/cpuinfo`
检查CPU类型: 如果是Intel的CPU, 输出VMX;如果是AMD的CPU, 输出SVM

`grep -E --color '(vmx|svm)' /proc/cpuinfo`
### **二、安装相关包**

```
apt update
apt install -y qemu-kvm libvirt-clients libvirt-daemon-system bridge-utils virtinst libvirt-daemon
```

安装GUI管理工具(可选)

没用桌面的不用装GUI

`apt install virt-manager -y`
### **三、配置网络**

要列出要在 KVM 虚拟机中使用的可用网络，请运行以下命令：

`virsh net-list --all`
默认情况下是不活跃的，需要启动

```
virsh net-start default
virsh net-autostart default
```
执行上述命令后结果如下

```
root@debian 13:14 ➜  ~ virsh net-list --all
 Name      State    Autostart   Persistent
--------------------------------------------
 default   active   yes         yes
```
加载内核模块vhost_net，提高网络性能
`modprobe vhost_net`
# 检查是否加载成功
`lsmod | grep vhost`
配置桥接网络
KVM自带有一个开箱即用的virbr0桥接网络接口, 但是默认分配ip不太满足需求，需要新建一个
编辑/etc/network/interfaces
根据实际情况新增如下, 我需要dhcp分配ip(可以是静态ip)
```
auto br0
iface br0 inet dhcp
  bridge_ports wlp1s0
```
更改完成后, 重启网络

```
systemctl restart networking.service
ip a
```
### **四、从镜像创建虚拟机**

创建Linux虚拟机

# 创建虚拟机磁盘
```
qemu-img create -f qcow2 /var/lib/libvirt/images/debian11.qcow2 20G

virt-install \
--name debian11 \
--memory memory=1024,currentMemory=512 \
--disk path=/var/lib/libvirt/images/debian11.qcow2,size=20 \
--vcpus 1 \
--os-type linux \
--os-variant debian10 \
--network bridge=br0,model=virtio \
--graphics none \
--console pty,target_type=serial \
--location 'http://mirrors.tencent.com/debian/dists/bullseye/main/installer-amd64/' \
--extra-args 'console=ttyS0,115200n8 serial' \
--debug
```

创建windows虚拟机
```
virt-install \
--name win10 \
--ram 2048 \
--disk path=/var/lib/libvirt/images/win10.qcow2,size=20 \
--vcpus 2 \
--os-type windows \
--os-variant win10 \
--network bridge=br0,model=virtio \
--graphics none \
--console pty,target_type=serial \
--cdrom /home/ysicing/Downloads/Win10_21H1_Chinese(Simplified)_x64.iso \
--extra-args 'console=ttyS0,115200n8 serial'
```

开启vnc

```
virsh vncdisplay win10
导入虚拟机
virt-install \
--name debian11 \
--ram 2048 \
--disk path=/var/lib/libvirt/images/debian11.qcow2,size=20 \
--vcpus 2 \
--os-type linux \
--os-variant debian11 \
--network bridge=br0,model=virtio \
--graphics none \
--console pty,target_type=serial \
--import
```

导出虚拟机为qcow2

`virt-clone --original debian11 --name debian11-clone --file /var/lib/libvirt/images/debian11-clone.qcow2`
### **五、常见高级操作**
查看虚拟机
`virsh list --all`
执行上述命令后结果如下

```
root@debian 13:14 ➜  ~ virsh list --all
 Id   Name      State
------------------------
 2    debian11  running
```
销毁虚拟机
```
virsh destroy debian11
virsh undefine debian11
```
关闭虚拟机
`virsh shutdown debian11`
### **一、挂载封装**

1.安装依赖项：

 确保你的系统上安装了QEMU和相应的工具。你可以使用以下命令在Ubuntu上安装它们：

```
sudo apt update
sudo apt install qemu-utils
```
2.挂载镜像文件：

使用`qemu-nbd`命令将镜像文件挂载到一个目录中：

```
sudo modprobe nbd max_part=8
sudo qemu-nbd --connect=/dev/nbd0 ubuntu.img
```
这会将`ubuntu.img`挂载到`/dev/nbd0`。

3.查看分区:

使用`lsblk`或其他相关命令查看挂载的镜像的分区情况：

`lsblk /dev/nbd0`
记下分区信息，以备后用。

4. 挂载分区：  将特定分区挂载到一个目录中：

` sudo mount /dev/nbd0p1 /mnt`
这里的`/dev/nbd0p1`可能需要替换为你实际的分区。

5.修改内容：   

在挂载的目录中进行任何你需要的更改，添加文件、修改配置等。

6.卸载分区和镜像： 

在完成修改后，确保卸载挂载的分区并断开QEMU NBD连接：

```
 sudo umount /mnt
 sudo qemu-nbd --disconnect /dev/nbd0
```
7.重新封装镜像：   

使用`qemu-img`命令重新封装镜像文件：

`qemu-img convert -O qcow2 -o compat=1.1 ubuntu.img ubuntu_modified.img`
这将生成一个新的QCOW2格式的镜像文件`ubuntu_modified.img`。

现在，你就成功修改并重新封装了QCOW2镜像文件。请注意，这些步骤中的具体命令可能需要根据你的系统和需求进行适当的调整。希望这些步骤对你有帮助，如果有任何问题，请随时向我询问。

### **二、镜像安装**

将制作好的QCOW2镜像导入到KVM并运行它的步骤如下：

1. 将镜像文件复制到KVM主机：

将制作好的QCOW2镜像文件（比如`ubuntu_modified.img`）复制到KVM主机的合适位置，例如`/var/lib/libvirt/images/`目录下。

2. 使用virt-manager创建虚拟机（可选）：

如果你使用virt-manager管理KVM虚拟机，可以通过virt-manager图形界面轻松创建虚拟机并导入镜像。打开virt-manager并按照向导指示操作，选择导入现有镜像并指定你的QCOW2镜像文件。

3. 通过命令行创建虚拟机：

如果你更喜欢使用命令行，可以使用`virt-install`命令创建虚拟机。以下是一个示例命令：

```
virt-install \
--name ubuntu-vm \  # 虚拟机名称
--memory 2048 \     # 内存大小（MB）
--vcpus 2 \         # 虚拟CPU数量
--disk path=/var/lib/libvirt/images/ubuntu_modified.img,format=qcow2 \  # 镜像文件路径和格式
--network bridge=br0 \  # 网络配置，根据你的网络设置调整
--graphics spice \   # 图形显示协议（可以选择其他选项）
--os-type linux \    # 操作系统类型
--os-variant ubuntu20.04 \  # 操作系统变种
--import             # 导入已有镜像
```
请根据你的实际情况调整参数，尤其是网络配置部分。

4. 启动虚拟机：

使用virt-manager或者`virsh`命令启动虚拟机：

`virsh start ubuntu-vm`
这会启动你的虚拟机，你可以使用`virt-manager`或者`virsh console`命令来管理和监视虚拟机的运行状态。

以上是将QCOW2镜像导入到KVM并运行的基本步骤。根据你的需求和环境，可能需要做一些额外的配置和调整。
# CentOS7.2部署KVM虚拟机


### 验证CPU是否支持KVM；如果结果中有vmx（Intel）或svm(AMD)字样，就说明CPU的支持的。

![CPU](/png/CPU.png)


### 关闭selinux

[root@localhost ~]# sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/sysconfig/selinux


### 安装依赖软件

[root@localhost ~]# yum -y install epel-release net-tools vim unzip zip wget ftp


### 安装KVM

[root@localhost ~]# yum -y install qemu-kvm libvirt virt-install bridge-utils 


### 验证安装结果，下图说明已经成功安装了

[root@localhost ~]# lsmod | grep kvm

![KVM](/png/KVM.png)


### 启动kvm服务：

[root@localhost ~]# systemctl start libvirtd

[root@localhost ~]# systemctl enable libvirtd

[root@localhost ~]# systemctl status libvirtd

[root@localhost ~]# systemctl is-enabled libvirtd


### 主机桥接网络配置：
[root@localhost ~]# vim /etc/sysconfig/network-scripts/ifcfg-eno1

NAME="eno1"

DEVICE="eno1"

NM_CONTROLLED=no

ONBOOT=yes

BOOTPROTO=static

BRIDGE=br0

[root@localhost ~]# vim /etc/sysconfig/network-scripts/ifcfg-br0 

BOOTPROTO=static

DEVICE=br0

TYPE=Bridge

NM_CONTROLLED=no

IPADDR=192.168.1.203

NETMASK=255.255.255.0

GATEWAY=192.168.1.1

DNS1=114.114.114.114

DNS2=8.8.8.8


### 重启网络服务

[root@localhost ~]# systemctl restart network


### 准备镜像文件：

[root@localhost ~]# ls /home/iso/CentOS-7-x86_64-DVD-1611.iso 

/home/iso/CentOS-7-x86_64-DVD-1611.iso


### 创建虚拟机文件存放的目录

[root@localhost ~]# mkdir -p /var/kvm-bak


### 创建虚拟机：

[root@localhost ~]# virt-install -n kvm1 -r 2048 --disk /var/kvm-bak/kvm.img,format=qcow2,size=50 --network bridge=br0 --os-type=linux --os-variant=rhel7.2 --cdrom /home/iso/CentOS-7-x86_64-DVD-1611.iso  --vnc --vncport=5910 --vnclisten=0.0.0.0


### 配置防火墙：

打开防火墙上的5910端口（如果防火墙为关闭则不用管）

[root@localhost ~]# firewall-cmd --zone=public --add-port=5910/tcp --permanent

[root@localhost ~]# firewall-cmd --reload


### 使用VNC连接该虚拟机

![VNC](/png/VNC.png)


### 登陆之后修改其网络配置文件并重启网络服务：

[root@localhost ~]# systemctl restart network

！[KVM](/png/KVM1.png)

之后就可以用xshell连接了

### virsh基本命令：

[root@localhost ~]# virsh start kvm1				//启动虚拟机

[root@localhost ~]# virsh shutdown kvm1				//关闭虚拟机

[root@localhost ~]# virsh reboot kvm1				//重启虚拟机

[root@localhost ~]# virsh undefine kvm1 			//删除虚拟机


### 克隆虚拟机：

虚拟机磁盘文件: /var/kvm-bak/kvm.img

虚拟机名称：kvm1 确保kvm1出于关闭状态

[root@localhost ~]# virt-clone -o kvm1 -n kvm2 -f /var/kvm-bak/kvm2.img


### 配置kvm2

[root@localhost ~]# vim /etc/libvirt/qemu/kvm2.xml 

<graphics type='vnc' port='5911' autoport='no' listen='0.0.0.0'>

### 重新定义虚拟机

virsh define /etc/libvirt/qemu/kvm2.xml	

启动kvm2虚拟机后并使用vnc连接，和上述方法一样。

![KVM](/png/KVM2.png)


【基本配置】
一、查看系统是否支持kvm虚拟化，如果支持会显示相关信息
#cat /proc/cpuinfo |grep -i -E '(vmx|svm)'

二、安装kvm模块
#modprobe  kvm

三、 安装kvm虚拟化需要的基本包
#yum install -y qemu-kvm libvirt libvirt-daemon virt-manager virt-viewer virt-install avahi

四、启动Libvirtd服务，设置开机自启动
#systemctl start Libvirtd
#systemctl enable Libvirtd

五、创建虚拟机存放路径
# mkdir -p /data/kvm/libvirt/{images,iso}

六、创建一个物理桥
#vim /etc/sysconfig/network-scripts/ifcfg-eth0
TYPE=Ethernet
NAME=eth0
DEVICE=eth0
ONBOOT=yes
BRIDGE=br0
NM_CONTROLLED=no
#vim  /etc/sysconfig/network-scripts/ifcfg-br0
TYPE=Bridge
BOOTPROTO=static
NAME=br0
DEVICE=br0
ONBOOT=yes
NM_CONTROLLED=no
IPADDR=172.17.60.210
NETMASK=255.255.255.0
GATEWAY=172.17.60.1
DNS1=172.17.60.100
#systemctl restart network

七、创建虚拟机磁盘
#qemu-img create -o preallocation=metadata -f qcow2 kvm_01.qcow2 20G

八、创建虚拟机，然后用vnc连接设置相关参数
#virt-install --name kvm_01 --hvm \
--ram 1024 \
--vcpus=1 \
-f /data/kvm/libvirt/images/kvm_01.qcow2 \
--network bridge=br0,model=virtio \
--location /data/kvm/libvirt/ios/CentOS-7_x86.iso  \
--vnc --vncport=5900 --vnclisten=0.0.0.0

九、console配置，执行之后重新启动，可以用console控制台连接虚拟机
#grubby --update-kernel=ALL --args="console=ttyS0"
#reboot

附录：命令
help	打印基本帮助信息。
list	列出所有客户端。
dumpxml	输出客户端 XML 配置文件。
create	从 XML 配置文件生成客户端并启动新客户端。
start	启动未激活的客户端。
destroy	强制客户端停止。
define	为客户端输出 XML 配置文件。
domid	显示客户端 ID。
domuuid	显示客户端 UUID。
dominfo	显示客户端信息。
domname	显示客户端名称。
domstate	显示客户端状态。
quit	退出这个互动终端。
reboot	重新启动客户端。
restore	恢复以前保存在文件中的客户端。
resume	恢复暂停的客户端。
save	将客户端当前状态保存到某个文件中。
shutdown	关闭某个域。
suspend	暂停客户端。
undefine	删除与客户端关联的所有文件。
migrate	将客户端迁移到另一台主机中。
命令	Description
setmem	为客户端设定分配的内存。
setmaxmem	为管理程序设定内存上限。
setvcpus	修改为客户端分配的虚拟 CPU 数目。
vcpuinfo	显示客户端的虚拟 CPU 信息。
vcpupin	控制客户端的虚拟 CPU 亲和性。
domblkstat	显示正在运行的客户端的块设备统计。
domifstat	显示正在运行的客户端的网络接口统计。
attach-device	使用 XML 文件中的设备定义在客户端中添加设备。
attach-disk	在客户端中附加新磁盘设备。
attach-interface	在客户端中附加新网络接口。
detach-device	从客户端中分离设备，使用同样的 XML 描述作为命令attach-device。
detach-disk	从客户端中分离磁盘设备。
detach-interface	从客户端中分离网络接口。
命令	Description
version	显示 virsh 版本
nodeinfo	有关管理程序的输出信息

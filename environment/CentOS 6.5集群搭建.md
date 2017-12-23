# CentOS 6.5集群搭建
1. Virtual Box
2. CentOS 6.5
3. JDK 1.7
4. Hadoop 2.4.1
5. Hive 0.13
6. ZooKeeper 3.4.5
7. kafka_2.9.2-0.8.1
8. Spark 1.3.0

## 安装virtual Box
> 去 https://www.virtualbox.org/ 下载virtual box安装即可。

## 安装CentOS
1. 使用CentOS 6.5镜像即可，CentOS-6.5-i386-minimal.iso。
2. 创建虚拟机：打开Virtual Box，点击“新建”按钮，点击“下一步”，输入虚拟机名称为spark1，选择操作系统为Linux，选择版本为Red Hat(32 bit)，分配1024MB内存，后面的选项全部用默认，在Virtual Disk File location and size中，一定要自己选择一个目录来存放虚拟机文件，最后点击“create”按钮，开始创建虚拟机。
3. 设置虚拟机网卡：选择创建好的spark1虚拟机，点击“设置”按钮，在网络一栏中，连接方式中，选择“Bridged Adapter”。
4. 安装虚拟机中的CentOS 6.5操作系统：选择创建好的虚拟机spark1，点击“开始”按钮，选择安装介质（即本地的CentOS 6.5镜像文件），选择第一项开始安装-Skip-欢迎界面Next-选择默认语言-Baisc Storage Devices-Yes, discard any data-主机名:spark1-选择时区-设置初始密码为hadoop-Replace Existing Linux System-Write changes to disk-CentOS 6.5自己开始安装。
5. 安装完以后，CentOS会提醒你要重启一下，就是reboot，你就reboot就可以了。

## CentOS网络配置
1. 先临时性设置虚拟机ip地址：ifconfig eth0 192.168.0.191，在/etc/hosts文件中配置本地ip（192.168.0.191）到host（spark1）的映射
2. 配置mac主机上的hosts文件：C/etc/hosts，192.168.0.191 spark1
3. d在mac上使用ssh命令连接虚拟机
4. 永久性配置CentOS网络
vi /etc/sysconfig/network-scripts/ifcfg-eth0
DEVICE=eth0
TYPE=Ethernet
ONBOOT=yes
BOOTPROTO=static
IPADDR=192.168.0.191
NETMASK=255.255.255.0
GATEWAY=192.168.0.1
5. 重启网卡
service network restart
6. 即使更换了ip地址，重启网卡，可能还是联不通网。那么可以先将IPADDR、NETMASK、GATEWAY给删除，将BOOTPROTO改成dhcp。然后用service network restart重启网卡。此时linux会自动给分配一个ip地址，用ifconfig查看分配的ip地址。然后再次按照之前说的，配置网卡，将ip改成自动分配的ip地址。最后再重启一次网卡。

## CentOS防火墙和DNS配置
1. 关闭防火墙<br>
service iptables stop<br>
chkconfig iptables off<br>
vi /etc/selinux/config<br>
SELINUX=disabled<br>
若有需要，可能同时要关闭mac的防火墙
2. 配置dns服务器
vi /etc/resolv.conf
nameserver 61.139.2.69
ping www.baidu.com

## CentOS 6.5 yum配置
1. 修改repo <br>
使用WinSCP（网上很多，自己下一个），将CentOS6-Base-163.repo上传到CentOS中的/usr/local目录下<br>
cd /etc/yum.repos.d/<br>
rm -rf *<br>
mv 自己的repo文件移动到/etc/yum.repos.d/目录中：cp /usr/local/CentOS6-Base-163.repo . <br>
修改repo文件，把所有gpgcheck属性修改为0<br>

2. 配置yum <br>
yum clean all<br>
yum makecache<br>
yum install telnet<br>

## JDK 1.7安装
1. 将jdk-7u60-linux-i586.rpm通过WinSCP上传到虚拟机中
2. 安装JDK：rpm -ivh jdk-7u65-linux-i586.rpm
3. 配置jdk相关的环境变量
``` shell
vi .bashrc
export JAVA_HOME=/usr/java/latest
export PATH=$PATH:$JAVA_HOME/bin
source .bashrc
```
4. 测试jdk安装是否成功：java -version
5. rm -f /etc/udev/rules.d/70-persistent-net.rules

## 安装第二台和第三台虚拟机
1. 使用visual box的复制功能复制两个虚拟机。(要勾选重新初始化所有网卡的MAC地址)
2. 在设置的网络配置页面找到新的mac地址，记录下来
3. 编辑/etc/udev/rules.d/70-persistent-net.rules文件将name改为“eth0”
4. 更改网卡配置文件，/etc/sysconfig/network-scripts/ifcfg-eth0,将HWADDR的值修改为/etc/udev/rules.d/70-persistent-net.rules文件中的新值(应该和步骤2中的mac地址值一样),同时修改IP地址为想要的地址（192.168.0.192，192.168.0.193）
5. 修改/etc/sysconfig/network，更改主机名（spark2,spark3）
6. 重启系统
### 
 在安装的时候，另外两台虚拟机的centos镜像文件必须重新拷贝一份，放在新的目录里，使用各自自己的镜像文件。虚拟机的硬盘文件也必须重新选择一个新的目录，以更好的区分。安装好之后，记得要在三台机器的/etc/hosts文件中，配置全三台机器的ip地址到hostname的映射，而不能只配置本机，这个很重要！在wmac的hosts文件中也要配置全三台机器的ip地址到hostname的映射。

## 配置集群ssh免密码登录
1. 首先在三台机器上配置对本机的ssh免密码登录<br>
生成本机的公钥，过程中不断敲回车即可，ssh-keygen命令默认会将公钥放在/root/.ssh目录下<br>
`ssh-keygen -t rsa`
将公钥复制为authorized_keys文件，此时使用ssh连接本机就不需要输入密码了<br>
`cd /root/.ssh`<br>
`cp id_rsa.pub authorized_keys`
2. 接着配置三台机器互相之间的ssh免密码登录<br>
使用`ssh-copy-id -i sparkX`命令将本机的公钥拷贝到指定机器的authorized_keys文件（方便好用）






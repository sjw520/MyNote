# 配置虚拟机网络

* 点击管理
  * 点击创建
  * 双击创建出来的网络
    * 手动配置网卡
    * IPv4地址：可以自定义网络地址（192.1.1.1）
  * 勾选创建出来网络上的启用
  * 关闭
* 点击设置
  * 找到网络
    * 网卡1选择的是NAT（勾选启用网络连接）
    * 点击网卡2
      * 启用网络连接
      * 连接方式：仅主机（Host-only）网络
    * 点击ok
* 启动虚拟机
* 命令 ip address
  * 可以看到IP地址（192.1.1.3）
* 命令 cd /etc/sysconfig/network-scripts/
  * vi ifcfg-enp0s3（有可能是ifcfg-enp0s5、ifcfg-enp0s6的文件名）
    * 将ONBOOT=no改成ONBOOT=yes
* 重启虚拟机网络service network restart



# 设置固定ip

* 命令cd /etc/sysconfig/network-scripts/
* cp ifcfg-enp0s3  ifcfg-enp0s8
* vi   ifcfg-enp0s8
  * 删除
    * PROXY_METHOD=none
    * BROWSER_ONLU=no
    * DEFROUTE=yes
    * IPv4_FALURE_FATAL=no
    * IPv6INIT=yes
    * IPv6_AUTOCONF=yes
    * IPv6DEFROUTE=yes
    * IPv6_FAILURE_FATAL=no
    * IPv6_ADDR_GEN_MODE=stable-privacy 
    * UUID=b0bdb05f-a549-434
  * 修改
    * BOOTPROTO=static
    * NAME=enp0s8
    * DEVICE=enp0s8
  * 添加
    * IPADDR=192.1.1.101
    * NETMASK=255.255.255.0
* 重启网络service network restart
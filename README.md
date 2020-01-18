# k8s-1.142
kubnetes 1.142版本部署，本地仓库，helm,dashboard 监控，nfs共享存储

## 一、系统优化
### 1、主机名修改，并设置本地解析

`vim /etc/hosts`
`192.168.206.131  node01`

### 2、系统服务优化
`systemctl disable firewalld && systemctl stop firewalld`
`sed -i s/SELINUX=enforcing/SELINUX=disabled/g /etc/selinux/config`
`setenforce 0`
`systemctl disable NetworkManager && systemctl stop NetworkManager`
`swapoff -a`
`sed -i 's/.*swap.*/#&/' /etc/fstab`

### 3、配置内核参数
配置内核参数，将桥接的IPv4流量传递到iptables的链

`modprobe br_netfilter`
`cat > /etc/sysctl.conf <<EOF`
`net.bridge.bridge-nf-call-ip6tables = 1`
`net.bridge.bridge-nf-call-iptables = 1`
`EOF`
`sysctl -p`

### 4、节点互信
因本次部署为单节点部署，所以节点互信配置省略

### 5、设置时间同步

`yum install -y chrony`
`vim /etc/chrony.conf`
`#server 0.centos.pool.ntp.org iburst`
`#server 1.centos.pool.ntp.org iburst`
`#server 2.centos.pool.ntp.org iburst`
`#server 3.centos.pool.ntp.org iburst`
`server 172.50.10.16 iburst`  设置为ntp server地址
启动服务并同步时间
`# systemctl enable chronyd && systemctl restart chronyd`
`# chronyc sources`
`210 Number of sources = 1`
`MS Name/IP ad dress         Stratum Poll Reach LastRx Last sample   `            
`===============================================================================`
`^* 172.50.10.16                  3   6    17    13   +103us[  +12us] +/-   24ms`




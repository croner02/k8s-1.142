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

## 二、kubernetes 1.142安装
注：在所有节点上进行如下操作
### 1、 配置国内yum源
`yum install -y wget`   
`mkdir /etc/yum.repos.d/bak && mv /etc/yum.repos.d/*.repo /etc/yum.repos.d/bak`   
`wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.cloud.tencent.com/repo/centos7_base.repo`   
`wget -O /etc/yum.repos.d/epel.repo http://mirrors.cloud.tencent.com/repo/epel-7.repo`   
`yum clean all && yum makecache`   
### 2、配置国内Kubernetes源
`cat <<EOF > /etc/yum.repos.d/kubernetes.repo`   
`[kubernetes]`   
`name=Kubernetes`   
`baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/`   
`enabled=1`   
`gpgcheck=1`   
`repo_gpgcheck=1`   
`gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg`   
`EOF`  

### 3、配置国内 docker 源
`wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo`   

### 4、安装docker
`yum install -y docker-ce-18.06.1.ce-3.el7`   
`systemctl enable docker && systemctl start docker`   
`docker --version`     
`Docker version 18.06.1-ce, build e68fc7a`   
docker服务为容器运行提供计算资源，是所有容器运行的基本平台。   

###  5、安装kubeadm、kubelet、kubectl
`yum install -y kubelet-1.14.2 kubeadm-1.14.2 kubectl-1.14.2`   
`systemctl enable kubelet`   
Kubelet负责与其他节点集群通信，并进行本节点Pod和容器生命周期的管理。Kubeadm是Kubernetes的自动化部署工具，   
降低了部署难度，提高效率。Kubectl是`Kubernetes集群管理工具。   

### 6、部署master 节点
注：在master节点上进行如下操作
在master进行Kubernetes集群初始化。   
`kubeadm init --kubernetes-version=1.14.2 \`   
`--apiserver-advertise-address=192.168.206.131 \`   
`--image-repository registry.aliyuncs.com/google_containers \`   
`--service-cidr=10.1.0.0/16 \`   
`--pod-network-cidr=10.244.0.0/16`   

定义POD的网段为: 10.244.0.0/16， api server地址就是master本机IP地址。   
这一步很关键，由于kubeadm 默认从官网k8s.grc.io下载所需镜像，国内无法访问，因此需要通过–image-repository指定阿里云镜像仓库地址，很多新手初次部署都卡在此环节无法进行后续配置。   
集群初始化成功后返回如下信息：   
记录生成的最后部分内容，此内容需要在其它节点加入Kubernetes集群时执行。   
`kubeadm join 192.168.206.131:6443 --token si9wm6.i3456f1h3jz5puqb \`   
`    --discovery-token-ca-cert-hash sha256:c4ffa34a083db53f9d7bf9d0df7e4c99fbd3e3b031c670eda190cce144a8e885`   
### 7、配置kubectl工具
`mkdir -p /root/.kube`   
`cp /etc/kubernetes/admin.conf /root/.kube/config`   
`kubectl get nodes`   
`kubectl get cs`   

## 三、部署flannel网络
`kubectl apply -f kube-flannel.yml`   

## 四、[部署helm](https://github.com/croner02/k8s-1.142/blob/master/helm/linux-amd64/HelmInstall_README.md)

## 五、[部署nginx-ingress](https://github.com/croner02/k8s-1.142/blob/master/ingress/README.md)

## 六、部署nfs持久化存储

## 七、[部署harbor本地仓库](https://github.com/croner02/k8s-1.142/blob/master/harbor/README.md)

## 八、[部署Dashboard](https://github.com/croner02/k8s-1.142/blob/master/dashboard/README.md)

## 九、[部署监控平台](https://github.com/croner02/k8s-1.142/blob/master/prometheus/prometheus/README.md)



# 部署helm
## 一、基本概念
对于复杂的应用中间件，需要设置镜像运行的需求、环境变量，并且需要定制存储、网络等设置，
最后设计和编写Deployment、Configmap、Service及Ingress等相关yaml配置文件，再提交给kubernetes进行部署。
这些复杂的过程将逐步被Helm应用包管理工具实现。Helm是一个由CNCF孵化和管理的项目，用于对需要在k8s上部署复杂应用进行定义、安装和更新。

Helm以Chart的方式对应用软件进行描述，可以方便地创建、版本化、共享和发布复杂的应用软件。

Chart：一个Helm包，其中包含了运行一个应用所需要的工具和资源定义，还可能包含kubernetes集群中的服务定义，
       类似于Homebrew中的formula、apt中的dpkg或者yum中的rpm文件。
       
Release：在K8S集群上运行一个Chart实例。在同一个集群上，一个Chart可以安装多次，例如有一个MySQL Chart，
         如果想在服务器上运行两个数据库，就可以基于这个Chart安装两次。每次安装都会生成新的Release，会有独立的Release名称。
         
Repository：用于存放和共享Chart的仓库。简单来说，Helm的任务是在仓库中查找需要的Chart，然后将Chart以Release的形式安装到K8S集群中。
## 二、Helm安装
Helm由两个组件组成：
　　  HelmClinet：客户端，拥有对Repository、Chart、Release等对象的管理能力。
　　  TillerServer：负责客户端指令和k8s集群之间的交互，根据Chart定义，生成和管理各种k8s的资源对象。
### 1、安装HelmClient
可以通过二进制文件或脚本方式进行安装。
`cp linux-amd64/helm linux-amd64/tiller /usr/local/bin/`  
`helm version`   
`Client: &version.Version{SemVer:"v2.11.0", GitCommit:"2e55dbe1fdb5fdb96b75ff144a339489417b146b", GitTreeState:"clean"}`   
`Error: could not find tiller`   

**因为没有安装tillerServer所以会报找不到tiller**

### 2、安装TillerServer
所有节点下载tiller:v[helm-version]镜像，helm-version为上面helm的版本2.11.0
`docker pull dotbalo/tiller:v2.11.0`   
`yum install socat -y`   

使用helm init安装tiller

`helm init --tiller-image dotbalo/tiller:v2.11.0`   
`Creating /root/.helm `   
`Creating /root/.helm/repository`    
`Creating /root/.helm/repository/cache`    
`Creating /root/.helm/repository/local `   
`Creating /root/.helm/plugins `   
`Creating /root/.helm/starters `   
`Creating /root/.helm/cache/archive`    
`Creating /root/.helm/repository/repositories.yaml`    
`Adding stable repo with URL: https://kubernetes-charts.storage.googleapis.com`   
`Adding local repo with URL: http://127.0.0.1:8879/charts `   
`$HELM_HOME has been configured at /root/.helm.`   
`Tiller (the Helm server-side component) has been installed into your Kubernetes Cluster.`    
`Please note: by default, Tiller is deployed with an insecure 'allow unauthenticated users' policy.`   
`To prevent this, run `helm init` with the --tiller-tls-verify flag.`   
`For more information on securing your installation see: https://docs.helm.sh/using_helm/#securing-your-helm-installation`   
`Happy Helming!`   

再次查看helm version及pod状态
`helm version`   
`Client: &version.Version{SemVer:"v2.11.0", GitCommit:"2e55dbe1fdb5fdb96b75ff144a339489417b146b", GitTreeState:"clean"}`   
`Server: &version.Version{SemVer:"v2.11.0", GitCommit:"2e55dbe1fdb5fdb96b75ff144a339489417b146b", GitTreeState:"clean"}`   
`kubectl get pod -n kube-system | grep tiller`   
`tiller-deploy-5d7c8fcd59-d4djx          1/1       Running   0          49s`   
`kubectl get pod,svc -n kube-system | grep tiller`   
`pod/tiller-deploy-5d7c8fcd59-d4djx          1/1       Running   0          3m`   
`service/tiller-deploy          ClusterIP   10.106.28.190    <none>        44134/TCP        5m`   






# 部署Dashboard

## 方案一：nodePort访问：
注：在master节点上进行如下操作

### 1、因搭建了本地仓库，可以先把所依赖镜像先都下载到本地镜像仓库。
需要在Dashboard Service内容加入nodePort： 30001和type: NodePort两项内容，将Dashboard访问端口映射为节点端口
`sed -i '/targetPort:/a\ \ \ \ \ \ nodePort: 30001\n\ \ type: NodePort' kubernetes-dashboard.yaml`   

### 2、部署Dashboard
`kubectl apply -f kubernetes-dashboard.yaml`    
### 3、创建完成后，检查相关服务运行状态
`kubectl get deployment kubernetes-dashboard -n kube-system`   
`kubectl get pods -n kube-system -o wide`   
`kubectl get services -n kube-system`   
`netstat -ntlp|grep 30001`   
### 4、在Firefox浏览器输入Dashboard访问地址：https://10.10.10.10:30001   

## 方案二： ingress访问：


### 编辑dashboadr-ingress.yaml文件
`apiVersion: extensions/v1beta1`   
`kind: Ingress`   
`metadata:`   
`  annotations:`   
`    kubernetes.io/ingress.class: "nginx"`   
`    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"`   
`  generation: 1`   
`  labels:`   
`    k8s-app: kubernetes-dashboard`   
`  name: kubernetes-dashboard-ingress`   
`  namespace: kube-system`   
`spec:`   
`  rules:`   
`  - host: kubernetes-dashboard.com`   
`    http:`   
`      paths:`   
`      - backend:`   
`          serviceName: kubernetes-dashboard`   
`          servicePort: 443`   
`        path: /`   
`  tls:`   
`  - hosts:`   
`    - kubernetes-dashboard.com`   
`    secretName: kube-dashboard-ssl  #需要手动创建或者直接复用ingress的secret`   

### 创建dashboard-ingress需要的secret
`openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout kube-dashboard.key -out kube-dashboard.crt -subj "/CN=dashboard.kube.com/O=dashboard.kube.com"`   
` kubectl create secret tls kube-dasboard-ssl --key kube-dashboard.key --cert kube-dashboard.crt -n kube-system`   


## 查看访问Dashboard的认证令牌


`kubectl create serviceaccount  dashboard-admin -n kube-system`   
`kubectl create clusterrolebinding  dashboard-admin --clusterrole=cluster-admin --serviceaccount=kube-system:dashboard-admin`   
`kubectl describe secrets -n kube-system $(kubectl -n kube-system get secret | awk '/dashboard-admin/{print $1}')`   

## 使用输出的token登录Dashboard。



# 部署监控平台
** 需提前部署好nfs持久化存储 **

## 一、修改 Service 端口设置
### 1、修改 prometheus-service.yaml 文件

  $ vim prometheus/prometheus-service.yaml   
  修改prometheus Service端口类型为NodePort，设置nodePort端口为32101   
  apiVersion: v1   
  kind: Service   
  metadata:   
    labels:   
      prometheus: k8s    
    name: prometheus-k8s     
    namespace: monitoring   
  spec:     
    type: NodePort   
    ports:   
    - name: web      
      port: 9090   
      targetPort: web   
      nodePort: 32101    
    selector:   
      app: prometheus   
      prometheus: k8s   
    sessionAffinity: ClientIP    

### 2、修改 Grafana Service
修改 grafana-service.yaml 文件   
  $ vim grafana/grafana-service.yaml   
  修改garafana Service端口类型为NodePort，设置nodePort端口为32102   
  apiVersion: v1   
  kind: Service   
  metadata:   
    labels:   
      app: grafana   
    name: grafana   
    namespace: monitoring    
  spec:   
    type: NodePort   
    ports:   
    - name: http   
      port: 3000   
      targetPort: http   
      nodePort: 32102   
    selector:   
      app: grafana   
## 二、修改数据持久化   
### 1、修改 Prometheus 持久化
修改 prometheus-prometheus.yaml 文件   
$ vim prometheus/prometheus-prometheus.yaml   
prometheus是一种 StatefulSet 有状态集的部署模式，所以直接将 StorageClass 配置到里面，在下面的yaml中最下面添加持久化配置：   
  ....   
    serviceMonitorSelector: {}   
    version: v2.7.2   
    storage:                  #----添加持久化配置，指定StorageClass为上面创建的fast   
      volumeClaimTemplate:   
        spec:    
          storageClassName: nfs-storage #---指定为nfs-storage   
          resources:   
            requests:   
              storage: 10Gi   

### 2、修改 Grafana 持久化配置
创建 grafana-pvc.yaml 文件   
由于 Grafana 是部署模式为 Deployment，所以我们提前为其创建一个 grafana-pvc.yaml 文件，加入下面 PVC 配置。   
  kind: PersistentVolumeClaim   
  apiVersion: v1   
  metadata:   
    name: grafana   
    namespace: monitoring  #---指定namespace为monitoring   
  spec:   
    storageClassName: nfs-storage   #---指定StorageClass为上面创建的nfs-storage   
    accessModes:   
      - ReadWriteOnce   
    resources:   
      requests:   
        storage: 5Gi    
修改 grafana-deployment.yaml 文件设置持久化配置，应用上面的 PVC   
$ vim grafana/grafana-deployment.yaml   
将 volumes 里面的 “grafana-storage” 配置注掉，新增如下配置，挂载一个名为 grafana 的 PVC   
  ......   
        volumes:    
        - name: grafana-storage       #-------新增持久化配置   
          persistentVolumeClaim:   
            claimName: grafana        #-------设置为创建的PVC名称   
        #- emptyDir: {}               #-------注释掉旧的配置    
        #  name: grafana-storage    
        - name: grafana-datasources   
          secret:   
            secretName: grafana-datasources   
        - configMap:   
            name: grafana-dashboards   
          name: grafana-dashboards   
  ......   

## 三、改 kubernetes 配置与创建对应 Service   
必须提前设置一些 Kubernetes 中的配置，否则 kube-scheduler 和 kube-controller-manager 无法监控到数据。   
### 1、更改 kubernetes 配置   
由于 Kubernetes 集群是由 kubeadm 搭建的，其中 kube-scheduler 和 kube-controller-manager   
默认绑定 IP 是 127.0.0.1 地址。Prometheus Operator 是通过节点 IP 去访问，所以我们将 kube-scheduler 绑定的地址更改成 0.0.0.0。   
- 修改 kube-scheduler 配置   
编辑 /etc/kubernetes/manifests/kube-scheduler.yaml 文件   
$ vim /etc/kubernetes/manifests/kube-scheduler.yaml   
将 command 的 bind-address 地址更改成 0.0.0.0   
`  ......`   
`  spec:`   
`    containers:`   
`    - command:`   
`      - kube-scheduler`   
`      - --bind-address=0.0.0.0  #改为0.0.0.0`   
`      - --kubeconfig=/etc/kubernetes/scheduler.conf`   
`      - --leader-elect=true`     
`  ......`    
- 修改 kube-controller-manager 配置   
编辑 /etc/kubernetes/manifests/kube-controller-manager.yaml 文件   
$ vim /etc/kubernetes/manifests/kube-controller-manager.yaml   
将 command 的 bind-address 地址更改成 0.0.0.0   
`  spec:`      
`    containers:`      
`    - command:`   
`      - kube-controller-manager`       
`      - --allocate-node-cidrs=true`        
`      - --authentication-kubeconfig=/etc/kubernetes/controller-manager.conf`      
`      - --authorization-kubeconfig=/etc/kubernetes/controller-manager.conf`       
`      - --bind-address=0.0.0.0  #改为0.0.0.0`      
`  ......`       
  
- 创建 kube-scheduler & controller-manager 对应 Service   
因为 Prometheus Operator 配置监控对象 serviceMonitor 是根据 label 选取 Service 来进行监控关联的，   
而通过 Kuberadm 安装的 Kubernetes 集群只创建了 kube-scheduler & controller-manager 的 Pod 并没有创建 Service，   
所以 Prometheus Operator 无法这两个组件信息，这里我们收到创建一下这俩个组件的 Service。   
[kube-system-service.yaml](https://github.com/croner02/k8s-1.142/blob/master/k8s/kube-system-service.yaml)   
   
如果是二进制部署还得创建对应的 Endpoints 对象将两个组件挂入到 kubernetes 集群内，然后通过 Service 提供访问，才能让 Prometheus 监控到。   
  
## 四、安装Prometheus Operator   

### 1、创建 namespace   
  $ kubectl apply -f 00namespace-namespace.yaml   
### 2、安装 Operator   
  $ kubectl apply -f operator/   
### 3、查看 Pod，等 pod 创建起来在进行下一步   
  $ kubectl get pods -n monitoring   
  NAME                                   READY   STATUS    RESTARTS   
  prometheus-operator-5d6f6f5d68-mb88p   1/1     Running   0     
### 4、安装其它组件    
  kubectl apply -f adapter/   
  kubectl apply -f alertmanager/   
  kubectl apply -f node-exporter/   
  kubectl apply -f kube-state-metrics/   
  kubectl apply -f grafana/   
  kubectl apply -f prometheus/   
  kubectl apply -f serviceMonitor/   
### 5、查看 Pod 状态   
  $ kubectl get pods -n monitoring   
  NAME                                   READY   STATUS    RESTARTS   
  alertmanager-main-0                    2/2     Running   0             
  alertmanager-main-1                    2/2     Running   0            
  alertmanager-main-2                    2/2     Running   0            
  grafana-b6bd6d987-2kr8w                1/1     Running   0   
  kube-state-metrics-6f7cd8cf48-ftkjw    4/4     Running   0             
  node-exporter-4jt26                    2/2     Running   0     
  node-exporter-h88mw                    2/2     Running   0             
  node-exporter-mf7rr                    2/2     Running   0    
  prometheus-adapter-df8b6c6f-jfd8m      1/1     Running   0             
  prometheus-k8s-0                       3/3     Running   0     
  prometheus-k8s-1                       3/3     Running   0     
  prometheus-operator-5d6f6f5d68-mb88p   1/1     Running   0     
## 五、查看 Prometheus & Grafana   
### 1、查看 Prometheus   
打开地址： http://192.168.101.131:32101 查看 Prometheus 采集的目标，看其各个采集服务状态有木有错误。   

### 2、查看 Grafana   
打开地址： http://192.168.101.131:32102 查看 Grafana 图表，看其 Kubernetes 集群是否能正常显示。   
•	默认用户名：admin    
•	默认密码：admin   





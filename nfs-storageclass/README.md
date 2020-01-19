## 一、部署nfs
`yum -y install nfs-utils rpcbind`   
`echo "/nfs-share *(rw,async,no_root_squash)" >> /etc/exports`   
`mkdir /nfs-share`   
`exportfs –r`   
`systemctl start rpcbind`   
`systemctl start nfs-server`   
`systemctl enable rpcbind`   
`systemctl enable nfs-server`   
`showmount -e localhost`   
## 二、创建 StorageClass
创建一个名称为 nfs-storage 的 StorageClass,不同的存储驱动创建的 StorageClass 配置也不同，下面提供基于”NFS”两种配置，
如果是NFS存储，请提前确认集群中是否存在”nfs-provisioner”应用。

[nfs-rbac.yaml](https://github.com/croner02/k8s-1.142/blob/master/nfs-storageclass/rbac.yaml)

`nfs-deployment配置`   
`      containers:`   
`        - name: nfs-client-provisioner`   
`          image: quay.io/external_storage/nfs-client-provisioner:latest`   
`          volumeMounts:`   
`            - name: nfs-client-root`   
`              mountPath: /persistentvolumes`   
`          env:`   
`            - name: PROVISIONER_NAME`   
`              value: nfs-client-provisioner   #和storageclass provisioner保持一致`   
`            - name: NFS_SERVER`   
`              value: 192.168.206.130   #nfs server地址`   
`            - name: NFS_PATH`   
`              value: /nfs-share        #nfs server挂载地址`   
`      volumes:`    
`        - name: nfs-client-root`   
`          nfs:`   
`            server: 192.168.206.130   #nfs server地址`   
`            path: /nfs-share    #nfs server挂载地址`   

NFS 存储的 StorageClass 配置

`apiVersion: storage.k8s.io/v1`   
`kind: StorageClass`   
`metadata:`   
`  name: nfs-storage`    
`provisioner: nfs-client-provisioner    #---动态卷分配应用设置的名称，必须和集群中的"nfs-provisioner"应用设置的变量名称保持一致`   
`parameters:`    
`  archiveOnDelete: "true"  #---设置为"false"时删除PVC不会保留数据,"true"则保留数据`  

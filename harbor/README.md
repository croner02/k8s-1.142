# Helm部署harbor

## 安装依赖
- 需要先部署helm
- 需要先部署 nginx-ingress
## 2、下载harbor chart
首先下载 Harbor Chart 包到要安装的集群上, 切换到我们需要安装的分支，比如我们这里使用 1.0.0分支：   
`$ git clone https://github.com/goharbor/harbor-helm`   
`cd harbor-helm`   
`git checkout 1.0.0`   

## 编辑 values配置文件
安装 Helm Chart 包最重要的当然是values.yaml文件了，我们可以通过覆盖该文件中的属性来改变配置：  

`expose:`    
`  # 设置暴露服务的方式。将类型设置为 ingress、clusterIP或nodePort并补充对应部分的信息。`    
`  type: ingress`    
`  tls:`       
`    # 是否开启 tls，注意：如果类型是 ingress 并且tls被禁用，则在pull/push镜像时，`   
`    # 则必须包含端口。详细查看文档：https://github.com/goharbor/harbor/issues/5291。`    
`    enabled: true`    
`    # 如果你想使用自己的 TLS 证书和私钥，请填写这个 secret 的名称，这个 secret 必须包含名为 `    
`    # tls.crt 和 tls.key 的证书和私钥文件，如果没有设置则会自动生成证书和私钥文件。`   
`    secretName: ""`   
`    # 默认 Notary 服务会使用上面相同的证书和私钥文件，如果你想用一个独立的则填充下面的字段，注意只有类型是 ingress 的时候才需要。`   
`    notarySecretName: ""`   
`    # common name 是用于生成证书的，当类型是 clusterIP 或者 nodePort 并且 secretName 为空的时候才需要`   
`    commonName: ""`    
`  ingress:`       
`    hosts:`   
`      core: registry.harbor.domain`   
`      notary: notary.harbor.domain`   
`    annotations:`   
`      ingress.kubernetes.io/ssl-redirect: "true"`   
`      nginx.ingress.kubernetes.io/ssl-redirect: "true"`   
`      ingress.kubernetes.io/proxy-body-size: "0"`   
`      nginx.ingress.kubernetes.io/proxy-body-size: "0"`   

`# Harbor 核心服务外部访问 URL。主要用于：`   
`# 1) 补全 portal 页面上面显示的 docker/helm 命令`   
`# 2) 补全返回给 docker/notary 客户端的 token 服务 URL`   
`# 格式：protocol://domain[:port]。`    
`# 1) 如果 expose.type=ingress，"domain"的值就是 expose.ingress.hosts.core 的值`   
`# 2) 如果 expose.type=clusterIP，"domain"的值就是 expose.clusterIP.name 的值`    
`# 3) 如 expose.type=nodePort，"domain"的值就是 k8s 节点的 IP 地址`   
`# 如果在代理后面部署 Harbor，请将其设置为代理的 URL`   
`externalURL: https://registry.harbor.domain`    
`# 默认情况下开启数据持久化，在k8s集群中需要动态的挂载卷默认需要一个StorageClass对象。`   
`# 如果你有已经存在可以使用的持久卷，需要在"storageClass"中指定你的 storageClass 或者设置 "existingClaim"。`    
`# 对于存储 docker 镜像和 Helm charts 包，你也可以用 "azure"、"gcs"、"s3"、"swift" 或者 "oss"，直接在 "imageChartStorage" 区域设置即可`    
`persistence:`    
`  enabled: true`    
`  # 设置成"keep"避免在执行 helm 删除操作期间移除 PVC，留空则在 chart 被删除后删除 PVC`    
`  resourcePolicy: "keep"`    
`  persistentVolumeClaim:`    
`    registry:`    
`      # 使用一个存在的 PVC(必须在绑定前先手动创建)`   
`      existingClaim: ""`   
`      # 指定"storageClass"，或者使用默认的 StorageClass 对象，设置成"-"禁用动态分配挂载卷`   
`      storageClass: "nfs-storage"`   
`      subPath: ""`   
`      accessMode: ReadWriteOnce`   
`      size: 5Gi`   
`    chartmuseum:`   
`      existingClaim: ""`   
`      storageClass: "nfs-storage"`   
`      subPath: ""`   
`      accessMode: ReadWriteOnce`   
`      size: 5Gi`   
`    jobservice:`   
`      existingClaim: ""`   
`      storageClass: "nfs-storage"`   
`      subPath: ""`   
`      accessMode: ReadWriteOnce`   
`      size: 1Gi`   
`    # 如果使用外部的数据库服务，下面的设置将会被忽略`   
`    database:`   
`      existingClaim: ""`   
`      storageClass: "nfs-storage"`   
`      subPath: ""`   
`      accessMode: ReadWriteOnce`   
`      size: 1Gi`   
`    # 如果使用外部的 Redis 服务，下面的设置将会被忽略`   
`    redis:`   
`      existingClaim: ""`   
`      storageClass: "nfs-storage"`   
`      subPath: ""`   
`      accessMode: ReadWriteOnce`   
`      size: 1Gi`   
`  # 定义使用什么存储后端来存储镜像和 charts 包，详细文档地址：`   
`  # https://github.com/docker/distribution/blob/master/docs/configuration.md#storage`   
`  imageChartStorage:`   
`    # 正对镜像和chart存储是否禁用跳转，对于一些不支持的后端(例如对于使用minio的`s3`存储)，`   
`    # 需要禁用它。为了禁止跳转，只需要设置`disableredirect=true`即可，`   
`    # 详细文档地址：https://github.com/docker/distribution/blob/master/docs/configuration.md#redirect`   
`    disableredirect: false`   
`    # 指定存储类型："filesystem", "azure", "gcs", "s3", "swift", "oss"，在相应的区域填上对应的信息。`   
`    # 如果你想使用 pv 则必须设置成"filesystem"类型   
`    type: filesystem`   
`    filesystem:`   
`      rootdirectory: /storage`   
`      #maxthreads: 100`   
` ….`   
`imagePullPolicy: IfNotPresent`   
`logLevel: debug`   
`# Harbor admin 初始密码，Harbor 启动后通过 Portal 修改该密码`   
`harborAdminPassword: "Harbor12345"`   
`# 用于加密的一个 secret key，必须是一个16位的字符串`   
`secretKey: "not-a-secure-key"`   
`# 如果你通过"ingress"保留服务，则下面的Nginx不会被使用`   
`nginx:`   
`  image:`   
`    repository: goharbor/nginx-photon`   
`    tag: v1.7.0`   
`  replicas: 1`   
`  # resources:`   
`  #  requests:`   
`  #    memory: 256Mi`   
`  #    cpu: 100m`   
`  nodeSelector: {}`   
`  tolerations: []`   
`  affinity: {}`   
`  ## 额外的 Deployment 的一些 annotations`   
`  podAnnotations: {}`   
`...`    

`jobservice:`   
`  image:`   
`    repository: goharbor/harbor-jobservice`   
`    tag: v1.7.0`   
`  replicas: 1`   
`  maxJobWorkers: 10`   
`  # jobs 的日志收集器："file", "database" or "stdout"`   
`  jobLogger: file`   
` ...`   
`database:`   
`  # 如果使用外部的数据库，则设置 type=external，然后填写 external 区域的一些连接信息`   
`  type: internal`   
`  internal:`   
`    image:`   
`      repository: goharbor/harbor-db`   
`      tag: v1.7.0`   
`    # 内部的数据库的初始化超级用户的密码`   
`    password: "changeit"`    
`    # resources:`   
`    #  requests:`   
`    #    memory: 256Mi`   
`    #    cpu: 100m`   
` ...`    
`redis:`   
`  # 如果使用外部的 Redis 服务，设置 type=external，然后补充 external 部分的连接信息。
`  type: internal`   
`  internal:`   
`    image:`   
`      repository: goharbor/redis-photon`   
`      tag: v1.7.0`   
`    # resources:`  
`    #  requests:`   
`    #    memory: 256Mi`   
`    #    cpu: 100m`   
`    nodeSelector: {}`   
`    tolerations: []`   
`    affinity: {}`   
`...`    
##  执行部署
`kubectl create namespace harbor`   
`kubectl create serviceaccount --namespace kube-system tiller`    
`kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller`   
`kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'`    
`helm install --debug --name harbor-v1 -f values.yaml  . --namespace harbor`   

上面是我们通过 Helm 安装所有涉及到的一些资源对象，稍微等一会儿，就可以安装成功了，查看对应的 Pod 状态：
`kubectl get pods -n harbor`   
现在都是Running状态了，都成功运行起来了，查看下对应的 Ingress 对象：

`kubectl get ingress -n kube-ops`   
`NAME                    HOSTS                                     ADDRESS   PORTS     AGE`   
`harbor-harbor-ingress   registry.harbor.domain,notary.harbor.domain             80, 443   50m`   

如果你有自己的真正的域名，则将上面的两个域名解析到你的任意一个 Ingress Controller 的 Pod 所在的节点即可，
我们这里为了演示方便，还是自己在本地的虚拟机hosts和windows的hosts里面添加上registry.harbor.domain和notary.harbor.domain的映射。

在第一次安装的时候比较顺畅，后面安装总是不成功，查看数据库的 Pod 日志出现database “registry” does not exist（）
的错误信息，如果 registry 数据库没有自动创建，我们可以进入数据库 Pod 中手动创建：
` 1. 进入数据库 Pod`   
`$ kubectl exec -it harbor-harbor-database-0 -n kube-ops /bin/bash`   
`# 2. 连接数据库`   
`root [ / ]# psql --username postgres`   
`psql (9.6.10)`   
`Type "help" for help.`   
`# 3. 创建 registry 数据库`   
`postgres=# CREATE DATABASE registry ENCODING 'UTF8';`   
`CREATE DATABASE`   
`postgres=# \c registry;`   
`You are now connected to database "registry" as user "postgres".`   
`registry=# CREATE TABLE schema_migrations(version bigint not null primary key, dirty boolean not null);`   
`CREATE TABLE`   
`registry-# \quit`   
### Harbor Portal
添加完成后，在浏览器中输入registry.harbor.domain就可以打开熟悉的 Harbor 的 Portal 界面了，当然我们配置的 
Ingress 中会强制跳转到 https，所以如果你的浏览器有什么安全限制的话，需要信任我们这里 Ingress 对应的证书，
证书文件可以通过查看 Secret 资源对象获取：
然后输入用户名：admin，密码：Harbor12345即可登录进入 Portal 首页：
我们可以看到有很多功能，默认情况下会有一个名叫library的项目，改项目默认是公开访问权限的，
进入项目可以看到里面还有 Helm Chart 包的管理，可以手动在这里上传，也可以对改项目里面的镜像进行一些配置，比如是否开启自动扫描镜像功能：
### docker cli
然后我们来测试下使用 docker cli 来进行 pull/push 镜像，由于上面我们安装的时候通过 Ingress
来暴露的 Harbor 的服务，而且强制使用了 https，所以如果我们要在终端中使用我们这里的私有仓库的话，就需要配置上相应的证书：

`docker login registry.harbor.domain`   
`Warning: failed to get default registry endpoint from daemon (Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?). Using system default: https://index.docker.io/v1/`   
`Username: admin`   
`Password: `   
`INFO[0007] Error logging in to v2 endpoint, trying next endpoint: Get https://registry.harbor.domain/v2/: x509: certificate has expired or is not yet valid`   
`INFO[0007] Error logging in to v1 endpoint, trying next endpoint: Get https://registry.harbor.domain/v1/users/: x509: certificate has expired or is not yet valid`   
`Get https://registry.harbor.domain/v1/users/: x509: certificate has expired or is not yet valid`    
这是因为我们没有提供证书文件，
修改daemon.json忽略证书的校验，并重启登陆
`cat /etc/docker/daemon.json`   
` {`   
`"insecure-registries":["https://registry.harbor-test.com:31470"]`   
`}`     
然后保存重启 docker，再使用 docker cli 就没有任何问题了：
`$ docker login registry.harbor.domain`   
`Username: admin`   
`Password:`   
`Login Succeeded`   
比如我们本地现在有一个名为 busybox 的镜像，现在我们想要将该镜像推送到我们的私有仓库中去，应该怎样操作呢？首先我们需要给该镜像重新打一个 registry.harbor.domain 的前缀，然后推送的时候就可以识别到推送到哪个镜像仓库：
`$ docker tag busybox registry.harbor.domain/library/busybox`   
`$ docker push registry.harbor.domain/library/busybox`   
`The push refers to repository [registry.harbor.domain/library/busybox]`   
`adab5d09ba79: Pushed`   
`latest: digest: sha256:4415a904b1aca178c2450fd54928ab362825e863c0ad5452fd020e92f7a6a47e size: 527`   
推送完成后，我们同样可以在 Portal 页面上看到这个镜像的信息：
镜像 push 成功，同样可以测试下 pull：

`$ docker rmi registry.harbor.domain/library/busybox`   
`Untagged: registry.harbor.domain/library/busybox:latest`   
`Untagged: registry.harbor.domain/library/busybox@sha256:4415a904b1aca178c2450fd54928ab362825e863c0ad5452fd020e92f7a6a47e`   
`$ docker pull registry.harbor.domain/library/busybox:latest`   
`latest: Pulling from library/busybox`   
`Digest: sha256:4415a904b1aca178c2450fd54928ab362825e863c0ad5452fd020e92f7a6a47e`    
`Status: Downloaded newer image for registry.harbor.domain/library/busybox:latest`   
`$ docker images |grep busybox`   
`busybox                                latest              d8233ab899d4        7 days ago          1.2MB`   
`registry.harbor.domain/library/busybox   latest              d8233ab899d4        7 days ago          1.2MB`   


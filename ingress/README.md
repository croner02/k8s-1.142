# 部署nginx-ingress

## 1、部署nginx-ingress controller
`kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/mandatory.yaml`   
## 2、部署nodeport的server
`kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/baremetal/service-nodeport.yaml`   
## 3、验证安装是否成功
`kubectl get pods --all-namespaces -l app.kubernetes.io/name=ingress-nginx --watch`   
## 4、如果你看到ingress controller的状态是running，就代表安装成功，现在你可以安装自己的ingress了。
## 5、查看nginx ingress controller暴露的端口
`$ kubectl get svc -n ingress-nginx`   
`NAME            TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE`   
`ingress-nginx   NodePort   172.19.88.130   <none>        80:32257/TCP,443:30839/TCP   6d`   

可以看到这里对外暴露出来的端口是32257和30839两个。

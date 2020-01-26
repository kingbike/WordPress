# docker-compose 建立 wordpress
docker-compose up -d --build
docker-compose ps
docker-compose stop
docker-compose rm

# 以上砍掉 container 實際上 volume 是不會消失的. 


# 看 volume 掛在哪裡 
@docker-compose  下面的路徑其實是掛在 windows linux 內的路徑 ref : https://dotblogs.com.tw/swater111/2017/02/07/181331
PS D:\workspace\wordpress> docker inspect -f '{{range .Mounts}}{{.Source}}{{end}}' 55120b57fa6d    
/var/lib/docker/volumes/wordpress_db_data/_data

- 看 volume 資訊 
$ docker volume ls
$ docker volume inspect wordpress_db_data
- 把 docker compose down 不會把 volume 砍掉 要下 
$ docker-compose down --volumes

- 砍掉所有 volume . 
S D:\workspace\wordpress> docker volume prune
WARNING! This will remove all local volumes not used by at least one container.
Are you sure you want to continue? [y/N] y


@k8s 下面是掛在  C:\Users\jerry\.docker\Volumes
PS D:\workspace\wordpress> docker inspect -f '{{range .Mounts}}{{.Source}}{{end}}' 53d70322b90f    
/var/lib/kubelet/pods/34968a80-3ce8-11ea-adb3-00155d6d0107/containers/mysql/7c736abb/host_mnt/c/Users/jerry/.docker/Volumes/mysql-pv-claim/pvc-349664c5-3ce8-11ea-adb3-00155d6d0107/var/lib/kubelet/pods/34968a80-3ce8-11ea-adb3-00155d6d0107/volumes/kubernetes.io~secret/default-token-x4mkh/var/lib/kubelet/pods/34968a80-3ce8-11ea-adb3-00155d6d0107/etc-hosts

# k8s
# 建立 deployment nginx
------------------------------------------------------------------------------------------------
PS C:\Users\jerry\Desktop\kkk> kubectl create -f nginx-deployment.yaml
deployment.apps/nginx-deployment created

PS C:\Users\jerry\Desktop\kkk> kubectl get deployments/nginx-deployment -o yaml

PS C:\Users\jerry\Desktop\kkk> kubectl get rs -o wide
NAME                          DESIRED   CURRENT   READY   AGE     CONTAINERS   IMAGES         SELECTOR
nginx-deployment-66f7f56f56   2         2         2       2m41s   nginx        nginx:1.15.4   app=nginx,pod-template-hash=66f7f56f56


PS C:\Users\jerry\Desktop\kkk> kubectl get pods -o wide
NAME                                READY   STATUS    RESTARTS   AGE   IP         NODE             NOMINATED NODE   READINESS GATES
nginx-deployment-66f7f56f56-h52m8   1/1     Running   0          13m   10.1.0.6   docker-desktop   <none>           <none>
nginx-deployment-66f7f56f56-pl5b6   1/1     Running   0          13m   10.1.0.5   docker-desktop   <none>           <none>


PS C:\Users\jerry\Desktop\kkk> kubectl exec -it nginx-deployment-66f7f56f56-h52m8 bash
root@nginx-deployment-66f7f56f56-h52m8:/# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var


# 建立 service hello
https://kubernetes.io/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/
------------------------------------------------------------------------------------------------
PS C:\Users\jerry\Desktop\kkk> kubectl create -f .\hello-service.yaml
service/hello created


PS C:\Users\jerry\Desktop\kkk> kubectl get service -o wide
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE    SELECTOR
hello        ClusterIP   10.99.166.10   <none>        80/TCP    13s    app=nginx
kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP   154m   <none>


PS C:\Users\jerry\Desktop\kkk> kubectl delete -f .\nginx-deployment.yaml
deployment.apps "nginx-deployment" deleted


PS D:\workspace\rfid\Kubernetes> kubectl logs frontend-895c8799c-ckdvc
2019/12/11 00:39:01 [emerg] 1#1: host not found in upstream "hello" in /etc/nginx/conf.d/frontend.conf:2
nginx: [emerg] host not found in upstream "hello" in /etc/nginx/conf.d/frontend.conf:2


# 啟動 wordpress , 分別啟動 wordpress 跟  mysql 
-------------------------------------
kubectl create secret generic mysql-pass --from-literal=password=chttl232
kubectl create -f .\mysql-deployment.yaml
kubectl create -f .\wordpress-deployment.yaml

# 也可以寫在一個 kustomization.yaml 檔案裡 
-------------
kubectl apply -k ./               //kustomization.yaml
kubectl delete -k ./

# loadbalance 是要搭配公雲； nodePort 可以讓外部存取. 
# 創建一個NodePort，讓在 Kubernetes Cluster外但在同一個 Node 上的其他服務，可以透過這個 NodePort 訪問到 Kubernetes Cluster 內正在運行中的 Pods。
# Windows 防火牆設定 將外來的 80 port 導到 nodePort 指定的 30036 去
netsh interface portproxy add v4tov4 listenport=80 listenaddress=122.116.214.159 connectport=30036 connectaddress=localhost
netsh interface portproxy delete v4tov4 listenport=80 listenaddress=122.116.214.159

# 停止 k8s
PS D:\workspace\wordpress\Kubernetes> kubectl get deployments
NAME              READY   UP-TO-DATE   AVAILABLE   AGE
wordpress         1/1     1            1           52m
wordpress-mysql   1/1     1            1           52m
PS D:\workspace\wordpress\Kubernetes> kubectl scale --replicas=0 deployments/wordpress
deployment.extensions/wordpress scaled
PS D:\workspace\wordpress\Kubernetes> kubectl scale --replicas=0 deployments/wordpress-mysql
deployment.extensions/wordpress-mysql scaled


# 查看 k8s 服務指應
kubectl get service -o wide
kubectl get secrets
kubectl get pvc
kubectl get deployments
kubectl get pods
kubectl get nodes

kubectl config view
kubectl config get-contexts
kubectl config use-context docker-for-desktop



# 參考資料: 
----------------------------------------------------------------------------------------------------------------------------------

Local Kubernetes for Windows — MiniKube vs Docker Desktop
https://medium.com/containers-101/local-kubernetes-for-windows-minikube-vs-docker-desktop-25a1c6d3b766


https://github.com/kubernetes/dashboard
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml
kubectl proxy

kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')


http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/#!/overview?namespace=default



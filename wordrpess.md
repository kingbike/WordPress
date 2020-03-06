
# Running services.
-----------------------------------------------------------------------------------------------------
##### docker-compose command to create containers
- $ docker-compose up -d --build
##### docker-compose command to show running containers
- $ docker-compose ps
##### docker-compose command to stop containers
- $ docker-compose stop
##### docker-compose command to remove containers
- $ docker-compose rm

##### if you modify the docker-compose.yaml file just restart the service. it will become effective
- $ docker-compose restar <service : db>


## volumes
-----------------------------------------------------------------------------------------------------
##### while you remove containers the volume dokcer-compose create will not be acturlly remove. show where volumes is ( the volume it acturlly path is in windows vm.  )
- PS D:\workspace\wordpress> docker inspect -f '{{range .Mounts}}{{.Source}}{{end}}' 55120b57fa6d    
/var/lib/docker/volumes/wordpress_db_data/_data

##### volume status 
- $ docker volume ls 
- $ docker volume inspect <wordpress_db_data>

##### let volmes  splite from container 
- $ docker-compose down --volumes
##### delete all volumes permanently  
- $ D:\workspace\wordpress> docker volume prune
##### WARNING! This will remove all local volumes not used by at least one container.
Are you sure you want to continue? [y/N] y

##### while using windows K8s the volume will locate in C:\Users\jerry\.docker\Volumes
- PS D:\workspace\wordpress> docker inspect -f '{{range .Mounts}}{{.Source}}{{end}}' 53d70322b90f    
/var/lib/kubelet/pods/34968a80-3ce8-11ea-adb3-00155d6d0107/containers/mysql/7c736abb/host_mnt/c/Users/jerry/.docker/Volumes/mysql-pv-claim/pvc-349664c5-3ce8-11ea-adb3-00155d6d0107/var/lib/kubelet/pods/34968a80-3ce8-11ea-adb3-00155d6d0107/volumes/kubernetes.io~secret/default-token-x4mkh/var/lib/kubelet/pods/34968a80-3ce8-11ea-adb3-00155d6d0107/etc-hosts



# K8s
### deployment nginx
------------------------------------------------------------------------------------------------
- PS C:\Users\jerry\Desktop\kkk> kubectl create -f nginx-deployment.yaml
deployment.apps/nginx-deployment created

- PS C:\Users\jerry\Desktop\kkk> kubectl get deployments/nginx-deployment -o yaml

- PS C:\Users\jerry\Desktop\kkk> kubectl get rs -o wide
NAME                          DESIRED   CURRENT   READY   AGE     CONTAINERS   IMAGES         SELECTOR
nginx-deployment-66f7f56f56   2         2         2       2m41s   nginx        nginx:1.15.4   app=nginx,pod-template-hash=66f7f56f56

- PS C:\Users\jerry\Desktop\kkk> kubectl get pods -o wide
NAME                                READY   STATUS    RESTARTS   AGE   IP         NODE             NOMINATED NODE   READINESS GATES
nginx-deployment-66f7f56f56-h52m8   1/1     Running   0          13m   10.1.0.6   docker-desktop   <none>           <none>
nginx-deployment-66f7f56f56-pl5b6   1/1     Running   0          13m   10.1.0.5   docker-desktop   <none>           <none>

- PS C:\Users\jerry\Desktop\kkk> kubectl exec -it nginx-deployment-66f7f56f56-h52m8 bash
root@nginx-deployment-66f7f56f56-h52m8:/# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

# deployment service hello 
###### (https://kubernetes.io/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/ )
------------------------------------------------------------------------------------------------
- PS C:\Users\jerry\Desktop\kkk> kubectl create -f .\hello-service.yaml
service/hello created

- PS C:\Users\jerry\Desktop\kkk> kubectl get service -o wide
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE    SELECTOR
hello        ClusterIP   10.99.166.10   <none>        80/TCP    13s    app=nginx
kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP   154m   <none>

- PS C:\Users\jerry\Desktop\kkk> kubectl delete -f .\nginx-deployment.yaml
deployment.apps "nginx-deployment" deleted

- PS D:\workspace\rfid\Kubernetes> kubectl logs frontend-895c8799c-ckdvc
2019/12/11 00:39:01 [emerg] 1#1: host not found in upstream "hello" in /etc/nginx/conf.d/frontend.conf:2
nginx: [emerg] host not found in upstream "hello" in /etc/nginx/conf.d/frontend.conf:2

### deploy wordpress and mysql sequencely  
- kubectl create secret generic mysql-pass --from-literal=password=chttl232
- kubectl create -f .\mysql-deployment.yaml
- kubectl create -f .\wordpress-deployment.yaml

### Or you can combine two yaml above into one kustomization.yaml then  
- $ kubectl apply -k ./               //kustomization.yaml
- $ kubectl delete -k ./

### 'type: loadbalance' is used for public cloud ;  'type: NodePort' let you export service pods  to other services in the same node.

- Windows Set portforwding 80 port to K8s port 30036, the you can access service through port 80 from internet 
-- $ netsh interface portproxy add v4tov4 listenport=80 listenaddress=122.116.214.159 connectport=30036 connectaddress=localhost
- nremove setting 
-- $ netsh interface portproxy delete v4tov4 listenport=80 listenaddress=122.116.214.159

#  Stop k8s
- PS D:\workspace\wordpress\Kubernetes> kubectl get deployments
NAME              READY   UP-TO-DATE   AVAILABLE   AGE
wordpress         1/1     1            1           52m
wordpress-mysql   1/1     1            1           52m
- PS D:\workspace\wordpress\Kubernetes> kubectl scale --replicas=0 deployments/wordpress
deployment.extensions/wordpress scaled
- PS D:\workspace\wordpress\Kubernetes> kubectl scale --replicas=0 deployments/wordpress-mysql
deployment.extensions/wordpress-mysql scaled

# K8s commands to show status
- kubectl get service -o wide
- kubectl get secrets
- kubectl get pvc
- kubectl get deployments
- kubectl get pods
- kubectl get nodes
- kubectl config view
- kubectl config get-contexts
- kubectl config use-context docker-for-desktop

# reference : 
-------------------
- Local Kubernetes for Windows — MiniKube vs Docker Desktop
https://medium.com/containers-101/local-kubernetes-for-windows-minikube-vs-docker-desktop-25a1c6d3b766

- https://github.com/kubernetes/dashboard
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml
kubectl proxy

- kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')

- http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/#!/overview?namespace=default



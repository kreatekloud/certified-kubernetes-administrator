# 🚀 Troubleshooting ArgoCD on K3s Kubernetes Cluster
This guide walks through troubleshooting ArgoCD on a K3s cluster and accessing its UI locally.

## Check argocd NodePort details
```
root@k3s-master:~# kubectl get svc -n argocd
NAME                                      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
argocd-applicationset-controller          ClusterIP   10.43.19.190    <none>        7000/TCP,8080/TCP            28d
argocd-dex-server                         ClusterIP   10.43.12.18     <none>        5556/TCP,5557/TCP,5558/TCP   28d
argocd-metrics                            ClusterIP   10.43.50.201    <none>        8082/TCP                     28d
argocd-notifications-controller-metrics   ClusterIP   10.43.160.26    <none>        9001/TCP                     28d
argocd-redis                              ClusterIP   10.43.221.173   <none>        6379/TCP                     28d
argocd-repo-server                        ClusterIP   10.43.2.227     <none>        8081/TCP,8084/TCP            28d
argocd-server                             NodePort    10.43.135.43    <none>        80:32632/TCP,443:31557/TCP   28d
argocd-server-metrics                     ClusterIP   10.43.194.127   <none>        8083/TCP                     28d
root@k3s-master:~#
```

## Access argocd UI using the NodePort

```
argocd-server                             NodePort    10.43.135.43    <none>        80:32632/TCP,443:31557/TCP   28d
go to browser and enter the server IP followed by NodePort details as mentioned above.
Example: https://172.31.252.59:32632/
Enter username and password: admin / devops@123
```
## Check if all the pods running in argocd namespace
Check if all the pods are running fine. if running but ready shows 0/1 then that needs to be fixed.

```
root@k3s-master:~# kubectl get po -n argocd
NAME                                                READY   STATUS    RESTARTS         AGE
argocd-application-controller-0                     1/1     Running   6 (11d ago)      28d
argocd-applicationset-controller-7dc85c9598-f98bp   1/1     Running   5 (11d ago)      28d
argocd-dex-server-7d88b5b798-52p6v                  1/1     Running   5 (11d ago)      28d
argocd-notifications-controller-5f95df7f8c-cvntb    1/1     Running   21 (2d15h ago)   28d
argocd-redis-65b5bbd8cb-sbp8d                       1/1     Running   5 (11d ago)      28d
argocd-repo-server-58ccbf46d6-6v7bv                 1/1     Running   7 (80s ago)      15m
argocd-server-58dbdffdcd-29m2d                      1/1     Running   5 (8m10s ago)    15m
root@k3s-master:~#
```
## Check pods log
```
kubectl -n argocd describe po argocd-server-58dbdffdcd-29m2d  
kubectl logs argocd-server-58dbdffdcd-29m2d -n argocd
```

## Fix pod failure
If any pods failure like argocd-server-xxx or argocd-repo-server-xxx then argocd UI will not work or github sync will not work to fix it follow the steps:

Try delete the respective pod and it will be redeployed. if consistently failing then try restarting the k3s service.
```
kubectl -n argocd delete po argocd-repo-server-58ccbf46d6-6v7bv 
kubectl -n argocd delete po argocd-server-58dbdffdcd-29m2d  
systemctl restart k3s
```

## Useful commands
```
kubectl get pods -A --> List all the pods from all namespaces
kubectl get nodes  --> List the cluster nodes
kubectl get svc kubernetes --> List the ClusterIP and NodePort
sudo systemctl restart containerd
sudo systemctl restart k3s
```

## Login to argocd using CLI and UI

Enter username and password like admin/devops@123 after successful login you should be able to run the argocd commands
```
argocd login 172.31.252.59:32632

root@k3s-master:~# argocd login 172.31.252.59:32632
WARNING: server certificate had error: error creating connection: tls: failed to verify certificate: x509: cannot validate certificate for 172.31.252.59 because it doesn't contain any IP SANs. Proceed insecurely (y/n)? y
Username: admin
Password: 
'admin:login' logged in successfully
Context '172.31.252.59:32632' updated
```
```
root@k3s-master:~# argocd account can-i create clusters '*'
yes
```
To login to UI, you can go to any browser and enter the above address and then enter the username and password.




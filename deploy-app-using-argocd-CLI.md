# 🚀 Deploy Application to Kubernetes using Argo CD CLI

This guide shows how to deploy an application to a Kubernetes cluster using the Argo CD CLI following GitOps principles.

## 🧠 Prerequisites

- Kubernetes cluster up and running
- Argo CD installed in the cluster
- Argo CD CLI installed
- Git repository with Kubernetes manifests

## 📦 Step 1: Start Port Forwarding

Change Service Type from ClusterIP to NodePort:

```
kubect edit svc argocd-server -n argocd
```
## 🔐 Step 2: Login to Argo CD
```
argocd login 172.27.111.151:32632 --insecure
```

## 📁 Step 3: Create Application in Argo CD
```
argocd app create guestbook-app \
--repo https://github.com/kreatekloud/argocd-sample-apps \
--path guestbook \
--dest-namespace default \
--dest-server https://kubernetes.default.svc
```

## 🔄 Step 4: Sync Application
```
argocd app sync guestbook-app
```

## 📊 Step 5: Check Application Status
```
argocd app get guestbook-app
```
## 📋 Step 6: List Applications
```
argocd app list
```
## 🧪 Step 7: Verify Deployment in Kubernetes
```
kubectl get pods
kubectl get svc
```
## 🔁 Step 8: Enable Auto-Sync (Optional)
```
argocd app set guestbook-app --sync-policy automated
```
## 🗑️ Step 9: Delete Application (Optional)
```
argocd app delete nginx-app
```

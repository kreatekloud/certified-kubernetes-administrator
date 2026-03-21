# 🚀 Install ArgoCD on K3s Kubernetes Cluster

This guide walks through installing ArgoCD on a K3s cluster and accessing its UI locally.

## 🧠 What is Argo CD?

Argo CD is a GitOps continuous delivery tool for Kubernetes. It helps you deploy and manage applications using Git as the source of truth.

## ⚙️ Prerequisites

- K3s cluster up and running
- kubectl access to the cluster
- Internet access from cluster nodes

Verify cluster:

```
kubectl get nodes
```

## 📦 Step 1: Create ArgoCD Namespace

Check for the nodes and name space:
```
kubect get nodes
kubect get ns
```

Create argoCD Namespace on your kubernetes cluster:
```
kubectl create namespace argocd
```

## 🚀 Step 2: Install Argo CD

Apply the official manifest:
```
kubectl apply -n argocd --server-side --force-conflicts -f https://raw.githubusercontent.com/argoproj/argo-cd/v3.3.4/manifests/install.yaml
```

## 📚 References

- [Argo CD Official Documentation](https://argo-cd.readthedocs.io/en/stable/)
- [Argo CD GitHub Releases](https://github.com/argoproj/argo-cd/releases)

## ⏳ Step 3: Wait for Pods to be Ready

```
kubectl get all -n argocd
kubectl get pods -n argocd
```
Wait until all pods show Running or Completed.

## 🌐 Step 4: Access ArgoCD UI (Port Forward)

```
kubect get svc -n argocd
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
## 🔧 Step 5: Change Service Type to NodePort
```
kubect edit svc argocd-server -n argocd
or
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'
```
Check the service again to check the Type and Port details to access it from the cluster IP:
```
for example: argocd-server  NodePort <cluster-IP>  Ports <80:30664>
```


Open browser:
```
https://172.27.111.151:30664
```

## 🔑 Step 6: Get Initial Admin Password

```
kubectl get secret -n argocd
kubectl get secret argocd-initial-admin-secret -n argocd -o json | jq .data.password -r | base64 -d
(or)
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d
```

## 🔐 Step 7: Login to Argo CD

```
Username: admin
Password: (output from previous command)
```

Check port:
```
kubectl get svc -n argocd
```
## 🧪 Step 8: Verify ArgoCD Components

```
kubectl get all -n argocd
```
## 🗑️ Step 9: Uninstall Argo CD (Optional)
```
kubectl delete namespace argocd
```
## 🖥️ Install Argo CD CLI

The Argo CD CLI allows you to interact with Argo CD from the command line.

---

### 📦 Step 1: Download Argo CD CLI

#### 👉 On Linux / WSL

```
wget https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
mv argocd-linux-amd64 argocd
```

## 🔐 Step 2: Make Binary Executable
```
chmod +x argocd
```
## 📁 Step 3: Move Binary to PATH
```
sudo mv argocd /usr/local/bin/
```
## ✅ Step 4: Verify Installation
```
argocd version
```
## 🔑 Login to Argo CD using CLI

Make sure port-forward is running:
```
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
## 🔐 Step 5: Login
```
argocd login 172.27.111.15
```
## 🔓 If you face SSL warning (expected in local setup)
```
argocd login 172.27.111.15 --insecure
```
## 📋 Step 6: List Applications
```
argocd app list
argocd cluster list
```
## 🧠 Notes

CLI is optional but very useful for automation

Use --insecure only for local/testing environments

For production, configure proper TLS

## 📚 References

Argo CD Releases
- [Argo CD CLI Documentation]((https://argo-cd.readthedocs.io/en/stable/getting_started/#2-download-argo-cd-cli))
- [Argo CD GitHub Releases](https://github.com/argoproj/argo-cd/releases)


## 🙌 Conclusion

You have successfully installed Argo CD on your K3s cluster and can now manage Kubernetes deployments using GitOps.

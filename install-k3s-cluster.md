# 🚀 Setup Production-like K3s Kubernetes Cluster on Windows (WSL + Multipass)

This guide walks through setting up a **multi-node Kubernetes cluster** using **K3s** on a Windows 11 laptop using WSL and Multipass.

## 🧠 Architecture

```text
Windows 11
├── WSL (Ubuntu - CLI tools)
├── Multipass (VMs)
│   ├── master (control plane)
│   ├── worker1
│   └── worker2
└── K3s Cluster (inside VMs)
```
---

## ⚙️ Prerequisites

- Windows 11
- Admin access
- Minimum 16 GB RAM (recommended)


## 🪟 Step 1: Install WSL

Open PowerShell (Admin):

```
wsl --install

```
Restart your system.

## 🐧 Step 2: Install Ubuntu in WSL

Install Ubuntu from Microsoft Store (22.04 or 24.04).

Verify:

wsl

## 🖥️ Step 3: Install Multipass

Download and install Multipass:
```

👉 https://multipass.run

```
Verify installation:

```
multipass list
```

## 🧱 Step 4: Create Virtual Machines

```
multipass launch --name master --cpus 2 --mem 4G --disk 20G
multipass launch --name worker1 --cpus 2 --mem 2G --disk 20G
multipass launch --name worker2 --cpus 2 --mem 2G --disk 20G
```
Check instances:

```
multipass list
```
## 🚀 Step 5: Install K3s on Master Node
```
multipass shell master
```

Install K3s:
```
curl -sfL https://get.k3s.io | sh -
```
Verify:
```
sudo kubectl get nodes
```
## 🔑 Step 6: Get Node Join Token
```
sudo cat /var/lib/rancher/k3s/server/node-token
```
Copy the token.

## 🌐 Step 7: Get Master Node IP

From host:
```
multipass list
```
Note the master IP.

## 🔗 Step 8: Join Worker Nodes
On worker1
```
multipass shell worker1
```
```
curl -sfL https://get.k3s.io | K3S_URL=https://<MASTER-IP>:6443 K3S_TOKEN=<TOKEN> sh -
```
On worker2
```
multipass shell worker2
curl -sfL https://get.k3s.io | K3S_URL=https://<MASTER-IP>:6443 K3S_TOKEN=<TOKEN> sh -
```
## ✅ Step 9: Verify Cluster
```
multipass shell master
sudo kubectl get nodes
```

Expected output:
```
master    Ready   control-plane
worker1   Ready
worker2   Ready
```
## ⚙️ Step 10: Configure kubectl (Optional)
```
mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $USER:$USER ~/.kube/config
```
Verify:
```
kubectl get nodes
```
## 🧪 Test Deployment (Optional)
```
kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --port=80 --type=NodePort
kubectl get pods -o wide
kubectl get svc
```
## 🔄 Multipass Commands

Start VMs:
```
multipass start master worker1 worker2
```
Stop VMs:
```
multipass stop master worker1 worker2
```
Delete all VMs:
```
multipass delete --all
multipass purge
```

# Kubernetes Cluster Setup Guide

## 1. Setting Up the Virtual Machines
- Used **Debian 12** and **Rocky Linux 9.4** for the Kubernetes cluster.
- Configured **static IPs** on all nodes.

---

## 2. Configuring Network Settings
### Install Network Manager
For Debian:
```bash
sudo apt install network-manager -y
```

For Rocky Linux:
```bash
sudo yum install NetworkManager -y
```

### Modify NetworkManager Configuration
Edit the configuration file:
```bash
sudo nano /etc/NetworkManager/NetworkManager.conf
```

Update the `[main]` section:
```ini
[main]
managed=true
```

Restart NetworkManager:
```bash
sudo systemctl restart NetworkManager
```

Set a **static IP** using:
```bash
sudo nmtui
```

---

## 3. Installing Kubernetes Control Plane (Master Node)
### Disable Swap
```bash
sudo swapoff -a
sudo sed -i '/swap/d' /etc/fstab
```

### Enable Kernel Modules
```bash
sudo modprobe overlay
sudo modprobe br_netfilter
```

Persist these settings:
```bash
echo "overlay" | sudo tee -a /etc/modules-load.d/k8s.conf
echo "br_netfilter" | sudo tee -a /etc/modules-load.d/k8s.conf
```

### Configure Sysctl Settings
```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF
```

Apply settings:
```bash
sudo sysctl --system
```

### Install Dependencies
```bash
sudo apt install -y apt-transport-https curl
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
```

### Add Kubernetes Repository
```bash
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

### Install Kubernetes
```bash
sudo apt update
sudo apt install -y kubelet kubeadm kubectl containerd
sudo apt-mark hold kubelet kubeadm kubectl
```

### Initialize Control Plane
```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

### Set Up `kubectl`
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### Deploy Flannel Network Plugin
```bash
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
```

### Verify Installation
```bash
kubectl get pods -n kube-system
```

---

## 4. Installing Kubernetes Worker Nodes (Debian & Red Hat)
### Perform the Same Kernel Module Setup
```bash
sudo swapoff -a
sudo modprobe overlay
sudo modprobe br_netfilter
```

Persist settings:
```bash
echo "overlay" | sudo tee -a /etc/modules-load.d/k8s.conf
echo "br_netfilter" | sudo tee -a /etc/modules-load.d/k8s.conf
```

Configure sysctl:
```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF
```

Apply settings:
```bash
sudo sysctl --system
```

### Install Kubernetes Packages
```bash
sudo apt update
sudo apt install -y kubelet kubeadm kubectl containerd
sudo apt-mark hold kubelet kubeadm kubectl
```

Enable and start services:
```bash
sudo systemctl enable --now kubelet containerd
```

---

## 5. Joining Worker Nodes to the Cluster
Use the **token and CA certificate hash** from the control plane:
```bash
sudo kubeadm join <MASTER_IP>:6443 --token <TOKEN> --discovery-token-ca-cert-hash sha256:<CA_HASH>
```

### Verify Worker Node Addition
```bash
kubectl get nodes -o wide
```

---

## 6. Installing Kubernetes Dashboard
### Deploy the Dashboard
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.5.1/aio/deploy/recommended.yaml
```

### Change the Service Type
```bash
kubectl -n kubernetes-dashboard edit service kubernetes-dashboard
```

Update the configuration to:
```yaml
type: NodePort
```

### Allow Access to the Dashboard
```bash
sudo ufw allow 32000/tcp
```

### Access the Dashboard
```
https://<NODE_IP>:32000
```

---

## 7. Configuring Firewall Rules
### Open Required Ports
```bash
sudo ufw allow 6443/tcp  # API Server
sudo ufw allow 32000/tcp  # Dashboard
```

---

## 8. Understanding Kubernetes Concepts
### Tokens
```bash
kubeadm token list
```

### Secrets
```bash
kubectl get secrets -n kubernetes-dashboard
```

### ClusterRoleBinding for Dashboard
```bash
kubectl create clusterrolebinding dashboard-admin-binding \
  --clusterrole=cluster-admin \
  --serviceaccount=kubernetes-dashboard:dashboard-admin
```

### Create a Service Account
```bash
kubectl create serviceaccount dashboard-admin -n kubernetes-dashboard
```

### Retrieve Authentication Token
```bash
kubectl -n kubernetes-dashboard get secret \
  $(kubectl -n kubernetes-dashboard get sa/dashboard-admin -o jsonpath="{.secrets[0].name}") \
  -o jsonpath="{.data.token}" | base64 --decode
```

---

## 9. Final Verification & Testing
### Check Node Status
```bash
kubectl get nodes -o wide
```

### Check Services
```bash
kubectl get svc -A
```

### Check Pods
```bash
kubectl get pods -A
```

Successfully accessed Kubernetes Dashboard!

---

## Conclusion
- Kubernetes Cluster setup (Master & Worker Nodes).
- Kubernetes Dashboard with external access.
- Authentication & firewall rules configured.
- Step-by-step setup guide for Kubernetes deployment.

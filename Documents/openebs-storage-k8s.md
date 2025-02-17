# 📖 Complete Guide: Setting Up OpenEBS with Kubernetes & Helm

---

## 1️⃣ What is OpenEBS?
OpenEBS is an open-source **Container Attached Storage (CAS)** solution designed for Kubernetes. It allows you to manage persistent storage dynamically.

### 🔹 Why Use OpenEBS?
- Enables **persistent storage** for Kubernetes applications.
- Supports **dynamic provisioning** of volumes.
- Works with different types of storage (**raw disks, directories, cloud storage**).
- Provides **data protection** with built-in snapshot & backup features.
- **Easily scalable** for production environments.

---

## 2️⃣ OpenEBS Storage Engines
OpenEBS offers different storage engines to match various use cases.

| Storage Engine         | Description                                      | Best For |
|------------------------|--------------------------------------------------|----------|
| **Local PV (Hostpath)**| Uses a directory on a node                       | Fast storage for single-node apps |
| **Local PV (Device)**  | Uses raw block storage devices                   | Fast, direct access to SSD/HDD |
| **Jiva**               | Replicated block storage in containers           | Small databases, HA workloads |
| **cStor**              | Pool-based block storage with snapshots & replication | Production databases, HA apps |
| **Mayastor**           | High-performance NVMe storage                    | Ultra-fast applications |

📌 For this guide, we use **Local PV (Hostpath)**, which is the simplest setup.

---

## 3️⃣ Installing Helm (Required for OpenEBS)

Helm is a package manager for Kubernetes that simplifies the installation of complex applications.

### 🔹 Install Helm on Linux
```bash
curl -fsSL https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

### 🔹 Verify Helm Installation
```bash
helm version
```
✅ If installed correctly, you should see output similar to:
```plaintext
version.BuildInfo{Version:"v3.x.x", GitCommit:"xxxxxx", GoVersion:"go1.xx.x"}
```

---

## 4️⃣ Installing OpenEBS using Helm

### 🔹 Add the OpenEBS Helm Repository
```bash
helm repo add openebs https://openebs.github.io/charts
helm repo update
```

### 🔹 Install OpenEBS
```bash
helm install openebs --namespace openebs openebs/openebs --create-namespace
```
✅ This installs OpenEBS into the **openebs** namespace.

---

## 5️⃣ Verifying OpenEBS Installation

### 🔹 Check OpenEBS Pods
```bash
kubectl get pods -n openebs -o wide
```

✅ Expected output:
```plaintext
NAME                                           READY   STATUS    RESTARTS   AGE   IP            NODE
openebs-localpv-provisioner-xxxxxx             1/1     Running   0          10m   10.244.1.10   kubeadm-wn01
openebs-ndm-operator-yyyyyy                    1/1     Running   0          10m   10.244.2.20   kubeadm-wn02
openebs-ndm-zzzzzz                             1/1     Running   0          10m   192.168.1.2   kubeadm-wn01
```

---

## 6️⃣ Creating Persistent Storage with OpenEBS

### 🔹 List Available Storage Classes
```bash
kubectl get sc
```
✅ Example output:
```plaintext
NAME               PROVISIONER        RECLAIMPOLICY   VOLUMEBINDINGMODE
openebs-device     openebs.io/local   Delete         WaitForFirstConsumer
openebs-hostpath   openebs.io/local   Delete         WaitForFirstConsumer
```
📌 We will use **openebs-hostpath**.

### 🔹 Create a Persistent Volume Claim (PVC)
📝 **pvc.yaml**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: openebs-pvc
spec:
  storageClassName: openebs-hostpath
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```

### 🔹 Apply the PVC
```bash
kubectl apply -f pvc.yaml
```

✅ Verify PVC is **Bound**:
```bash
kubectl get pvc
```
Expected output:
```plaintext
NAME          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS
openebs-pvc   Bound    pvc-abc123-xyz456-789def                  5Gi        RWO            openebs-hostpath
```

---

## 7️⃣ Using OpenEBS Storage in a Pod

📝 **pod.yaml**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  containers:
  - name: test-container
    image: busybox
    command: ["sleep", "3600"]
    volumeMounts:
    - mountPath: "/mnt/test"
      name: test-volume
  volumes:
  - name: test-volume
    persistentVolumeClaim:
      claimName: openebs-pvc
```

### 🔹 Apply the pod
```bash
kubectl apply -f pod.yaml
```

✅ Check if the pod is **running**:
```bash
kubectl get pods
```

✅ Verify storage inside the pod:
```bash
kubectl exec -it test-pod -- df -h /mnt/test
```
Expected output:
```plaintext
Filesystem      Size  Used Avail Use% Mounted on
/dev/sdb        5.0G  100M  4.9G   2% /mnt/test
```
🎉 **Success! OpenEBS storage is mounted in the pod!** 🎉

---

## 8️⃣ Troubleshooting Image Pull Issues (For containerd)

If OpenEBS fails to pull images due to network issues, follow these steps:

### 🔹 Step 1: Check Error Logs
```bash
kubectl describe pod <POD_NAME> -n openebs
```
If you see **ErrImagePull** or **ImagePullBackOff**, OpenEBS cannot download images.

### 🔹 Step 2: Manually Pull Images (For containerd)
```bash
ctr -n k8s.io image pull docker.io/openebs/node-disk-manager:2.1.0
ctr -n k8s.io image pull docker.io/openebs/provisioner-localpv:3.5.0
```

📌 If using Docker runtime:
```bash
docker pull openebs/node-disk-manager:2.1.0
docker pull openebs/provisioner-localpv:3.5.0
```

### 🔹 Step 3: Restart Kubernetes Pods
```bash
kubectl delete pod -n openebs --all
```
✅ Check if pods are running after restarting:
```bash
kubectl get pods -n openebs
```

---

## 9️⃣ Monitoring & Logs

### 🔹 Check OpenEBS Logs
```bash
kubectl logs -n openebs -l app=openebs
```

### 🔹 Check Block Devices
```bash
kubectl get blockdevices -n openebs
```

### 🔹 Check Custom Resource Definitions (CRDs)
```bash
kubectl get crds | grep openebs
```

---

## 🎯 Summary
✅ Installed Helm & OpenEBS.  
✅ Created Persistent Volumes & Claims.  
✅ Successfully mounted storage inside a pod.  
✅ Manually pulled images for containerd if needed.  
✅ Verified storage using logs and monitoring tools.  

🚀 **Next Steps**:
- Use OpenEBS with real workloads (e.g., databases like PostgreSQL, MySQL).
- Explore different OpenEBS storage engines (**cStor, Jiva, Mayastor**).
- Enable **snapshots & backups** for data protection.

🎉 **Congratulations! You have successfully set up OpenEBS for Kubernetes storage!** 🎉

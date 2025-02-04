# 📌 Understanding iSCSI: A Detailed Guide

## 🔹 What is iSCSI?
iSCSI (Internet Small Computer System Interface) is a block-level storage protocol that allows a system (initiator) to access remote storage (target) over a TCP/IP network. It enables disk drives to be exported from a storage server to remote clients as if they were locally attached.

### ✅ Key Concept:
It acts like a virtual disk over the network, providing centralized storage while allowing multiple clients to access it.

## 🔹 How Does iSCSI Work?

### 1️⃣ Key Components of iSCSI

| Component                 | Description                                         |
|---------------------------|-----------------------------------------------------|
| **iSCSI Target (Server)** | The system that hosts and shares storage over the network. |
| **iSCSI Initiator (Client)** | The system that connects to the target and mounts the storage. |
| **LUN (Logical Unit Number)** | The virtual disk shared by the iSCSI target. |
| **iSCSI Qualified Name (IQN)** | A unique identifier for iSCSI devices. |
| **ACL (Access Control List)** | Defines which initiators (clients) can access the target. |

---

## 🔹 Step-by-Step iSCSI Setup

✅ We set up iSCSI storage on a physical machine (HP02) and connected it to VMs. Below are the detailed steps.

---

### 🖥️ Step 1: Configure the iSCSI Target (Storage Server)

**👉 Performed on:** HP02 (Physical Machine)  
**👉 IP Address:** `192.168.1.9`  
**👉 Shared Disk:** `/dev/sdb1`

#### 1️⃣ Install iSCSI Target

For **Debian/Ubuntu**:

```bash
sudo apt update -y
sudo apt install tgt -y
```

For **RHEL/CentOS**:

```bash
sudo yum install targetcli -y
```

#### 2️⃣ Configure the iSCSI Target

```bash
sudo nano /etc/tgt/conf.d/iscsi-target.conf
```

💡 **Example iSCSI Configuration:**
```bash
<target iqn.2025-02.k8s:storage.disk1>
    backing-store /dev/sdb1
    incominguser iscsiuser TlrkWASUbzUC
    initiator-address 192.168.1.0/24
</target>
```

#### 3️⃣ Restart iSCSI Service

```bash
sudo systemctl restart tgt
sudo systemctl enable tgt
```

✅ **Verify the Target:**

```bash
sudo tgtadm --mode target --op show
```

✅ **Expected Output:**

```yaml
Target 1: iqn.2025-02.k8s:storage.disk1
    LUN: 1
        Type: disk
        Backing store: /dev/sdb1
    Account information:
        iscsiuser
    ACL information:
        192.168.1.0/24
```

---

### 🖥️ Step 2: Configure the iSCSI Initiator (Client)

**👉 Performed on:** `kubeadm-MN` and `kubeadm-WN01/WN02` (Worker Nodes)  
**👉 These machines will access the shared storage from HP02 (`192.168.1.9`).**

#### 1️⃣ Install the iSCSI Initiator

For **Debian/Ubuntu**:

```bash
sudo apt install open-iscsi -y
```

For **RHEL/CentOS**:

```bash
sudo yum install iscsi-initiator-utils -y
```

#### 2️⃣ Discover Available iSCSI Targets

```bash
sudo iscsiadm -m discovery -t st -p 192.168.1.9
```

✅ **Expected Output:**

```makefile
192.168.1.9:3260,1 iqn.2025-02.k8s:storage.disk1
```

#### 3️⃣ Set Authentication Credentials

```bash
echo "node.session.auth.username = iscsiuser" | sudo tee /etc/iscsi/iscsid.conf
echo "node.session.auth.password = TlrkWASUbzUC" | sudo tee -a /etc/iscsi/iscsid.conf
```

#### 4️⃣ Log into the iSCSI Target

```bash
sudo iscsiadm -m node --login
```

✅ **Expected Output:**

```yaml
Logging in to [iface: default, target: iqn.2025-02.k8s:storage.disk1, portal: 192.168.1.9,3260]
Login to [iface: default, target: iqn.2025-02.k8s:storage.disk1, portal: 192.168.1.9,3260] successful.
```

#### 5️⃣ Verify the Connection

```bash
lsblk
```

✅ **Expected Output:**

```pgsql
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda      8:0    0    50G  0 disk
└─sda1   8:1    0    487M  0 part /boot
sdb      8:16   0  931.5G  0 disk
```

🔹 **Note:** `sdb` should now be visible on the initiator.

---

## 🔹 How iSCSI Works in Kubernetes with OpenEBS

📌 **Scenario:**
- The **iSCSI Target (HP02)** provides storage.
- The **Worker Nodes (kubeadm-WN01, kubeadm-WN02)** connect to it via iSCSI.
- **OpenEBS** detects the block device and provisions Persistent Volumes (PVs) dynamically.

### 🔍 Verify iSCSI Storage in OpenEBS

```bash
kubectl get blockdevices -n openebs
```

✅ **Expected Output:**

```objectivec
NAME                           SIZE          CLAIMSTATE   STATUS
blockdevice-xyz123             931.5GiB      Unclaimed    Active
```

### 🛠️ Create a Persistent Volume Claim (PVC)

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: openebs-hostpath
  resources:
    requests:
      storage: 50Gi
```

Apply the PVC:

```bash
kubectl apply -f pvc.yaml
```

### 🛠️ Verify the Persistent Volume (PV)

```bash
kubectl get pv
```

✅ **Expected Output:**

```pgsql
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS
pvc-12345678-abcd-efgh-ijkl-mnopqrstuvwx   50Gi       RWO            Delete           Bound
```

🔹 Now, any pod that requests this PVC will get storage from your iSCSI target!

---

## 🔹 Summary

✅ **iSCSI Overview**
- A protocol that allows remote storage to be mounted like a local disk.
- Uses **targets (servers)** and **initiators (clients)**.

✅ **Steps We Did**
1️⃣ **Configured iSCSI Target (HP02):**
   - Installed `tgt`
   - Defined `/dev/sdb1` as a shared disk.
   - Allowed access via ACL and authentication.
  
2️⃣ **Configured iSCSI Initiators (Kubernetes Nodes):**
   - Installed `open-iscsi`
   - Discovered the target (`iscsiadm -m discovery`)
   - Logged in to storage (`iscsiadm -m node --login`)
  
3️⃣ **Integrated iSCSI with OpenEBS:**
   - OpenEBS detected the storage.
   - Persistent Volumes (PV) were created dynamically.

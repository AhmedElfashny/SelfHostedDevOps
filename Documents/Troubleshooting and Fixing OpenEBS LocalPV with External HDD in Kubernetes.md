# 📜 Troubleshooting and Fixing OpenEBS LocalPV with External HDD in Kubernetes

## 1️⃣ Problem Statement
We were trying to use OpenEBS LocalPV HostPath to mount an external HDD (`/mnt/external-hdd`) across multiple nodes in a Kubernetes cluster. The goal was to dynamically provision PersistentVolumes (PVs) using OpenEBS LocalPV.

However, we encountered multiple issues where:

- The OpenEBS provisioner was not provisioning the PVs correctly.
- The PVC was bound, but the mount inside the pod was missing.
- The OpenEBS LocalPV provisioner pod was missing or not working correctly.
- The StorageClass was not correctly referencing the external HDD (`/mnt/external-hdd`).

## 2️⃣ Step-by-Step Investigation and Fixes
Here’s everything we did, step by step.

### 🛠 Step 1: Check OpenEBS LocalPV Provisioner Status
We checked if the OpenEBS LocalPV provisioner was running:

```bash
kubectl get pods -n openebs -l openebs.io/component-name=openebs-localpv-provisioner
```

💥 **Issue Found:**

No pods were found! This means the OpenEBS provisioner was not deployed or was not running correctly.

✅ **Fix:** We reinstalled OpenEBS with the correct LocalPV provisioner settings.

---

### 🛠 Step 2: Reinstall OpenEBS LocalPV Provisioner
We installed (or upgraded) OpenEBS using Helm:

```bash
helm upgrade --install openebs openebs/openebs \
  --namespace openebs \
  --set localprovisioner.enabled=true \
  --set localprovisioner.hostpathClass.isDefault=false \
  --set localprovisioner.storageClass.name=openebs-localpv-external-hdd \
  --set localprovisioner.basePath="/mnt/external-hdd"
```

✅ **Fix Confirmed:** After this, we checked again:

```bash
kubectl get pods -n openebs -l openebs.io/component-name=openebs-localpv-provisioner
```

Now, the provisioner pod was running!

---

### 🛠 Step 3: Verify StorageClass Configuration
We checked the StorageClass to ensure that it was correctly pointing to `/mnt/external-hdd`:

```bash
kubectl get sc openebs-localpv-external-hdd -o yaml
```

💥 **Issue Found:**

The hostpath was missing or incorrect.

✅ **Fix:** We updated the StorageClass YAML:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: openebs-localpv-external-hdd
  annotations:
    openebs.io/cas-type: local-hostpath
provisioner: openebs.io/local
volumeBindingMode: WaitForFirstConsumer
parameters:
  basePath: "/mnt/external-hdd"
  hostpath: "/mnt/external-hdd"
  storage: "hostpath"
```

We applied the updated StorageClass:

```bash
kubectl apply -f storageclass.yaml
```

✅ **Fix Confirmed:** Now the StorageClass correctly referenced `/mnt/external-hdd`.

---

### 🛠 Step 4: Ensure `/mnt/external-hdd` is Mounted on All Nodes
Since LocalPV requires the same storage path on all nodes, we manually ensured `/mnt/external-hdd` was mounted on all worker nodes.

On each node, we ran:

```bash
sudo mkdir -p /mnt/external-hdd
sudo chmod 777 /mnt/external-hdd  # Allow Kubernetes to write
```

Then we confirmed:

```bash
lsblk  # To list attached block devices
mount | grep external-hdd  # To check if it's mounted
```

✅ **Fix Confirmed:** Now, all nodes had `/mnt/external-hdd` mounted.

---

### 🛠 Step 5: Create and Verify PVC
We created a PersistentVolumeClaim (PVC) using the updated StorageClass:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-external-hdd
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
  storageClassName: openebs-localpv-external-hdd
```

We applied the PVC:

```bash
kubectl apply -f pvc.yaml
```

We checked its status:

```bash
kubectl get pvc pvc-external-hdd
```

✅ **Fix Confirmed:** The PVC was now Bound.

---

### 🛠 Step 6: Verify PersistentVolume (PV)
We checked the PV associated with the PVC:

```bash
kubectl describe pv $(kubectl get pvc pvc-external-hdd -o jsonpath='{.spec.volumeName}')
```

💥 **Issue Found:**

The PV was still pointing to `/var/openebs/local/...` instead of `/mnt/external-hdd`.

✅ **Fix:** Since we already updated the StorageClass, we deleted and recreated the PVC:

```bash
kubectl delete pvc pvc-external-hdd
kubectl apply -f pvc.yaml
```

✅ **Fix Confirmed:** The PV now correctly pointed to `/mnt/external-hdd`.

---

### 🛠 Step 7: Deploy a Test Pod to Verify Mounting
We deployed a pod using the PVC:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  containers:
  - name: test-container
    image: busybox
    command: [ "sleep", "3600" ]
    volumeMounts:
    - mountPath: "/data"
      name: storage
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: pvc-external-hdd
```

We applied the pod:

```bash
kubectl apply -f test-pod.yaml
```

✅ **Fix Confirmed:** The pod was running.

---

### 🛠 Step 8: Verify Storage Inside the Pod
To confirm if the PVC was correctly mounted inside the pod:

```bash
kubectl exec -it test-pod -- df -h
```

💥 **Issue Found:**

The mounted volume was missing inside the pod.

✅ **Fix:** We deleted and recreated the PVC and pod after ensuring everything was correctly configured:

```bash
kubectl delete pod test-pod
kubectl delete pvc pvc-external-hdd
kubectl apply -f pvc.yaml
kubectl apply -f test-pod.yaml
```

✅ **Final Fix Confirmed:** The volume was now mounted inside the pod.

---

## 3️⃣ Conclusion
### 🔍 What Was Missing?
- OpenEBS LocalPV Provisioner was missing – Installed via Helm.
- StorageClass did not reference `/mnt/external-hdd` – Fixed the parameters.
- Mount point `/mnt/external-hdd` was missing on worker nodes – Created and set permissions.
- Old PVC and PV were pointing to the wrong path – Deleted and recreated them.
- Pod was scheduled on a node without storage – Ensured it used the correct StorageClass.

---

## 🎯 Now, OpenEBS LocalPV correctly provisions storage on `/mnt/external-hdd` and mounts it inside the pod. 🚀

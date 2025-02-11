# Troubleshooting Kubernetes Node Join Failure: Causes and Solutions

## Issue Overview

When attempting to join a worker node (`kubeadm-wm03`) to the Kubernetes cluster, the following error occurred:

```
[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
error execution phase kubelet-start: cannot get Node "kubeadm-wm03": nodes "kubeadm-wm03" is forbidden: User "system:bootstrap:yk9n13" cannot get resource "nodes" in API group "" at the cluster scope
```

This error indicated an RBAC (Role-Based Access Control) issue, where the user attempting to join the cluster did not have the necessary permissions to complete the node registration process.

---

## Root Causes and Solutions

### 1. RBAC (Role-Based Access Control) Issue

#### Understanding RBAC in Kubernetes
RBAC in Kubernetes is a security mechanism that regulates access to resources within the cluster. It consists of:

- **ClusterRoles**: Define permissions to access Kubernetes resources at the cluster level.
- **ClusterRoleBindings**: Bind a ClusterRole to a user or a group, granting them permissions.
- **Users and Groups**: Entities that require access to the cluster.

In our case, the error message indicated that the bootstrap token user `system:bootstrap:yk9n13` did not have permission to get nodes during the join process.

#### Solution: Granting the Correct Permissions

To resolve this, we created a `ClusterRoleBinding` that grants necessary permissions to the bootstrap token user.

#### Updated `node-viewer.yaml`:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-viewer
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: bootstrap-node-viewer
subjects:
- kind: User
  name: system:bootstrap:yk9n13  # Matching the token in the join command
  apiGroup: rbac.authorization.k8s.io
- kind: Group
  name: system:bootstrappers  # Granting access to all bootstrap users
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: node-viewer
  apiGroup: rbac.authorization.k8s.io
```

After applying the updated RBAC configuration:

```sh
kubectl apply -f node-viewer.yaml
```

The permissions were correctly assigned, resolving the RBAC-related join failure.

---

### 2. Bootstrap Token Issue

#### What is a Bootstrap Token?
A bootstrap token is a temporary authentication token used by worker nodes to join a Kubernetes cluster. These tokens are time-limited and must be valid during the join process.

#### Possible Issues
- **Token Expired**: The token used in the `kubeadm join` command may have expired.
- **Incorrect Token Used**: The token in the `ClusterRoleBinding` did not match the one used for the join process.

#### Solution: Checking and Regenerating a Token

To verify if the token was valid, we checked the available tokens on the master node:

```sh
kubeadm token list
```

##### Output:
```
TOKEN                     TTL         EXPIRES                USAGES                   DESCRIPTION                                                EXTRA GROUPS
yk9n13.zgo4cnxh3ur5hlo4   23h         2025-02-12T16:31:48Z   authentication,signing   <none>                                                     system:bootstrappers:kubeadm:default-node-token
```

Since the token was still valid, we proceeded with the join command. If it had expired, we would have generated a new one:

```sh
kubeadm token create --print-join-command
```

This command generates a new token and provides the correct `kubeadm join` command with the updated token and CA hash.

---

### 3. Preflight Check Failure: `ca.crt` Already Exists

#### Error Message:
```
[ERROR FileAvailable--etc-kubernetes-pki-ca.crt]: /etc/kubernetes/pki/ca.crt already exists
```

This occurs when the node was previously joined to a cluster and has leftover configuration files.

#### Solution: Reset the Node Before Rejoining

To fix this, we reset `kubeadm` on the worker node and removed old configurations:

```sh
kubeadm reset -f
rm -rf /etc/kubernetes/pki
```

Then, we retried the join command:

```sh
kubeadm join 192.168.1.8:6443 --token yk9n13.zgo4cnxh3ur5hlo4 --discovery-token-ca-cert-hash sha256:89fe4bc6370ff1ab565b2bb5f9f71446b59fe4fbc10fbf5d0b6d889ba932e3b8
```

---

### 4. Node Stuck in `NotReady` State

After successfully joining the cluster, the node remained in a `NotReady` state:

```sh
kubectl get nodes -o wide
```

##### Output:
```
kubeadm-wm03   NotReady   <none>   ...
```

This indicated an issue with the Container Network Interface (CNI).

#### Solution: Reinstalling the CNI Plugin

To fix this, we reinstalled the Flannel CNI:

```sh
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

Alternatively, for Calico:

```sh
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

After restarting `kubelet`, the node changed to `Ready`:

```sh
systemctl restart kubelet
kubectl get nodes -o wide
```

##### Final Output:
```
kubeadm-wm03   Ready   <none>   ...
```

---

## Conclusion

### Summary of Fixes:

| Issue | Cause | Solution |
|--------|--------|----------|
| RBAC Permission Error | Bootstrap token user lacked `get nodes` permission | Applied `ClusterRoleBinding` to `system:bootstrappers` |
| Bootstrap Token Issue | Token expired or mismatched | Verified and regenerated token using `kubeadm token create` |
| Preflight Check Failure (`ca.crt`) | Node was previously joined | Reset `kubeadm` and removed `/etc/kubernetes/pki` |
| Node `NotReady` State | CNI Plugin not initialized | Reinstalled Flannel/Calico and restarted `kubelet` |

These steps ensured a smooth resolution, successfully allowing the node to join and become `Ready` within the cluster. 🚀

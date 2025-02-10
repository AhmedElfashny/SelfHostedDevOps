# Kubernetes Cluster Network Issue: Kube-Proxy RBAC Misconfiguration

## Issue Summary

In our Kubernetes cluster, worker nodes could not reach the API server (`10.96.0.1:443`), and the Flannel CNI plugin was stuck in a `CrashLoopBackOff` state. The `kube-proxy` service was unable to function correctly, leading to CoreDNS failure and routing issues across the cluster.

## Root Cause

The error message from the `kube-proxy` logs indicated that the service account `system:serviceaccount:kube-system:kube-proxy` did not have permission to list services:

```plaintext
E0210 11:19:06.861103       1 reflector.go:147] k8s.io/client-go@v0.0.0/tools/cache/reflector.go:229: Failed to watch *v1.Service: failed to list *v1.Service: services is forbidden: User "system:serviceaccount:kube-system:kube-proxy" cannot list resource "services" in API group "" at the cluster scope
```

This meant that `kube-proxy` was unable to configure the necessary iptables rules to allow network communication between the cluster nodes and services. As a result:

- Flannel could not reach the API server.
- CoreDNS could not resolve service names.
- Worker nodes failed to communicate with the master node.

## Solution

To resolve the issue, we needed to restore the missing `ClusterRoleBinding` for `kube-proxy`.

### 1️⃣ Check if the ClusterRoleBinding Exists

We first checked if the `kube-proxy` ClusterRoleBinding was present:

```sh
kubectl get clusterrolebinding kube-proxy
```

If the binding was missing or misconfigured, we needed to recreate it.

### 2️⃣ Recreate the ClusterRoleBinding

To restore the correct permissions for `kube-proxy`, we applied the following command:

```sh
kubectl create clusterrolebinding kube-proxy \
  --clusterrole=system:node-proxier \
  --serviceaccount=kube-system:kube-proxy
```

### 3️⃣ Restart Kube-Proxy

After fixing the RBAC permissions, we restarted the `kube-proxy` pods:

```sh
kubectl delete pod -n kube-system -l k8s-app=kube-proxy
kubectl get pods -n kube-system | grep kube-proxy
```

This forced `kube-proxy` to restart and apply the new RBAC rules.

### 4️⃣ Verify Fix

We confirmed the API server was reachable:

```sh
curl -k https://10.96.0.1:443/version
```

If the connection succeeded, it meant `kube-proxy` was correctly handling service IP routing.

We also checked the `kube-proxy` logs for errors:

```sh
kubectl logs -n kube-system -l k8s-app=kube-proxy --tail=50
```

No further RBAC-related errors were present, confirming that the issue was resolved.

## Understanding ClusterRoleBinding in Kubernetes

A `ClusterRoleBinding` is an RBAC (Role-Based Access Control) resource that grants cluster-wide permissions to a user, group, or service account. In our case, `kube-proxy` required a `ClusterRoleBinding` to allow it to manage service IPs and configure `iptables` rules.

### Key Components of ClusterRoleBinding

- **ClusterRole**: Defines the set of permissions (e.g., `system:node-proxier`).
- **ServiceAccount**: The entity to which permissions are granted (`kube-system:kube-proxy`).
- **Binding**: Connects the `ClusterRole` to the `ServiceAccount`.

### Example ClusterRoleBinding for Kube-Proxy

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: kube-proxy
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:node-proxier
subjects:
- kind: ServiceAccount
  name: kube-proxy
  namespace: kube-system
```

This configuration ensures that `kube-proxy` can access and modify Kubernetes services, allowing it to set up `iptables` rules for service routing.

## Conclusion

This issue was caused by a missing `ClusterRoleBinding` for `kube-proxy`, preventing it from listing services and configuring networking. By restoring the RBAC permissions and restarting `kube-proxy`, we resolved the issue and restored network connectivity across the cluster.

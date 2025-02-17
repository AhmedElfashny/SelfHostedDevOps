## 📖 Kubernetes Dashboard Access – Full Documentation 🚀

### 🛠️ Overview
we deployed and accessed the Kubernetes Dashboard using an Ingress Controller and NodePort Service. Along the way, we faced and resolved several issues related to service ports, ingress configurations, and DNS settings.

This documentation covers:

- Kubernetes Service vs Ingress
- Deployment Steps
- Issues Faced & Solutions
- DNS Configuration
- Final Testing & Validation

### 📍 1. Kubernetes Service vs Ingress

#### 🧱 Kubernetes Service
A Kubernetes Service exposes applications running as pods to external or internal traffic.

- **Types:** ClusterIP, NodePort, LoadBalancer.
- **In our case:** We used a NodePort to expose the Kubernetes Dashboard on port 32000.

🔍 **Example:**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  type: NodePort
  ports:
    - port: 443
      targetPort: 8443
      nodePort: 32000
  selector:
    k8s-app: kubernetes-dashboard
```

#### ⚙️ Kubernetes Ingress
An Ingress manages access to services using Layer 7 (HTTP/HTTPS) traffic based on hostnames or URLs.

- We deployed an NGINX Ingress Controller to route traffic via `k8s-dashboard.selfhosteddevops`.
- Ingress simplifies access by routing external traffic to the correct internal service.

🔍 **Example:**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: kubernetes-dashboard-ingress
  namespace: kubernetes-dashboard
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: "/"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
spec:
  ingressClassName: nginx
  rules:
  - host: k8s-dashboard.selfhosteddevops
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: kubernetes-dashboard
            port:
              number: 443
```

### 📍 2. Deployment Steps

#### 🔹 Step 1: Install Ingress Controller
We installed the NGINX Ingress Controller using Helm:

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install nginx-ingress ingress-nginx/ingress-nginx   --namespace ingress-nginx   --create-namespace   --set controller.hostNetwork=true   --set controller.service.type=NodePort   --set controller.service.nodePorts.http=30080   --set controller.service.nodePorts.https=30443
```

#### 🔹 Step 2: Deploy the Kubernetes Dashboard
The Kubernetes Dashboard was already deployed, so we just needed to configure the Ingress.

#### 🔹 Step 3: Apply the Ingress Configuration
We applied the Ingress Resource:

```bash
kubectl apply -f kubernetes-dashboard-ingress.yaml
```

🔍 **YAML Configuration (Final Version):**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: kubernetes-dashboard-ingress
  namespace: kubernetes-dashboard
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: "/"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
spec:
  ingressClassName: nginx
  rules:
  - host: k8s-dashboard.selfhosteddevops
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: kubernetes-dashboard
            port:
              number: 443
```

#### 🔹 Step 4: Add DNS Entry
We added a local DNS entry in Windows:

```bash
echo "192.168.1.2 k8s-dashboard.selfhosteddevops" | sudo tee -a /etc/hosts
echo "192.168.1.3 k8s-dashboard.selfhosteddevops" | sudo tee -a /etc/hosts
```

📍 **Windows hosts file path:**
```
C:\Windows\System32\drivers\etc\hosts
```

---

### 📍 3. Issues Faced & Solutions

#### ❌ Issue 1: 503 Service Temporarily Unavailable
- **Root Cause:** The Ingress tried to forward traffic to port 80, while the dashboard listens on port 8443.
- **Fix:** Updated the Ingress YAML to use port 443 (NodePort for 8443) and added the `backend-protocol: HTTPS` annotation.

#### ❌ Issue 2: HTTP 400 Bad Request
- **Root Cause:** The dashboard expected HTTPS traffic, but we accessed it over HTTP.
- **Fix:** Added `nginx.ingress.kubernetes.io/ssl-redirect: "true"` to force HTTPS.

#### ❌ Issue 3: No Route to Host (NodePort Unreachable)
- **Root Cause:** Firewall rules blocked NodePort traffic.
- **Fix:** Allowed ports 30080 and 30443 in the firewall:

```bash
firewall-cmd --zone=public --add-port=30080/tcp --permanent
firewall-cmd --zone=public --add-port=30443/tcp --permanent
firewall-cmd --reload
```

#### ❌ Issue 4: DNS Resolution Issues
- **Root Cause:** Missing host entries on the client machine.
- **Fix:** Manually added `k8s-dashboard.selfhosteddevops` to the hosts file.

---

### 📍 4. DNS Configuration Explained

We used DNS resolution with a custom domain name:

```
k8s-dashboard.selfhosteddevops
```

This works as follows:

1. The browser sends a request to `k8s-dashboard.selfhosteddevops`.
2. The OS checks the hosts file and maps it to `192.168.1.2` or `192.168.1.3`.
3. Traffic hits the NodePort (30080/30443) on these nodes.
4. The Ingress Controller routes traffic to the dashboard service.
5. The dashboard service forwards the request to the dashboard pod.

**🌐 Why Use DNS & Ingress?**
- Without Ingress, we'd have to remember IP addresses and NodePorts.
- With Ingress, we use easy-to-remember domain names.

---

### 📍 5. Final Testing & Validation

#### 🛎️ Check Ingress Controller:

```bash
kubectl get ingress -n kubernetes-dashboard
```

**Expected Output:**

```bash
NAME                           CLASS    HOSTS                           ADDRESS         PORTS
kubernetes-dashboard-ingress   nginx    k8s-dashboard.selfhosteddevops   192.168.1.2     80, 443
```

#### 🌐 Access Dashboard:

- Open browser:

```
https://k8s-dashboard.selfhosteddevops
```

- Command-line test:

```bash
curl -k https://k8s-dashboard.selfhosteddevops
```

---

### 🎯 Key Takeaways

- Services provide connectivity to pods; Ingress provides domain-based access.
- The Kubernetes Dashboard requires HTTPS and port 8443.
- NodePort services and Ingress Controllers are essential for external access.
- DNS entries make access user-friendly.
- Firewall adjustments are critical for NodePort traffic.

🚀 **Happy Kubernetes Dashboarding!** 🛠️

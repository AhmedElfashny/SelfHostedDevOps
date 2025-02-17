
# 📖 Comprehensive Guide to DNS, DHCP, and Dnsmasq Installation & Configuration

## 🌐 1. What is DNS (Domain Name System)?

The Domain Name System (DNS) is like the phonebook of the internet, translating human-readable domain names (e.g., example.com) into IP addresses (e.g., 192.168.1.1).

### 🛠️ How DNS Works

- **User Query**: You enter a domain name (e.g., google.com).
- **DNS Resolver**: The request is sent to a DNS resolver.
- **Recursive Lookup**: The resolver queries root, TLD, and authoritative DNS servers.
- **Response**: The DNS server returns the IP address.
- **Caching**: The result is cached to speed up future lookups.

## ⚔️ 2. DNS Server Software Comparison

| DNS Server | Primary Role       | Key Features                    | Complexity | Best For            |
|------------|--------------------|----------------------------------|------------|---------------------|
| BIND9      | Authoritative & Caching | Full-featured, DNSSEC, IPv6 support | High       | Large Enterprises  |
| Dnsmasq    | Caching & Forwarding   | Lightweight, DHCP support         | Low        | Small/Home Networks |
| PowerDNS   | Authoritative DNS       | Database-driven backend            | Medium     | ISPs, Dynamic DNS   |
| Unbound    | Recursive Resolver      | DNS-over-TLS/HTTPS, DNSSEC         | Low        | Privacy-Focused Setups |

### Recommendations:

- **Small Networks or Kubernetes**: Dnsmasq
- **Enterprise-Grade**: BIND9
- **Dynamic DNS Management**: PowerDNS
- **Privacy-Focused**: Unbound

## 🛠️ 3. What is DHCP (Dynamic Host Configuration Protocol)?

DHCP automatically assigns network settings (IP address, subnet mask, gateway, DNS servers) to devices.

### 🔄 DHCP Process:

1. **DHCPDISCOVER**: Device broadcasts a request.
2. **DHCPOFFER**: Server responds with an available IP.
3. **DHCPREQUEST**: Device requests the offered IP.
4. **DHCPACK**: Server acknowledges and assigns the IP.

*Without DHCP: You must manually configure every device.*

## ⚙️ 4. Installing Dnsmasq on Debian (for Kubernetes Cluster)

### 🛠️ Step 1: Update & Install Dnsmasq

```bash
sudo apt update
sudo apt install dnsmasq -y
```

### 🔍 Step 2: Verify Installation

```bash
dnsmasq --version
```

## ✏️ 5. Configuring Dnsmasq for Kubernetes DNS

### Step 1: Edit Configuration File

```bash
sudo nano /etc/dnsmasq.conf
```

Add the following lines:

```bash
# Listen on local and network interface
listen-address=127.0.0.1,192.168.1.8

# Kubernetes domain
local=/cluster.local/

# Custom domain for Kubernetes Dashboard
address=/k8s-dashboard.selfhosteddevops/192.168.1.2

# Forward all other queries to external DNS servers
server=8.8.8.8
server=1.1.1.1

# Enable DNS caching
cache-size=10000

# Bind to a specific interface
bind-interfaces
```

### Step 2: Restart Dnsmasq

```bash
sudo systemctl restart dnsmasq
sudo systemctl status dnsmasq
```

### Step 3: Verify Configuration

```bash
dig @127.0.0.1 k8s-dashboard.selfhosteddevops
```

Expected:

```
k8s-dashboard.selfhosteddevops. 0 IN    A       192.168.1.2
```

## 🖥️ 6. Configuring Windows 10 to Use Dnsmasq DNS

### Method 1: Control Panel

1. Open Control Panel → Network and Sharing Center.
2. Select your network adapter → Properties.
3. Open IPv4 Settings → Properties.
4. Set DNS servers:
   - Preferred DNS Server: `8.8.8.8`
   - Alternate DNS Server: `192.168.1.8`
5. Click OK → Close.

### Method 2: Command Line

```bash
netsh interface ip set dns name="Wi-Fi" static 192.168.1.8
```

### Test DNS Resolution

```bash
nslookup k8s-dashboard.selfhosteddevops
```

## 🔍 7. Troubleshooting DNS Issues

### 1️⃣ Check if dnsmasq is Listening

```bash
sudo netstat -plnt | grep :53
```

### 2️⃣ Test DNS Resolution from Debian

```bash
dig @127.0.0.1 k8s-dashboard.selfhosteddevops
```

### 3️⃣ Test DNS from Windows

```bash
nslookup k8s-dashboard.selfhosteddevops 192.168.1.8
```

### 4️⃣ Flush DNS Cache on Windows

```bash
ipconfig /flushdns
```

## 🧠 8. Understanding the dig Command

`dig` is a DNS lookup utility used to test and troubleshoot DNS configurations.

### Common Usage:

```bash
dig [@server] [domain] [query_type] [options]
```

### Examples:

- Query Kubernetes Dashboard:

```bash
dig @127.0.0.1 k8s-dashboard.selfhosteddevops
```

- Query Google with Full Trace:

```bash
dig google.com +trace
```

- Short Answer:

```bash
dig google.com +short
```

## 🎯 Conclusion

By following this guide, you’ve installed, configured, and tested Dnsmasq as a DNS server for your Kubernetes cluster and integrated it with your Windows 10 machine.

- 🔹 **Dnsmasq** helps resolve internal Kubernetes services without relying on external DNS servers.
- 🔹 **DNS & DHCP** are essential for automated, reliable network configurations.

**Need help adding more records or advanced DNS configurations? Let me know! 😊**

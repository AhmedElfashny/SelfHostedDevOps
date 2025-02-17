# 🚀 Complete Step-by-Step Guide to Installing Jenkins on Debian with Firewalld

This guide covers everything you need to install Jenkins step by step on Debian, including Java setup, firewall configuration, and troubleshooting.

---

## 🔹 Step 1: Update System Packages

Before installing anything, update your system:

```bash
sudo apt update && sudo apt upgrade -y
```

---

## 🔹 Step 2: Install Java 17 (Required for Jenkins)

Jenkins requires Java 17 or higher. Install OpenJDK 17:

```bash
sudo apt install openjdk-17-jdk -y
```

Verify the installation:

```bash
java -version
```

Expected output:

```bash
openjdk version "17.0.X"
```

### Set Java 17 as Default (If Needed)

If multiple Java versions exist, set Java 17 as default:

```bash
sudo update-alternatives --config java
```

Select Java 17.

Set JAVA_HOME permanently:

```bash
echo 'export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64' | sudo tee -a /etc/profile
echo 'export PATH=$JAVA_HOME/bin:$PATH' | sudo tee -a /etc/profile
source /etc/profile
```

Verify:

```bash
echo $JAVA_HOME
```

Expected output:

```bash
/usr/lib/jvm/java-17-openjdk-amd64
```

---

## 🔹 Step 3: Add Jenkins Repository

Jenkins is not available in Debian’s default repositories, so we add it manually.

### 1️⃣ Add Jenkins GPG key:

```bash
wget -O- https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc
```

### 2️⃣ Add the Jenkins repository:

```bash
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
```

### 3️⃣ Update package lists:

```bash
sudo apt update
```

---

## 🔹 Step 4: Install Jenkins

Now install Jenkins:

```bash
sudo apt install jenkins -y
```

---

## 🔹 Step 5: Configure Jenkins to Use Port 8081

By default, Jenkins runs on port 8080, but we will change it to 8081.

### 1️⃣ Edit the Jenkins configuration file:

```bash
sudo nano /etc/default/jenkins
```

Find this line:

```ini
HTTP_PORT=8080
```

Change it to:

```ini
HTTP_PORT=8081
```

Save and exit (`CTRL+X`, `Y`, `Enter`).

### 2️⃣ Edit the systemd service file:

```bash
sudo nano /lib/systemd/system/jenkins.service
```

Find any line with:

```ini
ExecStart=/usr/bin/jenkins --httpPort=8080
```

Change 8080 to 8081:

```ini
ExecStart=/usr/bin/jenkins --httpPort=8081
```

Save and exit (`CTRL+X`, `Y`, `Enter`).

### 3️⃣ Reload systemd to apply changes:

```bash
sudo systemctl daemon-reload
```

---

## 🔹 Step 6: Open Port 8081 in Firewalld

If `firewalld` is running, allow port 8081:

### 1️⃣ Start and enable firewalld (if not already running):

```bash
sudo systemctl start firewalld
sudo systemctl enable firewalld
```

### 2️⃣ Allow Jenkins through the firewall:

```bash
sudo firewall-cmd --add-port=8081/tcp --permanent
sudo firewall-cmd --reload
```

### 3️⃣ Verify firewall rules:

```bash
sudo firewall-cmd --list-ports
```

You should see:

```bash
8081/tcp
```

---

## 🔹 Step 7: Start Jenkins

Now start the Jenkins service:

```bash
sudo systemctl start jenkins
```

Enable it to start on boot:

```bash
sudo systemctl enable jenkins
```

Check the status:

```bash
sudo systemctl status jenkins
```

Expected output:

```bash
Active: active (running)
```

---

## 🔹 Step 8: Verify Jenkins is Running

Check if Jenkins is listening on port 8081:

```bash
sudo netstat -tulnp | grep 8081
```

or

```bash
sudo lsof -i :8081
```

If Jenkins is running, you should see something like:

```bash
tcp  LISTEN  <PID>/java  0.0.0.0:8081
```

---

## 🔹 Step 9: Access Jenkins Web UI

Now open your browser and go to:

```bash
http://<your-server-ip>:8081
```

Example:

```bash
http://192.168.1.100:8081
```

---

## 🔹 Step 10: Retrieve the Initial Admin Password

Jenkins requires an initial password for setup. Get it with:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Copy the password and paste it into the Jenkins setup page.

---

## 🔹 Step 11: Complete Jenkins Setup

1. Choose **"Install Suggested Plugins"** (Recommended).
2. Create an **Admin User**.
3. Configure **Jenkins URL** (leave default or set your domain/IP).
4. Start using Jenkins!

---

## 🔹 Step 12: Troubleshooting (If Needed)

### If Jenkins fails to start, check logs:

```bash
sudo journalctl -u jenkins --no-pager | tail -50
```

or

```bash
sudo cat /var/log/jenkins/jenkins.log
```

### If another process is using port 8080, find and stop it:

```bash
sudo netstat -tulnp | grep 8080
sudo kill -9 <PID>
```

(Replace `<PID>` with the actual process ID.)

### If Jenkins is not accessible, check firewall rules:

```bash
sudo firewall-cmd --list-ports
```

Make sure **8081/tcp** is allowed.

---

## 🎯 Summary of Commands Used

Here is a quick summary of all commands used:

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Java 17
sudo apt install openjdk-17-jdk -y
java -version

# Add Jenkins repository
wget -O- https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update

# Install Jenkins
sudo apt install jenkins -y

# Change Jenkins port to 8081
sudo nano /etc/default/jenkins
sudo nano /lib/systemd/system/jenkins.service
sudo systemctl daemon-reload

# Allow Jenkins in firewall
sudo systemctl start firewalld
sudo systemctl enable firewalld
sudo firewall-cmd --add-port=8081/tcp --permanent
sudo firewall-cmd --reload

# Start and enable Jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins
sudo systemctl status jenkins

# Get Jenkins initial admin password
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

---

## 🎉 Congratulations! Jenkins is Now Installed!

Jenkins is now running on **port 8081** and ready for **CI/CD automation**.

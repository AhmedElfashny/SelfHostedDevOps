# Hipster Shop Project Setup with GitLab and SSH

## 1. GitLab User and Project Creation

**Created GitLab User:**  
- **Username:** ahmedehab  
- **Hostname:** DESKTOP-G0HNRFS  

**Created GitLab Project:**  
- **Project Name:** hipster-shop  
- **Repository URL (SSH):** `git@192.168.1.8:ahmedehab/hipster-shop.git`

---

## 2. SSH Key Generation and Configuration

### Step 1: Check Existing SSH Key
```bash
ls ~/.ssh/id_rsa.pub
```
**Result:** Existing SSH key found at `/c/Users/hp/.ssh/id_rsa.pub`

### Step 2: Added SSH Key to GitLab
Copied the SSH key to clipboard:
```bash
cat ~/.ssh/id_rsa.pub | clip
```
Added the key to GitLab:
- **GitLab:** Preferences > SSH Keys > Pasted Key  
- **Title:** DESKTOP-G0HNRFS

### Step 3: Verified SSH Connection
```bash
ssh -T git@192.168.1.8
```
**Result:** Welcome to GitLab, ahmedehab!

---

## 3. Git Repository Configuration

### Step 1: Cloned Repository from GitLab
```bash
git clone git@192.168.1.8:ahmedehab/hipster-shop.git
cd hipster-shop
```

### Step 2: Configured Git User Identity
```bash
git config --global user.name "ahmedehab"
git config --global user.email "ahmedehab@example.com"
```

### Step 3: Copied Hipster Shop Source Code
```bash
cp -r microservices-demo/* ./
```

### Step 4: Staged and Committed Changes
```bash
git add .
git commit -m "Added Hipster Shop microservices project"
```

### Step 5: Pushed Project to GitLab
```bash
git push origin main
```

---

## ✅ Status Check
- **User:** `ahmedehab` created on GitLab.  
- **SSH:** Key successfully added and verified.  
- **Repository:** `hipster-shop` created and code pushed.  

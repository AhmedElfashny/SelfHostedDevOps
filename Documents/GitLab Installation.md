# GitLab Installation, Configuration, and Troubleshooting Documentation

## Introduction

This document provides a detailed guide on how to install GitLab locally, configure it properly, troubleshoot common issues, and document important changes. The goal is to maintain a well-documented record of the installation process, configuration settings, and potential modifications to ensure a smooth experience when managing GitLab.

---

## 1. Installation of GitLab

### 1.1 What is GitLab?

GitLab is a DevOps platform that provides source code management (SCM), continuous integration (CI/CD), issue tracking, and more. It can be installed locally to manage code repositories and collaborate with teams.

### 1.2 Installation Type Used

For this setup, we installed **GitLab Community Edition (CE)**, which is an open-source, self-hosted version of GitLab.

### 1.3 Steps to Install GitLab on a Local Server

#### 1.3.1 Install Required Dependencies

Ensure that your system has the required dependencies installed:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl openssh-server ca-certificates tzdata perl
```

#### 1.3.2 Add GitLab Repository and Install GitLab

```bash
curl -fsSL https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
sudo apt install -y gitlab-ce
```

#### 1.3.3 Configure GitLab URL

Modify the GitLab external URL in its configuration file:

```bash
sudo nano /etc/gitlab/gitlab.rb
```

Find the following line and modify it:

```plaintext
external_url 'http://192.168.1.8'
```

Save and exit (`CTRL+X`, `Y`, `Enter`).

#### 1.3.4 Apply Configuration and Start GitLab

```bash
sudo gitlab-ctl reconfigure
sudo gitlab-ctl restart
```

This step generates necessary configurations and starts GitLab services.

---

## 2. Understanding GitLab Services and Configuration

### 2.1 What is `gitlab-ctl`?

`gitlab-ctl` is a command-line tool used to manage GitLab services. Some of the most important commands are:

- **Check GitLab status:**
  
  ```bash
  sudo gitlab-ctl status
  ```

- **Restart GitLab:**
  
  ```bash
  sudo gitlab-ctl restart
  ```

- **View logs for troubleshooting:**
  
  ```bash
  sudo gitlab-ctl tail
  ```

- **Stop and start GitLab services:**
  
  ```bash
  sudo gitlab-ctl stop
  sudo gitlab-ctl start
  ```

### 2.2 GitLab Components

- **Puma** → Application server that serves GitLab’s web interface.
- **Nginx** → Reverse proxy that routes requests to Puma.
- **PostgreSQL** → Database backend for GitLab.
- **Redis** → Caching system for improved performance.
- **Sidekiq** → Background job processing.
- **Gitaly** → Service that manages Git repositories.

### 2.3 Modifying Ports and Puma Configuration

By default, GitLab runs on port **80**. If a different port is required:

Modify GitLab external URL:

```bash
sudo nano /etc/gitlab/gitlab.rb
```

Example:

```plaintext
external_url 'http://192.168.1.8:8080'
```

Modify Puma’s Port (if needed):

```bash
sudo nano /var/opt/gitlab/gitlab-rails/etc/puma.rb
```

Set:

```plaintext
puma['port'] = 8080
```

Reapply the configuration:

```bash
sudo gitlab-ctl reconfigure
sudo gitlab-ctl restart
```

---

## 3. Troubleshooting Issues Faced

### 3.1 Issue: 502 Bad Gateway (GitLab Not Starting)

**Cause:**
- Puma service is not running or is misconfigured.
- Nginx cannot connect to Puma.
- PostgreSQL is not running.

**Solution:**
- Check Puma’s status:

  ```bash
  sudo gitlab-ctl status puma
  ```

- Manually restart Puma:

  ```bash
  sudo gitlab-ctl restart puma
  ```

- Check Nginx logs:

  ```bash
  sudo gitlab-ctl tail nginx
  ```

- Ensure Puma is listening on the correct port:

  ```bash
  sudo ss -tulnp | grep puma
  ```

---

### 3.2 Issue: Root Password Expired

**Solution:**
Open the GitLab Rails console:

```bash
sudo gitlab-rails console -e production
```

Reset the root password:

```ruby
user = User.find_by_username('root')
user.password = 'NewSecurePassword'
user.password_confirmation = 'NewSecurePassword'
user.save!
exit
```

Restart GitLab:

```bash
sudo gitlab-ctl restart
```

---

### 3.3 Issue: GitLab Not Responding on Custom Port

**Solution:**
- Ensure the port is set correctly in `gitlab.rb`.

- Run:

  ```bash
  sudo gitlab-ctl reconfigure
  sudo gitlab-ctl restart
  ```

- Verify the port is in use:

  ```bash
  sudo ss -tulnp | grep 8080
  ```

---

## 4. Final Setup & Security Considerations

- Change the **default root password** after installation.
- Set up **SSH keys** for Git operations:

  ```bash
  ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
  ```

- Enable **HTTPS** for secure access (optional).

---

## 5. Summary of Key Learnings

- GitLab requires multiple services to run (**Puma, PostgreSQL, Redis, etc.**).
- `gitlab-ctl` is the primary tool for managing GitLab services.
- Modifying ports requires updating **both Puma and external_url settings**.
- Troubleshooting involves checking logs, verifying running services, and restarting components.

---

## 🎯 Conclusion

This document provides a step-by-step guide on how to install, configure, and troubleshoot GitLab on a local server. By following these instructions, you can maintain a fully functional GitLab instance for local version control and collaboration.

🚀 **Enjoy using GitLab!** Let me know if you need further enhancements to the documentation.

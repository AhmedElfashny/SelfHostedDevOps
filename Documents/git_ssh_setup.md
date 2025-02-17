
# Git SSH Setup Documentation

## 1. Commands Used for Git Configuration

### 1.1 Configure Git User

```bash
git config --global user.name "ahmed ElFashny"
git config --global user.email "ahmedehabezzatkamel@gmail.com"
```

**Explanation:**

- Sets your name and email for all git operations on the machine.

### 1.2 Check Git Configuration

```bash
git config --global --list
```

**Explanation:**

- Displays your global git configuration settings.

### 1.3 Initialize a Repository (If Needed)

```bash
git init
```

**Explanation:**

- Initializes a new git repository in the current directory.

### 1.4 Stage and Commit Changes

```bash
git add .
git commit -m "Commit message"
```

**Explanation:**

- Stages all changes for commit.
- Commits the changes with the provided message.

### 1.5 Push Changes to Remote Repository

```bash
git push
```

**Explanation:**

- Pushes local commits to the remote repository.

### 1.6 Change Remote URL to SSH

```bash
git remote set-url origin git@github.com:AhmedElfashny/SelfHostedDevOps.git
```

**Explanation:**

- Updates the remote URL from HTTPS to SSH to use SSH key-based authentication.

## 2. SSH Key Setup Commands

### 2.1 Generate SSH Key

```bash
ssh-keygen -t ed25519 -C "ahmedalfashny1994@gmail.com"
```

**Explanation:**

- Generates a new SSH key pair using the ed25519 algorithm.
- The `-C` flag adds an email comment to the key.

### 2.2 Start the SSH Agent

```bash
eval "$(ssh-agent -s)"
```

**Explanation:**

- Starts the SSH agent, which runs in the background to manage SSH keys.

**SSH Agent Role:**

- Caches private keys so you don’t have to enter the passphrase every time.
- Once added, the agent retains the key for the duration of your session.

**How It Works:**

- `eval`: Executes the command output by `ssh-agent -s`.
- `ssh-agent -s`: Starts the SSH agent in the current session.

### 2.3 Add SSH Key to the Agent

```bash
ssh-add ~/.ssh/id_ed25519
```

**Explanation:**

- Adds the private key to the SSH agent.

**Why?**

- GitHub requires the private key for SSH authentication.
- Without adding it, git operations would fail even if the key exists.

### 2.4 Test SSH Connection

```bash
ssh -T git@github.com
```

**Expected Output:**

```
Hi AhmedElfashny! You've successfully authenticated, but GitHub does not provide shell access.
```

**Explanation:**

- Tests SSH connectivity to GitHub.
- Ensures the SSH key is correctly added to GitHub.

### 2.5 File Permissions

```bash
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
```

**Explanation:**

- Sets appropriate permissions for SSH keys to prevent unauthorized access.

## 3. Troubleshooting Tips

### Ensure SSH Agent is Running

If you get authentication errors, rerun:

```bash
eval "$(ssh-agent -s)"
```

### Re-Add SSH Key

If the key is missing after a reboot:

```bash
ssh-add ~/.ssh/id_ed25519
```

### Verify Remote URL

Confirm it uses SSH:

```bash
git remote -v
```

### GitHub Key Addition

- Make sure the public key is added to GitHub under **Settings > SSH and GPG keys**.

## 📌 Summary

By switching from HTTPS to SSH, you avoid GitHub's password authentication restrictions and ensure secure, passwordless access using SSH keys.

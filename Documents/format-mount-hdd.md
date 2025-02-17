# 📌 Complete Guide: Formatting & Mounting an External HDD for Kubernetes

This guide explains every step in detail, including what each command does and why we use it.

---

## 1️⃣ Understanding Key Concepts

### 🔹 What is `mount`?
The `mount` command attaches a filesystem (HDD, USB, network drive, etc.) to a specific directory in Linux.

#### Example:
```bash
sudo mount /dev/sdc1 /mnt/k8s-storage
```
This means: **Attach `/dev/sdc1` (the HDD) to `/mnt/k8s-storage` so we can use it.**

### 🔹 What is UUID?
UUID (Universally Unique Identifier) is a unique ID assigned to each partition. Instead of referring to a disk by `/dev/sdc1` (which may change after reboot), we use the UUID.

#### Example:
```ini
UUID=7d175e42-e22d-4f4b-9dbd-42179ac41cb6 /mnt/k8s-storage ext4 defaults 0 2
```
This ensures that Linux always mounts the correct disk at boot.

### 🔹 What is `fdisk`?
`fdisk` is a command-line tool to create, delete, and manage disk partitions.

#### Example:
```bash
sudo fdisk /dev/sdc
```
This opens the partition manager for the `/dev/sdc` disk.

---

## 2️⃣ Step-by-Step Guide

### 🛠️ Step 1: Identify the External HDD
#### Command:
```bash
lsblk
```
🔹 This lists all storage devices attached to the system.

#### 📌 Example Output:
```plaintext
NAME    MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda       8:0    0 238.5G  0 disk
└─sda1    8:1    0 238.5G  0 part /
sdc       8:32   0 931.5G  0 disk
```
✅ Here, we see that `/dev/sdc` is our external HDD.

---

### 🛠️ Step 2: Unmount Any Existing Mounts
#### Command:
```bash
sudo umount /mnt/k8s-storage
```
✅ This ensures that no process is using the disk.

---

### 🛠️ Step 3: Delete Existing Partitions
#### Command:
```bash
sudo fdisk /dev/sdc
```
📌 Inside `fdisk`, follow these steps:
- Press `p` → To list partitions.
- Press `d` → To delete existing partitions (repeat if multiple exist).
- Press `n` → To create a new partition.
- Press `p` → To choose Primary.
- Press `Enter` (twice) → To accept default first & last sector (use full disk).
- Press `w` → To save changes.

✅ Now, `/dev/sdc1` is created as a new partition.

---

### 🛠️ Step 4: Format the Disk as EXT4
#### Command:
```bash
sudo mkfs.ext4 /dev/sdc1
```
📌 This command:
- Formats `/dev/sdc1` as EXT4.
- Erases any old filesystem signatures.

✅ The disk is now clean and ready to use.

---

### 🛠️ Step 5: Check UUID
#### Command:
```bash
sudo blkid /dev/sdc1
```
📌 Example output:
```plaintext
/dev/sdc1: UUID="7d175e42-e22d-4f4b-9dbd-42179ac41cb6" TYPE="ext4"
```
✅ Copy the UUID for the next step.

---

### 🛠️ Step 6: Create a Mount Point
#### Command:
```bash
sudo mkdir -p /mnt/k8s-storage
```
✅ This creates the directory `/mnt/k8s-storage`.

---

### 🛠️ Step 7: Mount the Disk
#### Command:
```bash
sudo mount /dev/sdc1 /mnt/k8s-storage
```
✅ The HDD is now accessible from `/mnt/k8s-storage`.

To verify:
```bash
df -h | grep k8s-storage
```
📌 Example output:
```plaintext
/dev/sdc1  931G  1%  930G  1% /mnt/k8s-storage
```
✅ This confirms the disk is correctly mounted.

---

### 🛠️ Step 8: Make the Mount Persistent (Auto-Mount After Reboot)
#### Edit `/etc/fstab`:
```bash
sudo nano /etc/fstab
```
Add this line at the bottom:
```ini
UUID=7d175e42-e22d-4f4b-9dbd-42179ac41cb6 /mnt/k8s-storage ext4 defaults 0 2
```
Save & Exit (`CTRL + X`, then `Y`, then `Enter`).

✅ This tells Linux to automatically mount the disk on every boot.

---

### 🛠️ Step 9: Reload `fstab` and Verify
#### Command:
```bash
sudo systemctl daemon-reload
sudo mount -a
```
✅ This ensures Linux recognizes the new mount settings.

To verify:
```bash
df -h | grep k8s-storage
```
✅ If the output shows `/mnt/k8s-storage`, everything is working correctly!

---

## 📌 Summary: Commands Used

1️⃣ Check Available Disks:
```bash
lsblk
```

2️⃣ Unmount Existing Mounts (If Needed):
```bash
sudo umount /mnt/k8s-storage
```

3️⃣ Delete Old Partitions & Create a New One:
```bash
sudo fdisk /dev/sdc
```
(Follow the `fdisk` steps)

4️⃣ Format the Partition as EXT4:
```bash
sudo mkfs.ext4 /dev/sdc1
```

5️⃣ Get UUID:
```bash
sudo blkid /dev/sdc1
```

6️⃣ Create a Mount Point:
```bash
sudo mkdir -p /mnt/k8s-storage
```

7️⃣ Mount the Disk:
```bash
sudo mount /dev/sdc1 /mnt/k8s-storage
```

8️⃣ Make the Mount Persistent:
Edit `/etc/fstab`:
```bash
sudo nano /etc/fstab
```
Add:
```ini
UUID=YOUR-UUID /mnt/k8s-storage ext4 defaults 0 2
```

9️⃣ Reload `fstab` and Verify:
```bash
sudo systemctl daemon-reload
sudo mount -a
df -h | grep k8s-storage
```

✅ Now the disk is fully mounted and ready for Kubernetes! 🚀

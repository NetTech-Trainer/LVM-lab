# LVM-lab



---

# 🧩 A Practical Guide to Linux LVM (Logical Volume Management)

This repository provides a **hands-on, end-to-end guide to Linux Logical Volume Management (LVM)** using real commands and real scenarios.
It is designed for **students, system administrators, and Linux learners** who want to understand LVM not just theoretically, but practically.

---

## 📌 What You Will Learn

* Understanding disks, partitions, and filesystems
* Creating LVM from scratch (PV → VG → LV)
* Formatting and mounting logical volumes
* Extending logical volumes **online**
* Adding new disks to existing volume groups
* Safely reducing logical volumes (**offline**)
* Removing LVM components cleanly
* Managing swap space
* Backing up and restoring LVM metadata

---

## 🖥️ Environment Assumptions

* Linux system (CentOS / RHEL / Rocky / Alma compatible)
* Root access
* Virtual disks added (e.g., via VMware / VirtualBox)
* LVM utilities installed (`lvm2`)

---

## 🔍 Initial System Checks

### 1️⃣ Check Mounted Filesystems

```bash
df -h
```

Example output:

```
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda2        18G  2.8G   14G  17% /
/dev/sda1       291M   34M  242M  13% /boot
tmpfs           497M  228K  497M   1% /dev/shm
```

---

### 2️⃣ Check Available Disks

```bash
fdisk -l
```

Example:

* `/dev/sda` → OS disk
* `/dev/sdb` → New 2GB disk
* `/dev/sdc` → New 2GB disk (used later)

---

# 🚀 Phase 1: Creating LVM from Scratch

## Step 1: Prepare Disk Partition (LVM Type)

```bash
fdisk /dev/sdb
```

Actions inside `fdisk`:

* Create new primary partition
* Use full disk
* Change partition type to **8e (Linux LVM)**
* Write changes

Refresh kernel partition table:

```bash
partprobe -s
# or
sync
```

---

## Step 2: Create Physical Volume (PV)

```bash
pvcreate /dev/sdb1
pvdisplay
```

---

## Step 3: Create Volume Group (VG)

```bash
vgcreate myvol /dev/sdb1
vgdisplay
```

This creates a storage pool named **myvol**.

---

## Step 4: Create Logical Volume (LV)

```bash
lvcreate -L 500M -n lv1 myvol
lvdisplay
```

LV path:

```
/dev/myvol/lv1
```

---

# 📁 Phase 2: Formatting and Mounting

## Step 1: Create Filesystem

```bash
mkfs.ext3 /dev/myvol/lv1
```

*(ext4 or xfs can also be used)*

---

## Step 2: Mount Logical Volume

```bash
mkdir /Dell
mount /dev/myvol/lv1 /Dell
df -h
```

---

## Step 3: Permanent Mount (`/etc/fstab`)

```bash
/dev/myvol/lv1  /Dell  ext3  defaults  0 0
```

---

# ⚡ Phase 3: Managing LVM (Real Power)

## Scenario A: Extend Logical Volume (Online)

### Extend LV size

```bash
lvextend -l +100%FREE /dev/myvol/lv1
```

### Resize filesystem

```bash
resize2fs /dev/myvol/lv1
df -h
```

✔ Online extension supported for ext3/ext4.

---

## Scenario B: Extend Volume Group with New Disk

### Prepare new disk `/dev/sdc`

* Partition as type **8e**

```bash
pvcreate /dev/sdc1
vgextend myvol /dev/sdc1
vgdisplay
```

Now the VG has more free space.

---

## Scenario C: Reduce Logical Volume (⚠️ Offline & Risky)

> ⚠️ **Always take backups before reducing volumes**

### Steps:

```bash
umount /Dell
e2fsck -f /dev/myvol/lv1
resize2fs /dev/myvol/lv1 1000M
lvreduce -L 1000M /dev/myvol/lv1
mount -a
df -h
```

📌 **Filesystem must be resized before LVM reduction**

---

# 🧹 Phase 4: Removing LVM Components

Reverse order of creation:

```bash
umount /Dell
lvremove /dev/myvol/lv1
vgremove myvol
pvremove /dev/sdb1 /dev/sdc1
```

Also remove entry from `/etc/fstab`.

---


* Linux students
* System administrators
* DevOps beginners
* Interview preparation
* Home lab practice

---

## 📜 License

Free to use for **learning and educational purposes**.

---

## 🙌 Acknowledgement

Created by **Tushar Jadhav**
If you find this useful, ⭐ star the repo and share it with others!

---

If you want, I can also:

* Split this into multiple markdown files
* Add diagrams (PV → VG → LV)
* Convert it into a **blog-style README**
* Add troubleshooting & interview questions

Just tell me 👍

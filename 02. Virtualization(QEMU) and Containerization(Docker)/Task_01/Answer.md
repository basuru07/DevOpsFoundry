# Linux Administration and Troubleshooting Assignment

## Secure Web and Database Deployment with LVM, RAID, User Policies, and SELinux Monitoring

---

## Initial System Setup
Update and Upgrade System
```bash
sudo apt update && sudo apt upgrade -y
```

### Install Required Packages
```bash
sudo apt install -y mdadm lvm2 apache2 mysql-server vim net-tools
```

---

## 1. Create the Loop Devices

### Create First Disk Image (30GB)
```bash
sudo dd if=/dev/zero of=/disk1.img bs=1G count=30
```

### Create Second Disk Image (30GB)
```bash
sudo dd if=/dev/zero of=/disk2.img bs=1G count=30
```

**Parameters:**
- `if=/dev/zero`: Reads zeros as input
- `of=disk1.img`: Specifies the output file name
- `bs=1G count=30`: Creates 30GB disk image

### Attach Loop Devices
Verify the attachment location for disk1:
```bash
sudo losetup --find --show /disk1.img
```

Verify the attachment location for disk2:
```bash
sudo losetup --find --show /disk2.img
```

---

## 2. Combine Disks into RAID 0 Using mdadm

### Create RAID 0 Array
```bash
sudo mdadm --create /dev/md0 \
  --level=0 \
  --raid-devices=2 \
  /dev/loop13 /dev/loop15
```

### Verify RAID Configuration
```bash
lsblk
```

### Save RAID Configuration
```bash
sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf
sudo update-initramfs -u
```

---

## 3. Create Logical Volumes

### Step 1: Create Physical Volume
```bash
sudo pvcreate /dev/md0
```

### Step 2: Create Volume Group
```bash
sudo vgcreate myvg /dev/md0
```

### Step 3: Create Logical Volumes
Create lv_apps (2GB):
```bash
sudo lvcreate -L 2G -n lv_apps myvg
```

Create lv_data (2GB):
```bash
sudo lvcreate -L 2G -n lv_data myvg
```

### Step 4: Create Mount Points
```bash
sudo mkdir /apps /data
```

### Step 5: Create File Systems
Format lv_apps:
```bash
sudo mkfs.ext4 /dev/myvg/lv_apps
```

Format lv_data:
```bash
sudo mkfs.ext4 /dev/myvg/lv_data
```

### Step 6: Mount Logical Volumes
Mount lv_apps:
```bash
sudo mount /dev/myvg/lv_apps /apps
```

Mount lv_data:
```bash
sudo mount /dev/myvg/lv_data /data
```

### Step 7: Verify Mounts
```bash
df -h
```

---

## 4. Deploy the Web Page

### Create the Sample Web Page
```bash
echo "<html><body><h1>Sample Web Page</h1></body></html>" | sudo tee /apps/index.html
```

### Change the Apache DocumentRoot Path
Edit the Apache configuration file:
```bash
sudo vi /etc/apache2/sites-enabled/000-default.conf
```

Change `DocumentRoot /var/www/html` to `DocumentRoot /apps`

### Restart Apache
```bash
sudo systemctl restart apache2
```

---

## 5. Install and Configure MySQL

### Install MySQL Server
```bash
sudo apt install mysql-server -y
```

### Secure MySQL Installation
```bash
sudo mysql_secure_installation
```
(Set root password, remove anonymous users, disable remote root login, etc.)

### Stop MySQL Service
```bash
sudo systemctl stop mysql
```

### Create MySQL Data Directory
```bash
sudo mkdir /data/mysql
```

### Copy Existing Data
```bash
sudo rsync -av /var/lib/mysql /data/
```

### Update MySQL Configuration
Edit the MySQL configuration file:
```bash
sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf
```

Change `datadir = /var/lib/mysql` to `datadir = /data/mysql`

### Set Proper Ownership
```bash
sudo chown -R mysql:mysql /data/mysql
```

### Start MySQL Service
```bash
sudo systemctl start mysql
```

---

## 6. Create Users with Sudo and Directory Access

### Create Users
Create appuser:
```bash
sudo useradd -m appuser
```

Create dbuser:
```bash
sudo useradd -m dbuser
```

### Set Passwords
Set password for appuser (same as username):
```bash
sudo passwd appuser
```

Set password for dbuser (same as username):
```bash
sudo passwd dbuser
```

### Grant Sudo Privileges
Edit the sudoers file:
```bash
sudo visudo
```

Add the following lines:
```
appuser ALL=(ALL) NOPASSWD:ALL
dbuser ALL=(ALL) ALL
```

### Restrict Directory Access

Set ownership and permissions for `/apps`:
```bash
sudo chown appuser:appuser /apps
sudo chmod 700 /apps
```

Set ownership and permissions for `/data`:
```bash
sudo chown dbuser:dbuser /data
sudo chmod 700 /data
```

Override MySQL subdirectory ownership:
```bash
sudo chown -R mysql:mysql /data/mysql
```

### Test Directory Access
Test appuser access to /data (should deny):
```bash
su - appuser
ls /data
```

Test dbuser access to /apps (should deny):
```bash
su - dbuser
ls /apps
```

---

## 7. Install and Configure SELinux

### Install SELinux Packages
```bash
sudo apt install policycoreutils selinux-utils selinux-basics -y
```

### Activate SELinux
```bash
sudo selinux-activate
sudo reboot
```

### Set Enforcing Mode
```bash
sudo selinux-config-enforcing
sudo reboot
```

### Disable AppArmor (Conflicts with SELinux)
Stop AppArmor:
```bash
sudo systemctl stop apparmor
```

Disable AppArmor permanently:
```bash
sudo systemctl disable apparmor
```

### Map dbuser to SELinux Context
```bash
sudo semanage login -a -s guest_u dbuser
```

### Verify SELinux Configuration
```bash
sudo semanage login -l
```

---

## 8. Kernel Dump and Performance Analysis

### Enable KDump
```bash
sudo apt install linux-crashdump -y
```

Reboot the system:
```bash
sudo reboot
```

### Verify KDump Status
```bash
sudo systemctl status kdump-tools
```

### Trigger a Controlled Crash

Enable SysRq:
```bash
echo 1 | sudo tee /proc/sys/kernel/sysrq
```

Trigger the crash (system will crash and reboot automatically):
```bash
echo c | sudo tee /proc/sysrq-trigger
```

### Analyze the Kernel Dump

Install debug tools:
```bash
sudo apt install crash linux-image-$(uname -r)-dbg -y
```

Analyze the dump file:
```bash
sudo crash /var/crash/<timestamp>/vmcore /usr/lib/debug/boot/vmlinux-$(uname -r)
```

---

## Summary

This comprehensive guide covers:
- Loop device and RAID 0 setup for storage redundancy
- LVM configuration for flexible storage management
- Apache web server deployment on dedicated logical volume
- MySQL database installation with custom data directory
- User creation with granular permission control
- SELinux security enforcement
- Kernel dump analysis for troubleshooting
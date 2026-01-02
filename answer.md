# Linux Administration and Troubleshooting Assignment

---

## Assignment Answer

### Assignment Title
Secure Web and Database Deployment with LVM, RAID, User Policies, and SELinux Monitoring

### Assignment Steps
1. First setup your Ubuntu OS and Update and upgrade the system using the following command
```bash
sudo apt update && sudo apt upgrade -y
```
2. Install all of necessary things
```bash
sudo apt install -y mdadm lvm2 apache2 mysql-server vim net-tools
```

### 1. Create the Loop Devices
```bash
sudo dd if=/dev/zero of=/disk1.img bs=1G count=30
```
```bash
sudo dd if=/dev/zero of=/disk2.img bs=1G count=30
```
    - `if`=/dev/zero: Reads zeros as input.
    - `of`=disk1.img: Specifies the output file name.
    - `bs` = 1GB then count 30 (30GB)
verify the attach location 
```bash
sudo losetup --find --show /disk1.img
```
```bash
sudo losetup --find --show /disk2.img
```
### 2. Combine RAID 0 using mdadm
Then after both disk1.img and disk2.img combine into RAID 0 using mdadm
```bash
sudo mdadm --create /dev/md0 \  --level=0 \  --raid-devices=2 \ /dev/loop13 /dev/loop15
```
then after verify it and save config
```bash
lsblk
```
```bash
sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf and sudo update-initramfs -u
```

### 3. Create Logical volumes
1. Create physical volume
```bash
sudo pvcreate /dev/md0
```
2. Create volume group
```bash
sudo vgcreate myvg /dev/md0
```
3. Create Logical volumes 
```bash
sudo lvcreate -L 2G -n lv_apps myvg
```
```bash
sudo lvcreate -L 2G -n lv_data myvg
```
4. Create mount point 
```bash
sudo mkdir /apps /data
```
5. Create the fileSystem 
```bash
sudo mkfs.ext4 /dev/myvg/lv_apps
```
```bash
sudo mkfs.ext4 /dev/myvg/lv_data
```
6. Then Mount
```bash
sudo mount /dev/myvg/lv_apps /apps
```
```bash
sudo mount /dev/myvg/lv_data /data
```
7. After verify it 
```bash
df -h
```















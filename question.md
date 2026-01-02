# Linux Administration and Troubleshooting Assignment

## Assignment Details

**Operating System:** Ubuntu 22.04 LTS or latest

---

## Assignment Overview

### Assignment Title
Secure Web and Database Deployment with LVM, RAID, User Policies, and SELinux Monitoring

### Assignment Path
In this assignment you will deploy a sample web application server and database in a Linux machine using LVM with RAID disk partitioning. Different mountpoints to be used for web application and database with controlled access for application user (appuser) and database user (dbuser). Finally you will enable SELinux policies and troubleshoot the system behaviour using a kernel dump.
- Deploy a sample web page and MySQL database. 
- Configure Logical Volume Management with RAID 0. 
- Partition storage using LVM: /apps (20GB), /data (30GB) from a 50GB volume group. 
- Create and configure appuser and dbuser with sudo and SELinux policies. 
- Enforce SELinux policies on dbuser with audit logging and restricted policy. 
- Analyze system behavior with kernel dump. 
---

## Assignment Steps

### Step 1: Setup the Environment
Install Ubuntu 22.04 or 24.04 on a virtual machine with the following specifications:

- **vCPUs:** 2
- **RAM:** 4 GB
- **Disk Space:** Minimum 60 GB

### Step 2: Create and Mount LVM Partitions with RAID 0
- Create 2 disks using loop devices (or attach real block devices). 
- Combine them into RAID 0 using mdadm. 
- Create a Volume Group (VG) with the RAID device. 
- Create Logical Volumes: 
- - /apps (20GB) 
- - /data (30GB) 
- Format with ext4 and mount: 
- - /apps → /dev/<vg_name>/lv_apps 
- - /data → /dev/<vg_name>/lv_data 

### Step 3: Deploy Web Page and MySQL 
- Install Apache or Nginx, place a sample HTML in /apps 
- Install MySQL and store data in /data

## Step 4: Create Users with Sudo and Directory Access 
- Create local user accounts as  appuser and dbuser. 
- Grant sudo privileges: 
- Modify /etc/sudoers using visudo. 
 - - Configure appuser with → NOPASSWD option 
 - - Configure dbuser with → PASSWORD option  
- Restrict /data mountpoint access from appuser (dbuser cannot be able to access the /apps) 
- Configure proper ownership and permissions (appuser needs to have access only to /apps and dbuser needs to have access only to /data)

### Step 5: Install and Configure SELinux 
- Install SELinux 
- Set SELinux to enforcing mode 
- Map dbuser to SELinux user (guest_u temporarily) 
- Review SELinux logs and show dbuser is restricted for the commands temporarily 

### Step 6: Kernel Dump and Performance Analysis 
- Enable kdump (sudo apt install linux-crashdump) 
- Reboot and trigger a controlled system crash for a demo purposes (echo c | sudo tee /proc/sysrq-trigger) 
- Use crash or kdump-tools to analyze the dump (Analyze CPU, Memory and system bottlenecks, show the output of crash command to identify the cause for the crash)  


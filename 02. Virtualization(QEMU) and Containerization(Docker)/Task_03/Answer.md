# Task 03 - QEMU Virtual Machine Setup Guide

## Overview

QEMU (Quick Emulator) is a powerful, open-source machine emulator and virtualizer that allows you to run operating systems as if they were running on real hardware. It can emulate CPU, memory, storage, and various devices. When combined with **KVM (Kernel-based Virtual Machine)**, it leverages hardware virtualization capabilities for near-native performance.

This guide provides step-by-step instructions to set up an Ubuntu virtual machine using QEMU and KVM, with SSH access via port forwarding.

### Key Features

- **Hardware Emulation**: Emulates CPU, memory, storage, and network devices
- **KVM Integration**: Hardware virtualization for near-native performance
- **Remote Access**: SSH and VNC (Virtual Network Computing) support
- **Flexible Configuration**: Customizable memory, CPU cores, and storage

---

## Prerequisites

- Linux system with virtualization support (Intel VT-x or AMD-V)
- Sudo/root access
- Ubuntu ISO file (24.04 LTS recommended)
- Sufficient disk space (minimum 20GB for the VM)

---

## Installation Steps

### Step 1: System Update

Update your package manager to ensure you have the latest software versions:

```bash
sudo apt-get update && sudo apt-get upgrade -y
```

### Step 2: Install QEMU, KVM, and Related Tools

Install the necessary virtualization packages:

```bash
sudo apt install qemu-system-x86 qemu-utils libvirt-daemon-system libvirt-clients bridge-utils virt-manager -y
```

**Package Descriptions:**
- `qemu-system-x86`: QEMU emulator for x86/x64 systems
- `qemu-utils`: QEMU utilities for image management
- `libvirt-daemon-system`: Libvirt daemon for VM management
- `libvirt-clients`: CLI tools for interacting with libvirt
- `bridge-utils`: Network bridging utilities
- `virt-manager`: GUI tool for managing virtual machines

### Step 3: Verify KVM Support

Check if your CPU supports hardware virtualization. A return value greater than 0 indicates support:

```bash
egrep -c '(vmx|svm)' /proc/cpuinfo
```

**Expected Output:** A number > 0 (e.g., 4, 8, 16) indicates virtualization support is available.

- **vmx**: Intel VT-x support
- **svm**: AMD-V support

### Step 4: Create the Hard Disk Image

Create a 20GB QCOW2 (QEMU Copy-On-Write) format disk image. QCOW2 is space-efficient and supports snapshots:

```bash
qemu-img create -f qcow2 ubuntu-vm.qcow2 20G
```

You can verify the image creation:

```bash
ls -lh ubuntu-vm.qcow2
```

### Step 5: Install Ubuntu on the VM

Boot the VM from the Ubuntu ISO to begin installation. The VM will have 4GB RAM, hardware-accelerated CPU, and KVM enabled:

```bash
qemu-system-x86_64 \
  -m 4G \
  -cpu host \
  -enable-kvm \
  -drive file=ubuntu-vm.qcow2,format=qcow2 \
  -cdrom /path/to/ubuntu-24.04-live-server-amd64.iso \
  -boot d \
  -vga virtio \
  -display vnc=:1
```

**Command Parameters:**
- `-m 4G`: Allocate 4GB RAM (adjust as needed)
- `-cpu host`: Use host CPU features for better performance
- `-enable-kvm`: Enable KVM hardware virtualization
- `-drive file=ubuntu-vm.qcow2,format=qcow2`: Specify the disk image
- `-cdrom /path/to/iso`: Path to the Ubuntu ISO file
- `-boot d`: Boot from CD-ROM (d = DVD/CD)
- `-vga virtio`: Use virtio graphics for better performance
- `-display vnc=:1`: Enable VNC display on port 5901

**Installation Access:**
Connect to the VM via VNC at `localhost:5901` using a VNC client during installation.

### Step 6: Boot the VM After Installation

Once Ubuntu is installed, boot the VM normally from the disk:

```bash
qemu-system-x86_64 \
  -m 4G \
  -cpu host \
  -enable-kvm \
  -drive file=ubuntu-vm.qcow2,format=qcow2 \
  -boot c \
  -vga virtio \
  -display vnc=:1
```

**Key Change:**
- `-boot c`: Boot from the disk (c = primary disk)

### Step 7: Enable SSH via Port Forwarding

To access the VM via SSH, configure QEMU user-mode networking with port forwarding. This forwards TCP port 2222 on the host to port 22 (SSH) on the VM:

```bash
qemu-system-x86_64 \
  -m 4G \
  -cpu host \
  -enable-kvm \
  -drive file=ubuntu-vm.qcow2,format=qcow2 \
  -boot c \
  -vga virtio \
  -net user,hostfwd=tcp::2222-:22 \
  -net nic \
  -display vnc=:1
```

**Networking Parameters:**
- `-net user,hostfwd=tcp::2222-:22`: Enable user-mode networking with port forwarding (host 2222 → VM 22)
- `-net nic`: Create a virtual network interface card

**Inside the VM:** Ensure SSH is installed and running:

```bash
sudo apt install openssh-server -y
sudo systemctl start ssh
sudo systemctl enable ssh
```

### Step 8: Connect via SSH

From the host machine, connect to the VM using SSH:

```bash
ssh username@localhost -p 2222
```

**Example:**
```bash
ssh ubuntu@localhost -p 2222
```

Replace `ubuntu` with your actual username from the VM installation.

---
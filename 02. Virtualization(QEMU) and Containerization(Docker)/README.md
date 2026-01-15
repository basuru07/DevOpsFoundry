# Linux Containers

## Overview

Linux containers share the host kernel but are isolated from each other. This isolation is achieved using namespaces, which are kernel features that partition system resources such as processes, networks, and filesystems.

## Creating a Linux Container

Creating a Linux container involves two main steps: setting up namespaces and establishing network connectivity.

### Step 1: Namespaces

Namespaces are kernel features that isolate and virtualize system resources. They are the foundation of container isolation.

#### Common Namespaces

- **PID Namespace** – Each container has its own process IDs.
- **UTS Namespace** – Containers can have their own hostname.
- **Mount Namespace** – Provides an isolated filesystem view.
- **Network Namespace** – Each container gets its own network interface, routing table, and ports.
- **User Namespace** – Isolates user and group IDs for security.

### Step 2: Networking with veth

The second step is connecting the container to the host using a virtual ethernet pair (veth).

```
container -----------------> host
  (veth0)        connect     (veth1)
     |____________________________|
            veth pair (virtual cable)
```

#### How veth Works

A veth pair acts like a virtual network cable:

- One end is placed inside the container
- The other end remains on the host
- This allows bidirectional communication between the container and host

#### Configuration Steps

After setting up the veth pair, complete the following:

1. Assign an IP address to the container interface
2. Test connectivity using `ping <IP_Address>`
3. Bring the interface up
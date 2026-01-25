# 03. Deploying DNS and DHCP on Ubuntu Server with Client Verification 

## Assignment Details

# DNS and DHCP Server Configuration Lab

## Objective

Deploy and configure a DNS and DHCP server on an Ubuntu Server VM. Verify that two Ubuntu Desktop VMs receive IP addresses via DHCP and can resolve domain names both for internet sites and internal network devices.

## Lab Setup Requirements

- Virtual Box
- Ubuntu Server 22.04 LTS Image
- Ubuntu Desktop 22.04 LTS Image
- Internet access to all VMs

## Assignment Steps

### 1. Setup two Linux Servers and two Linux Desktops on Virtual Box 

- Add one network interface for internet connectivity with NAT
- Configure other network adapter to internal network

### 2. Deploy Two Ubuntu Server VMs with Static IP

- **DNS Server IP**: 192.168.50.1/24 (Use DNS server as 127.0.0.1)
- **DHCP Server IP**: 192.168.50.2/24 (Use DNS server as 192.168.50.1)

### 3. Install and Configure DNS (BIND9) with linuxtraining25.com domain.

#### Add Forward Lookup Zone

Create A records for clients and servers:

- `dns.linuxtraining25.com` → 192.168.50.1
- `dhcp.linuxtraining25.com` → 192.168.50.2

#### Reverse Lookup Zone to DNS

Add PTR records corresponding to the A records above.

### 4. Verify DNS Server Records (From DNS server)

Run the following commands from the DNS server:

```bash
nslookup dns.linuxtraining25.com
nslookup 192.168.50.1
```

### 5. Install and Configure DHCP (ISC-DHCP-SERVER)

Configure the DHCP server with the following parameters:

- **Subnet**: 192.168.50.0/24
- **DHCP Scope**: 192.168.50.100 to 192.168.50.200
- **DNS Server**: 192.168.50.1
- **DNS Domain Name**: linuxtraining25.com

### 6. Deploy Two Ubuntu Desktop VMs (Client1, Client2)

Deploy Client1 and Client2 with the following configuration:

- Set network mode to internal network in VirtualBox
- Configure each client's network interface to use Automatic (DHCP)
- Verify that the DNS server IP (192.168.50.1) is automatically assigned via DHCP configuration
- Verify IP addresses assignment

### 7. Add the following records to the DNS server forward and reverse lookup zones accordingly. 

Add the following records to the DNS server forward and reverse lookup zones:

- `client1.linuxtraining25.com` → {IP address assigned to client1 via DHCP}
- `client2.linuxtraining25.com` → {IP address assigned to client2 via DHCP}

### 8. Verification (Required screenshot in the detailed document) 

All verification steps require screenshots in the detailed document.

#### IP Assignment Verification

Verify that the IP address falls within the correct range (192.168.50.100 to 192.168.50.200):

```bash
ifconfig
# or
ip a
```

#### Inter-Client Communication

When using nslookup/dig commands, ensure that the DNS server resolves to 192.168.50.1.

**Test from Client1 desktop to Client2 by DNS name:**

```bash
ping client2.linuxtraining25.com
nslookup client2.linuxtraining25.com
dig client2.linuxtraining25.com
```

**Test from Client2 desktop to Client1 by DNS name:**

```bash
ping client1.linuxtraining25.com
nslookup client1.linuxtraining25.com
dig client1.linuxtraining25.com
```

**Check PTR records for reverse lookup:**

```bash
dig -x <client1_ip_address>
dig -x <client2_ip_address>
```
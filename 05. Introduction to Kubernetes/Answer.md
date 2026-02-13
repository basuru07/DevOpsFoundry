# DNS and DHCP Server Setup Guide

## Phase 1: Virtual Machine Network Configuration

### Step 1: Configure VM Network Adapters

For each VM in VirtualBox settings:

1. Go to **Settings → Network**
2. **Adapter 1**: Set to `NAT` (enabled)
3. **Adapter 2**: Set to `Internal Network` (enabled)
   - This creates a private LAN between VMs as required by the task

---

## Phase 2: DNS Server Configuration

### Step 2: Configure DNS Server Network Interface

Login to the DNS server and configure netplan for a fixed IP address.

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

Replace with:

```yaml
network:
  version: 2
  ethernets:
    enp0s8:
      dhcp4: no
      addresses:
        - 192.168.50.1/24
      gateway4: 192.168.50.1
      nameservers:
        addresses: [127.0.0.1]
```

Apply and verify:

```bash
sudo netplan apply
ip a
```

---

### Step 3: Configure DHCP Server Network Interface

Login to the DHCP server and configure netplan.

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

Replace with:

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s8:
      dhcp4: no
      addresses:
        - 192.168.50.2/24
      nameservers:
        addresses: [192.168.50.1]
```

Apply and verify:

```bash
sudo netplan apply
ip a
```

### Expected Network Configuration

| Server | Interface | IP Address |
|--------|-----------|-----------|
| main (DNS) | enp0s8 | 192.168.50.1 |
| k8s-worker2 (DHCP) | enp0s8 | 192.168.50.2 |

---

## Phase 3: Install and Configure BIND9 (DNS Server)

### Step 4: Install BIND9

```bash
sudo apt install bind9 bind9utils -y
```

### Step 5: Define DNS Zones

```bash
sudo nano /etc/bind/named.conf.local
```

Add:

```bind
zone "linuxtraining25.com" {
    type master;
    file "/etc/bind/db.linuxtraining25.com";
};

zone "50.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/db.192.168.50";
};
```

#### Zone Type Reference

- **Forward Zone**: Maps hostnames → IP addresses
- **Reverse Zone**: Maps IP addresses → hostnames

---

### Step 6: Create Forward Zone File

Copy the template:

```bash
sudo cp /etc/bind/db.local /etc/bind/db.linuxtraining25.com
sudo nano /etc/bind/db.linuxtraining25.com
```

Replace with:

```bind
;
; BIND data file for linuxtraining25.com
;
$TTL    604800
@       IN      SOA     dns.linuxtraining25.com. root.linuxtraining25.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      linuxtraining25.com.
dns     IN      A       192.168.50.1
dhcp    IN      A       192.168.50.2
@       IN      AAAA    ::1
```

---

### Step 7: Create Reverse Zone File

Copy the template:

```bash
sudo cp /etc/bind/db.127 /etc/bind/db.192.168.50
sudo nano /etc/bind/db.192.168.50
```

Replace with:

```bind
$TTL    604800
@   IN  SOA dns.linuxtraining25.com. root.linuxtraining25.com. (
        2
        604800
        86400
        2419200
        604800 )
@   IN  NS  dns.linuxtraining25.com.
1   IN  PTR dns.linuxtraining25.com.
2   IN  PTR dhcp.linuxtraining25.com.
```

---

### Step 8: Restart and Test DNS Server

Restart BIND9:

```bash
sudo systemctl restart bind9
```

Test forward lookup:

```bash
nslookup dns.linuxtraining25.com
```

Test reverse lookup:

```bash
nslookup 192.168.50.1
```

---

## Phase 4: Install and Configure DHCP Server

### Step 9: Install ISC DHCP Server

```bash
sudo apt install isc-dhcp-server -y
```

### Step 10: Configure DHCP Server Interface

Edit the DHCP server configuration to listen only on the internal NIC:

```bash
sudo nano /etc/default/isc-dhcp-server
```

Set:

```
INTERFACESv4="enp0s8"
```

### Step 11: Configure DHCP Pool and Settings

```bash
sudo nano /etc/dhcp/dhcpd.conf
```

Add:

```dhcp
subnet 192.168.50.0 netmask 255.255.255.0 {
  range 192.168.50.100 192.168.50.200;
  option routers 192.168.50.1;
  option domain-name-servers 192.168.50.1;
  option domain-name "linuxtraining25.com";
}
```

### Step 12: Restart DHCP Server

```bash
sudo systemctl restart isc-dhcp-server
```

---

## Phase 5: Add Client DNS Records

### Step 13: Add Clients to Forward Zone

After clients receive DHCP IPs (e.g., 192.168.50.101, 192.168.50.102):

```bash
sudo nano /etc/bind/db.linuxtraining25.com
```

Add:

```bind
client1 IN A 192.168.50.101
client2 IN A 192.168.50.102
```

### Step 14: Add Clients to Reverse Zone

```bash
sudo nano /etc/bind/db.192.168.50
```

Add:

```bind
101 IN PTR client1.linuxtraining25.com.
102 IN PTR client2.linuxtraining25.com.
```

### Step 15: Restart DNS Server

```bash
sudo systemctl restart bind9
```

---

## Phase 6: Verification and Testing

### Verification Commands (Run on Clients)

Forward DNS lookup:

```bash
ping client2.linuxtraining25.com
nslookup client2.linuxtraining25.com
dig client2.linuxtraining25.com
```

Reverse DNS lookup:

```bash
dig -x 192.168.50.101
```

---

## Summary

This setup creates a complete DNS and DHCP infrastructure with:
- **DNS Server**: BIND9 managing forward and reverse zones for `linuxtraining25.com`
- **DHCP Server**: ISC DHCP distributing IPs in the 192.168.50.100-200 range
- **Private Network**: Internal network adapter creates isolated LAN between VMs
- **Name Resolution**: Both forward and reverse DNS lookups functional for all clients

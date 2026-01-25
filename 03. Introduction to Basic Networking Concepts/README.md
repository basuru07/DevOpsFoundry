# DNS and DHCP - Simple Theory Guide

## What is DHCP?

**DHCP = Automatic IP Address Giver**

Instead of manually assigning IP addresses to every computer, DHCP automatically gives each device an IP address when it connects to the network.

**How it works:**
1. Computer boots up and asks "Can anyone give me an IP?"
2. DHCP server responds "Yes! Use 192.168.50.100"
3. Computer accepts and uses that IP
4. IP is "leased" for a set time (default: 10 minutes to 24 hours)

**DHCP gives 3 things:**
- IP address (e.g., 192.168.50.100)
- DNS server address (e.g., 192.168.50.1)
- Gateway/Router address (e.g., 192.168.50.1)

---

## What is DNS?

**DNS = Phone Directory for Internet**

When you type "google.com", your computer doesn't know what IP address that is. DNS converts names into IP addresses.

**Example:**
- You ask: "What IP is client1.linuxtraining25.com?"
- DNS answers: "It's 192.168.50.100"
- Your computer connects to 192.168.50.100

**Two types of DNS lookup:**

1. **Forward Lookup** (Name → IP)
   - Question: What IP is client1.linuxtraining25.com?
   - Answer: 192.168.50.100

2. **Reverse Lookup** (IP → Name)
   - Question: What name is 192.168.50.100?
   - Answer: client1.linuxtraining25.com

---

## DNS Records (Simple Explanation)

**A Record**: Maps domain name to IP address
```
client1.linuxtraining25.com  →  192.168.50.100
```

**PTR Record**: Maps IP address back to domain name
```
192.168.50.100  →  client1.linuxtraining25.com
```

**NS Record**: Says "This is your DNS server"
```
nameserver: dns.linuxtraining25.com
```

**SOA Record**: Contains zone information (less important for now)

---

## Network Setup (Two Network Cards)

Each server has 2 network adapters:

**Network Adapter 1 (NAT)**
- Purpose: Internet access
- Gets IP automatically
- Can browse internet

**Network Adapter 2 (Internal Network)**
- Purpose: Talk to other VMs
- Gets static IP on servers
- Gets DHCP IP on clients

---

## Static IP vs Dynamic IP

**Static IP** (For Servers)
- You manually set it (e.g., 192.168.50.1)
- It never changes
- Clients always know where to find the server
- Set in: `/etc/netplan/` config files

**Dynamic IP** (For Clients)
- DHCP server assigns it automatically
- Changes when you reboot or lease expires
- Clients don't need to know in advance
- Configured to use DHCP in network settings

---

## How DNS and DHCP Work Together

1. **Client boots up**
   - Sends DHCP request: "Give me an IP!"

2. **DHCP server responds**
   - "Here's your IP: 192.168.50.100"
   - "Use this DNS server: 192.168.50.1"

3. **Client stores DNS server address**
   - Remembers: DNS is at 192.168.50.1

4. **Client needs to access something by name**
   - Asks DNS: "What IP is client2.linuxtraining25.com?"
   - DNS answers: "It's 192.168.50.101"

5. **Client can now connect** to client2

---

## Basic DHCP Configuration

```
Subnet: 192.168.50.0/24
IP Range: 192.168.50.100 to 192.168.50.200
DNS Server: 192.168.50.1
Lease Time: 10 minutes
```

This means:
- Give clients IPs between .100 and .200
- Tell all clients their DNS is at .1
- Each IP is valid for 10 minutes before renewal

---

## Basic DNS Configuration

**For DNS Server (192.168.50.1):**
```
Domain: linuxtraining25.com
Nameserver: dns.linuxtraining25.com (192.168.50.1)

Forward Records (Name to IP):
dns.linuxtraining25.com    → 192.168.50.1
dhcp.linuxtraining25.com   → 192.168.50.2
client1.linuxtraining25.com → 192.168.50.100
client2.linuxtraining25.com → 192.168.50.101

Reverse Records (IP to Name):
192.168.50.1   → dns.linuxtraining25.com
192.168.50.2   → dhcp.linuxtraining25.com
192.168.50.100 → client1.linuxtraining25.com
192.168.50.101 → client2.linuxtraining25.com
```

---

## Testing Commands (What They Do)

**ping client1.linuxtraining25.com**
- Asks DNS for IP of client1
- Sends signal to that IP
- Shows if computer is reachable

**nslookup client1.linuxtraining25.com**
- Asks DNS server: "What IP is this?"
- DNS answers with the IP address
- Shows which DNS server answered

**dig client1.linuxtraining25.com**
- More detailed than nslookup
- Shows exactly what DNS server answered
- Shows all details of the response

**dig -x 192.168.50.100**
- Reverse lookup
- Asks: "What name goes with this IP?"

---

## The 4 VMs in This Lab

| VM | Type | IP | Purpose |
|---|---|---|---|
| DNS Server | Ubuntu Server | 192.168.50.1 | Translates names to IPs |
| DHCP Server | Ubuntu Server | 192.168.50.2 | Assigns IPs to clients |
| Client 1 | Ubuntu Desktop | Auto (100-200) | Gets IP from DHCP, resolves names via DNS |
| Client 2 | Ubuntu Desktop | Auto (100-200) | Gets IP from DHCP, resolves names via DNS |

---

## Key Points to Remember

1. **DHCP = Automatic IP assignment**
2. **DNS = Convert names to IP addresses**
3. **Servers need static IPs** (so clients can find them)
4. **Clients use DHCP** (automatic IP assignment)
5. **DHCP tells clients where DNS is** (via DHCP options)
6. **Clients ask DNS for other computer names** (to find them by name)
7. **Two network cards per VM**: One for internet (NAT), one for internal network
8. **Always increment serial number** in DNS zone files when you make changes
9. **Test with dig/nslookup/ping** to verify everything works

---

## Common Files You'll Edit

| File | Purpose |
|---|---|
| `/etc/netplan/*.yaml` | Set static IP on servers |
| `/etc/dhcp/dhcpd.conf` | DHCP configuration |
| `/etc/bind/named.conf` | DNS main configuration |
| `/var/lib/bind/db.linuxtraining25.com` | DNS forward lookup zone |
| `/var/lib/bind/db.50.168.192` | DNS reverse lookup zone |

---
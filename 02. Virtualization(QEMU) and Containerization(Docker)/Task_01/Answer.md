# Task 01 - Build and Run a Custom Linux Container with Networking

# Linux Container Setup

```
Install Dependencies
        ↓
Clone Repository
        ↓
Build Project
        ↓
Create Container Environment
        ↓
Run Container
        ↓
Create Namespaces
        ↓
Enter Container Namespace
        ↓
Create Virtual Ethernet Pair
        ↓
Move veth1 to Container
        ↓
Assign IP to Host
        ↓
Assign IP to Container
        ↓
Test Connectivity
```

## 01. Update System and Install Dependencies

Update your system packages and install the required dependencies:

```bash
sudo apt-get update && sudo apt-get upgrade
sudo apt install -y build-essential util-linux iproute2 net-tools
```

**Dependency Details:**
- `build-essential` – Tools needed to compile C code
- `iproute2` and `net-tools` – Utilities for configuring networking

## 02. Clone the Repository

Clone the simple-container project from GitHub:

```bash
git clone https://github.com/gnudeep/simple-container.git
cd simple-container
```

## 03. Build the Project

Build the project using make. This will compile the C code and create the executable binary:

```bash
make
```

The build creates a binary file named `simple-container`. Execute it with:

```bash
./simple-container
```

**Note:** This binary uses Linux namespaces including PID, UTS, Mount, and Network isolation.

## 04. Create the Container Environment

Set up the container environment by running the initialization script:

```bash
sudo ./make-dev.sh
```

## 05. Run the Container

Launch the container and enter an isolated environment:

```bash
sudo ./simple-container
```

You are now inside an isolated container environment.

## 06. Enter the Container Namespace (nsenter)

Use nsenter to enter the container namespaces. This provides full isolation:

```bash
nsenter --uts --pid --mount --net -t <PID>
```

**Namespace Flags:**
- `--uts` – Hostname isolation
- `--pid` – Process isolation
- `--mount` – Filesystem isolation
- `--net` – Network isolation

## 07. Create Virtual Ethernet Pair (Host)

Create a virtual ethernet pair that acts like a virtual network cable:

```bash
sudo ip link add veth0 type veth peer name veth1
```

**Allocation:**
- `veth0` – Stays on the host
- `veth1` – Will be moved into the container

## 08. Move veth1 into Container Network Namespace

Transfer the container-side interface into the container's network namespace (replace `1678` with the actual container PID):

```bash
sudo ip link set veth1 netns 1678
```

**Ownership:**
- Host owns `veth0`
- Container owns `veth1`

## 10. Assign IP Address (Host)

Configure the host-side virtual interface with an IP address:

```bash
sudo ifconfig veth0 192.168.1.1 up
```

## 11. Assign IP Address (Inside Container)

From inside the container, configure the container-side interface:

```bash
ifconfig veth1 192.168.1.2 up
```

## 12. Test Network Connectivity

Verify that the host and container can communicate with each other.

**Ping from Host to Container:**

```bash
ping 192.168.1.2
```

**Ping from Container to Host:**

```bash
ping 192.168.1.1
```

If both pings are successful, your container networking is properly configured!
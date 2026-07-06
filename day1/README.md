# OpenStack Day 1 — Prerequisites & Initial Setup

## OpenStack Architecture Overview

### 3-Layer Service Architecture

![OpenStack 3-Layer Architecture](3l.png)

### Installer and Deployment Options

![OpenStack Installer Options](installer.png)

### Ansible + Container-Based Deployment

![Container-Based Deployment](cont1.png)

---

## Prerequisites for 3-Node Deployment

### Initial Setup on All Machines

#### Step 1: Escalate to Root User

```bash
sudo -i
whoami  # Output: root
```

#### Step 2: Update Ubuntu Package Repository

```bash
apt update
```

**Output:**
```
Get:1 http://security.ubuntu.com/ubuntu jammy-security InRelease [129 kB]
Hit:2 http://archive.ubuntu.com/ubuntu jammy InRelease  
Get:3 http://security.ubuntu.com/ubuntu jammy-security/main amd64 Packages [3,351 kB]
Get:4 http://security.ubuntu.com/ubuntu jammy-security/main Translation-en [472 kB]
Get:5 http://security.ubuntu.com/ubuntu jammy-security/main amd64 c-n-f Metadata [14.6 kB]
Get:6 http://security.ubuntu.com/ubuntu jammy-security/restricted amd64 Packages [5,977 kB]
Get:7 http://archive.ubuntu.com/ubuntu jammy-updates InRelease [128 kB]
```

#### Step 3: Install Essential Tools

```bash
apt install vim net-tools
```

#### Step 4: Set Hostname Permanently

```bash
hostname  # Check current hostname
hostnamectl set-hostname node1  # Set new hostname
exit
sudo -i
hostname  # Verify: should show "node1"
```

---

## Network Configuration

### Node1 and Node2: Multi-NIC Setup

Nodes 1 and 2 each have 2 network interfaces (NICs) for separated management and data traffic.

#### Check Network Interfaces

```bash
ifconfig
ifconfig -a
```

**Output:**

```
ens18: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.0.19.76  netmask 255.0.0.0  broadcast 10.255.255.255
        inet6 fe80::be24:11ff:fef9:b735  prefixlen 64  scopeid 0x20<link>
        ether bc:24:11:f9:b7:35  txqueuelen 1000  (Ethernet)
        RX packets 35911  bytes 29222088 (29.2 MB)
        RX errors 0  dropped 4182  overruns 0  frame 0
        TX packets 2198  bytes 237056 (237.0 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 130  bytes 11713 (11.7 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 130  bytes 11713 (11.7 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

ens19: flags=4098<BROADCAST,MULTICAST>  mtu 1500
        ether bc:24:11:52:1f:3b  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

### Configure Static IP (Node1)

Edit the Netplan configuration file:

```bash
cd /etc/netplan/
ls
cat 01-netcfg.yaml
```

**Initial Configuration:**

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    ens18:
      dhcp4: no
      addresses:
        - 10.0.19.76/8
      routes:
        - to: default
          via: 10.0.0.2
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4
```

**Updated Configuration (with ens19 disabled):**

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    ens19:
      dhcp4: no
      dhcp6: no
    ens18:
      dhcp4: no
      addresses:
        - 10.0.19.76/8
      routes:
        - to: default
          via: 10.0.0.2
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4
```

Apply the configuration:

```bash
netplan apply
```

### Configure Hostname-Based Local DNS

Edit `/etc/hosts` to add hostname-to-IP mappings for all nodes:

```bash
nano /etc/hosts
```

**File Content:**

```
127.0.0.1 localhost
127.0.1.1 student

# IPv6
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

# Node mappings
10.0.19.76  node1
10.0.19.77  node2
10.0.19.78  node3
```

Verify connectivity:

```bash
ping node1   # Local node
ping node2   # Remote node
ping node3   # Remote node
```

**Output:**

```
PING node1 (10.0.19.76) 56(84) bytes of data.
64 bytes from node1 (10.0.19.76): icmp_seq=1 ttl=64 time=0.020 ms
--- node1 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.020/0.020/0.020/0.000 ms

PING node2 (10.0.19.77) 56(84) bytes of data.
64 bytes from node2 (10.0.19.77): icmp_seq=1 ttl=64 time=0.435 ms
64 bytes from node2 (10.0.19.77): icmp_seq=2 ttl=64 time=0.449 ms
--- node2 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1022ms
rtt min/avg/max/mdev = 0.435/0.442/0.449/0.007 ms
```

### Sync Time Across All Nodes

Install Chrony for NTP time synchronization:

```bash
apt install chrony
```

---

## Install Docker CE on All 3 Nodes

Add Docker's GPG key and repository:

```bash
# Update package manager
sudo apt update
sudo apt install ca-certificates curl

# Create GPG keyring directory
sudo install -m 0755 -d /etc/apt/keyrings

# Add Docker's GPG key
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

Add Docker repository to APT sources:

```bash
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update
```

Install Docker and related packages:

```bash
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

---

## Setup Python Environment on Node1 (Ansible Controller)

### Install Python Dependencies

```bash
apt install python3-venv python3-pip python3-docker
```

### Create Virtual Environment

```bash
python3 -m venv ~/openstack-setup
source openstack-setup/bin/activate
```

### Install/Upgrade Python Tools

```bash
pip install -U pip setuptools wheel
```

**Output:**

```
Collecting pip
  Downloading pip-26.1.2-py3-none-any.whl (1.8 MB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 1.8/1.8 MB 14.4 MB/s eta 0:00:00
Collecting setuptools
  Downloading setuptools-82.0.1-py3-none-any.whl (1.0 MB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 1.0/1.0 MB 10.4 MB/s eta 0:00:00
```

### Install Ansible and Kolla-Ansible

```bash
pip install "ansible-core>=2.13,<2.14"
pip install "kolla-ansible==15.*"
```

Verify installation:

```bash
(openstack-setup) root@node1:~# pip list | grep -i ansible
ansible-core       2.13.13
kolla-ansible      15.6.0

(openstack-setup) root@node1:~# pip list | grep -i setup
setuptools         82.0.1
```


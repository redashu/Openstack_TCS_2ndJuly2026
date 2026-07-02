## Openstack  3 layer service architecture 

<img src="3l.png">

### Openstack Installer and Deployment options 

<img src="installer.png">

## Adopting ansible + container based openstack component deployment

<img src="cont1.png">

## Checking prerequisite details for all the machines 

## 3 machines 

## check and install few common things 

### login as root user 

```
student@student:~$ sudo -i

root@student:~# whoami
root
root@student:~# 

```
### updating repo of Ubuntu machine 

```
apt  update 
Get:1 http://security.ubuntu.com/ubuntu jammy-security InRelease [129 kB]
Hit:2 http://archive.ubuntu.com/ubuntu jammy InRelease  
Get:3 http://security.ubuntu.com/ubuntu jammy-security/main amd64 Packages [3,351 kB]
Get:4 http://security.ubuntu.com/ubuntu jammy-security/main Translation-en [472 kB]
Get:5 http://security.ubuntu.com/ubuntu jammy-security/main amd64 c-n-f Metadata [14.6 kB]
Get:6 http://security.ubuntu.com/ubuntu jammy-security/restricted amd64 Packages [5,977 kB]
Get:7 http://archive.ubuntu.com/ubuntu jammy-updates InRelease [128 kB]       

```

### Installing tools 

```
apt  install  vim net-tools 
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done

```

### setting hostname permanently 

```
oot@student:~# hostname
student
root@student:~# hostnamectl set-hostname node1
root@student:~# exit
logout
student@student:~$ sudo -i
root@node1:~# hostname
node1
root@node1:~# 

```

### Node1 and Node2 are having 2 NIC 

```
root@node1:~# ifconfig
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

root@node1:~# ifconfig -a
ens18: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.0.19.76  netmask 255.0.0.0  broadcast 10.255.255.255
        inet6 fe80::be24:11ff:fef9:b735  prefixlen 64  scopeid 0x20<link>
        ether bc:24:11:f9:b7:35  txqueuelen 1000  (Ethernet)
        RX packets 36030  bytes 29234095 (29.2 MB)
        RX errors 0  dropped 4211  overruns 0  frame 0
        TX packets 2205  bytes 238926 (238.9 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

ens19: flags=4098<BROADCAST,MULTICAST>  mtu 1500
        ether bc:24:11:52:1f:3b  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536

```

### static IP details for vm1

```
oot@node1:~# cd  /etc/netplan/
root@node1:/etc/netplan# ls
01-netcfg.  01-netcfg.yaml  50-cloud-init.yaml
root@node1:/etc/netplan# cat 01-netcfg.yaml 
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
root@node1:/etc/netplan# 


===> change this file and test again 

root@node1:/etc/netplan# cat  01-netcfg.yaml 
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
root@node1:/etc/netplan# netplan  apply 
WARNING:root:Cannot call Open vSwitch: ovsdb-server.service is not running.
```
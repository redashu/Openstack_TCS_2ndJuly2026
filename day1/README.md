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

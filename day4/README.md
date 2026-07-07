# Day 4 Notes

## Recheck Installer Python Virtual Environment

### 1) Verify virtual environment layout

```bash
root@node1:~# ls
openstack-setup  snap

root@node1:~# ls openstack-setup/
bin  include  lib  lib64  pyvenv.cfg  share

root@node1:~# ls openstack-setup/bin/
activate       ansible-config      ansible-inventory  cinder     jsonpointer     kolla-writepwd  openstack-inventory    pip3         python3
activate.csh   ansible-connection  ansible-playbook   idna       kolla-ansible   markdown-it     oslo-config-generator  pip3.10      python3.10
activate.fish  ansible-console     ansible-pull       jp.py      kolla-genpwd    netaddr         oslo-config-validator  __pycache__  wheel
Activate.ps1   ansible-doc         ansible-test       jsondiff   kolla-mergepwd  normalizer      pbr                    pygmentize
ansible        ansible-galaxy      ansible-vault      jsonpatch  kolla-readpwd   openstack       pip                    python

root@node1:~# ls openstack-setup/share/
kolla-ansible

root@node1:~# ls openstack-setup/share/kolla-ansible/
ansible  doc  etc_examples  init-runonce  init-vpn  requirements.yml  setup.cfg  tools
```

### 2) Activate virtual environment

```bash
root@node1:~# source openstack-setup/bin/activate
(openstack-setup) root@node1:~#
```

### 3) Check installed Python packages

```bash
(openstack-setup) root@node1:~# pip list
Package                Version
---------------------- ---------
ansible-core           2.13.13
autopage               0.6.0
backports.strenum      1.3.1
certifi                2026.6.17
cffi                   2.0.0
charset-normalizer     3.4.7
cliff                  4.14.0
cmd2                   3.5.1
cryptography           49.0.0
debtcollector          3.1.0
decorator              5.3.1
dogpile.cache          1.5.0
hvac                   2.4.0
idna                   3.18
iso8601                2.1.0
Jinja2                 3.1.6
jmespath               1.1.0
jsonpatch              1.33
jsonpointer            3.1.1
keystoneauth1          5.14.0
kolla-ansible          15.6.0
markdown-it-py         4.2.0
MarkupSafe             3.0.3
mdurl                  0.1.2
msgpack                1.2.1
netaddr                1.3.0
openstacksdk           4.13.0
os-service-types       1.8.2
osc-lib                4.6.0
oslo.config            10.4.0
oslo.i18n              6.8.0
oslo.serialization     5.10.0
oslo.utils             10.1.1
packaging              26.2
pbr                    7.0.3
pip                    26.1.2
platformdirs           4.10.0
prettytable            3.18.0
psutil                 7.2.2
pycparser              3.0
Pygments               2.20.0
pyparsing              3.3.2
pyperclip              1.11.0
python-cinderclient    9.9.0
python-keystoneclient  5.8.0
python-openstackclient 10.0.0
PyYAML                 6.0.3
requests               2.34.2
resolvelib             0.8.1
rfc3986                2.0.0
rich                   15.0.0
rich-argparse          1.8.0
setuptools             82.0.1
stevedore              5.8.0
typing_extensions      4.15.0
urllib3                2.7.0
wcwidth                0.8.2
wheel                  0.47.0
wrapt                  2.2.2
```

### 4) Confirm ansible-core package details

```bash
(openstack-setup) root@node1:~# pip show ansible-core
Name: ansible-core
Version: 2.13.13
Summary: Radically simple IT automation
Home-page: https://ansible.com/
Author: Ansible, Inc.
Author-email: info@ansible.com
License: GPLv3+
Location: /root/openstack-setup/lib/python3.10/site-packages
Requires: cryptography, jinja2, packaging, PyYAML, resolvelib
Required-by:
```

## Give Docker Access to Non-Root User

### Issue observed

```bash
student@node1:~$ docker ps
permission denied while trying to connect to the docker API at unix:///var/run/docker.sock
```

### Commands executed

```bash
student@node1:~$ sudo -i
root@node1:~# usermod -aG docker student
root@node1:~# chmod 777 /var/run/docker.sock
root@node1:~# exit
logout
```

### Verification

```bash
student@node1:~$ docker version
Client: Docker Engine - Community
 Version:           29.6.1
 API version:       1.55
 Go version:        go1.26.4
 Git commit:        8900f1d
 Built:             Fri Jun 26 11:40:26 2026
 OS/Arch:           linux/amd64
 Context:           default

Server: Docker Engine - Community
 Engine:
  Version:          29.6.1
  API version:      1.55 (minimum version 1.40)
  Go version:       go1.26.4
  Git commit:       8ec5ab3
  Built:            Fri Jun 26 11:40:26 2026
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          v2.2.5
  GitCommit:        e53c7c1516c3b2bff98eb76f1f4117477e6f4e66
```

> Note: `chmod 777 /var/run/docker.sock` is not recommended for long-term security. Prefer adding the user to the `docker` group and re-logging (or rebooting) so group membership takes effect.


### recheck with openstack user login option 

```
tudent@node1:~$ sudo -i
root@node1:~# ls
openstack-setup  snap
root@node1:~# source  openstack-setup/bin/activate
(openstack-setup) root@node1:~# ls
openstack-setup  snap
(openstack-setup) root@node1:~# 
(openstack-setup) root@node1:~# openstack service list
Missing value auth-url required for auth plugin password
(openstack-setup) root@node1:~# ls  /etc/kolla/
admin-openrc.sh  globals.yml   horizon          keystone-ssh          multinode                  neutron-server       nova-novncproxy        placement-api
clouds.yaml      haproxy       jack-user.sh     kolla-toolbox         neutron-dhcp-agent         nova-api             nova-scheduler         rabbitmq
cron             heat-api      keepalived       mariadb               neutron-l3-agent           nova-api-bootstrap   openvswitch-db-server
fluentd          heat-api-cfn  keystone         mariadb-clustercheck  neutron-metadata-agent     nova-cell-bootstrap  openvswitch-vswitchd
glance-api       heat-engine   keystone-fernet  memcached             neutron-openvswitch-agent  nova-conductor       passwords.yml
(openstack-setup) root@node1:~# 
(openstack-setup) root@node1:~# 
(openstack-setup) root@node1:~# source  /etc/kolla/admin-openrc.sh  
(openstack-setup) root@node1:~# 
(openstack-setup) root@node1:~# openstack service list
+----------------------------------+-----------+----------------+
| ID                               | Name      | Type           |
+----------------------------------+-----------+----------------+
| 08b4431534be4f8587b13e8ac4c4a07d | placement | placement      |
| 4eef5e18b5b04baa8b46ff0709f4d048 | heat      | orchestration  |
| 885bb275217c4a5f83854f108b9fa820 | glance    | image          |
| 8b6d6e5ebb474ccc8bea2d8737512cdb | keystone  | identity       |
| 9fbbfe8249444183b8020ea6aa3a1249 | neutron   | network        |
| b231a7cfd2a9480d835abd1116672988 | nova      | compute        |
| ba0aa15283934cd6a1018938c148e24f | heat-cfn  | cloudformation |
+----------------------------------+-----------+----------------+
(openstack-setup) root@node1:~# openstack token issue -f json 
{
  "expires": "2026-07-08T04:00:54+0000",
  "id": "gAAAAABqTHn2m-sMyTrork3K7OGO-T-igYLvQiLcr0Xv-rAh8iqH488149mYy10oIWjgyxQK6EA3kkozOMQ6Mv3wHGcskPmqexvk3f0o9NahhI_sWtDRtWWzIFZyNEWXUC6i3p55jfOtQVYuyndlYAV05ing3jD859Xl4oJBkiL7BEuhakdDbpw",
  "project_id": "c5e84cfbf4e5432c92c200d87c8667db",
  "user_id": "985c845632814153935af665b7464997"
}
(openstack-setup) root@node1:~# 

```

### accessing horizon container details 

```
oot@node1:/etc/kolla/horizon# ls
config.json  custom_local_settings  horizon.conf  local_settings
root@node1:/etc/kolla/horizon# 
root@node1:/etc/kolla/horizon# docker  exec -it horizon  bash 
(horizon)[root@node1 /]# ls /etc/openstack-dashboard/
cinder_policy.yaml     default_policies    heat_policy.yaml      local_settings       nova_policy.d
custom_local_settings  glance_policy.yaml  keystone_policy.yaml  neutron_policy.yaml  nova_policy.yaml
(horizon)[root@node1 /]# 

```

### checking horizon container status 

```
root@node1:/etc/kolla/horizon# docker  ps  | grep -i horizon 
51eec1f9053a   quay.io/openstack.kolla/horizon:zed-rocky-9                     "dumb-init --single-…"   3 days ago   Up 4 hours (healthy)             horizon
root@node1:/etc/kolla/horizon# 
root@node1:/etc/kolla/horizon# 
root@node1:/etc/kolla/horizon# docker  stats horizon 

====>

oot@node1:/etc/kolla/horizon# docker  ps  | grep -i horizon 
51eec1f9053a   quay.io/openstack.kolla/horizon:zed-rocky-9                     "dumb-init --single-…"   3 days ago   Up 4 hours (healthy)             horizon
root@node1:/etc/kolla/horizon# 
root@node1:/etc/kolla/horizon# 
root@node1:/etc/kolla/horizon# docker  ps  | grep -i horizon 
51eec1f9053a   quay.io/openstack.kolla/horizon:zed-rocky-9                     "dumb-init --single-…"   3 days ago   Up 4 hours (healthy)             horizon
root@node1:/etc/kolla/horizon# 
root@node1:/etc/kolla/horizon# docker  stop horizon 
horizon
root@node1:/etc/kolla/horizon# 
root@node1:/etc/kolla/horizon# docker  ps  | grep -i horizon 
root@node1:/etc/kolla/horizon# docker ps
CONTAINER ID   IMAGE                                                           COMMAND                  CREATED      STATUS                 PORTS     NAMES
3c57c242c367   quay.io/openstack.kolla/heat-engine:zed-rocky-9                 "dumb-init --single-…"   3 days ago   Up 4 hours (healthy)             heat_engine
9edd84e4cf61   quay.io/openstack.kolla/heat-api-cfn:zed-rocky-9                "dumb-init --single-…"   3 days ago   Up 4 hours (healthy)             heat_api_cfn
root@node1:/etc/kolla/horizon# 
root@node1:/etc/kolla/horizon# 
root@node1:/etc/kolla/horizon# 
root@node1:/etc/kolla/horizon# openstack service list
Missing value auth-url required for auth plugin password
root@node1:/etc/kolla/horizon# 
root@node1:/etc/kolla/horizon# source  /etc/kolla/admin-openrc.sh 
root@node1:/etc/kolla/horizon# 
root@node1:/etc/kolla/horizon# openstack service list
+----------------------------------+-----------+----------------+
| ID                               | Name      | Type           |
+----------------------------------+-----------+----------------+
| 08b4431534be4f8587b13e8ac4c4a07d | placement | placement      |
| 4eef5e18b5b04baa8b46ff0709f4d048 | heat      | orchestration  |
| 885bb275217c4a5f83854f108b9fa820 | glance    | image          |
| 8b6d6e5ebb474ccc8bea2d8737512cdb | keystone  | identity       |
| 9fbbfe8249444183b8020ea6aa3a1249 | neutron   | network        |
| b231a7cfd2a9480d835abd1116672988 | nova      | compute        |
| ba0aa15283934cd6a1018938c148e24f | heat-cfn  | cloudformation |
+----------------------------------+-----------+----------------+
root@node1:/etc/kolla/horizon# openstack domain  list
+----------------------------------+------------------+---------+--------------------+
| ID                               | Name             | Enabled | Description        |
+----------------------------------+------------------+---------+--------------------+
| 55f971b99fe241ffaddb31c0f219d29d | training         | True    |                    |
| aead084565024688a8ae84b6f0d24356 | heat_user_domain | True    |                    |
| default                          | Default          | True    | The default domain |
+----------------------------------+------------------+---------+--------------------+
root@node1:/etc/kolla/horizon# docker  start horizon 
horizon
root@node1:/etc/kolla/horizon# docker ps  | grep -i hori
51eec1f9053a   quay.io/openstack.kolla/horizon:zed-rocky-9                     "dumb-init --single-…"   3 days ago   Up 5 seconds (health: starting)             horizon
root@node1:/etc/kolla/horizon# docker ps  | grep -i hori
51eec1f9053a   quay.io/openstack.kolla/horizon:zed-rocky-9                     "dumb-init --single-…"   3 days ago   Up 24 seconds (healthy)             horizon
root@node1:/etc/kolla/horizon# 
```

## Getting stared with Image management system with Glance 

<img src="glance1.png">

### glance flow 

<img src="glance2.png">

### checking glance service and EP 

```
oot@node1:/etc/kolla/horizon# openstack service list
+----------------------------------+-----------+----------------+
| ID                               | Name      | Type           |
+----------------------------------+-----------+----------------+
| 08b4431534be4f8587b13e8ac4c4a07d | placement | placement      |
| 4eef5e18b5b04baa8b46ff0709f4d048 | heat      | orchestration  |
| 885bb275217c4a5f83854f108b9fa820 | glance    | image          |
| 8b6d6e5ebb474ccc8bea2d8737512cdb | keystone  | identity       |
| 9fbbfe8249444183b8020ea6aa3a1249 | neutron   | network        |
| b231a7cfd2a9480d835abd1116672988 | nova      | compute        |
| ba0aa15283934cd6a1018938c148e24f | heat-cfn  | cloudformation |
+----------------------------------+-----------+----------------+
root@node1:/etc/kolla/horizon# openstack catalog list
+-----------+----------------+-----------------------------------------------------------------------+
| Name      | Type           | Endpoints                                                             |
+-----------+----------------+-----------------------------------------------------------------------+
| placement | placement      | RegionOne                                                             |
|           |                |   internal: http://10.0.39.1:8780                                     |
|           |                | RegionOne                                                             |
|           |                |   public: http://10.0.39.1:8780                                       |
|           |                |                                                                       |
| heat      | orchestration  | RegionOne                                                             |
|           |                |   internal: http://10.0.39.1:8004/v1/c5e84cfbf4e5432c92c200d87c8667db |
|           |                | RegionOne                                                             |
|           |                |   public: http://10.0.39.1:8004/v1/c5e84cfbf4e5432c92c200d87c8667db   |
|           |                |                                                                       |
| glance    | image          | RegionOne                                                             |
|           |                |   internal: http://10.0.39.1:9292                                     |
|           |                | RegionOne                                                             |
|           |                |   public: http://10.0.39.1:9292                                       |
|           |                |                                                  

```

### exploring internal image location inside docker container 

```
root@node1:/etc/kolla/horizon# docker ps  | grep -i glan
add760030436   quay.io/openstack.kolla/glance-api:zed-rocky-9                  "dumb-init --single-…"   3 days ago   Up 6 hours (healthy)             glance_api
root@node1:/etc/kolla/horizon# 
root@node1:/etc/kolla/horizon# 
root@node1:/etc/kolla/horizon# 
root@node1:/etc/kolla/horizon# docker exec -it glance_api  bash 
(glance-api)[glance@node1 /]$ cd /etc/
(glance-api)[glance@node1 /etc]$ ls
adjtime                 dnf           gshadow-     ld.so.cache    mime.types         passwd-         rocky-release-upstream  statetab.d          tmpfiles.d
aliases                 environment   gss          ld.so.conf     modprobe.d         pkcs11          rpc                     subgid              tpm2-tss
alternatives            ethertypes    host.conf    ld.so.conf.d   motd               pkgconfig       rpm                     subgid-             virc
bash_completion.d       exports       hostname     libaudit.conf  motd.d             pki             rwtab.d                 subuid              X11
bashrc                  filesystems   hosts        libibverbs.d   mtab               pm              sasl2                   subuid-             xattr.conf
bindresvport.blacklist  fonts         httpd        libnl          multipath          popt.d          securetty               sudo.conf           xdg
BUILDTIME               fstab.script  inittab      libreport      netconfig          printcap        security                sudoers             yum
ceph                    gcrypt        inputrc      libuser.conf   networks           profile         selinux                 sudoers.d           yum.conf
crypto-policies         glance        iproute2     locale.conf    nsswitch.conf      profile.d       services                sudo-ldap.conf      yum.repos.d
csh.cshrc               gnupg         iscsi        localtime      nsswitch.conf.bak  protocols       shadow                  sysconfig
csh.login               GREP_COLORS   issue        login.defs     openldap           rc.d            shadow-                 systemd
dbus-1                  groff         issue.d      logrotate.d    opt                rc.local        shells                  system-release
debuginfod              group         issue.net    lvm            os-release         redhat-release  skel                    system-release-cpe
default                 group-        krb5.conf    machine-id     pam.d              resolv.conf     ssh                     terminfo
depmod.d                gshadow       krb5.conf.d  mailcap        passwd             rocky-release   ssl                     timezone
(glance-api)[glance@node1 /etc]$ cd /var/lib/
(glance-api)[glance@node1 /var/lib]$ ls
alternatives  dnf  games  glance  httpd  iscsi  kolla  misc  private  rpm  rpm-state  selinux  systemd  tpm2-tss
(glance-api)[glance@node1 /var/lib]$ cd glance/
(glance-api)[glance@node1 /var/lib/glance]$ ls
images  staging  tasks_work_dir
(glance-api)[glance@node1 /var/lib/glance]$ cd images/
(glance-api)[glance@node1 /var/lib/glance/images]$ ls
(glance-api)[glance@node1 /var/lib/glance/images]$ 

```
### in kolla-installer location checking things 

```
oot@node1:/etc/kolla/glance-api# ls
config.json  glance-api.conf
root@node1:/etc/kolla/glance-api# cat glance-api.conf 
[DEFAULT]
debug = False
log_file = /var/log/kolla/glance/glance-api.log
use_forwarded_for = true
worker_self_reference_url = http://10.0.19.76:9292
bind_host = 10.0.19.76
bind_port = 9292
workers = 4
enabled_backends = file:file, http:http
cinder_catalog_info = volume:cinder:internalURL
transport_url = rabbit://openstack:Ddmr1rvTJZTa6D295QYDPpGPAr5Vm7EGRjPt4gvW@10.0.19.76:5672//

[database]
connection = mysql+pymysql://glance:S3b200qo2dQx5avWmAQmJv6tbhhKd4uNgJk465ql@10.0.39.1:3306/glance
connection_recycle_time = 10
max_pool_size = 1
max_retries = -1

[keystone_authtoken]
service_type = image
www_authenticate_uri = http://10.0.39.1:5000
auth_url = http://10.0.39.1:5000
auth_type = password
project_domain_id = default
user_domain_id = default
project_name = service
username = glance
password = Z6kBRAvCxSwkORvZuWcRXATcARwezqdqhj6nzraE
cafile =
region_name = RegionOne
memcache_security_strategy = ENCRYPT
memcache_secret_key = ZO6sHLVhKBNMvQ4Li2xN7Tv0FhJh5cZaluFmr1Eb
memcached_servers = 10.0.19.76:11211

[paste_deploy]
flavor = keystone

[glance_store]
default_backend = file

[file]
filesystem_store_datadir = /var/lib/glance/images/

```

### checking mariadb container for image entries

```
openstack-setup) root@node1:~# grep -iw database_password  /etc/kolla/passwords.yml 
database_password: 0B4tBYdkFRCe1VyP0ANp4ayGNt6vb0EMbIbo1f0k
(openstack-setup) root@node1:~# 
(openstack-setup) root@node1:~# 
(openstack-setup) root@node1:~# 
(openstack-setup) root@node1:~# docker  exec -it mariadb bash 
(mariadb)[mysql@node1 /]$ 
(mariadb)[mysql@node1 /]$ mysql -u root -p
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 33525
Server version: 10.6.18-MariaDB-log MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| glance             |
| heat               |
| information_schema |
| keystone           |
| mysql              |
| neutron            |
| nova               |
| nova_api           |
| nova_cell0         |
| performance_schema |
| placement          |
| sys                |
+--------------------+
12 rows in set (0.002 sec)

MariaDB [(none)]> use glance;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [glance]> show tables;
+----------------------------------+
| Tables_in_glance                 |
+----------------------------------+
| alembic_version                  |
| image_locations                  |
| image_members                    |
| image_properties                 |
| image_tags                       |
| images                           |
| metadef_namespace_resource_types |
| metadef_namespaces               |
| metadef_objects                  |
| metadef_properties               |
| metadef_resource_types           |
| metadef_tags                     |
| task_info                        |
| tasks                            |
+----------------------------------+
14 rows in set (0.001 sec)

MariaDB [glance]> desc images;
+------------------+-----------------------------------------------+------+-----+---------+-------+
| Field            | Type                                          | Null | Key | Default | Extra |
+------------------+-----------------------------------------------+------+-----+---------+-------+
| id               | varchar(36)                                   | NO   | PRI | NULL    |       |
| name             | varchar(255)                                  | YES  |     | NULL    |       |
| size             | bigint(20)                                    | YES  |     | NULL    |       |
| status           | varchar(30)                                   | NO   |     | NULL    |       |
| created_at       | datetime                                      | NO   | MUL | NULL    |       |
| updated_at       | datetime                                      | YES  | MUL | NULL    |       |
| deleted_at       | datetime                                      | YES  |     | NULL    |       |
| deleted          | tinyint(1)                                    | NO   | MUL | NULL    |       |
| disk_format      | varchar(20)                                   | YES  |     | NULL    |       |
| container_format | varchar(20)                                   | YES  |     | NULL    |       |
| checksum         | varchar(32)                                   | YES  | MUL | NULL    |       |
| owner            | varchar(255)                                  | YES  | MUL | NULL    |       |
| min_disk         | int(11)                                       | NO   |     | NULL    |       |
| min_ram          | int(11)                                       | NO   |     | NULL    |       |
| protected        | tinyint(1)                                    | NO   |     | 0       |       |
| virtual_size     | bigint(20)                                    | YES  |     | NULL    |       |
| visibility       | enum('private','public','shared','community') | NO   | MUL | shared  |       |
| os_hidden        | tinyint(1)                                    | NO   | MUL | 0       |       |
| os_hash_algo     | varchar(64)                                   | YES  |     | NULL    |       |
| os_hash_value    | varchar(128)                                  | YES  | MUL | NULL    |       |
+------------------+-----------------------------------------------+------+-----+---------+-------+
20 rows in set (0.001 sec)

MariaDB [glance]> select * from images;
Empty set (0.001 sec)

```

### history 

```
 grep -i database_password  /etc/kolla/passwords.yml 
  616  grep -iw database_password  /etc/kolla/passwords.yml 
  617  docker  exec -it mariadb bash 
  618  openstack user list
  619  openstack domain  list
  620  history   | grep -i ashu
  621  docker  exec -it mariadb bash 
  622  docker exec -it glance_api  bash 

```

### uploading image using openstack-cli 

```
openstack-setup) root@node1:~# openstack service list
+----------------------------------+-----------+----------------+
| ID                               | Name      | Type           |
+----------------------------------+-----------+----------------+
| 08b4431534be4f8587b13e8ac4c4a07d | placement | placement      |
| 4eef5e18b5b04baa8b46ff0709f4d048 | heat      | orchestration  |
| 885bb275217c4a5f83854f108b9fa820 | glance    | image          |
| 8b6d6e5ebb474ccc8bea2d8737512cdb | keystone  | identity       |
| 9fbbfe8249444183b8020ea6aa3a1249 | neutron   | network        |
| b231a7cfd2a9480d835abd1116672988 | nova      | compute        |
| ba0aa15283934cd6a1018938c148e24f | heat-cfn  | cloudformation |
+----------------------------------+-----------+----------------+
(openstack-setup) root@node1:~# openstack catalog list
+-----------+----------------+-----------------------------------------------------------------------+
| Name      | Type           | Endpoints                                                             |
+-----------+----------------+-----------------------------------------------------------------------+
| placement | placement      | RegionOne                                                             |
|           |                |   internal: http://10.0.39.1:8780                                     |
|           |                | RegionOne                                                             |
|           |                |   public: http://10.0.39.1:8780                                       |
|           |                |                                                                       |
(openstack-setup) root@node1:~# 
(openstack-setup) root@node1:~# 
(openstack-setup) root@node1:~# 
(openstack-setup) root@node1:~# 
(openstack-setup) root@node1:~# wget https://cloud-images.ubuntu.com/jammy/current/jammy-server-cloudimg-amd64-disk-kvm.img
--2026-07-07 06:49:44--  https://cloud-images.ubuntu.com/jammy/current/jammy-server-cloudimg-amd64-disk-kvm.img
Resolving cloud-images.ubuntu.com (cloud-images.ubuntu.com)... 185.125.190.40, 185.125.190.37, 2620:2d:4000:1::17, ...
Connecting to cloud-images.ubuntu.com (cloud-images.ubuntu.com)|185.125.190.40|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 665280000 (634M) [application/octet-stream]
Saving to: ‘jammy-server-cloudimg-amd64-disk-kvm.img’

jammy-server-cloudimg-amd64-disk-k 100%[===============================================================>] 634.46M  17.6MB/s    in 39s     

2026-07-07 06:50:24 (16.1 MB/s) - ‘jammy-server-cloudimg-amd64-disk-kvm.img’ saved [665280000/665280000]

(openstack-setup) root@node1:~# 
(openstack-setup) root@node1:~# ls
jammy-server-cloudimg-amd64-disk-kvm.img  openstack-setup  snap
(openstack-setup) root@node1:~# openstack  image create --disk-format raw --public --file jammy-server-cloudimg-amd64-disk-kvm.img                ashu-ubuntu-22image
+------------------+----------------------------------------------------------------------------------------------------------------------+
| Field            | Value                                                                                                                |
+------------------+----------------------------------------------------------------------------------------------------------------------+
| checksum         | f3a282f0b941771e1b5c45e69c20c14d                                                                                     |
| container_format | bare                                                                                                                 |
| created_at       | 2026-07-07T06:51:23Z                                                                                                 |
| disk_format      | raw                                                                                                                  |
| file             | /v2/images/b22af30f-52df-4633-8931-3cc5c4c7560a/file                                                                 |
| id               | b22af30f-52df-4633-8931-3cc5c4c7560a                                                                                 |
| min_disk         | 0                                                                                                                    |
| min_ram          | 0                                                                                                                    |
| name             | ashu-ubuntu-22image                                                                                                  |
| owner            | c5e84cfbf4e5432c92c200d87c8667db                                                                                     |
| properties       | os_hash_algo='sha512', os_hash_value='e798716313039b307521cb7caeffde65945cc24ed2b5895170c08a865bdb8d595198965c5bd733 |
|                  | 1a6688da96da75ac33ab6fac74a1a3f8e5380c92a9f52660fe', os_hidden='False', owner_specified.openstack.md5='',            |
|                  | owner_specified.openstack.object='images/ashu-ubuntu-22image', owner_specified.openstack.sha256='', stores='file'    |
| protected        | False                                                                                                                |
| schema           | /v2/schemas/image                                                                                                    |
| size             | 665280000                                                                                                            |
| status           | active                                                                                                               |
| tags             |                                                                                                                      |
| updated_at       | 2026-07-07T06:51:26Z                                                                                                 |
| virtual_size     | 665280000                                                                                                            |
| visibility       | public                                                                                                               |
+------------------+----------------------------------------------------------------------------------------------------------------------+
(openstack-setup) root@node1:~# openstack image list
+--------------------------------------+---------------------+--------+
| ID                                   | Name                | Status |
+--------------------------------------+---------------------+--------+
| 78a6f549-ea4a-4b58-bc63-4c03c3818eca | ashu-cirros-os      | active |
| b22af30f-52df-4633-8931-3cc5c4c7560a | ashu-ubuntu-22image | active |
+--------------------------------------+---------------------+--------+
(openstack-setup) root@node1:~# 

```

## Neutron concpet and understanding with container 

<img src="net1.png">

### checking some details about  Neutron bridge

```
openstack-setup) root@node1:/etc/kolla# docker  exec -it  openvswitch_vswitchd  bash 
(openvswitch-vswitchd)[root@node1 /]# ovs-vsctl  show 
e489dd9d-13c8-452c-a8d9-649b11547536
    Manager "ptcp:6640:127.0.0.1"
        is_connected: true
    Bridge br-int
        Controller "tcp:127.0.0.1:6633"
            is_connected: true
(openvswitch-vswitchd)[root@node1 /]# 
(openvswitch-vswitchd)[root@node1 /]# 
(openvswitch-vswitchd)[root@node1 /]# 
(openvswitch-vswitchd)[root@node1 /]# ovs-vsctl  list-br 
br-ex
br-int
br-tun
(openvswitch-vswitchd)[root@node1 /]# ovs-vsctl  list-ports  br-int
int-br-ex
patch-tun
(openvswitch-vswitchd)[root@node1 /]# ovs-vsctl  list-ports  br-tun
patch-int
(openvswitch-vswitchd)[root@node1 /]# ovs-vsctl  list-ports  br-ex
ens19
phy-br-ex

```

### in external flat network type the provider name is  physnet1 only 

```
oot@node1:~# docker ps  | grep -i openvsw
c3405ac2ebbe   quay.io/openstack.kolla/neutron-openvswitch-agent:zed-rocky-9   "dumb-init --single-…"   4 days ago   Up 9 hours (healthy)             neutron_openvswitch_agent
dbeadb1d241f   quay.io/openstack.kolla/openvswitch-vswitchd:zed-rocky-9        "dumb-init --single-…"   4 days ago   Up 9 hours (healthy)             openvswitch_vswitchd
9d630fce627e   quay.io/openstack.kolla/openvswitch-db-server:zed-rocky-9       "dumb-init --single-…"   4 days ago   Up 9 hours (healthy)             openvswitch_db
root@node1:~# docker  exec -it neutron_openvswitch_agent bash 
(neutron-openvswitch-agent)[neutron@node1 /]$ 
(neutron-openvswitch-agent)[neutron@node1 /]$ 
(neutron-openvswitch-agent)[neutron@node1 /]$ cd /etc/neutron/
(neutron-openvswitch-agent)[neutron@node1 /etc/neutron]$ ls
api-paste.ini  oslo-config-generator  plugins                 README.txt     rootwrap.d
neutron.conf   oslo-policy-generator  README.policy.yaml.txt  rootwrap.conf
(neutron-openvswitch-agent)[neutron@node1 /etc/neutron]$ cd plugins/
(neutron-openvswitch-agent)[neutron@node1 /etc/neutron/plugins]$ ls
ml2  neutron
(neutron-openvswitch-agent)[neutron@node1 /etc/neutron/plugins]$ cd ml2/
(neutron-openvswitch-agent)[neutron@node1 /etc/neutron/plugins/ml2]$ ls
openvswitch_agent.ini
(neutron-openvswitch-agent)[neutron@node1 /etc/neutron/plugins/ml2]$ cat openvswitch_agent.ini 
[agent]
tunnel_types = vxlan
l2_population = true
arp_responder = true
enable_distributed_routing = True

[securitygroup]
firewall_driver = neutron.agent.linux.iptables_firewall.OVSHybridIptablesFirewallDriver

[ovs]
bridge_mappings = physnet1:br-ex
datapath_type = system
ovsdb_connection = tcp:127.0.0.1:6640
ovsdb_timeout = 10
local_ip = 10.0.19.76
```


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
## Revision process

<img src="rev1.png">

### Understanding the logs location for openstack services

```
openstack-setup) root@node1:/etc/kolla# cd  /var/log/
(openstack-setup) root@node1:/var/log# ls
alternatives.log       auth.log.1     chrony                 dmesg.0     dpkg.log       journal        landscape  syslog.2.gz
alternatives.log.1     auth.log.2.gz  cloud-init.log         dmesg.1.gz  dpkg.log.1     kern.log       lastlog    ubuntu-advantage.log
alternatives.log.2.gz  bootstrap.log  cloud-init-output.log  dmesg.2.gz  dpkg.log.2.gz  kern.log.1     private    unattended-upgrades
apt                    btmp           dist-upgrade           dmesg.3.gz  faillog        kern.log.2.gz  syslog     wtmp
auth.log               btmp.1         dmesg                  dmesg.4.gz  installer      kolla          syslog.1
(openstack-setup) root@node1:/var/log# cd kolla
(openstack-setup) root@node1:/var/log/kolla# ls
ansible.log  cinder  fluentd  glance  haproxy  heat  horizon  keystone  mariadb  neutron  nova  openvswitch  placement  rabbitmq
(openstack-setup) root@node1:/var/log/kolla# 
(openstack-setup) root@node1:/var/log/kolla# 
(openstack-setup) root@node1:/var/log/kolla# ls  glance/
glance-api.log
(openstack-setup) root@node1:/var/log/kolla# ls  neutron/
dnsmasq.log             neutron-l3-agent.log        neutron-openvswitch-agent.log  privsep-helper.log
neutron-dhcp-agent.log  neutron-metadata-agent.log  neutron-server.log
(openstack-setup) root@node1:/var/log/kolla# ls  nova/
apache-access.log  nova-api-access.log  nova-api.log        nova-manage.log           nova-metadata-error.log  nova-scheduler.log
apache-error.log   nova-api-error.log   nova-conductor.log  nova-metadata-access.log  nova-novncproxy.log
(openstack-setup) root@node1:/var/log/kolla# 

```

## the mindset of troubleshooting VM related issues 

<img src="vmtr1.png">


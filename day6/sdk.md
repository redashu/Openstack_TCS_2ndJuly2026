# OpenStack SDK Architecture

## Overview

```
Python Application
        │
        ▼
OpenStack SDK (openstacksdk)
        │
        ▼
REST API (HTTPS)
        │
        ▼
Keystone Authentication
        │
        ▼
-----------------------------------
| Nova | Neutron | Glance | Cinder | Swift |
-----------------------------------
```

## SDK Components

The OpenStack SDK provides a unified interface to interact with:

- **Nova** - Compute services
- **Neutron** - Networking services
- **Glance** - Image services
- **Cinder** - Block storage services
- **Swift** - Object storage services

## Authentication

All requests are authenticated through **Keystone**, the OpenStack Identity service, ensuring secure access to all OpenStack services.

### validating python based option to connection openstack 

```
openstack-setup) root@node1:~# pip list  | grep -i openstack 
openstacksdk           4.13.0
python-openstackclient 10.0.0
(openstack-setup) root@node1:~# 
(openstack-setup) root@node1:~# 
(openstack-setup) root@node1:~# python3
Python 3.10.12 (main, Jun 22 2026, 18:55:27) [GCC 11.4.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> 
>>> import openstack 
>>> dir(openstack)
['Any', '__all__', '__builtins__', '__cached__', '__doc__', '__file__', '__loader__', '__name__', '__package__', '__path__', '__spec__', '_log', '_services_mixin', 'accelerator', 'argparse', 'baremetal', 'baremetal_introspection', 'block_storage', 'cloud', 'clustering', 'common', 'compute', 'config', 'connect', 'connection', 'container_infrastructure_management', 'database', 'dns', 'enable_logging', 'exceptions', 'fields', 'format', 'identity', 'image', 'instance_ha', 'key_manager', 'load_balancer', 'message', 'network', 'object_store', 'openstack', 'orchestration', 'placement', 'proxy', 'resource', 'service_description', 'shared_file_system', 'types', 'utils', 'version', 'warnings', 'workflow']
>>> 

```
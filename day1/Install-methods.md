# OpenStack Deployment Models and Installer Methods

When preparing for an OpenStack interview, it helps to group deployment options into three categories:

- Learning / Development
- POC / Lab
- Production Enterprise Deployment

The OpenStack community supports multiple deployment frameworks, and organizations choose installers based on scale, operations, and lifecycle requirements.

## 1. Single Node Deployment (All-in-One)

Everything runs on one server.

```text
+------------------------------------------------+
|                 Single Server                  |
|------------------------------------------------|
| Keystone                                       |
| Nova API                                       |
| Glance                                         |
| Neutron                                        |
| Cinder                                         |
| Horizon                                        |
| RabbitMQ                                       |
| MariaDB                                        |
| Compute (nova-compute)                         |
| KVM                                            |
+------------------------------------------------+
```

### Used For

- Learning
- Home Lab
- Development
- API Testing
- CI/CD

### Advantages

- Easy
- Low hardware requirements
- Fast deployment

### Disadvantages

- No HA
- No scalability
- Single point of failure

## 2. Multi Node Deployment

Most enterprise deployments separate roles.

```text
                +----------------+
                |  Load Balancer |
                +-------+--------+
                        |
      +-----------------+-----------------+
      |                                   |
+-------------+                   +-------------+
| Controller1 |                   | Controller2 |
+-------------+                   +-------------+
      |
      |
+-------------+
| Controller3 |
+-------------+

        |
        |
---------------- Management Network ----------------

+-------------+    +-------------+    +-------------+
| Compute-01  |    | Compute-02  |    | Compute-03  |
+-------------+    +-------------+    +-------------+

        |
        |

+-------------+
| Storage     |
| Ceph        |
+-------------+
```

### Typical Roles

- Controller Nodes
- Compute Nodes
- Storage Nodes
- Network Nodes (optional)
- Monitoring Node
- Deployment Node

This is the most common production architecture.

## 3. Highly Available (HA) OpenStack

Enterprise deployments typically include:

- 3 Controller Nodes
- HAProxy
- Keepalived (Virtual IP)
- MariaDB Galera Cluster
- RabbitMQ Cluster
- Multiple Compute Nodes
- Ceph Storage Cluster

```text
VIP
 |
HAProxy
 |
--------------------------------
|        |          |
Ctrl1   Ctrl2     Ctrl3

Galera Cluster

RabbitMQ Cluster

Ceph Cluster

Multiple Compute Nodes
```

### Benefits

- No single point of failure
- Rolling upgrades
- Automatic failover
- High availability

## Popular OpenStack Installers

## 1. DevStack ⭐

### Purpose

Development only.

### Installation Method

Shell scripts.

### Deploys

Single node.

### Best For

- Developers
- OpenStack contributors
- Students

### Advantages

- Very easy
- Latest code
- Fast installation

### Disadvantages

- Not production-ready
- No HA
- No upgrade path

### Interview Answer

DevStack is intended for OpenStack development and testing environments and is not recommended for production deployments.

## 2. PackStack

### Purpose

POC / Lab.

### Installation

Puppet-based installer.

### Supports

Mostly RHEL/CentOS ecosystems.

### Deploys

Mostly single node.

### Advantages

- Easy
- Quick
- GUI installer options

### Disadvantages

- Limited scalability
- Not recommended for large production deployments

### Best For

- Proof of Concept
- Training labs

## 3. Kolla-Ansible ⭐⭐⭐⭐⭐ (Most Popular)

### Purpose

Production.

### Deployment Model

- Docker containers
- Ansible automation

### Architecture (Containerized Services)

```text
+----------------------+
| Nova Container       |
+----------------------+

+----------------------+
| Neutron Container    |
+----------------------+

+----------------------+
| Keystone Container   |
+----------------------+

+----------------------+
| Glance Container     |
+----------------------+
```

### Advantages

- Containerized services
- Easy upgrades
- HA support
- Multi-node support
- Production-ready
- Large community adoption

### Typical Deployment Commands

```bash
kolla-ansible bootstrap-servers
kolla-ansible prechecks
kolla-ansible deploy
kolla-ansible post-deploy
```

### Best For

- Enterprise
- Telco
- Private Cloud
- Large deployments

This is currently one of the most widely recommended official deployment methods for production OpenStack.

## 4. OpenStack-Ansible

### Purpose

Production.

### Uses

- Ansible
- LXC containers or bare metal

### Advantages

- Highly customizable
- Mature
- Flexible networking

### Disadvantages

- Steeper learning curve
- More operational complexity than Kolla-Ansible

### Best For

- Operators needing deep customization

## 5. OpenStack Charms (Canonical)

### Uses

- Juju
- Charms
- MAAS integration

### Supports

- HA
- Day-2 operations
- Upgrades
- Scaling

### Best For

- Ubuntu-based enterprise deployments

## 6. MicroStack

### Purpose

- Developer workstation or edge deployments

### Uses

- Snap packages

### Deploys

- Single node
- Small clusters (newer releases)

### Advantages

- Very quick setup
- Great for demos and labs

### Note

- Not intended for large enterprise production clouds

## 7. Manual Installation

Install every service individually:

MariaDB
RabbitMQ
Keystone
Glance
Nova
Neutron
Cinder
Horizon

Pros

Learn every component
Complete control

Cons

Time-consuming
Error-prone
Rarely used in production today
Commercial OpenStack Distributions

Many enterprises use supported distributions rather than installing upstream OpenStack directly:

Canonical Charmed OpenStack
Mirantis OpenStack
Red Hat OpenStack Platform (legacy/lifecycle varies by version)

These provide enterprise support, tested lifecycle management, and integrated tooling.

Which installer should you choose?
Installer	Single Node	Multi Node	HA	Production	Typical Use
DevStack	✅	❌	❌	❌	Development
PackStack	✅	Limited	❌	⚠️ Small POC	Lab / Demo
MicroStack	✅	Small	Limited	⚠️ Edge	Learning
Kolla-Ansible	✅	✅	✅	✅	Enterprise (most common)
OpenStack-Ansible	✅	✅	✅	✅	Enterprise with high customization
OpenStack Charms	✅	✅	✅	✅	Ubuntu enterprise deployments
Manual Install	✅	✅	Possible	Rare	Learning / Custom environments
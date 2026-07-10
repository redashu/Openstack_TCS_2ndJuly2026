# OpenStack High Availability (HA)

## Why Do We Need HA?

Imagine your OpenStack environment request path:

```text
Users
 ↓
Horizon
 ↓
Keystone
 ↓
Nova
 ↓
Neutron
 ↓
Glance
 ↓
Cinder
```

Now imagine `Controller1` crashes.

Without HA:

- Horizon unavailable
- Authentication fails
- VM creation fails
- Networking operations fail
- APIs unavailable

Even though all VMs continue running on compute nodes, your cloud is effectively down because the control plane is unavailable.

High Availability ensures that the control plane remains available even if one or more controller nodes fail.

## Single Controller vs HA Controller

### Single Controller

```text
                 Users
                    │
              +-----------+
              |Controller1|
              +-----------+
                    │
      -----------------------------
      | Nova  Neutron  Keystone |
      | RabbitMQ MariaDB Horizon|
      -----------------------------
```

Problem:

If `Controller1` crashes:

```text
Users
  ×
Cloud Down
```

### HA Architecture

```text
                    VIP
              192.168.1.100
                    │
        --------------------------
        │                        │
   Controller1              Controller2
        │                        │
        └────── Controller3 ─────┘

     Compute1      Compute2
```

Now one controller can fail and users continue using the Virtual IP (VIP).

## Typical Production Architecture

```text
                    VIP
                     │
               Keepalived
                     │
               HAProxy Cluster
         ┌──────────┴──────────┐

 Controller1   Controller2   Controller3
         │          │            │
---------------------------------------------
MariaDB Galera Cluster
RabbitMQ Cluster
Keystone
Nova API
Neutron API
Glance API
Cinder API
```

Notice:

- Every controller runs the API services.

## Components

## 1. Controller HA Architecture

### What Runs on Controllers?

Typically:

- Keystone
- Nova API
- Neutron Server
- Glance API
- Cinder API
- Horizon
- RabbitMQ
- MariaDB
- Memcached
- HAProxy
- Keepalived

All controllers provide identical API endpoints.

### Without HA

```text
Controller1
↓
API
```

### With HA

```text
Controller1 API
Controller2 API
Controller3 API
```

All serve requests.

### How Users Connect

Instead of:

- `http://controller1:5000`

Users connect to:

- `http://192.168.1.100:5000`

which is the VIP.

### Why 3 Controllers?

Because most HA components require quorum.

Example:

1. **1 Controller**: No HA
2. **2 Controllers**: Split-brain risk
3. **3 Controllers**: Majority voting

This is why production deployments almost always recommend three controllers.

## 2. Galera Cluster

### What Is Galera?

Galera is a synchronous multi-master MySQL cluster.

OpenStack stores almost everything in databases.

Examples:

- **Nova**
  - VM metadata
  - Flavor
  - Migration
  - Instances
- **Neutron**
  - Networks
  - Routers
  - Ports
  - Security Groups
- **Keystone**
  - Users
  - Projects
  - Roles
  - Tokens

All of these are stored in MariaDB.

### Without Cluster

```text
MariaDB
↓
Controller1
Failure
↓
Everything stops.
```

### With Galera

```text
Controller1 (MariaDB)
      │
Controller2 (MariaDB)
      │
Controller3 (MariaDB)
```

Every node has the same data.

### Multi-Master

Unlike traditional MySQL:

```text
Master
↓
Slave
```

Galera allows:

```text
Write
↓
Any Node
```

Every write is replicated synchronously.

### Why Synchronous?

Traditional replication:

```text
Insert
↓
Master
↓
Later
↓
Replica
```

Possible data loss.

Galera:

```text
Insert
↓
Controller1
↓
Controller2
↓
Controller3
↓
Success
```

No committed transaction is acknowledged until the cluster agrees, which greatly reduces the risk of data divergence.

### Quorum

Suppose:

- Controller1
- Controller2
- Controller3

If `Controller3` dies, remaining nodes still have majority and cluster continues.

If `Controller1` and `Controller2` die, only one node remains and Galera refuses writes.

Reason:

- Prevent split-brain.

## 3. RabbitMQ Cluster

RabbitMQ is the message broker.

OpenStack services communicate through RabbitMQ.

Example:

```text
Nova API
↓
RabbitMQ
↓
Nova Scheduler
↓
Nova Compute
```

VM creation flow:

```text
User
↓
Nova API
↓
RabbitMQ
↓
Scheduler
↓
Compute
```

RabbitMQ carries all those messages.

### Without RabbitMQ Cluster

```text
RabbitMQ
↓
Controller1
```

If it dies, Nova cannot communicate.

### With Cluster

```text
RabbitMQ
 Controller1
 Controller2
 Controller3
```

Each node participates in the cluster.

### Message Example

```text
Create VM
↓
RabbitMQ
↓
Scheduler
↓
Compute
```

### Queue Mirroring / Quorum Queues

If `Controller1` dies, messages still exist on `Controller2` and `Controller3`.

Modern RabbitMQ deployments generally recommend quorum queues instead of the older mirrored queues.

## 4. HAProxy

HAProxy is the load balancer.

Instead of users connecting directly to controllers, users connect to HAProxy.

HAProxy distributes requests.

Example:

```text
API Request
↓
HAProxy
↓
Controller2
```

Next request:

```text
↓
Controller3
```

Round-robin example:

```text
Request1 → Controller1
Request2 → Controller2
Request3 → Controller3
```

### Health Checks

Suppose `Controller2` dies.

HAProxy detects it and traffic automatically goes to:

- Controller1
- Controller3

No manual intervention.

## 5. Keepalived

Question:

How do users reach HAProxy?

Via a Virtual IP (VIP).

Example:

- Controller1: `10.0.0.11`
- Controller2: `10.0.0.12`
- Controller3: `10.0.0.13`
- VIP: `10.0.0.10`

Users always connect to:

- `10.0.0.10`

Keepalived manages that VIP.

### Normal Operation

```text
VIP
↓
Controller1
```

`Controller1` is MASTER.

If `Controller1` crashes, Keepalived immediately moves the VIP:

```text
VIP
↓
Controller2
```

Users do not need to change URLs.

### VRRP

Keepalived uses VRRP (Virtual Router Redundancy Protocol) to elect the MASTER node.

### Failover Example

```text
MASTER (Controller1)
↓
Failure
↓
Controller2 becomes MASTER
↓
VIP moves automatically
```

## Complete Request Flow

```text
User
↓
VIP (Keepalived)
↓
HAProxy
↓
Keystone API
↓
RabbitMQ
↓
Nova Scheduler
↓
Nova Compute
↓
Galera Database
↓
VM Created
```

## What Happens if a Controller Fails?

### Controller1 Crash

- **Keepalived**: Detects failure and moves VIP to another controller.
- **HAProxy**: Stops forwarding traffic to `Controller1`; sends requests only to healthy controllers.
- **Galera**: Continues with quorum (assuming at least two controllers remain).
- **RabbitMQ**: Continues serving messages through remaining cluster nodes.

### User Impact

- Existing VMs keep running.
- API availability continues.
- New VM creation and other control-plane operations continue with little or no interruption.

## Responsibilities Summary

| Component | Purpose | Why Required |
| --- | --- | --- |
| Keepalived | Provides Virtual IP (VIP) failover | Users always connect to one stable IP |
| HAProxy | Load balances API requests | Distributes traffic and removes failed controllers |
| Galera Cluster | Highly available MariaDB database | Keeps OpenStack databases synchronized across controllers |
| RabbitMQ Cluster | Message broker cluster | Ensures reliable inter-service communication |
| Controller HA | Multiple controller nodes | Eliminates single points of failure in the control plane |

## Typical Production HA Deployment

```text
                     Users
                       │
                 10.0.0.100 (VIP)
                       │
                 Keepalived (VRRP)
                       │
                 HAProxy Cluster
                       │
      ┌────────────┬────────────┬────────────┐
      │            │            │
 Controller1   Controller2   Controller3
      │            │            │
      ├────────────┼────────────┤
      │   Galera (MariaDB)      │
      │   RabbitMQ Cluster      │
      │   Keystone              │
      │   Nova API              │
      │   Neutron Server        │
      │   Glance API            │
      │   Cinder API            │
      └────────────┼────────────┘
                   │
        ┌──────────┴──────────┐
        │                     │
   Compute1              Compute2
```

## Key Takeaway

Think of the HA stack as four layers working together:

- **Keepalived** answers: "Which controller owns the cloud's IP address?"
- **HAProxy** answers: "Which healthy controller should receive this API request?"
- **RabbitMQ** answers: "How do OpenStack services reliably communicate with each other?"
- **Galera** answers: "How do all controllers share the same database without losing data?"

Together, they ensure that the OpenStack control plane remains available even when an individual controller node fails, while compute nodes continue running tenant workloads uninterrupted.
# Hystax Acura Migration Workflow (VMware → OpenStack)

Hystax Acura is an enterprise migration platform that automates VMware to OpenStack, Bare Metal to OpenStack, Cloud-to-Cloud, and OpenStack-to-OpenStack migrations. Its key capabilities include continuous replication, orchestration, test migrations, and a final cutover with minimal downtime.

## High-Level Architecture

```text
                 Source Datacenter
          +----------------------------+
          | VMware vCenter / ESXi      |
          | Windows/Linux VMs          |
          +-------------+--------------+
                        |
            Replication Agent (CBT)
                        |
            Continuous Replication
                        |
==================== WAN =====================
                        |
                Hystax Acura Server
         (Migration Orchestration Engine)
                        |
            OpenStack APIs (Keystone)
                        |
        +---------------+---------------+
        |      OpenStack Cloud          |
        | Glance  Nova  Neutron Cinder  |
        +---------------+---------------+
                        |
                    Migrated VMs
```

## Complete Migration Workflow

```text
Assessment
      │
      ▼
Deploy Hystax
      │
      ▼
Install Replication Agents
      │
      ▼
Initial Full Replication
      │
      ▼
Continuous Incremental Sync
      │
      ▼
Migration Plan Creation
      │
      ▼
Test Migration
      │
      ▼
Application Validation
      │
      ▼
Final Synchronization
      │
      ▼
Cutover
      │
      ▼
Production on OpenStack
```

This aligns with Hystax's documented five-step process: **Start replication** → **Store data** → **Automate orchestration** → **Test migration** → **Final cutover**.

## Phase 1 – Discovery & Assessment

Before migrating, collect information about:

### VMware Inventory

- vCenter
- ESXi hosts
- Datastores
- Port Groups
- VLANs
- Resource Pools
- Clusters

### Virtual Machines

- CPU
- RAM
- OS
- IP addresses
- Disk size
- Applications
- Dependencies

### Example

```text
VM1
Windows
4 vCPU
8 GB RAM
100 GB Disk

↓

Need OpenStack:

Flavor
Image
Volume
Network
```

## Phase 2 – Deploy Hystax Acura

Deploy the Hystax management appliance.

### Connections

- VMware vCenter
- OpenStack APIs
- Replication agents
- Target cloud

### Responsibilities

- Inventory collection
- Replication management
- Migration orchestration
- Scheduling
- Reporting

## Phase 3 – Install Replication Agents

There are two common approaches:

### VMware Agent

- Installed once per ESXi host
- Agentless for guest VMs using VMware APIs/CBT

### Windows/Linux Agents

- Used for physical servers or supported workloads
- Continuously send changed blocks to the target cloud

## Phase 4 – Initial Replication

First synchronization copies the complete VM.

```text
VMware VM
100 GB

↓

OpenStack Storage
100 GB
```

- This is the longest step
- Production VMs continue running during replication

## Phase 5 – Incremental Replication

After the initial copy, only changed blocks are transferred:

```text
Changed Blocks Only

↓

5 MB

↓

20 MB

↓

50 MB
```

Instead of copying 100 GB again, only the deltas are sent.

### Benefits

- Lower bandwidth
- Faster synchronization
- Minimal production impact

## Phase 6 – Store Data

Replicated data is stored on the OpenStack side using:

- Cinder Volumes
- Snapshots
- Temporary migration storage

No VM is started yet. The target environment simply maintains an up-to-date copy of the source VM.

## Phase 7 – Create Migration Plan

Migration plans define:

- VM order
- Boot sequence
- Network mapping
- Storage mapping
- Startup delays
- Dependencies

### Migration Order Example

```text
Database
↓
Application
↓
Web Server
```

This ensures dependent services start in the correct order.

## Phase 8 – Network Mapping

One of the most important tasks. Maps VMware networking to OpenStack Neutron.

| VMware      | OpenStack           |
|-------------|---------------------|
| VLAN 100    | Provider Network    |
| VLAN 200    | VXLAN Network       |
| Port Group  | Neutron Network     |
| vSwitch     | Open vSwitch / OVN  |
| NSX Segment | Neutron Network     |

## Phase 9 – Storage Mapping

| VMware     | OpenStack               |
|------------|-------------------------|
| VMDK       | Cinder Volume           |
| Datastore  | Cinder Backend          |
| VMFS       | LVM / Ceph              |
| Thick Disk | Thick Volume            |
| Thin Disk  | Thin Provisioned Volume |

## Phase 10 – Compute Mapping

| VMware          | OpenStack         |
|-----------------|-------------------|
| Cluster         | Availability Zone |
| ESXi Host       | Nova Compute      |
| Resource Pool   | Project Quota     |
| VM              | Nova Instance     |

## Phase 11 – Test Migration

One of Hystax's strongest features is the ability to run non-disruptive test migrations.

```text
Production VM
↓
Snapshot
↓
Temporary VM
↓
Testing
↓
Delete
```

You can verify:

- Boot success
- Application startup
- Database connectivity
- Network connectivity
- Performance

Production continues running during testing.

## Phase 12 – Validation

Typical validation includes:

### Infrastructure

- VM boots
- CPU
- RAM
- Disk
- Network connectivity (Ping, DNS, Gateway, Internet)

### Applications

- IIS
- Apache
- Tomcat
- SQL Server
- Oracle
- MySQL

## Phase 13 – Final Synchronization

Before cutover:

```text
Production
↓
Last Incremental Sync
↓
Target Updated
```

Only the most recent changed blocks are copied, minimizing downtime.

## Phase 14 – Final Cutover

```text
Shutdown VMware VM
↓
Final Sync
↓
Launch OpenStack VM
↓
Update DNS
↓
Application Live
```

Downtime is typically limited to the maintenance window required for the last synchronization and startup.

## Phase 15 – Post Migration

Validate:

- Application functionality
- User access
- Network
- Storage
- Security Groups
- Floating IPs
- Backups
- Monitoring

## Workflow Diagram

```text
VMware
   │
   ▼
Deploy Hystax
   │
   ▼
Install Agent
   │
   ▼
Initial Replication
   │
   ▼
Incremental Replication
   │
   ▼
Migration Plan
   │
   ▼
Test Migration
   │
   ▼
Validation
   │
   ▼
Final Sync
   │
   ▼
Cutover
   │
   ▼
OpenStack Production
```

## Advantages

- Minimal downtime
- Continuous replication
- Automated orchestration
- Multiple test migrations
- Rollback capability before final cutover
- Network and storage mapping
- VMware API integration
- Large-scale migration support (thousands of VMs)

## VMware → OpenStack Resource Mapping

| VMware              | OpenStack                      |
|---------------------|--------------------------------|
| vCenter             | Keystone + Nova                |
| ESXi                | Nova Compute                   |
| VM                  | Instance                       |
| VMDK                | Cinder Volume                  |
| VMFS Datastore      | Cinder Backend (LVM/Ceph)      |
| vSwitch / Port Group| Neutron Network                |
| VLAN                | Provider Network               |
| Snapshot            | Cinder Snapshot                |
| Resource Pool       | Project / Flavor / Quotas      |
| HA Cluster          | Availability Zone + Nova Scheduler |

## End-to-End Migration Summary

```text
Assessment
      │
      ▼
Deploy Hystax
      │
      ▼
Discover VMware Inventory
      │
      ▼
Continuous Replication
      │
      ▼
Store Replicas in OpenStack
      │
      ▼
Create Migration Plan
      │
      ▼
Map Compute, Storage & Network
      │
      ▼
Run Test Migration
      │
      ▼
Validate Applications
      │
      ▼
Final Incremental Sync
      │
      ▼
Shutdown Source VM
      │
      ▼
Launch VM on OpenStack
      │
      ▼
Production Cutover
```

## Summary

This workflow is suitable for enterprise VMware-to-OpenStack migrations because it:

- Minimizes downtime through continuous replication
- Supports repeated testing before cutover
- Automates orchestration of compute, storage, and networking in the target OpenStack environment
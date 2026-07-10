# Heat Demo 1 – Create a Single Network

## Objective

Create a single Neutron network using a Heat template.

## Architecture

```
Heat Stack
    │
    ▼
training-net
```

## Template: network.yaml

```yaml
heat_template_version: 2021-04-16

description: Create a simple private network

resources:

  training_network:
    type: OS::Neutron::Net
    properties:
      name: training-net
```

## Deploy the Stack

```bash
source /etc/kolla/admin-openrc.sh

openstack stack create \
-t network.yaml \
network-demo
```

Heat creates a Stack named `network-demo`, and the `training-net` network appears in Neutron.

## Verify

Students should verify:

- Stack exists
- Network exists
- Stack status becomes `CREATE_COMPLETE`

**Useful checks include:**

- Stack list
- Stack events
- Network list

Heat also provides stack events that show each resource creation step.



---

# Heat Demo 2 – Network + Subnet

## Objective

Demonstrate resource dependency.

## Architecture

```
Stack
  │
  ├── Network
  └── Subnet
```

## Template

```yaml
heat_template_version: 2021-04-16

description: Network and Subnet

resources:

  training_network:
    type: OS::Neutron::Net
    properties:
      name: training-net

  training_subnet:
    type: OS::Neutron::Subnet
    properties:
      name: training-subnet
      network_id:
        get_resource: training_network
      cidr: 192.168.100.0/24
      gateway_ip: 192.168.100.1
      ip_version: 4
```

## Concept

Notice this line:

```yaml
network_id:
  get_resource: training_network
```

Heat automatically understands:

```
Network
  ↓
Subnet
```

The subnet cannot be created until the network exists. Heat builds a dependency graph and provisions resources in the correct order.



---

# Heat Demo 3 – Complete Networking

## Objective

Create an entire tenant network.

## Architecture

```
Provider Network
        ▲
        │
     Router
        ▲
        │
  Private Subnet
        ▲
        │
  Private Network
```

## Resources

- Network
- Subnet
- Router
- Router Interface
- Router Gateway

This demonstrates that Heat can orchestrate multiple dependent Neutron resources in one stack.



---

# Heat Demo 4 – Launch a VM

## Architecture

```
Stack
  │
  ├── Network
  ├── Subnet
  ├── Security Group
  └── VM
```

## Example VM Resource

```yaml
training_vm:
  type: OS::Nova::Server
  properties:
    name: heat-vm
    image: cirros
    flavor: training.small
    key_name: training-key
    networks:
      - network:
          get_resource: training_network
```

## What Heat Does

```
Read Template
     ↓
Create Network
     ↓
Create Subnet
     ↓
Launch VM
     ↓
Stack Complete
```

Heat invokes Nova only after the required networking resources are available.



---

# Heat Demo 5 – Complete Enterprise Stack ⭐

This is the demo I'd use for your training.

## Architecture

```
Heat Stack
  │
  ├── Security Group
  ├── Network
  ├── Subnet
  ├── Router
  ├── Router Gateway
  ├── VM
  ├── Floating IP
  └── Floating IP Association
```

One click (or one CLI command) provisions an entire application environment.

## Stack Lifecycle Demo

After creating the stack, demonstrate lifecycle operations.

### Show Stack

Students inspect:

- Stack status
- Creation time
- Resources
- Outputs

### Show Stack Resources

They should observe:

- `training-net`
- `training-subnet`
- `training-router`
- `heat-vm`
- `floating-ip`

### Show Stack Events

This is one of Heat's best troubleshooting features.

Students can watch:

```
Network Created
     ↓
Subnet Created
     ↓
Router Created
     ↓
VM Created
     ↓
Floating IP Created
     ↓
Stack Complete
```

Every resource transition is recorded as a stack event.

## Update Demo

Modify the template by:

- Changing the flavor
- Renaming the VM
- Adding another VM

Then perform a stack update.

Students will observe that Heat compares the desired state with the current state and changes only the necessary resources instead of recreating everything.

## Delete Demo

Finally:

```
Delete Stack
     ↓
Heat deletes VM
     ↓
Deletes Floating IP
     ↓
Deletes Router
     ↓
Deletes Subnet
     ↓
Deletes Network
```

↓

Stack Removed

Students learn that deleting the stack automatically removes all managed resources in the correct dependency order.
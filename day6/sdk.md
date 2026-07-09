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
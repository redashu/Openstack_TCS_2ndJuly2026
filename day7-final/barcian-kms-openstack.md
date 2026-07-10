# OpenStack Barbican (Concept Only)

Barbican is OpenStack's Key Management Service (KMS).

Its primary purpose is to securely store, manage, and distribute secrets used by OpenStack services and applications.

Think of Barbican as OpenStack's centralized vault for passwords, encryption keys, SSL certificates, and other sensitive information.

## Real-World Problem

Imagine an enterprise OpenStack cloud with:

- 500 virtual machines
- 200 SSL certificates
- 300 database passwords
- 100 API keys
- 100 encryption keys

Without Barbican, administrators may store secrets in:

- Configuration files
- Environment variables
- Scripts
- Databases
- Plain text

Example:

```ini
db_password=Admin@123
ssl_private_key=....
api_key=ABCXYZ123
```

Problems:

- Easy to steal
- Difficult to rotate
- No auditing
- No centralized management
- Security risk

## Solution

Instead of storing secrets everywhere, everything is stored inside OpenStack Barbican.

Applications request secrets when required.

## What Is a Secret?

A secret can be:

- Password
- SSH key
- SSL certificate
- Private key
- API token
- Encryption key
- Database credential
- License key

## Barbican Architecture

```text
               Users / Applications

                      │

             OpenStack REST API

                      │

                 Barbican API

                      │

             Secret Management

                      │

          ----------------------------

          Database

          Secret Store

          HSM (Optional)
```

## Barbican Components

### Barbican API

Provides REST APIs, including:

- Store secret
- Retrieve secret
- Delete secret
- Update secret

### Secret Store

Stores:

- Passwords
- Certificates
- Keys

Can use:

- Database
- Hardware Security Module (HSM)
- KMIP server
- HashiCorp Vault (through integrations)
- Vendor key management systems

### Database

Stores:

- Metadata
- Secret references
- Ownership
- ACL information

The actual secret material may be stored in the database or in an external backend depending on the configured plugin.

## How Barbican Works

Suppose an application needs a database password.

### Without Barbican

```text
Application
↓
config.ini
↓
Password
```

### With Barbican

```text
Application
↓
Barbican API
↓
Retrieve Secret
↓
Password
```

The password is never hardcoded inside the application.

## OpenStack Services Using Barbican

Many OpenStack services can integrate with Barbican.

### Cinder

Stores volume encryption keys.

```text
Encrypted Volume
↓
Encryption Key
↓
Barbican
```

Without Barbican, encryption keys have nowhere secure to live.

### Glance

Can protect image encryption keys.

```text
Encrypted Image
↓
Key
↓
Barbican
```

### Octavia (Load Balancer)

Stores SSL certificates.

```text
HTTPS Load Balancer
↓
SSL Certificate
↓
Barbican
```

Instead of storing certificates on disk.

### Nova

Can integrate with encrypted disks and other services that use Barbican-managed keys.

### Manila

Can use Barbican for encryption-related workflows depending on backend capabilities.

## Example Architecture

```text
             User

               │

        Create HTTPS LB

               │

           Octavia

               │

           Barbican

               │

        SSL Certificate

               │

      Secure Secret Store
```

## Secret Lifecycle

```text
Create Secret
      │
Store Secret
      │
Access Secret
      │
Use Secret
      │
Rotate Secret
      │
Delete Secret
```

## Secret Types

| Type | Example |
| --- | --- |
| Password | Database password |
| Certificate | SSL certificate |
| Private Key | RSA private key |
| SSH Key | Linux SSH key |
| Symmetric Key | AES key |
| Asymmetric Key | RSA key pair |

## Configuration Exposure Example

Application needs a database password.

### Without Barbican (Config File)

```text
Application
↓
config.yaml
↓
Password
```

Anyone reading `config.yaml` gets the password.

### With Barbican (API Retrieval)

```text
Application
↓
REST API
↓
Barbican
↓
Password
```

Password never exists permanently inside configuration files.

## Barbican with Cinder Encryption

```text
User
↓
Create Encrypted Volume
↓
Cinder
↓
Barbican
↓
Generate AES Key
↓
Store Key
↓
Encrypt Volume

Later

Attach Volume
↓
Retrieve Key
↓
Decrypt
↓
VM Access
```

## Barbican with Octavia

```text
User
↓
Upload SSL Certificate
↓
Barbican
↓
Certificate Stored
↓
Octavia
↓
HTTPS Load Balancer
```

Private certificates remain protected.

## Barbican with HSM

Large enterprises often use hardware security modules.

```text
Barbican
↓
HSM
↓
Encryption Keys
```

Advantages:

- Keys never leave hardware
- Higher security
- Compliance
- FIPS certification

## Authentication and Authorization

### Authentication

Barbican integrates with Keystone.

Only authenticated users can access secrets.

### Authorization

Policies define who can:

- Create secrets
- Read secrets
- Delete secrets
- Share secrets

## Secret Rotation

Instead of one password forever, organizations can rotate secrets:

```text
Generate New Password
↓
Update Application
↓
Delete Old Password
```

This is important for security compliance.

## High Availability

Barbican is stateless.

Multiple API servers can run behind:

```text
HAProxy
↓
Barbican API
↓
Galera Database
```

Just like other OpenStack API services.

## Advantages

- Centralized secret management
- Secure key storage
- REST API
- Keystone integration
- Access control
- Auditing
- Encryption support
- HSM support
- Certificate management
- Secret rotation

## Limitations

- Additional service to deploy and maintain
- External HSM integration adds complexity
- Applications and OpenStack services must be configured to use Barbican

## Typical Enterprise Use Cases

| Service | Secret Stored in Barbican |
| --- | --- |
| Cinder | Volume encryption keys |
| Octavia | SSL certificates and private keys |
| Glance | Image encryption keys |
| Applications | Database passwords |
| DevOps | API tokens |
| Kubernetes | TLS certificates (through integrations) |
| Enterprise Security | HSM-managed encryption keys |

## Complete Architecture

```text
                 Users

                   │

              Keystone

                   │

              Barbican API

                   │

        Secret Management Service

                   │

      ┌────────────┼─────────────┐
      │            │             │
  Database      HSM/KMIP     External Vault
   Backend       Backend        (Optional)

                   │

        OpenStack Services

      ┌────────┬────────┬────────┐
      │        │        │        │
   Cinder   Octavia  Glance  Applications
```

## Key Takeaways

- Barbican is OpenStack's centralized KMS.
- It securely manages passwords, API keys, certificates, private keys, and encryption keys.
- It integrates with Keystone for authentication and authorization.
- Services like Cinder (encrypted volumes) and Octavia (TLS certificates) rely on Barbican to avoid storing sensitive material in configuration files.
- In enterprise deployments, Barbican can use HSMs or external key management systems for stronger security and regulatory compliance.

## Simple Analogy

- Keystone = Who are you? (Identity and authentication)
- Neutron = How do you connect? (Networking)
- Cinder = Where is your block storage? (Volumes)
- Glance = Where are your VM images? (Image repository)
- Barbican = Where are your secrets and encryption keys stored securely? (Key and secret management)
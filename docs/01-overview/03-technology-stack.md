---
title: "Technology Stack"
version: "1.0"
last_updated: "2025-11-16"
maintained_by: "E-Global SCM Development Team"
status: "Active"
---

# Technology Stack

## Overview

Cetmix Tower is built on a proven technology stack that combines Odoo's enterprise-grade application framework with specialized Python libraries for SSH communication, cloud integration, and data processing. This document provides detailed information about each technology component, version requirements, and integration patterns.

---

## Platform Foundation

### Odoo 17.0

**Version**: 17.0 (Community or Enterprise)
**License**: LGPL-3.0 (Community) / Proprietary (Enterprise)
**Website**: [https://www.odoo.com](https://www.odoo.com)

Odoo provides the foundational platform for Cetmix Tower, offering:

**Core Framework Features**:
- **ORM (Object-Relational Mapping)**: Python-based ORM for database operations
- **MVC Architecture**: Clean separation of models, views, and controllers
- **Security Framework**: Multi-level security with groups, rules, and ACLs
- **Web Framework**: Modern web interface with JavaScript/XML views
- **API System**: XML-RPC and JSON-RPC APIs for integration
- **Module System**: Plugin architecture for extensibility
- **Workflow Engine**: Built-in workflow and automation capabilities
- **Multi-tenancy**: Support for multiple databases from single installation
- **Internationalization**: Multi-language support out of the box
- **Cron System**: Built-in task scheduler

**Why Odoo 17.0?**
- Latest stable long-term support version
- Modern Python 3.10+ support
- Improved performance over previous versions
- Enhanced security features
- Better UI/UX framework
- Active community and commercial support

**Odoo Components Used by Tower**:
```python
# Core Odoo modules utilized
- models.Model         # Base model class
- api.depends          # Computed fields
- api.model           # Model methods
- fields.*            # All field types
- http.request        # HTTP request handling
- http.route          # URL routing
- tools.safe_eval     # Safe Python evaluation
- tools.misc          # Utility functions
- osv.expression      # Domain expressions
```

**Minimum Requirements**:
- Odoo 17.0.0 or higher
- Python 3.10, 3.11, or 3.12
- PostgreSQL 12.0 or higher

---

## Python Environment

### Python Version

**Required Version**: Python 3.10, 3.11, or 3.12
**Recommended**: Python 3.11

**Key Python Features Used**:
- Type hints for better code quality
- Async/await for asynchronous operations (queue jobs)
- Context managers for resource management
- Decorators for API methods
- List/dict comprehensions for data processing
- F-strings for string formatting

### Python Dependencies

All Python dependencies are managed via `pip` and defined in each module's `__manifest__.py` file under `external_dependencies`.

---

## Core Python Libraries

### 1. Paramiko (SSH Library)

**Version**: < 4.0 (pinned to maintain compatibility)
**Purpose**: SSH2 protocol implementation for remote command execution
**License**: LGPL-2.1
**Website**: [https://www.paramiko.org](https://www.paramiko.org)

**Used By**: `cetmix_tower_server` (core module)

**Key Features Utilized**:
- SSH client implementation
- SFTP client for file transfers
- Public key authentication
- Password authentication
- Host key verification
- Channel management for command execution

**Usage in Tower**:
```python
# Connection establishment
client = paramiko.SSHClient()
client.set_missing_host_key_policy(paramiko.AutoAddPolicy())
client.connect(
    hostname=server.ssh_host,
    port=server.ssh_port,
    username=server.ssh_username,
    password=server.ssh_password,  # or
    key_filename=private_key_path,
    timeout=connection_timeout
)

# Command execution
stdin, stdout, stderr = client.exec_command(command)
output = stdout.read().decode()
error = stderr.read().decode()
exit_code = stdout.channel.recv_exit_status()

# File transfer (SFTP)
sftp = client.open_sftp()
sftp.put(local_path, remote_path)
sftp.get(remote_path, local_path)
```

**Why Paramiko?**
- Pure Python implementation (no system SSH required)
- Cross-platform compatibility
- Well-maintained and widely used
- Complete SSH2 protocol support
- SFTP support for file transfers
- Programmatic control over all aspects of SSH

**Version Constraint Note**:
- Version pinned to `< 4` for compatibility with legacy installations
- Paramiko 3.x is stable and well-tested
- Future versions will support Paramiko 4.x

---

### 2. TLDExtract (Domain Parsing)

**Version**: Latest stable
**Purpose**: Extract domain components from URLs and hostnames
**License**: BSD-3-Clause
**Website**: [https://github.com/john-kurkowski/tldextract](https://github.com/john-kurkowski/tldextract)

**Used By**: `cetmix_tower_server`

**Key Features Utilized**:
- Extract subdomain, domain, and suffix from URLs
- Public suffix list support
- Handle complex domain structures

**Usage in Tower**:
```python
import tldextract

# Parse server hostname
extracted = tldextract.extract('app.staging.example.com')
# Result:
# subdomain: 'app.staging'
# domain: 'example'
# suffix: 'com'
# registered_domain: 'example.com'
```

**Use Cases in Tower**:
- Server hostname validation
- Domain-based server grouping
- SSL certificate domain matching
- DNS management features

---

### 3. DNSPython (DNS Toolkit)

**Version**: Latest stable
**Purpose**: DNS query and manipulation library
**License**: ISC License
**Website**: [https://www.dnspython.org](https://www.dnspython.org)

**Used By**: `cetmix_tower_server`

**Key Features Utilized**:
- DNS queries (A, AAAA, MX, TXT, etc.)
- DNS resolution
- Reverse DNS lookups
- DNS zone file parsing

**Usage in Tower**:
```python
import dns.resolver

# Resolve hostname to IP
answers = dns.resolver.resolve(hostname, 'A')
ip_address = str(answers[0])

# Reverse DNS lookup
rev_name = dns.reversename.from_address(ip_address)
answers = dns.resolver.resolve(rev_name, 'PTR')
hostname = str(answers[0])
```

**Use Cases in Tower**:
- Server hostname resolution
- Validate server connectivity before attempting SSH
- DNS-based server discovery
- Network troubleshooting features

---

### 4. PyYAML (YAML Parser)

**Version**: Latest stable
**Purpose**: YAML serialization and deserialization
**License**: MIT
**Website**: [https://pyyaml.org](https://pyyaml.org)

**Used By**: `cetmix_tower_yaml`

**Key Features Utilized**:
- Parse YAML files
- Generate YAML from Python objects
- Safe loading (prevent code execution)
- Custom representers and constructors

**Usage in Tower**:
```python
import yaml

# Export Tower object to YAML
data = {
    'servers': [...],
    'commands': [...],
    'flight_plans': [...]
}
yaml_content = yaml.dump(data, default_flow_style=False)

# Import from YAML
with open(yaml_file, 'r') as f:
    data = yaml.safe_load(f)
```

**Use Cases in Tower**:
- Export configurations to YAML files
- Import configurations from YAML files
- Configuration as code
- Version control integration
- Configuration templates

**Safety Features**:
- Uses `yaml.safe_load()` to prevent code execution
- Validates structure before import
- Error handling for malformed YAML

---

### 5. Boto3 (AWS SDK)

**Version**: Latest stable
**Purpose**: Amazon Web Services (AWS) SDK for Python
**License**: Apache-2.0
**Website**: [https://boto3.amazonaws.com](https://boto3.amazonaws.com)

**Used By**: `cetmix_tower_aws` (optional module)

**Key Features Utilized**:
- EC2 instance management
- Instance lifecycle operations (start, stop, terminate)
- Instance metadata retrieval
- Resource tagging
- IAM integration

**Usage in Tower**:
```python
import boto3

# Create EC2 client
ec2 = boto3.client('ec2',
    aws_access_key_id=access_key,
    aws_secret_access_key=secret_key,
    region_name=region
)

# List instances
instances = ec2.describe_instances()

# Start instance
ec2.start_instances(InstanceIds=[instance_id])

# Stop instance
ec2.stop_instances(InstanceIds=[instance_id])
```

**Use Cases in Tower**:
- Manage AWS EC2 instances from Tower
- Sync instance information to server records
- Automate instance lifecycle
- Cost optimization through scheduled start/stop
- Integration with AWS infrastructure

**Authentication Methods**:
- AWS access key and secret key
- IAM roles (when running on EC2)
- AWS credentials file
- Environment variables

---

### 6. OVH SDK (OVH API)

**Version**: Latest stable
**Purpose**: OVH cloud services API client
**License**: BSD-3-Clause
**Website**: [https://github.com/ovh/python-ovh](https://github.com/ovh/python-ovh)

**Used By**: `cetmix_tower_ovh` (optional module)

**Key Features Utilized**:
- OVH API authentication
- Cloud instance management
- Resource provisioning
- API request handling

**Usage in Tower**:
```python
import ovh

# Create OVH client
client = ovh.Client(
    endpoint=endpoint,
    application_key=app_key,
    application_secret=app_secret,
    consumer_key=consumer_key
)

# List cloud instances
instances = client.get('/cloud/project/{}/instance'.format(project_id))

# Instance operations
client.post('/cloud/project/{}/instance/{}/reboot'.format(
    project_id, instance_id
))
```

**Use Cases in Tower**:
- Manage OVH cloud instances
- Provision new instances
- Monitor instance status
- Automate OVH infrastructure
- European cloud provider integration

---

## Optional Python Libraries

### Queue Job Framework

**Module**: `queue_job` (OCA module)
**Purpose**: Asynchronous job execution
**Required For**: `cetmix_tower_server_queue`

**Benefits**:
- Non-blocking command execution
- Background job processing
- Job queue management
- Retry mechanisms
- Job monitoring and status tracking

**Integration**:
```python
from odoo.addons.queue_job.job import job

@job
def execute_command_async(self):
    """Execute command in background job"""
    self.execute_command()
```

---

### Web Notify

**Module**: `web_notify` (OCA module)
**Purpose**: Browser notifications
**Required For**: `cetmix_tower_server_notify_backend`

**Benefits**:
- Real-time notifications in browser
- Success/failure alerts
- Custom notification messages
- Non-intrusive user feedback

**Integration**:
```python
self.env.user.notify_success(
    message="Command executed successfully",
    title="Command Complete"
)
```

---

### RPC Helper

**Module**: `rpc_helper`
**Purpose**: Remote procedure call utilities
**Required For**: `cetmix_tower_server`

**Benefits**:
- Simplified RPC calls
- Connection management
- Error handling
- API abstraction

---

## Database Layer

### PostgreSQL

**Minimum Version**: 12.0
**Recommended Version**: 14.x or 15.x
**License**: PostgreSQL License (permissive)
**Website**: [https://www.postgresql.org](https://www.postgresql.org)

**Why PostgreSQL?**
- Required by Odoo (no alternative databases supported)
- Excellent performance for relational data
- ACID compliance for data integrity
- Advanced indexing capabilities
- JSON/JSONB support for flexible data
- Mature replication and backup tools
- Strong community and commercial support

**PostgreSQL Features Used**:
- **JSONB fields**: Store variable data structures
- **Indexes**: B-tree, GIN indexes for performance
- **Foreign Keys**: Referential integrity
- **Transactions**: ACID guarantees
- **Sequences**: Auto-incrementing IDs
- **Triggers**: Automatic data processing
- **Views**: Query optimization

**Database Schema Highlights**:
```sql
-- Server table with encrypted passwords
CREATE TABLE cx_tower_server (
    id SERIAL PRIMARY KEY,
    name VARCHAR NOT NULL,
    ssh_host VARCHAR NOT NULL,
    ssh_port INTEGER DEFAULT 22,
    ssh_username VARCHAR,
    ssh_password VARCHAR,  -- Encrypted by Odoo
    ssh_auth_mode VARCHAR,
    host_key TEXT,
    -- ... other fields
    UNIQUE(ssh_host, ssh_port)
);

-- Command logs with indexes for performance
CREATE TABLE cx_tower_command_log (
    id SERIAL PRIMARY KEY,
    command_id INTEGER REFERENCES cx_tower_command(id),
    server_id INTEGER REFERENCES cx_tower_server(id),
    create_date TIMESTAMP,
    duration FLOAT,
    exit_code INTEGER,
    -- ... other fields
);
CREATE INDEX idx_cmd_log_create_date ON cx_tower_command_log(create_date);
CREATE INDEX idx_cmd_log_server ON cx_tower_command_log(server_id);
CREATE INDEX idx_cmd_log_command ON cx_tower_command_log(command_id);

-- Vault storage (encrypted)
CREATE TABLE cx_tower_key_value (
    id SERIAL PRIMARY KEY,
    key_id INTEGER REFERENCES cx_tower_key(id),
    name VARCHAR NOT NULL,
    value TEXT,  -- Encrypted by Odoo
    -- ... other fields
);
```

**Performance Optimization**:
- Regular VACUUM and ANALYZE
- Proper indexing on frequently queried fields
- Connection pooling
- Query optimization
- Partitioning for large log tables (optional)

**Backup Recommendations**:
- Daily full backups
- Point-in-time recovery (WAL archiving)
- Test restore procedures regularly
- Separate filestore backups

---

## External System Integrations

### Git

**Purpose**: Source code version control integration
**Used By**: `cetmix_tower_git`

**Integration Methods**:
- Execute Git commands via SSH on remote servers
- Clone repositories
- Pull updates
- Manage credentials

**Typical Workflow**:
```bash
# Commands executed on remote servers
git clone <repository_url> <destination_path>
git pull origin main
git checkout <branch>
git reset --hard <commit>
```

**Credential Management**:
- SSH keys for Git authentication (stored in Vault)
- HTTPS with tokens (stored in Vault)
- Deploy keys for read-only access

---

### AWS (Amazon Web Services)

**Purpose**: Cloud infrastructure management
**Used By**: `cetmix_tower_aws`

**Supported Services**:
- **EC2**: Elastic Compute Cloud instance management
- **IAM**: Authentication and authorization

**Integration Pattern**:
```
Tower → Boto3 → AWS API → EC2 Instances
  ↓
Server Records (synced with EC2 metadata)
```

**Use Cases**:
- Sync EC2 instances to Tower server records
- Start/stop instances on schedule
- Cost optimization
- Infrastructure automation

---

### OVH Cloud

**Purpose**: European cloud provider integration
**Used By**: `cetmix_tower_ovh`

**Supported Services**:
- OVH Public Cloud instances
- VPS management
- Instance lifecycle operations

**Integration Pattern**:
```
Tower → OVH SDK → OVH API → Cloud Instances
  ↓
Server Records (synced with OVH)
```

---

### Webhooks (HTTP/HTTPS)

**Purpose**: Event-driven automation and external integrations
**Used By**: `cetmix_tower_webhook`

**Supported Operations**:
- Receive POST requests
- Parse JSON payloads
- Extract variables from webhooks
- Trigger commands or flight plans
- Authentication via tokens or signatures

**Integration Examples**:
- **GitHub**: Trigger deployments on push events
- **GitLab**: CI/CD pipeline integration
- **Custom Systems**: Any system that can send HTTP requests

**Webhook Endpoint**:
```
POST https://your-odoo-instance.com/tower/webhook/<webhook_reference>

Headers:
  Content-Type: application/json
  X-Webhook-Token: <authentication_token>

Body:
{
  "event": "deployment",
  "branch": "main",
  "commit": "abc123",
  "environment": "production"
}
```

---

## Frontend Technologies

### Odoo Web Framework

**Components**:
- **OWL (Odoo Web Library)**: JavaScript framework for views
- **XML Views**: Declarative view definitions
- **QWeb**: Template engine
- **JavaScript Actions**: Client-side logic
- **SCSS**: Styling

**Tower-Specific Frontend Features**:
```javascript
// Custom JavaScript widgets
- Command execution progress indicator
- Real-time log viewer
- Secret masking in UI
- Flight plan diagram visualization

// SCSS styling
- Tower-specific color schemes
- Custom button styles
- Enhanced form layouts
```

**Technologies Used**:
- **JavaScript**: ES6+ syntax
- **XML**: View definitions
- **SCSS**: Styling
- **Bootstrap**: CSS framework (via Odoo)
- **Font Awesome**: Icons

---

## Development and Testing Tools

### Code Quality Tools

**Pre-commit Hooks**:
- `pylint-odoo`: Odoo-specific linting
- `black`: Code formatting
- `isort`: Import sorting
- `flake8`: Style guide enforcement
- `prettier`: XML/JavaScript formatting

**Testing Framework**:
- Odoo's built-in test framework
- Python `unittest` module
- TransactionCase for database tests
- Mock for external API testing

**CI/CD**:
- GitHub Actions workflows
- Automated testing on pull requests
- Code coverage reporting (codecov)
- Pre-commit checks

---

## Security Technologies

### Encryption

**Database Encryption**:
- Odoo's built-in field encryption for sensitive data
- AES encryption for passwords and secrets
- Transparent to application code

**Transport Encryption**:
- **SSH**: Encrypted connections to servers (SSH2 protocol)
- **HTTPS**: Encrypted web traffic (TLS 1.2+)
- **Database**: PostgreSQL connections can use SSL

**Secret Management**:
- Cetmix Tower Vault (encrypted at rest)
- In-memory decryption only
- Automatic masking in logs and UI
- No plain text storage

### Authentication and Authorization

**Odoo Security Framework**:
- **Groups**: Role-based access control
- **Record Rules**: Row-level security
- **Access Rights**: Model-level permissions
- **Field Security**: Field-level access control

**External Authentication** (via Odoo):
- OAuth2 (Google, Microsoft, etc.)
- LDAP/Active Directory
- SAML
- Multi-factor authentication (via modules)

---

## Infrastructure Requirements

### Minimum System Requirements

**For Development**:
- **CPU**: 2 cores
- **RAM**: 4 GB
- **Storage**: 20 GB
- **OS**: Linux (Ubuntu 20.04+, Debian 10+), macOS, Windows (WSL2)

**For Production (Small)**:
- **CPU**: 4 cores
- **RAM**: 8 GB
- **Storage**: 50 GB SSD
- **OS**: Linux (Ubuntu 22.04 LTS recommended)

**For Production (Medium - 100 servers, 10 concurrent users)**:
- **CPU**: 8 cores
- **RAM**: 16 GB
- **Storage**: 100 GB SSD
- **OS**: Linux (Ubuntu 22.04 LTS)

**For Production (Large - 500+ servers, 50+ concurrent users)**:
- **CPU**: 16+ cores
- **RAM**: 32+ GB
- **Storage**: 500 GB SSD (consider separate log storage)
- **OS**: Linux (Ubuntu 22.04 LTS)
- **Database**: Separate PostgreSQL server
- **High Availability**: Load balanced Odoo instances

### Network Requirements

**Firewall Rules**:
```
# Inbound (Tower Server)
- 8069/tcp  - Odoo HTTP (or custom port)
- 443/tcp   - HTTPS (if using reverse proxy)

# Outbound (Tower Server)
- 22/tcp    - SSH to managed servers
- 443/tcp   - HTTPS for external APIs (AWS, OVH, Git)
- 53/udp    - DNS
- Custom    - Any other ports for managed services

# Database (if separate)
- 5432/tcp  - PostgreSQL (only from Odoo servers)
```

**SSH Access**:
- Tower server must be able to reach managed servers on SSH port (default 22)
- No inbound SSH required on Tower server (but recommended for administration)
- Consider bastion hosts or VPN for secure access

### Supported Operating Systems

**Tower Server (Odoo)**:
- Ubuntu 20.04 LTS, 22.04 LTS (recommended)
- Debian 10, 11, 12
- Red Hat Enterprise Linux 8, 9
- CentOS Stream 8, 9
- Fedora (latest)
- macOS (development only)
- Windows with WSL2 (development only)

**Managed Servers**:
- Any Linux/Unix system with SSH server
- Windows with OpenSSH server (limited support)
- Network devices with SSH access

---

## Browser Compatibility

**Supported Browsers** (via Odoo):
- Google Chrome 90+ (recommended)
- Mozilla Firefox 88+
- Microsoft Edge 90+
- Safari 14+ (macOS/iOS)

**Not Supported**:
- Internet Explorer (all versions)
- Legacy browsers

---

## Licensing Summary

| Component | License | Type |
|-----------|---------|------|
| Cetmix Tower | AGPL-3.0 | Open Source |
| Odoo Community | LGPL-3.0 | Open Source |
| Odoo Enterprise | Proprietary | Commercial |
| PostgreSQL | PostgreSQL License | Open Source |
| Paramiko | LGPL-2.1 | Open Source |
| Boto3 | Apache-2.0 | Open Source |
| PyYAML | MIT | Open Source |
| OVH SDK | BSD-3-Clause | Open Source |
| Python | PSF License | Open Source |

**Important Notes**:
- Cetmix Tower uses AGPL-3.0 license, which requires source code distribution for modifications
- Each module may have specific licensing (check `__manifest__.py`)
- External dependencies have their own licenses
- Commercial support available from Cetmix

---

## Version Compatibility Matrix

| Component | Minimum Version | Recommended Version | Maximum Version |
|-----------|----------------|---------------------|-----------------|
| Odoo | 17.0.0 | 17.0 (latest) | 17.0.x |
| Python | 3.10 | 3.11 | 3.12 |
| PostgreSQL | 12.0 | 14.x, 15.x | Latest |
| Paramiko | 2.7 | 3.x | < 4.0 |
| PyYAML | 5.3 | Latest | Latest |
| Boto3 | 1.17 | Latest | Latest |
| OVH SDK | 0.5 | Latest | Latest |

---

## Next Steps

- [Module List](04-module-list.md) - Detailed module information
- [Installation Guide](../02-getting-started/01-installation.md) - Install dependencies and Tower
- [Configuration](../02-getting-started/02-configuration.md) - Configure your installation

---

**Last Updated**: 2025-11-16
**Version**: 1.0
**Maintained By**: E-Global SCM Development Team

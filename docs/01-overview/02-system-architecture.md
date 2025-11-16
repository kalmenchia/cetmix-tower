---
title: "System Architecture"
version: "1.0"
last_updated: "2025-11-16"
maintained_by: "E-Global SCM Development Team"
status: "Active"
---

# System Architecture

## Overview

Cetmix Tower is built on a modular architecture that leverages Odoo's proven MVC (Model-View-Controller) framework. The system is designed for scalability, security, and extensibility, allowing organizations to manage from a single server to thousands of servers from a centralized Odoo instance.

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        Odoo 17.0 Platform                       │
│                     (Web Server + Framework)                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│  │   Web UI     │  │     API      │  │   Queue      │         │
│  │  (Backend)   │  │  (Webhooks)  │  │   Jobs       │         │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘         │
│         │                  │                  │                 │
│  ┌──────┴──────────────────┴──────────────────┴───────┐        │
│  │         Cetmix Tower Core Business Logic           │        │
│  │                                                     │        │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐           │        │
│  │  │ Server  │  │ Command │  │  Flight │           │        │
│  │  │  Mgmt   │  │  Engine │  │  Plans  │           │        │
│  │  └─────────┘  └─────────┘  └─────────┘           │        │
│  │                                                     │        │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐           │        │
│  │  │  File   │  │Variable │  │  Vault  │           │        │
│  │  │  Mgmt   │  │  Mgmt   │  │(Secrets)│           │        │
│  │  └─────────┘  └─────────┘  └─────────┘           │        │
│  └─────────────────────────────────────────────────┬─┘        │
│                                                     │          │
│  ┌──────────────────────────────────────────────────┴─┐       │
│  │         Integration Layer                          │       │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐           │       │
│  │  │   Git   │  │   AWS   │  │   OVH   │           │       │
│  │  └─────────┘  └─────────┘  └─────────┘           │       │
│  └────────────────────────────────────────────────────┘       │
│                                                                │
├────────────────────────────────────────────────────────────────┤
│                     PostgreSQL Database                        │
│              (Models, Logs, Configurations, Vault)             │
└────────────────────────────────────────────────────────────────┘
                              │
                              │ SSH / API Connections
                              ▼
        ┌─────────────────────────────────────────────┐
        │          Remote Servers                     │
        │  ┌─────────┐  ┌─────────┐  ┌─────────┐     │
        │  │Server 1 │  │Server 2 │  │Server N │     │
        │  └─────────┘  └─────────┘  └─────────┘     │
        └─────────────────────────────────────────────┘
```

---

## Core Components

### 1. Server Management Component

The Server Management component is the foundation of Cetmix Tower, responsible for managing server connections and metadata.

**Key Models**:
- `cx.tower.server` - Core server model
- `cx.tower.server.template` - Server templates for quick provisioning
- `cx.tower.os` - Operating system profiles
- `cx.tower.tag` - Tags for server organization
- `cx.tower.server.log` - Server activity logs

**Responsibilities**:
- Maintain server inventory and metadata
- Manage SSH connections (paramiko-based)
- Handle authentication (password and key-based)
- Track server status and availability
- Organize servers with tags and templates
- Log all server-related activities

**Connection Architecture**:
```
Server Record → SSH Client → Paramiko Library → SSH Connection → Remote Server
        │              │
        │              └─ Connection Pool (reusable connections)
        │
        └─ Authentication:
           - Password (encrypted in database)
           - SSH Key (from Vault)
           - Host Key Verification
```

### 2. Command Execution Engine

The Command Execution Engine handles all remote command execution with sophisticated features.

**Key Models**:
- `cx.tower.command` - Command definitions
- `cx.tower.command.log` - Command execution logs

**Execution Flow**:
```
1. Command Triggered (UI/API/Schedule/Flight Plan)
                │
                ▼
2. Variable Substitution (${var_name} → actual value)
                │
                ▼
3. Secret Injection (from Vault, with masking)
                │
                ▼
4. SSH Connection Established
                │
                ▼
5. Command Executed on Remote Server
                │
                ▼
6. Output Captured (stdout/stderr)
                │
                ▼
7. Secret Masking Applied to Logs
                │
                ▼
8. Results Stored in cx.tower.command.log
                │
                ▼
9. Notifications Sent (if configured)
```

**Command Types**:
- **SSH Commands**: Execute shell commands remotely
- **Python Code**: Execute Python code with Odoo context
- **File Upload**: Upload files to servers
- **File Download**: Download files from servers

**Variable Substitution Engine**:
```python
# Variables can come from multiple sources (priority order):
1. Custom values (passed at execution time)
2. Server-specific values
3. Command-specific default values
4. Global variable defaults

# Syntax: ${variable_name}
# Example: "cd ${app_path} && git pull"
# Resolved: "cd /opt/myapp && git pull"
```

### 3. Flight Plan Orchestration

Flight Plans provide workflow automation capabilities with conditional logic and error handling.

**Key Models**:
- `cx.tower.plan` - Flight plan definitions
- `cx.tower.plan.line` - Individual steps in a flight plan
- `cx.tower.plan.line.action` - Conditional actions within steps
- `cx.tower.plan.log` - Flight plan execution logs

**Flight Plan Architecture**:
```
Flight Plan
    │
    ├─ Plan Line 1: Check disk space
    │   ├─ Action: If success → Continue
    │   └─ Action: If failure → Stop with error
    │
    ├─ Plan Line 2: Backup database
    │   ├─ Action: If success → Continue
    │   └─ Action: If failure → Execute rollback plan
    │
    ├─ Plan Line 3: Deploy new code
    │   ├─ Action: If success → Continue
    │   └─ Action: If failure → Restore backup
    │
    ├─ Plan Line 4: Restart services
    │   └─ Action: Always continue
    │
    └─ Plan Line 5: Verify health
        ├─ Action: If success → Complete
        └─ Action: If failure → Alert admin
```

**Execution States**:
- `draft` - Being edited
- `ready` - Ready for execution
- `running` - Currently executing
- `done` - Completed successfully
- `error` - Failed with error
- `terminated` - Manually terminated

**Conditional Logic**:
- Success/failure of previous command
- Variable value comparisons
- Custom Python expressions
- Combined conditions (AND/OR logic)

### 4. File Management System

Manages bidirectional file synchronization between Tower and servers.

**Key Models**:
- `cx.tower.file` - File records
- `cx.tower.file.template` - File templates with variables

**File Operations Architecture**:
```
┌─────────────────────────────────────────────────────┐
│                 File Management                     │
├─────────────────────────────────────────────────────┤
│                                                     │
│  File Template (with Jinja2 variables)             │
│         │                                           │
│         ▼                                           │
│  Variable Substitution                              │
│         │                                           │
│         ▼                                           │
│  ┌──────────┐         ┌──────────┐                │
│  │   Push   │────────▶│ Server   │                │
│  │          │         │  Files   │                │
│  │   Pull   │◀────────│          │                │
│  └──────────┘         └──────────┘                │
│         │                                           │
│         ▼                                           │
│  Version Tracking (reference codes)                │
│                                                     │
└─────────────────────────────────────────────────────┘
```

**File Template Engine**:
- Uses Jinja2 for templating
- Supports all Tower variables
- Can include conditional sections
- Generates files dynamically at execution time

**Synchronization Modes**:
- **One-time**: Manual push/pull
- **Scheduled**: Automatic sync at intervals
- **On-demand**: Triggered by commands or flight plans

### 5. Variable Management System

Provides flexible configuration management through variables.

**Key Models**:
- `cx.tower.variable` - Variable definitions
- `cx.tower.variable.value` - Variable values (server-specific or global)
- `cx.tower.variable.option` - Selection options for choice variables

**Variable Types**:
- `char` - Text values
- `text` - Multi-line text
- `selection` - Dropdown choices
- `boolean` - True/false flags

**Variable Scope Architecture**:
```
Global Variable Definition
        │
        ├─ Default Value (if not server-specific)
        │
        └─ Server-Specific Values
                │
                ├─ Server A: value_a
                ├─ Server B: value_b
                └─ Server C: value_c

Runtime Resolution:
  Command Execution → Check for custom value
                           │
                           ├─ Yes: Use custom value
                           │
                           └─ No: Check server-specific value
                                     │
                                     ├─ Yes: Use server value
                                     │
                                     └─ No: Use default value
```

### 6. Vault (Secret Management)

Cetmix Tower Vault provides secure storage for sensitive data.

**Key Models**:
- `cx.tower.key` - Key definitions (grouping of secrets)
- `cx.tower.key.value` - Individual secret key-value pairs

**Security Architecture**:
```
┌─────────────────────────────────────────────────────┐
│                 Cetmix Tower Vault                  │
├─────────────────────────────────────────────────────┤
│                                                     │
│  Secret Storage:                                    │
│  ┌──────────────────────────────────────────────┐  │
│  │ Key: SSH_PRIVATE_KEY                         │  │
│  │ Value: -----BEGIN RSA PRIVATE KEY-----       │  │
│  │        (encrypted in database)               │  │
│  │ Reference: ${secret:ssh_key}                 │  │
│  └──────────────────────────────────────────────┘  │
│                                                     │
│  Injection Process:                                │
│  1. Command contains: ${secret:db_password}        │
│  2. Vault retrieves encrypted value                │
│  3. Value decrypted in memory                      │
│  4. Injected into command                          │
│  5. Command executed                               │
│  6. Secret masked in logs as: ****                 │
│                                                     │
│  Access Control:                                   │
│  - Odoo security groups                            │
│  - Record rules by ownership                       │
│  - Audit logging of access                         │
│                                                     │
└─────────────────────────────────────────────────────┘
```

**Secret References**:
- Syntax: `${secret:reference_code}`
- Automatic injection during command execution
- Automatic masking in all logs
- No secrets stored in plain text in logs

### 7. Scheduled Task System

Automates recurring operations using Odoo's cron system.

**Key Models**:
- `cx.tower.scheduled.task` - Scheduled task definitions
- `cx.tower.scheduled.task.cv` - Custom variable values for scheduled tasks

**Scheduling Architecture**:
```
Odoo Cron System
        │
        ▼
Scheduled Task Record
        │
        ├─ Execution Type: Command or Flight Plan
        ├─ Target Servers (tags or specific servers)
        ├─ Custom Variables (override defaults)
        └─ Schedule (cron syntax)
        │
        ▼
Execution Triggered
        │
        ▼
Queue Job Created (if queue_job enabled)
        │
        ▼
Command/Flight Plan Executed
        │
        ▼
Logs Created
        │
        ▼
Notifications Sent (if configured)
```

### 8. Logging System

Comprehensive logging for audit trails and troubleshooting.

**Log Types**:
- **Command Logs** (`cx.tower.command.log`): Every command execution
- **Flight Plan Logs** (`cx.tower.plan.log`): Flight plan executions
- **Server Logs** (`cx.tower.server.log`): Server-related activities

**Log Data Captured**:
- Execution timestamp (start and end)
- User who triggered the execution
- Server(s) involved
- Command/plan executed
- Exit code
- stdout output (with secret masking)
- stderr output (with secret masking)
- Duration
- Variable values used
- Success/failure status

**Log Retention**:
- Configurable retention periods
- Automatic cleanup via scheduled tasks
- Export capabilities for archiving

---

## Module Architecture and Dependencies

### Module Dependency Graph

```
                    cetmix_tower (Meta Module)
                           │
                ┌──────────┴──────────┐
                │                     │
    cetmix_tower_server_queue  cetmix_tower_git
                │                     │
                │              cetmix_tower_webhook
                │                     │
                │              cetmix_tower_yaml
                │                     │
        cetmix_tower_server_notify_backend
                │
                │
        cetmix_tower_server (Core)
                │
        ┌───────┼───────┐
        │       │       │
cetmix_tower  cetmix_tower  cetmix_tower
   _aws         _ovh          (others)
```

### Core Module: `cetmix_tower_server`

The foundation module providing:
- All core models (Server, Command, Plan, File, Variable, Vault)
- SSH connection management
- Command execution engine
- Flight plan orchestration
- Security framework
- Base UI views

**Dependencies**:
- `mail` - Odoo's messaging system
- `rpc_helper` - RPC communication helper
- `paramiko` - SSH library (Python)
- `tldextract` - Domain parsing (Python)
- `dnspython` - DNS utilities (Python)

### Queue Module: `cetmix_tower_server_queue`

Adds asynchronous execution capabilities:
- Long-running command execution without blocking UI
- Background job processing
- Job queue management
- Job status tracking

**Dependencies**:
- `cetmix_tower_server` - Core module
- `queue_job` - OCA queue job framework

**Auto-install**: Yes (when queue_job is available)

### Notification Module: `cetmix_tower_server_notify_backend`

Provides real-time browser notifications:
- Command completion notifications
- Flight plan status updates
- Error alerts
- Success confirmations

**Dependencies**:
- `cetmix_tower_server` - Core module
- `web_notify` - Web notification framework

**Auto-install**: Yes (when web_notify is available)

### YAML Module: `cetmix_tower_yaml`

Enables configuration as code:
- Export Tower objects to YAML
- Import configurations from YAML
- Version control integration
- Configuration templates

**Dependencies**:
- `cetmix_tower_server` - Core module
- `pyyaml` - YAML parser (Python)

**Exportable Objects**:
- Servers and server templates
- Commands
- Flight plans
- Files and file templates
- Variables
- Scheduled tasks
- Tags
- SSH keys

### Git Module: `cetmix_tower_git`

Git repository management:
- Clone repositories to servers
- Pull updates
- Manage Git credentials
- Repository synchronization

**Dependencies**:
- `cetmix_tower_yaml` - YAML module (extends it)

**Features**:
- Git project management
- Git source definitions
- Git remote configurations
- Integration with flight plans

### Webhook Module: `cetmix_tower_webhook`

HTTP webhook integration:
- Receive webhooks from external systems
- Trigger commands/flight plans via webhooks
- Webhook authentication
- Webhook logging

**Dependencies**:
- `cetmix_tower_yaml` - YAML module

**Features**:
- Webhook authenticators
- Webhook endpoints
- Variable extraction from webhooks
- Webhook execution logs

### AWS Module: `cetmix_tower_aws` (Optional)

AWS EC2 integration:
- Manage EC2 instances
- Start/stop instances
- Retrieve instance information
- Integration with server management

**Dependencies**:
- `cetmix_tower_server` - Core module
- `boto3` - AWS SDK (Python)

### OVH Module: `cetmix_tower_ovh` (Optional)

OVH cloud integration:
- Manage OVH cloud instances
- OVH API integration
- Instance lifecycle management

**Dependencies**:
- `cetmix_tower_server` - Core module
- `ovh` - OVH SDK (Python)

### Meta Module: `cetmix_tower`

Installation convenience module:
- No code, just dependencies
- Installs recommended modules together
- Simplifies deployment

**Dependencies**:
- `cetmix_tower_server`
- `cetmix_tower_server_queue`
- `cetmix_tower_server_notify_backend`
- `cetmix_tower_git`
- `cetmix_tower_webhook`

---

## Data Flow

### Command Execution Data Flow

```
1. User Interface
        │ (Trigger: Button click, Schedule, API call)
        ▼
2. Controller Layer
        │ (Validate permissions, prepare execution context)
        ▼
3. Command Model
        │ (Load command definition)
        ▼
4. Variable Resolution System
        │ (Resolve all ${variables})
        ▼
5. Secret Injection System
        │ (Inject secrets from Vault)
        ▼
6. Queue Job System (if enabled)
        │ (Create background job)
        ▼
7. SSH Connection Manager
        │ (Establish SSH connection via paramiko)
        ▼
8. Remote Server
        │ (Execute command)
        ▼
9. Output Capture
        │ (Capture stdout/stderr)
        ▼
10. Secret Masking
        │ (Mask all secrets in output)
        ▼
11. Log Storage
        │ (Create cx.tower.command.log record)
        ▼
12. Notification System
        │ (Send browser notification if enabled)
        ▼
13. User Interface Update
        (Display results)
```

### Flight Plan Execution Data Flow

```
1. Flight Plan Triggered
        │
        ▼
2. Flight Plan Log Created (status: running)
        │
        ▼
3. Load Plan Lines (sequential order)
        │
        ▼
4. For Each Plan Line:
        │
        ├─ Evaluate Conditions (previous results, variables)
        │       │
        │       ├─ Condition True: Execute line
        │       │       │
        │       │       ├─ Execute Command
        │       │       │       │
        │       │       │       └─ Create Command Log
        │       │       │
        │       │       └─ Execute Actions based on result
        │       │               │
        │       │               ├─ Success Action
        │       │               │
        │       │               └─ Failure Action
        │       │
        │       └─ Condition False: Skip line
        │
        ▼
5. All Lines Processed
        │
        ▼
6. Update Flight Plan Log (status: done/error)
        │
        ▼
7. Send Notifications
        │
        ▼
8. Update UI
```

---

## SSH Connection Architecture

### Connection Management

Cetmix Tower uses the Paramiko library for SSH connections with sophisticated connection management:

**Connection Lifecycle**:
```
┌─────────────────────────────────────────────────────┐
│            SSH Connection Lifecycle                 │
├─────────────────────────────────────────────────────┤
│                                                     │
│  1. Connection Request                             │
│     │                                               │
│     ▼                                               │
│  2. Retrieve Server Credentials                    │
│     - Username                                      │
│     - Authentication method (password/key)          │
│     - Host key                                      │
│     │                                               │
│     ▼                                               │
│  3. Create SSH Client (Paramiko)                   │
│     │                                               │
│     ▼                                               │
│  4. Configure Client                               │
│     - Set timeout                                   │
│     - Configure host key checking                  │
│     - Load system host keys                        │
│     │                                               │
│     ▼                                               │
│  5. Establish Connection                           │
│     - Connect to server:port                       │
│     - Verify host key                              │
│     - Authenticate (password or key)               │
│     │                                               │
│     ▼                                               │
│  6. Connection Established                         │
│     │                                               │
│     ▼                                               │
│  7. Execute Commands                               │
│     │                                               │
│     ▼                                               │
│  8. Close Connection                               │
│                                                     │
└─────────────────────────────────────────────────────┘
```

**Authentication Methods**:
1. **Password Authentication**:
   - Password encrypted in database
   - Decrypted at connection time
   - Passed to paramiko for authentication

2. **Key Authentication**:
   - Private key stored in Vault
   - Retrieved securely at connection time
   - Passphrase support (if key is encrypted)
   - Public key derived for display purposes

**Security Features**:
- Host key verification prevents MITM attacks
- Failed connection attempts logged
- Connection timeouts prevent hanging connections
- Secure credential storage

---

## Security Architecture

### Multi-Layer Security Model

```
┌─────────────────────────────────────────────────────┐
│               Security Layers                       │
├─────────────────────────────────────────────────────┤
│                                                     │
│  Layer 1: Odoo Authentication                      │
│  - User login (database, OAuth, LDAP, etc.)        │
│  - Session management                              │
│  - MFA support (via Odoo modules)                  │
│                                                     │
│  Layer 2: Odoo Authorization (RBAC)               │
│  - Security groups:                                │
│    • Tower Administrator                          │
│    • Tower Manager                                │
│    • Tower User                                   │
│  - Record rules (row-level security)              │
│  - Field-level access control                     │
│                                                     │
│  Layer 3: Model-Level Security                    │
│  - ir.model.access (CRUD permissions)             │
│  - Record rules by owner/creator                  │
│  - Custom access rules                            │
│                                                     │
│  Layer 4: Vault Encryption                        │
│  - Encrypted storage for secrets                  │
│  - In-memory decryption only                      │
│  - No plain text in logs                          │
│                                                     │
│  Layer 5: SSH Security                            │
│  - Host key verification                          │
│  - Encrypted SSH connections                      │
│  - Key-based authentication preferred             │
│  - Connection timeout controls                    │
│                                                     │
│  Layer 6: Audit Logging                           │
│  - All actions logged                             │
│  - User tracking                                  │
│  - Timestamp recording                            │
│  - Immutable logs                                 │
│                                                     │
└─────────────────────────────────────────────────────┘
```

### Security Groups

| Group | Permissions | Use Case |
|-------|-------------|----------|
| **Tower Administrator** | Full access to all records and settings | System administrators |
| **Tower Manager** | Create/edit servers, commands, flight plans | DevOps team leads |
| **Tower User** | Execute commands and flight plans | Developers, operators |
| **Tower YAML Manager** | Import/export YAML configurations | Configuration managers |

---

## Integration Points

### Internal Odoo Integrations

1. **Mail Module Integration**:
   - Activity tracking on records
   - Email notifications
   - Message threading
   - Follower system

2. **Queue Job Integration**:
   - Background job execution
   - Job monitoring
   - Retry logic
   - Job prioritization

3. **Web Notification Integration**:
   - Real-time browser notifications
   - Success/failure alerts
   - Custom notification messages

### External Integrations

1. **Git Integration**:
   - Clone repositories
   - Pull updates
   - Manage credentials
   - Branch management

2. **AWS EC2 Integration**:
   - Instance lifecycle management
   - Start/stop instances
   - Instance information retrieval
   - Integration with server records

3. **OVH Integration**:
   - Cloud instance management
   - API-based control
   - Instance provisioning

4. **Webhook Integration**:
   - Inbound webhooks
   - Trigger automation
   - External system integration
   - Event-driven architecture

---

## Extensibility Architecture

### Extension Points

Cetmix Tower is designed for extensibility:

1. **Model Inheritance**:
   - All models can be extended via Odoo's inheritance
   - Add custom fields
   - Override methods
   - Add new behaviors

2. **View Customization**:
   - All views can be inherited and modified
   - Add custom buttons and actions
   - Modify layouts
   - Add custom widgets

3. **Custom Command Types**:
   - Create new command execution types
   - Implement custom runners
   - Add specialized commands

4. **Flight Plan Actions**:
   - Add custom conditional actions
   - Implement custom logic
   - Extend action types

5. **Integration Modules**:
   - Create new integration modules (like AWS, OVH)
   - Add API integrations
   - Implement custom protocols

### Plugin Architecture

```
Base Module (cetmix_tower_server)
        │
        ├─ Provides hooks for extensions
        ├─ Defines abstract methods
        └─ Implements core functionality
        │
        ▼
Extension Modules
        │
        ├─ Inherit base models
        ├─ Implement hooks
        ├─ Add custom features
        └─ Integrate with external systems
```

---

## Performance Considerations

### Scalability

- **Connection Pooling**: Reusable SSH connections reduce overhead
- **Queue Jobs**: Long-running tasks don't block UI
- **Pagination**: Large result sets are paginated
- **Database Indexing**: Proper indexes on frequently queried fields
- **Lazy Loading**: Data loaded only when needed

### Optimization Strategies

1. **Command Execution**:
   - Parallel execution when possible
   - Connection reuse for multiple commands
   - Timeout controls prevent hanging

2. **Flight Plans**:
   - Conditional execution skips unnecessary steps
   - Error handling prevents cascade failures
   - Progress tracking for long-running plans

3. **Logging**:
   - Asynchronous log writing
   - Automatic log cleanup
   - Configurable retention periods

4. **Database**:
   - Indexes on key fields (server_id, create_date, etc.)
   - Archive old logs to reduce active data
   - Regular vacuum and analyze

---

## Disaster Recovery Architecture

### Backup Strategy

**Data to Backup**:
1. PostgreSQL database (all Tower data)
2. Filestore (if custom files stored)
3. Configuration files

**Recovery Points**:
- All server configurations
- All commands and flight plans
- All execution history
- Vault contents (encrypted)
- User permissions

### High Availability Considerations

For production deployments:
1. **Odoo HA**: Run multiple Odoo instances behind load balancer
2. **Database HA**: PostgreSQL replication
3. **Filestore HA**: Shared filestore or object storage
4. **Queue Jobs**: Multiple queue job workers

---

## Next Steps

- [Technology Stack](03-technology-stack.md) - Detailed technology information
- [Module List](04-module-list.md) - Complete module reference
- [Installation Guide](../02-getting-started/01-installation.md) - Set up your instance

---

**Last Updated**: 2025-11-16
**Version**: 1.0
**Maintained By**: E-Global SCM Development Team

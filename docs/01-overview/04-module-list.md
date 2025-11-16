---
title: "Module List"
version: "1.0"
last_updated: "2025-11-16"
maintained_by: "E-Global SCM Development Team"
status: "Active"
---

# Module List

## Overview

Cetmix Tower consists of 9 interconnected modules that work together to provide comprehensive server management and automation capabilities. The modules are designed with a modular architecture, allowing you to install only the components you need while maintaining the flexibility to add more functionality later.

---

## Complete Module Table

| Module Name | Version | Type | Status | Dependencies |
|-------------|---------|------|--------|--------------|
| `cetmix_tower` | 17.0.2.0.0 | Meta | Beta | 5 core modules |
| `cetmix_tower_server` | 17.0.2.0.5 | Core | Beta | mail, rpc_helper |
| `cetmix_tower_server_queue` | 17.0.1.2.3 | Core | Beta | cetmix_tower_server, queue_job |
| `cetmix_tower_server_notify_backend` | 17.0.1.1.0 | Core | Beta | cetmix_tower_server, web_notify |
| `cetmix_tower_yaml` | 17.0.2.0.1 | Core | Beta | cetmix_tower_server |
| `cetmix_tower_git` | 17.0.1.0.5 | Core | Beta | cetmix_tower_yaml |
| `cetmix_tower_webhook` | 17.0.1.0.0 | Core | Beta | cetmix_tower_yaml |
| `cetmix_tower_aws` | 17.0.1.0.0 | Optional | Beta | cetmix_tower_server |
| `cetmix_tower_ovh` | 17.0.1.0.0 | Optional | Beta | cetmix_tower_server |

---

## Module Details

### 1. cetmix_tower (Meta Module)

**Technical Name**: `cetmix_tower`
**Version**: 17.0.2.0.0
**Type**: Meta Module
**Category**: Productivity
**License**: AGPL-3.0
**Auto-Install**: No
**Application**: Yes

#### Description

The main Cetmix Tower module is a meta-module designed to simplify installation. It has no functionality of its own but declares dependencies on all recommended core modules. Installing this module will install a complete, fully-functional Cetmix Tower system.

#### Dependencies

**Odoo Modules**:
- `cetmix_tower_server` - Core server management
- `cetmix_tower_server_queue` - Asynchronous execution
- `cetmix_tower_server_notify_backend` - Browser notifications
- `cetmix_tower_git` - Git integration
- `cetmix_tower_webhook` - Webhook support

**Python Packages**: None (all in dependent modules)

#### Key Features

- One-click installation of recommended modules
- Ensures compatible versions of all components
- Simplifies deployment process

#### When to Install

Install this module if you want a complete Cetmix Tower installation with all recommended features. If you need only specific functionality, install individual modules instead.

#### Related Documentation

- [Installation Guide](../02-getting-started/01-installation.md)
- [System Architecture](02-system-architecture.md)

---

### 2. cetmix_tower_server (Core Module)

**Technical Name**: `cetmix_tower_server`
**Version**: 17.0.2.0.5
**Type**: Core Module
**Category**: Productivity
**License**: AGPL-3.0
**Auto-Install**: No
**Application**: No

#### Description

The foundation module of Cetmix Tower, providing all core functionality for server management, command execution, flight plans, file management, variable system, and the Cetmix Tower Vault for secure secret storage.

#### Dependencies

**Odoo Modules**:
- `mail` - Odoo's messaging and activity system
- `rpc_helper` - Remote procedure call utilities

**Python Packages**:
- `paramiko<4` - SSH2 protocol implementation
- `tldextract` - Domain component extraction
- `dnspython` - DNS toolkit

#### Key Features

**Server Management**:
- Server CRUD operations
- SSH connection management
- Server templates for quick provisioning
- Operating system profiles
- Server tagging and organization
- Connection testing and host key verification
- Server activity logging

**Command Execution**:
- SSH command execution
- Python code execution with Odoo context
- Variable substitution system
- Secret injection from Vault
- Sudo support
- Parallel execution control
- Command templates
- Timeout management
- Comprehensive execution logging

**Flight Plans (Workflows)**:
- Sequential command execution
- Conditional logic and branching
- Error handling strategies
- Nested plan execution
- Multi-server execution
- Variable passing between steps
- Real-time execution tracking
- Flight plan templates

**File Management**:
- Bidirectional file sync (push/pull)
- File templates with Jinja2
- Automatic periodic sync
- Binary and text file support
- Version tracking via references
- Permission management
- Batch operations

**Variable System**:
- Variable definitions (char, text, selection, boolean)
- Global and server-specific values
- Variable options for selections
- Custom values at runtime
- Variable references in commands

**Cetmix Tower Vault**:
- Encrypted secret storage
- SSH key management
- Secret injection in commands
- Automatic secret masking in logs
- Key-value pair storage
- Reference-based access

**Scheduled Tasks**:
- Cron-based scheduling
- Multi-server task execution
- Custom variable values per task
- Execution history tracking
- Task templates

**Security**:
- Security groups (Administrator, Manager, User)
- Record-level access rules
- Field-level security
- Audit logging

#### Data Models

**Core Models** (19 models):
- `cx.tower.server` - Server records
- `cx.tower.server.template` - Server templates
- `cx.tower.server.log` - Server activity logs
- `cx.tower.os` - Operating system profiles
- `cx.tower.tag` - Tags for organization
- `cx.tower.command` - Command definitions
- `cx.tower.command.log` - Command execution logs
- `cx.tower.plan` - Flight plan definitions
- `cx.tower.plan.line` - Flight plan steps
- `cx.tower.plan.line.action` - Conditional actions
- `cx.tower.plan.log` - Flight plan execution logs
- `cx.tower.file` - File records
- `cx.tower.file.template` - File templates
- `cx.tower.variable` - Variable definitions
- `cx.tower.variable.value` - Variable values
- `cx.tower.variable.option` - Selection options
- `cx.tower.key` - Vault key groups
- `cx.tower.key.value` - Vault secret values
- `cx.tower.scheduled.task` - Scheduled tasks
- `cx.tower.scheduled.task.cv` - Custom variable values for tasks
- `cx.tower.shortcut` - Quick access shortcuts

#### Views and Interface

**Menus**:
- Servers
- Commands
- Flight Plans
- Files
- Variables
- Vault
- Scheduled Tasks
- Logs
- Configuration
- Settings

**View Types**:
- Tree views (lists)
- Form views (detail)
- Kanban views (cards)
- Search views (filters)
- Calendar views (schedules)
- Graph views (analytics)

#### Wizards

- Command Run Wizard - Execute commands with custom parameters
- Flight Plan Run Wizard - Execute flight plans with options
- Server Template Creation Wizard - Create servers from templates
- Host Key Wizard - Retrieve and save SSH host keys

#### Cron Jobs

- File sync scheduler - Periodic file synchronization
- Log cleanup - Automatic log retention management

#### Configuration

**Settings** (`res.config.settings` extension):
- Default SSH port
- Connection timeout
- Default command timeout
- Log retention periods
- Secret masking patterns
- Parallel execution limits

#### When to Install

This is the **required core module** for Cetmix Tower. All other Tower modules depend on this module. Install this if you want just the core functionality without optional features.

#### Related Documentation

- [Server Management Guide](../04-feature-guides/server-management/README.md)
- [Command Execution Guide](../04-feature-guides/command-execution/README.md)
- [Flight Plans Guide](../04-feature-guides/flight-plans/README.md)
- [Module Reference](../05-module-references/cetmix-tower-server.md)

---

### 3. cetmix_tower_server_queue (Queue Integration)

**Technical Name**: `cetmix_tower_server_queue`
**Version**: 17.0.1.2.3
**Type**: Core Module (Optional)
**Category**: Productivity
**License**: AGPL-3.0
**Auto-Install**: Yes (when queue_job is available)
**Application**: No

#### Description

Integrates Cetmix Tower with the OCA `queue_job` module to enable asynchronous command and flight plan execution. This prevents long-running operations from blocking the Odoo interface and allows users to continue working while commands execute in the background.

#### Dependencies

**Odoo Modules**:
- `cetmix_tower_server` - Core Tower module
- `queue_job` - OCA job queue framework

**Python Packages**: None (all in queue_job)

#### Key Features

**Asynchronous Execution**:
- Commands execute in background jobs
- Flight plans run asynchronously
- Non-blocking UI operations
- Progress tracking in job queue

**Job Management**:
- Job status monitoring
- Job cancellation capability
- Retry on failure
- Job prioritization
- Job history

**Performance Benefits**:
- Multiple commands can execute simultaneously
- UI remains responsive during long operations
- Better resource utilization
- Scalable execution

#### Enhancements to Core Models

**Command Log**:
- Link to queue job
- Job status display
- Quick access to job details

**Flight Plan Log**:
- Link to queue job
- Real-time status updates

**File Operations**:
- Asynchronous file sync
- Background file uploads/downloads

#### When to Install

Install this module if:
- You execute long-running commands (> 10 seconds)
- You need to run multiple commands simultaneously
- You want non-blocking UI during operations
- You have the `queue_job` module available

**Note**: This module auto-installs when `queue_job` is present in your Odoo instance.

#### Configuration

Configure queue jobs via Odoo's standard queue_job settings:
- Number of workers
- Job channels
- Job retry policies
- Job failure handling

#### Related Documentation

- [Queue Job Module (OCA)](https://github.com/OCA/queue)
- [Asynchronous Execution Guide](../04-feature-guides/command-execution/03-async-execution.md)

---

### 4. cetmix_tower_server_notify_backend (Notification Integration)

**Technical Name**: `cetmix_tower_server_notify_backend`
**Version**: 17.0.1.1.0
**Type**: Core Module (Optional)
**Category**: Productivity
**License**: AGPL-3.0
**Auto-Install**: Yes (when web_notify is available)
**Application**: No

#### Description

Integrates Cetmix Tower with the OCA `web_notify` module to provide real-time browser notifications for command executions, flight plan completions, and other Tower events. Notifications appear as non-intrusive pop-ups in the Odoo interface.

#### Dependencies

**Odoo Modules**:
- `cetmix_tower_server` - Core Tower module
- `web_notify` - OCA web notification framework

**Python Packages**: None

#### Key Features

**Real-Time Notifications**:
- Command completion notifications
- Flight plan status updates
- Success/failure alerts
- Error notifications
- Custom notification messages

**Notification Types**:
- Success notifications (green)
- Warning notifications (yellow)
- Error notifications (red)
- Info notifications (blue)

**User Experience**:
- Non-blocking notifications
- Auto-dismiss after timeout
- Click to view details
- Notification history
- User preferences for notification types

#### Notification Triggers

- Command execution complete
- Command execution failed
- Flight plan completed
- Flight plan failed
- Scheduled task completed
- File sync completed
- Server connection failed
- Custom trigger points (extensible)

#### When to Install

Install this module if:
- Users execute commands and want immediate feedback
- You want to improve user experience with real-time updates
- You have the `web_notify` module available

**Note**: This module auto-installs when `web_notify` is present in your Odoo instance.

#### Configuration

Notifications are automatically enabled upon installation. Users can configure notification preferences in their user settings (if extended).

#### Related Documentation

- [Web Notify Module (OCA)](https://github.com/OCA/web)
- [User Interface Guide](../03-user-guides/01-navigation.md)

---

### 5. cetmix_tower_yaml (YAML Import/Export)

**Technical Name**: `cetmix_tower_yaml`
**Version**: 17.0.2.0.1
**Type**: Core Module
**Category**: Productivity
**License**: AGPL-3.0
**Auto-Install**: No
**Application**: No

#### Description

Provides YAML import and export capabilities for Cetmix Tower configurations. This module enables "Configuration as Code" workflows, allowing you to version control your server configurations, commands, and flight plans in Git repositories and deploy them across different Tower instances.

#### Dependencies

**Odoo Modules**:
- `cetmix_tower_server` - Core Tower module

**Python Packages**:
- `pyyaml` - YAML parser and serializer

#### Key Features

**Export Capabilities**:
- Export servers to YAML
- Export server templates to YAML
- Export commands to YAML
- Export flight plans to YAML
- Export files and file templates to YAML
- Export variables to YAML
- Export scheduled tasks to YAML
- Export SSH keys to YAML
- Export tags to YAML
- Batch export of multiple objects
- Export with dependencies

**Import Capabilities**:
- Import from YAML files
- Validate YAML structure before import
- Create or update records
- Import with dependencies
- Conflict resolution options
- Dry-run mode (preview changes)
- Import logs and error reporting

**YAML Manifest System**:
- Define manifest templates
- Track manifest authors
- Version control integration
- Standardized YAML structure

**Configuration as Code**:
- Store configurations in Git
- Version control your infrastructure
- Deploy same configuration across environments
- Share configurations between instances
- Template-based provisioning

#### YAML Structure Example

```yaml
# Server export
servers:
  - name: "Production Web Server"
    reference: "prod-web-01"
    ssh_host: "192.168.1.100"
    ssh_port: 22
    ssh_username: "deploy"
    ssh_auth_mode: "key"
    tags:
      - "production"
      - "web"
    variables:
      app_path: "/opt/myapp"
      python_version: "3.11"

# Command export
commands:
  - name: "Deploy Application"
    reference: "deploy-app"
    code: |
      cd ${app_path}
      git pull origin main
      pip install -r requirements.txt
      systemctl restart myapp
    sudo: true
    timeout: 300

# Flight plan export
flight_plans:
  - name: "Full Deployment"
    reference: "full-deploy"
    lines:
      - sequence: 10
        command: "deploy-app"
        on_success: "continue"
        on_failure: "stop"
      - sequence: 20
        command: "verify-deployment"
        on_success: "continue"
        on_failure: "rollback"
```

#### Wizards

**Export Wizard**:
- Select objects to export
- Choose export options
- Download YAML file
- Preview YAML content

**Import Wizard**:
- Upload YAML file
- Preview changes
- Map secrets (secure import)
- Execute import
- View import results

#### Security Groups

- **Tower YAML Manager**: Full import/export access
- Integrated with core Tower security groups

#### When to Install

Install this module if you need:
- Configuration as code workflows
- Version control for Tower configurations
- Multi-environment deployments (dev, staging, prod)
- Configuration templates for quick setup
- Sharing configurations between Tower instances
- Backup and restore via YAML

#### Related Documentation

- [YAML Import/Export Guide](../04-feature-guides/yaml-import-export/README.md)
- [Configuration as Code Workflow](../08-development-workflows/configuration-as-code.md)
- [Module Reference](../05-module-references/cetmix-tower-yaml.md)

---

### 6. cetmix_tower_git (Git Integration)

**Technical Name**: `cetmix_tower_git`
**Version**: 17.0.1.0.5
**Type**: Core Module
**Category**: Productivity
**License**: AGPL-3.0
**Auto-Install**: No
**Application**: No

#### Description

Provides Git repository management capabilities within Cetmix Tower. This module allows you to manage Git repositories on remote servers, including cloning, pulling updates, and managing Git credentials. It extends Tower's file management with Git-aware features.

#### Dependencies

**Odoo Modules**:
- `cetmix_tower_yaml` - YAML module (extends it)

**Python Packages**: None (Git commands executed on remote servers)

#### Key Features

**Git Project Management**:
- Define Git projects
- Store repository URLs
- Manage branches
- Configure Git credentials
- Track project metadata

**Git Source Definitions**:
- Define Git sources (repositories)
- Multiple sources per project
- Source authentication
- Branch/tag specification

**Git Remote Management**:
- Configure Git remotes
- Multiple remotes per repository
- Remote URL management
- Remote authentication

**Integration with Tower Objects**:
- Link servers to Git projects
- Link files to Git repositories
- Link file templates to Git sources
- Git operations in flight plans

**Git Operations**:
- Clone repositories
- Pull updates
- Checkout branches/tags
- Reset to specific commits
- Manage Git credentials securely

#### Data Models

- `cx.tower.git.project` - Git project definitions
- `cx.tower.git.source` - Git source repositories
- `cx.tower.git.remote` - Git remotes configuration
- `cx.tower.git.project.rel` - Project-server relationships
- `cx.tower.git.project.file.template.rel` - Project-template relationships

#### Views and Menus

**New Menus**:
- Git Projects
- Git Sources
- Git Remotes

**Extended Views**:
- Server form (Git projects tab)
- File form (Git source field)
- File template form (Git source field)
- Flight plan lines (Git-specific actions)

#### Common Use Cases

1. **Application Deployment**:
   - Clone application repository to servers
   - Pull latest changes during deployment
   - Checkout specific branches per environment

2. **Configuration Management**:
   - Store configuration files in Git
   - Deploy configurations to servers
   - Track configuration changes

3. **Infrastructure as Code**:
   - Manage infrastructure scripts in Git
   - Deploy scripts to servers
   - Version control infrastructure changes

#### Example Flight Plan with Git

```yaml
flight_plan:
  name: "Deploy from Git"
  lines:
    - sequence: 10
      command: "git_clone"
      code: |
        git clone ${git_repo_url} ${app_path}
    - sequence: 20
      command: "git_pull"
      code: |
        cd ${app_path}
        git pull origin ${git_branch}
    - sequence: 30
      command: "restart_app"
      code: |
        systemctl restart myapp
```

#### When to Install

Install this module if you:
- Deploy applications from Git repositories
- Manage infrastructure code in Git
- Need version-controlled configuration deployment
- Want to integrate Git workflows with Tower automation

#### Related Documentation

- [Git Integration Guide](../04-feature-guides/git-integration/README.md)
- [Deployment Workflows](../08-development-workflows/deployment-workflows.md)
- [Module Reference](../05-module-references/cetmix-tower-git.md)

---

### 7. cetmix_tower_webhook (Webhook Integration)

**Technical Name**: `cetmix_tower_webhook`
**Version**: 17.0.1.0.0
**Type**: Core Module
**Category**: Productivity
**License**: AGPL-3.0
**Auto-Install**: No
**Application**: No

#### Description

Enables webhook-based automation in Cetmix Tower. This module allows external systems to trigger Tower commands and flight plans via HTTP webhooks, enabling event-driven automation and integration with CI/CD pipelines, Git hosting platforms, and custom applications.

#### Dependencies

**Odoo Modules**:
- `cetmix_tower_yaml` - YAML module

**Python Packages**: None

#### Key Features

**Webhook Endpoints**:
- Create webhook endpoints with unique URLs
- Accept POST requests with JSON payloads
- Parse webhook data
- Extract variables from payloads
- Trigger commands or flight plans

**Webhook Authentication**:
- Token-based authentication
- Signature verification (HMAC)
- IP whitelisting (extensible)
- Multiple authenticators per webhook

**Variable Extraction**:
- Extract data from JSON payloads
- Map webhook fields to Tower variables
- Support for nested JSON paths
- Default values for missing fields

**Webhook Logging**:
- Log all webhook requests
- Store request payloads
- Track execution results
- Debug failed webhooks

**Integration Options**:
- Execute specific commands
- Execute flight plans
- Target specific servers or server groups
- Pass custom variables from webhook data

#### Data Models

- `cx.tower.webhook` - Webhook definitions
- `cx.tower.webhook.authenticator` - Authentication methods
- `cx.tower.webhook.log` - Webhook request logs

#### Webhook URL Format

```
POST https://your-odoo-instance.com/tower/webhook/<webhook_reference>

Headers:
  Content-Type: application/json
  X-Webhook-Token: <authentication_token>

Body:
{
  "event": "deployment",
  "repository": "myapp",
  "branch": "main",
  "commit": "abc123def456",
  "author": "john@example.com"
}
```

#### Configuration Settings

**System Parameters**:
- Webhook secret key for signature verification
- Webhook timeout settings
- Log retention for webhook logs

#### Common Integration Examples

**1. GitHub Webhook**:
```yaml
webhook:
  name: "GitHub Push Webhook"
  reference: "github-push"
  authentication: "token"
  token: "secret-token-123"
  variable_mappings:
    - webhook_field: "repository.name"
      tower_variable: "repo_name"
    - webhook_field: "ref"
      tower_variable: "branch"
  action: "execute_flight_plan"
  flight_plan: "deploy-from-github"
```

**2. GitLab CI/CD**:
```yaml
webhook:
  name: "GitLab Pipeline Webhook"
  reference: "gitlab-pipeline"
  authentication: "token"
  variable_mappings:
    - webhook_field: "project.name"
      tower_variable: "project"
    - webhook_field: "object_attributes.ref"
      tower_variable: "branch"
  action: "execute_command"
  command: "deploy-application"
```

**3. Custom Application**:
```yaml
webhook:
  name: "Custom App Webhook"
  reference: "custom-app"
  authentication: "signature"
  variable_mappings:
    - webhook_field: "environment"
      tower_variable: "env"
    - webhook_field: "version"
      tower_variable: "app_version"
  action: "execute_flight_plan"
  flight_plan: "custom-deployment"
  target_servers: ["production"]
```

#### Views and Menus

**New Menus**:
- Webhooks
- Webhook Authenticators
- Webhook Logs

**Features**:
- Webhook configuration form
- Test webhook button
- View recent webhook logs
- Webhook URL copy button

#### When to Install

Install this module if you need:
- Event-driven automation
- Integration with Git hosting platforms (GitHub, GitLab, Bitbucket)
- CI/CD pipeline integration
- External system triggers for deployments
- Automation triggered by external events
- Custom application integration

#### Security Considerations

- Always use authentication (token or signature)
- Use HTTPS in production
- Regularly rotate webhook tokens
- Monitor webhook logs for suspicious activity
- Consider IP whitelisting for sensitive webhooks

#### Related Documentation

- [Webhook Integration Guide](../04-feature-guides/webhook-integration/README.md)
- [CI/CD Integration Guide](../07-technical-guides/cicd-integration.md)
- [Module Reference](../05-module-references/cetmix-tower-webhook.md)

---

### 8. cetmix_tower_aws (AWS Integration) [Optional]

**Technical Name**: `cetmix_tower_aws`
**Version**: 17.0.1.0.0
**Type**: Optional Module
**Category**: Productivity
**License**: AGPL-3.0
**Auto-Install**: No
**Application**: No

#### Description

Integrates Cetmix Tower with Amazon Web Services (AWS) EC2, enabling management of AWS EC2 instances directly from Tower. This module allows you to sync EC2 instances to Tower server records, and perform instance lifecycle operations (start, stop, terminate) from the Tower interface.

#### Dependencies

**Odoo Modules**:
- `cetmix_tower_server` - Core Tower module

**Python Packages**:
- `boto3` - AWS SDK for Python

#### Key Features

**EC2 Instance Management**:
- Sync EC2 instances to Tower servers
- View instance metadata in Tower
- Start/stop instances from Tower
- Terminate instances
- Retrieve instance information
- Tag-based instance filtering

**AWS Authentication**:
- AWS access key and secret key authentication
- IAM role support (when running on EC2)
- Multiple AWS accounts/regions support

**Integration with Tower**:
- EC2 instances appear as Tower servers
- Use Tower commands on EC2 instances
- Include EC2 instances in flight plans
- Schedule EC2 instance operations

**Cost Optimization**:
- Schedule automatic instance start/stop
- Reduce costs by stopping unused instances
- Track instance usage via Tower logs

#### Use Cases

1. **Development Environment Management**:
   - Stop dev servers during non-business hours
   - Start servers on-demand
   - Reduce AWS costs

2. **Testing Infrastructure**:
   - Provision test instances
   - Run tests via Tower flight plans
   - Terminate instances after testing

3. **Disaster Recovery**:
   - Quickly start backup instances
   - Automate failover procedures
   - Test DR scenarios

4. **Multi-Cloud Management**:
   - Manage AWS and on-premise servers from one interface
   - Unified command execution across clouds
   - Consistent automation workflows

#### Configuration

**AWS Credentials**:
- Configure in Tower settings or environment variables
- Support for multiple AWS regions
- IAM role-based authentication

**Instance Sync**:
- Manual or scheduled sync
- Filter instances by tags
- Map EC2 metadata to Tower fields

#### When to Install

Install this module if you:
- Use AWS EC2 for infrastructure
- Want to manage EC2 instances from Tower
- Need cost optimization through scheduled start/stop
- Want unified management of AWS and other servers
- Integrate EC2 with Tower automation workflows

#### Related Documentation

- [AWS Integration Guide](../04-feature-guides/cloud-integration/aws.md)
- [Cloud Integration Overview](../04-feature-guides/cloud-integration/README.md)
- [Module Reference](../05-module-references/cetmix-tower-aws.md)

---

### 9. cetmix_tower_ovh (OVH Integration) [Optional]

**Technical Name**: `cetmix_tower_ovh`
**Version**: 17.0.1.0.0
**Type**: Optional Module
**Category**: Productivity
**License**: AGPL-3.0
**Auto-Install**: No
**Application**: No
**Maintainer**: GSLabIt (Giovanni Serra)

#### Description

Integrates Cetmix Tower with OVH cloud services, enabling management of OVH Public Cloud and VPS instances from Tower. This module is particularly useful for European organizations using OVH as their cloud provider, offering the same integration capabilities as the AWS module but for OVH infrastructure.

#### Dependencies

**Odoo Modules**:
- `cetmix_tower_server` - Core Tower module

**Python Packages**:
- `ovh` - OVH API Python wrapper

#### Key Features

**OVH Instance Management**:
- Sync OVH instances to Tower servers
- View instance metadata
- Instance lifecycle operations
- Manage OVH VPS and Public Cloud
- Multi-region support

**OVH API Authentication**:
- Application key authentication
- Consumer key management
- Multiple OVH accounts support
- Region-specific endpoints

**Integration with Tower**:
- OVH instances as Tower servers
- Execute commands on OVH instances
- Include in flight plans
- Schedule OVH operations

#### Use Cases

1. **European Data Sovereignty**:
   - Manage GDPR-compliant infrastructure
   - Keep data in European data centers
   - OVH-specific compliance requirements

2. **OVH-Specific Services**:
   - Manage OVH Public Cloud
   - Control OVH VPS instances
   - Integrate with OVH ecosystem

3. **Multi-Region Deployment**:
   - Manage instances across OVH regions
   - European multi-region setups
   - Disaster recovery across OVH DCs

#### Configuration

**OVH API Credentials**:
- Application key
- Application secret
- Consumer key
- Endpoint (ovh-eu, ovh-ca, etc.)

**Instance Sync**:
- Sync OVH instances to Tower
- Filter by project ID
- Map OVH metadata to Tower

#### When to Install

Install this module if you:
- Use OVH for cloud infrastructure
- Need European cloud provider integration
- Want unified OVH and other server management
- Require GDPR-compliant infrastructure management
- Integrate OVH with Tower workflows

#### Related Documentation

- [OVH Integration Guide](../04-feature-guides/cloud-integration/ovh.md)
- [Cloud Integration Overview](../04-feature-guides/cloud-integration/README.md)
- [Module Reference](../05-module-references/cetmix-tower-ovh.md)

---

## Module Dependency Graph

### Visual Dependency Hierarchy

```
┌─────────────────────────────────────────────┐
│         cetmix_tower (Meta Module)          │
│              Version: 17.0.2.0.0            │
└─────────────────┬───────────────────────────┘
                  │
      ┌───────────┼───────────┬────────────────┬───────────┐
      │           │           │                │           │
      ▼           ▼           ▼                ▼           ▼
┌──────────┐ ┌────────┐ ┌─────────┐ ┌──────────────┐ ┌─────────┐
│  server  │ │  git   │ │ webhook │ │    queue     │ │ notify  │
│  _queue  │ │        │ │         │ │              │ │ backend │
└────┬─────┘ └───┬────┘ └────┬────┘ └──────┬───────┘ └────┬────┘
     │           │           │             │              │
     │      ┌────┴───────────┘             │              │
     │      │                              │              │
     │      ▼                              │              │
     │ ┌─────────┐                        │              │
     │ │  yaml   │                        │              │
     │ └────┬────┘                        │              │
     │      │                             │              │
     └──────┴─────────────────────────────┴──────────────┘
            │
            ▼
  ┌──────────────────────────────────────┐
  │    cetmix_tower_server (Core)        │
  │        Version: 17.0.2.0.5           │
  └──────────────┬───────────────────────┘
                 │
        ┌────────┼────────┐
        │        │        │
        ▼        ▼        ▼
   ┌────────┐ ┌────┐ ┌────────┐
   │  aws   │ │ovh │ │ (ext)  │
   │(opt)   │ │(opt)│ │        │
   └────────┘ └────┘ └────────┘

Legend:
  [Box] = Module
  (opt) = Optional
  (ext) = External Odoo modules
```

### Dependency Table

| Module | Depends On | Type |
|--------|------------|------|
| `cetmix_tower` | server, queue, notify, git, webhook | Meta |
| `cetmix_tower_server` | mail, rpc_helper | Core |
| `cetmix_tower_server_queue` | cetmix_tower_server, queue_job | Add-on |
| `cetmix_tower_server_notify_backend` | cetmix_tower_server, web_notify | Add-on |
| `cetmix_tower_yaml` | cetmix_tower_server | Core |
| `cetmix_tower_git` | cetmix_tower_yaml | Core |
| `cetmix_tower_webhook` | cetmix_tower_yaml | Core |
| `cetmix_tower_aws` | cetmix_tower_server | Optional |
| `cetmix_tower_ovh` | cetmix_tower_server | Optional |

---

## Installation Recommendations

### Minimal Installation

For basic server management without optional features:

```bash
# Install only core module
pip3 install paramiko tldextract dnspython
# In Odoo: Install cetmix_tower_server
```

**Provides**:
- Server management
- Command execution
- Flight plans
- File management
- Variables
- Vault
- Scheduled tasks

### Recommended Installation

For full functionality with recommended features:

```bash
# Install Python dependencies
pip3 install paramiko tldextract dnspython pyyaml

# Install OCA dependencies
# Install queue_job and web_notify from OCA

# In Odoo: Install cetmix_tower meta module
```

**Provides**:
- All core features
- Asynchronous execution
- Browser notifications
- YAML import/export
- Git integration
- Webhook support

### Full Installation

For complete functionality including cloud integrations:

```bash
# Install all Python dependencies
pip3 install paramiko tldextract dnspython pyyaml boto3 ovh

# Install OCA dependencies
# Install queue_job and web_notify from OCA

# In Odoo: Install all modules
```

**Provides**:
- All features from recommended installation
- AWS EC2 integration
- OVH cloud integration

---

## Module Installation Order

When installing manually (not using meta module):

1. **Install external dependencies first**:
   - Python packages via pip
   - OCA modules (queue_job, web_notify, rpc_helper)

2. **Install core module**:
   - `cetmix_tower_server`

3. **Install auto-install modules** (if dependencies present):
   - `cetmix_tower_server_queue` (auto-installs with queue_job)
   - `cetmix_tower_server_notify_backend` (auto-installs with web_notify)

4. **Install YAML module**:
   - `cetmix_tower_yaml`

5. **Install additional core modules**:
   - `cetmix_tower_git`
   - `cetmix_tower_webhook`

6. **Install optional modules** (as needed):
   - `cetmix_tower_aws`
   - `cetmix_tower_ovh`

7. **Install meta module** (optional, for convenience):
   - `cetmix_tower`

---

## Version Compatibility

### Cross-Module Compatibility

All modules in version 17.0.x.x.x are compatible with each other. Mix and match modules based on your needs.

### Odoo Version Support

| Odoo Version | Tower Version | Support Status |
|--------------|---------------|----------------|
| 17.0 | 17.0.x.x.x | ✅ Current (this documentation) |
| 16.0 | Not available | ❌ Requires migration |
| 15.0 | Not available | ❌ Requires migration |
| 14.0 | 14.0.x.x.x | ⚠️ Legacy (separate branch) |

### Upgrade Path

For organizations on older Odoo versions:
1. Plan migration to Odoo 17.0
2. Export configurations using YAML (if available in old version)
3. Migrate Odoo database to 17.0
4. Install Cetmix Tower 17.0 modules
5. Import configurations from YAML

---

## Module Comparison Matrix

| Feature | Required Module | Optional Modules |
|---------|----------------|------------------|
| Server management | cetmix_tower_server | - |
| Command execution | cetmix_tower_server | - |
| Flight plans | cetmix_tower_server | - |
| File management | cetmix_tower_server | - |
| Variables | cetmix_tower_server | - |
| Vault (secrets) | cetmix_tower_server | - |
| Scheduled tasks | cetmix_tower_server | - |
| Asynchronous execution | - | cetmix_tower_server_queue |
| Browser notifications | - | cetmix_tower_server_notify_backend |
| YAML import/export | - | cetmix_tower_yaml |
| Git integration | - | cetmix_tower_git (requires yaml) |
| Webhook integration | - | cetmix_tower_webhook (requires yaml) |
| AWS EC2 management | - | cetmix_tower_aws |
| OVH cloud management | - | cetmix_tower_ovh |

---

## Next Steps

- [Installation Guide](../02-getting-started/01-installation.md) - Install Cetmix Tower
- [Configuration Guide](../02-getting-started/02-configuration.md) - Configure modules
- [Module References](../05-module-references/README.md) - Detailed technical documentation
- [System Architecture](02-system-architecture.md) - Understand how modules work together

---

**Last Updated**: 2025-11-16
**Version**: 1.0
**Maintained By**: E-Global SCM Development Team

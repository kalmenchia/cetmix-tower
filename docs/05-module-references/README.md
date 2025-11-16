---
title: "Module References"
section: "05-module-references"
audience: "Developers, Technical Consultants"
version: "17.0"
last_updated: "2025-11-16"
---

# Module References

Comprehensive technical documentation for all Cetmix Tower modules, including models, views, security, and APIs.

## Purpose

Module references provide detailed technical documentation for developers and technical consultants who need to:

- Understand module structure and dependencies
- Integrate with Tower modules
- Extend or customize modules
- Troubleshoot module issues
- Contribute to module development

## Target Audience

- **Module Developers**: Building new Tower modules
- **Technical Consultants**: Implementing and customizing Tower
- **System Integrators**: Integrating Tower with other systems
- **Contributors**: Contributing to Cetmix Tower project

## How to Use Module References

Each module reference follows a consistent structure:

1. **Module Overview**: Purpose, features, dependencies
2. **Models**: Complete model documentation with fields, methods, relationships
3. **Views**: Form, tree, kanban, calendar views
4. **Security**: Access rights, groups, record rules
5. **Wizards**: Transient models and wizards
6. **Data**: Default data, demo data
7. **API**: Public methods and integration points
8. **Configuration**: Settings and options
9. **Dependencies**: Module and Python dependencies
10. **Examples**: Common usage patterns

---

## Module Organization

Modules are organized into two categories:

### Core Modules

Essential modules required for Tower functionality:

| Module | Description | Status |
|--------|-------------|--------|
| [cetmix_tower](./core-modules/cetmix_tower.md) | Base module, menus, core functionality | Core |
| [cetmix_tower_server](./core-modules/cetmix_tower_server.md) | Server management, commands, plans, files | Core |

**Core modules** provide fundamental Tower functionality and are required for Tower to operate.

---

### Optional Modules

Enhancement modules that add specific features:

| Module | Description | Dependencies |
|--------|-------------|--------------|
| [cetmix_tower_yaml](./optional-modules/cetmix_tower_yaml.md) | YAML-based configuration | cetmix_tower_server |
| [cetmix_tower_server_queue](./optional-modules/cetmix_tower_server_queue.md) | Job queue integration | cetmix_tower_server, queue_job |
| [cetmix_tower_server_notify_backend](./optional-modules/cetmix_tower_server_notify_backend.md) | Backend notifications | cetmix_tower_server |
| [cetmix_tower_git](./optional-modules/cetmix_tower_git.md) | Git integration | cetmix_tower_server |
| [cetmix_tower_webhook](./optional-modules/cetmix_tower_webhook.md) | Webhook support | cetmix_tower_server |
| [cetmix_tower_aws](./optional-modules/cetmix_tower_aws.md) | AWS cloud integration | cetmix_tower_server |
| [cetmix_tower_ovh](./optional-modules/cetmix_tower_ovh.md) | OVH cloud integration | cetmix_tower_server |

**Optional modules** extend Tower with additional capabilities and can be installed as needed.

---

## Module Dependency Graph

```
cetmix_tower (base)
    ↓
cetmix_tower_server (core)
    ↓
    ├─→ cetmix_tower_yaml
    ├─→ cetmix_tower_server_queue
    ├─→ cetmix_tower_server_notify_backend
    ├─→ cetmix_tower_git
    ├─→ cetmix_tower_webhook
    ├─→ cetmix_tower_aws
    └─→ cetmix_tower_ovh
```

---

## Core Modules

### cetmix_tower

**Location:** `/home/user/cetmix-tower/cetmix_tower/`

Base module providing:
- Main menu structure
- Application configuration
- Base models and utilities
- Common dependencies

**Key Features:**
- Application menu
- Base configuration
- Module organization

**See:** [cetmix_tower Module Reference](./core-modules/cetmix_tower.md)

---

### cetmix_tower_server

**Location:** `/home/user/cetmix-tower/cetmix_tower_server/`

Core server management functionality:
- Server configuration and connectivity
- Command execution and logging
- Flight plan orchestration
- File management and synchronization
- Variable management
- SSH key management
- Security and access control

**Key Features:**
- 30+ models for comprehensive management
- SSH-based server communication
- Template rendering with variables
- Vault-based secret storage
- Role-based access control
- Execution logging and auditing

**See:** [cetmix_tower_server Module Reference](./core-modules/cetmix_tower_server.md) ⭐ **Start Here**

---

## Optional Modules

### cetmix_tower_yaml

**Location:** `/home/user/cetmix-tower/cetmix_tower_yaml/`

YAML-based configuration for Infrastructure as Code:
- Export Tower configuration to YAML
- Import configuration from YAML
- Version control Tower setup
- Environment migration

**Use Cases:**
- Infrastructure as Code
- Configuration backup
- Multi-environment management
- Version control integration

**See:** [cetmix_tower_yaml Module Reference](./optional-modules/cetmix_tower_yaml.md)

---

### cetmix_tower_server_queue

**Location:** `/home/user/cetmix-tower/cetmix_tower_server_queue/`

Integration with OCA `queue_job`:
- Asynchronous command execution
- Job queue management
- Better handling of long-running tasks
- Job monitoring and retry

**Dependencies:**
- `cetmix_tower_server`
- `queue_job` (OCA)

**See:** [cetmix_tower_server_queue Module Reference](./optional-modules/cetmix_tower_server_queue.md)

---

### cetmix_tower_server_notify_backend

**Location:** `/home/user/cetmix-tower/cetmix_tower_server_notify_backend/`

Backend notification system:
- Real-time notifications
- Command completion alerts
- Plan execution status
- Error notifications

**See:** [cetmix_tower_server_notify_backend Module Reference](./optional-modules/cetmix_tower_server_notify_backend.md)

---

### cetmix_tower_git

**Location:** `/home/user/cetmix-tower/cetmix_tower_git/`

Git repository integration:
- Clone repositories
- Deploy from Git
- Manage Git projects
- Automated deployments

**See:** [cetmix_tower_git Module Reference](./optional-modules/cetmix_tower_git.md)

---

### cetmix_tower_webhook

**Location:** `/home/user/cetmix-tower/cetmix_tower_webhook/`

Webhook support for external integrations:
- Receive webhooks from external systems
- Trigger Tower actions from webhooks
- Custom webhook handlers
- Authentication and security

**See:** [cetmix_tower_webhook Module Reference](./optional-modules/cetmix_tower_webhook.md)

---

### cetmix_tower_aws

**Location:** `/home/user/cetmix-tower/cetmix_tower_aws/`

AWS cloud integration:
- EC2 instance management
- AWS-specific commands
- Cloud resource discovery

**See:** [cetmix_tower_aws Module Reference](./optional-modules/cetmix_tower_aws.md)

---

### cetmix_tower_ovh

**Location:** `/home/user/cetmix-tower/cetmix_tower_ovh/`

OVH cloud integration:
- OVH server management
- OVH-specific commands
- Cloud resource management

**See:** [cetmix_tower_ovh Module Reference](./optional-modules/cetmix_tower_ovh.md)

---

## Module Reference Template

When documenting a new module, follow this template:

```markdown
# Module: module_name

## Overview
- Purpose
- Features
- Version
- License

## Installation
- Dependencies
- Installation steps
- Configuration

## Models
- Model list
- Field documentation
- Methods documentation
- Relationships

## Views
- View types
- Screenshots/examples

## Security
- Groups
- Access rights
- Record rules

## Configuration
- Settings
- Options

## API
- Public methods
- Integration points
- Examples

## Examples
- Common usage
- Code snippets

## Troubleshooting
- Common issues
- Solutions
```

---

## Quick Navigation

### By Use Case

**I want to...**

- **Manage servers remotely** → [cetmix_tower_server](./core-modules/cetmix_tower_server.md)
- **Use YAML configuration** → [cetmix_tower_yaml](./optional-modules/cetmix_tower_yaml.md)
- **Deploy from Git** → [cetmix_tower_git](./optional-modules/cetmix_tower_git.md)
- **Integrate with webhooks** → [cetmix_tower_webhook](./optional-modules/cetmix_tower_webhook.md)
- **Manage AWS instances** → [cetmix_tower_aws](./optional-modules/cetmix_tower_aws.md)
- **Handle long-running tasks** → [cetmix_tower_server_queue](./optional-modules/cetmix_tower_server_queue.md)

### By Topic

**Understanding...**

- **Models and data structure** → [Technical Guide: Models and ORM](../07-technical-guides/01-models-and-orm.md)
- **Security and access** → [Technical Guide: Security](../07-technical-guides/03-security-and-access.md)
- **UI and views** → [Technical Guide: Views and UI](../07-technical-guides/02-views-and-ui.md)
- **API integration** → [API References](../06-api-references/README.md)

---

## Contributing

### Adding Module Documentation

To add documentation for a new module:

1. **Create module reference file:**
   - Core modules: `core-modules/module_name.md`
   - Optional modules: `optional-modules/module_name.md`

2. **Follow the template** (see above)

3. **Update this README** with module link and description

4. **Cross-reference** from related documentation

### Documentation Standards

- Use absolute file paths
- Include code examples
- Reference actual files and line numbers
- Test all code examples
- Keep examples up-to-date

---

## Related Documentation

- [Technical Guides](../07-technical-guides/README.md): In-depth technical topics
- [API References](../06-api-references/README.md): API documentation
- [Feature Guides](../04-feature-guides/README.md): Feature-specific guides
- [Development Workflows](../08-development-workflows/README.md): Development processes

---

## Support and Resources

- **Official Website:** https://tower.cetmix.com
- **Documentation:** https://tower.cetmix.com/documentation
- **Source Code:** https://github.com/cetmix/cetmix-tower
- **Issue Tracker:** https://github.com/cetmix/cetmix-tower/issues

---

**Navigation:**
- [← Documentation Home](../README.md)
- [Next: cetmix_tower_server Reference →](./core-modules/cetmix_tower_server.md)

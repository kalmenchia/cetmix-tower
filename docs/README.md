---
title: "Cetmix Tower Documentation"
version: "1.0"
last_updated: "2025-11-16"
maintained_by: "E-Global SCM Development Team"
status: "Active"
---

# Cetmix Tower Documentation

Welcome to the comprehensive documentation for **Cetmix Tower** - an Odoo-based server management and automation platform.

## Overview

**Cetmix Tower** is a powerful SAAS server application management tool built on Odoo 17.0. It provides comprehensive capabilities for managing multiple servers, automating deployment workflows, executing commands remotely, managing files, and orchestrating complex automation sequences called "Flight Plans".

**Current Version**: 17.0.2.0.0
**License**: AGPL-3.0
**Website**: [https://tower.cetmix.com](https://tower.cetmix.com)

---

## Quick Navigation

### 🚀 Getting Started
- [Project Overview](01-overview/01-project-overview.md)
- [System Architecture](01-overview/02-system-architecture.md)
- [Technology Stack](01-overview/03-technology-stack.md)
- [Module List](01-overview/04-module-list.md)
- [Installation Guide](02-getting-started/01-installation.md)
- [Configuration](02-getting-started/02-configuration.md)
- [First Steps](02-getting-started/03-first-steps.md)

### 👥 For End Users
- [User Guide Overview](03-user-guides/README.md)
- [Navigation & Interface](03-user-guides/01-navigation.md)
- [Basic Operations](03-user-guides/02-basic-operations.md)
- [Common Tasks](03-user-guides/03-common-tasks.md)

### 📚 Feature Guides
- [Setup & Configuration](04-feature-guides/setup-configuration/)
- [Server Management](04-feature-guides/server-management/)
- [Command Execution](04-feature-guides/command-execution/)
- [Flight Plans (Workflows)](04-feature-guides/flight-plans/)
- [File Management](04-feature-guides/file-management/)
- [Variable Management](04-feature-guides/variable-management/)
- [Secret & Key Management](04-feature-guides/secret-management/)
- [Scheduling & Automation](04-feature-guides/scheduling-automation/)
- [Logging & Monitoring](04-feature-guides/logging-monitoring/)
- [Maintenance Operations](04-feature-guides/maintenance-operations/)

### 🔧 For Developers
- [Module References](05-module-references/README.md)
- [API References](06-api-references/README.md)
- [Technical Guides](07-technical-guides/README.md)
- [Development Workflows](08-development-workflows/README.md)
- [Customization Guide](11-customization-extensions/README.md)

### 🛠️ Operations
- [Deployment & Operations](09-deployment-operations/README.md)
- [Security & Compliance](10-security-compliance/README.md)
- [Known Issues](12-issues-bugs/README.md)
- [Upgrade Planning](13-potential-upgrade/README.md)

---

## Key Features

### 🖥️ Server Management
- Manage multiple servers from a single interface
- SSH-based connectivity (password or key authentication)
- Server templates for quick provisioning
- Server status tracking and monitoring
- Tag-based organization

### ⚡ Command Execution
- Execute SSH commands remotely
- Python code execution with Odoo context
- Variable substitution in commands
- Secret injection with automatic masking
- Parallel execution control

### 🛫 Flight Plans (Automation Workflows)
- Sequential command execution
- Conditional logic and branching
- Error handling strategies
- Nested plan execution
- Variable passing between commands

### 📁 File Management
- Bidirectional file synchronization (push/pull)
- Template-based file creation
- Automatic periodic sync
- Binary and text file support
- Server-side version tracking

### 🔐 Security & Secrets
- Encrypted vault for sensitive data
- SSH key management
- Secret injection in commands
- Automatic secret masking in logs
- Role-based access control

### 📅 Scheduling & Automation
- Cron-based task scheduling
- Multi-server task execution
- Custom variable values per task
- Execution history tracking

### 📊 Logging & Monitoring
- Command execution logs
- Flight plan execution tracking
- Server application logs
- Duration and performance metrics

---

## Module Architecture

Cetmix Tower consists of 9 interconnected modules:

| Module | Purpose | Status |
|--------|---------|--------|
| **cetmix_tower** | Main application (meta-module) | Core |
| **cetmix_tower_server** | Server management & command execution | Core |
| **cetmix_tower_server_queue** | Asynchronous task execution | Core |
| **cetmix_tower_server_notify_backend** | Backend notifications | Core |
| **cetmix_tower_git** | Git repository management | Core |
| **cetmix_tower_webhook** | Webhook integration | Core |
| **cetmix_tower_yaml** | YAML import/export | Core |
| **cetmix_tower_aws** | AWS EC2 integration | Optional |
| **cetmix_tower_ovh** | OVH API integration | Optional |

---

## Documentation Structure

This documentation follows the **three-audience approach**:

1. **👨‍💼 System Administrators**: Installation, configuration, maintenance
2. **👥 End Users**: Daily operations, step-by-step guides
3. **👨‍💻 Developers**: Architecture, APIs, customization

Each feature guide is structured to serve all three audiences with appropriate detail levels.

---

## Version Compatibility

| Odoo Version | Tower Version | Status | Notes |
|--------------|---------------|--------|-------|
| 17.0 | 17.0.2.0.0 | ✅ Current | Fully supported |
| 16.0 | Not available | ❌ | Migration required |
| 15.0 | Not available | ❌ | Migration required |
| 14.0 | Available | ⚠️ Legacy | Separate branch |

**Note**: This documentation covers Odoo 17.0 implementation. For version-specific differences, see [Upgrade Planning](13-potential-upgrade/README.md).

---

## Getting Help

- **Documentation Issues**: Open an issue on the GitHub repository
- **Bug Reports**: Use the [Issues section](12-issues-bugs/10-bug-reporting-guide.md)
- **Feature Requests**: Contact Cetmix support
- **Website**: [https://tower.cetmix.com](https://tower.cetmix.com)

---

## Contributing

For information on contributing to Cetmix Tower:
- See [Development Workflows](08-development-workflows/README.md)
- Review [Coding Standards](08-development-workflows/coding-standards.md)
- Follow [Git Workflow](08-development-workflows/git-workflow.md)

---

## License

Cetmix Tower is licensed under **AGPL-3.0**. Each module may have specific licensing - consult individual `__manifest__.py` files.

---

**Last Updated**: 2025-11-16
**Next Review**: 2026-02-16
**Maintained By**: E-Global SCM Development Team

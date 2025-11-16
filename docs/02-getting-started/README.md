---
title: "Getting Started Section"
version: "1.0"
last_updated: "2025-11-16"
maintained_by: "E-Global SCM Development Team"
status: "Active"
---

# Getting Started with Cetmix Tower

Welcome to Cetmix Tower! This section will guide you through the initial setup and help you start managing your servers effectively.

## Contents

1. [**Installation**](01-installation.md)
   - System prerequisites and requirements
   - Installing dependencies
   - Module installation (Apps menu or manual)
   - Development environment setup
   - Installation verification
   - Troubleshooting common issues

2. [**Configuration**](02-configuration.md)
   - Initial configuration wizard
   - Settings and parameters
   - User and access rights setup
   - Cron job configuration
   - Security settings
   - Personal preferences

3. [**First Steps**](03-first-steps.md)
   - Adding your first server
   - Testing SSH connectivity
   - Creating your first command
   - Running commands and viewing results
   - Understanding the interface
   - Basic workflow walkthrough

4. [**Common Tasks**](04-common-tasks.md)
   - Quick reference guide
   - Adding servers
   - Creating and running commands
   - Creating Flight Plans
   - Managing files
   - Working with variables
   - Storing secrets
   - Scheduling tasks
   - Viewing logs

---

## Quick Start Summary

### Prerequisites Checklist

Before you begin, ensure you have:

- [ ] **Odoo 17.0** installed and running
- [ ] **Python 3.10+** with pip package manager
- [ ] **PostgreSQL 12+** database server
- [ ] **Git** version control system
- [ ] **SSH access** to servers you want to manage
- [ ] **Admin rights** in Odoo (or appropriate Cetmix Tower access level)

### Installation Steps (Quick Overview)

1. **Install Python Dependencies**
   ```bash
   pip install paramiko<4 tldextract dnspython pyyaml boto3 ovh
   ```

2. **Install Core Module**
   - Go to Apps menu in Odoo
   - Remove "Apps" filter
   - Search for "Cetmix Tower"
   - Click Install

3. **Configure Basic Settings**
   - Navigate to Settings → Cetmix Tower
   - Set command timeout (default: 300 seconds)
   - Configure cron jobs as needed

4. **Add Your First Server**
   - Go to Tower → Servers → Create
   - Fill in connection details (IP, port, username)
   - Test SSH connection
   - Save

5. **Run Your First Command**
   - Go to Tower → Commands → Create
   - Enter command code (e.g., `df -h`)
   - Assign to your server
   - Click "Run Command"
   - View results in logs

---

## Installation Paths

Choose the installation path that suits your needs:

### Minimal Installation (Core Only)

Install just the essential modules for basic server management:
- `cetmix_tower_server` - Core functionality
- `cetmix_tower_server_queue` - Asynchronous task execution
- `cetmix_tower_server_notify_backend` - Backend notifications

**Use case**: Small teams, basic SSH command execution, limited servers

### Standard Installation (Recommended)

Install the main Cetmix Tower module with all standard features:
- All minimal installation modules
- `cetmix_tower` - Main application bundle
- `cetmix_tower_git` - Git repository management
- `cetmix_tower_webhook` - Webhook support

**Use case**: Most production scenarios, comprehensive server management

### Full Installation (All Features)

Install all available modules including cloud provider integrations:
- All standard installation modules
- `cetmix_tower_aws` - AWS EC2 integration
- `cetmix_tower_ovh` - OVH cloud integration
- `cetmix_tower_yaml` - YAML export/import

**Use case**: Large deployments, multi-cloud environments, advanced automation

---

## Who Should Use This Section?

### System Administrators
- Installing and configuring Cetmix Tower
- Setting up user access and security
- Managing system-wide settings
- Troubleshooting installation issues

### End Users
- Learning the basic interface
- Running commands on servers
- Viewing logs and results
- Managing personal preferences

### Developers
- Setting up development environment
- Installing from source
- Understanding configuration parameters
- Extending functionality

---

## Access Levels Overview

Cetmix Tower has three main access levels:

| Access Level | Permissions |
|--------------|-------------|
| **User** | Basic actions for selected servers (view, run assigned commands) |
| **Manager** | Create and modify selected servers (manage servers, commands, plans) |
| **Root** | Full control over all servers (all Manager permissions + system configuration) |

Access is configured in Settings → Users & Companies → Users → Access Rights → Cetmix Tower

---

## Learning Path

We recommend following this learning path:

```
1. Installation (15-30 minutes)
   ↓
2. Configuration (10-20 minutes)
   ↓
3. First Steps (30-45 minutes)
   ↓
4. Common Tasks Reference (ongoing)
   ↓
5. Feature Guides (as needed)
```

---

## Need Help?

- **Installation Issues**: See [Installation Troubleshooting](01-installation.md#troubleshooting)
- **Configuration Questions**: Check [Configuration Guide](02-configuration.md)
- **Usage Help**: Review [First Steps](03-first-steps.md) and [Common Tasks](04-common-tasks.md)
- **Technical Issues**: Visit [Issues & Bugs](../12-issues-bugs/README.md)
- **Feature Documentation**: Browse [Feature Guides](../04-feature-guides/README.md)

---

## What's Next?

Once you complete the getting started section, explore:

- [**User Guides**](../03-user-guides/README.md) - Role-based workflows and scenarios
- [**Feature Guides**](../04-feature-guides/README.md) - Detailed feature documentation
- [**Module References**](../05-module-references/README.md) - Technical module documentation
- [**Development Workflows**](../08-development-workflows/README.md) - Development best practices

---

**Ready to begin?** → [Start with Installation](01-installation.md)

**Navigate to**:
- [Installation Guide →](01-installation.md)
- [Back to Main Documentation](../README.md)

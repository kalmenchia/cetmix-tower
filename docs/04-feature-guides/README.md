# Feature Guides

Welcome to the Cetmix Tower Feature Guides. This section provides comprehensive documentation for all features in Cetmix Tower, organized by functionality and tailored to different audiences.

## About Feature Guides

Feature guides are structured documentation that explains how specific features work in Cetmix Tower. Each guide is organized into three audience-specific sections:

- **For Administrators**: Configuration, management, and setup tasks
- **For End Users**: Day-to-day usage and operations
- **For Developers**: Technical implementation details, APIs, and code examples

## How to Use These Guides

### Finding What You Need

1. **Browse by Feature Category**: Navigate to the relevant subdirectory below
2. **Look for README files**: Each category has an overview README
3. **Check numbered guides**: Within each category, guides are numbered in logical order

### Understanding the Structure

Each feature guide follows this pattern:

```
feature-name/
├── README.md              # Overview of the feature category
├── 01-feature-basics.md   # Introduction and basic usage
├── 02-advanced-topics.md  # Advanced usage patterns
└── 03-integration.md      # Integration with other features
```

## Feature Guide Organization

### Server Management

Manage servers, configure SSH connections, organize servers with tags and partners.

- [Server Management Overview](./server-management/README.md)
- [Server Basics](./server-management/01-server-basics.md) - Server creation, configuration, lifecycle

**Key Topics**: Server creation, SSH configuration, server templates, status management

### Command Execution

Execute commands on servers using different execution methods.

- [Command Execution Overview](./command-execution/README.md)
- [Command Types](./command-execution/01-command-types.md) - SSH, Python, file templates, flight plans

**Key Topics**: SSH commands, Python code execution, command compatibility, access control

### Flight Plans

Orchestrate complex multi-step operations across servers.

- [Flight Plans Overview](./flight-plans/README.md)
- [Flight Plan Basics](./flight-plans/01-flight-plan-basics.md) - Creating and running flight plans

**Key Topics**: Plan creation, execution order, error handling, plan logs

### Variable Management

Define and manage variables for use in commands, paths, and templates.

- [Variable Management Overview](./variable-management/README.md)
- [Variable Basics](./variable-management/01-variable-basics.md) - Variable types, scope, and usage

**Key Topics**: Variable types, scope (global, server, template), validation, applied expressions

### File Management

Manage files on servers, create files from templates, sync between Tower and servers.

- [File Management Overview](./file-management/README.md)

**Key Topics**: File templates, file sync, server directories

### Secret Management

Securely store and manage secrets, SSH keys, and sensitive data.

- [Secret Management Overview](./secret-management/README.md)

**Key Topics**: SSH keys, vault integration, secret values, inline secrets

### Scheduling & Automation

Schedule tasks, automate recurring operations, trigger actions.

- [Scheduling & Automation Overview](./scheduling-automation/README.md)

**Key Topics**: Scheduled tasks, cron expressions, task execution

### Logging & Monitoring

Track command execution, monitor flight plan progress, analyze logs.

- [Logging & Monitoring Overview](./logging-monitoring/README.md)

**Key Topics**: Command logs, flight plan logs, server logs, duration tracking

### Setup & Configuration

Initial system configuration, settings, and preferences.

- [Setup & Configuration Overview](./setup-configuration/README.md)

**Key Topics**: System settings, timeout configuration, access control

### Maintenance & Operations

Routine maintenance tasks, backup, cleanup, and operational procedures.

- [Maintenance & Operations Overview](./maintenance-operations/README.md)

**Key Topics**: Zombie command cleanup, log archival, server deletion

## Navigating Between Guides

### Related Documentation

- **[Overview](../01-overview/README.md)**: High-level system overview
- **[Getting Started](../02-getting-started/README.md)**: Quick start and tutorials
- **[User Guides](../03-user-guides/README.md)**: Task-oriented how-to guides
- **[Module References](../05-module-references/README.md)**: Detailed module documentation
- **[API References](../06-api-references/README.md)**: API and method documentation

### Cross-References

Feature guides often reference each other. Look for links to:
- Related features
- Prerequisites
- Advanced integrations
- Code examples

## Audience-Specific Reading Paths

### For Administrators

Recommended reading order:
1. [Server Basics](./server-management/01-server-basics.md)
2. [Command Types](./command-execution/01-command-types.md)
3. [Variable Basics](./variable-management/01-variable-basics.md)
4. [Flight Plan Basics](./flight-plans/01-flight-plan-basics.md)

### For End Users

Recommended reading order:
1. [Server Basics](./server-management/01-server-basics.md) - End Users section
2. [Command Types](./command-execution/01-command-types.md) - End Users section
3. [Flight Plan Basics](./flight-plans/01-flight-plan-basics.md) - End Users section

### For Developers

Recommended reading order:
1. [Server Basics](./server-management/01-server-basics.md) - Developers section
2. [Command Types](./command-execution/01-command-types.md) - Developers section
3. [Flight Plan Basics](./flight-plans/01-flight-plan-basics.md) - Developers section
4. [Variable Basics](./variable-management/01-variable-basics.md)

## Quick Reference

### Common Tasks by Feature

| Task | Feature Guide |
|------|--------------|
| Add a new server | [Server Basics](./server-management/01-server-basics.md) |
| Run a command | [Command Types](./command-execution/01-command-types.md) |
| Create a flight plan | [Flight Plan Basics](./flight-plans/01-flight-plan-basics.md) |
| Define variables | [Variable Basics](./variable-management/01-variable-basics.md) |
| Manage SSH keys | [Secret Management](./secret-management/README.md) |
| Schedule tasks | [Scheduling & Automation](./scheduling-automation/README.md) |
| View logs | [Logging & Monitoring](./logging-monitoring/README.md) |

## Contributing to Feature Guides

Feature guides should be:
- **Audience-aware**: Clearly separate content for admins, users, and developers
- **Example-rich**: Include code examples with file paths
- **Technically accurate**: Use actual model names, field names, and methods
- **Cross-referenced**: Link to related features and documentation

## Getting Help

If you can't find what you're looking for:
1. Check the [User Guides](../03-user-guides/README.md) for task-specific instructions
2. Review [Module References](../05-module-references/README.md) for detailed technical information
3. Consult the [API References](../06-api-references/README.md) for method signatures
4. Visit the [Issues & Bugs](../12-issues-bugs/README.md) section for troubleshooting

## Conventions Used in These Guides

- **File paths**: Absolute paths to source files (e.g., `cetmix_tower_server/models/cx_tower_server.py:123`)
- **Model names**: Odoo model names (e.g., `cx.tower.server`)
- **Field references**: Model fields in code (e.g., `server.ip_v4_address`)
- **Methods**: Function names with parentheses (e.g., `run_command()`)
- **Variables**: Template variables use Jinja syntax (e.g., `{{ variable_name }}`)

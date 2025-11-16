---
title: "Module: cetmix_tower_server"
module_name: "cetmix_tower_server"
version: "17.0.2.0.5"
category: "Productivity"
license: "AGPL-3"
author: "Cetmix"
website: "https://tower.cetmix.com"
last_updated: "2025-11-16"
section: "05-module-references"
---

# Module Reference: cetmix_tower_server

## Table of Contents

1. [Module Overview](#module-overview)
2. [Installation and Dependencies](#installation-and-dependencies)
3. [Models](#models)
4. [Views and Wizards](#views-and-wizards)
5. [Security](#security)
6. [Configuration](#configuration)
7. [API and Integration](#api-and-integration)
8. [Usage Examples](#usage-examples)
9. [Related Documentation](#related-documentation)

---

## Module Overview

### Purpose

`cetmix_tower_server` is the core module of Cetmix Tower, providing comprehensive server management capabilities through Odoo. It enables remote server administration, command execution, file management, and automated workflows via SSH connectivity.

### Key Features

- **Server Management**: Configure and manage remote servers with SSH connectivity
- **Command Execution**: Run shell scripts, Python code, and custom commands on servers
- **Flight Plans**: Orchestrate multi-step automated workflows
- **File Management**: Synchronize files between Tower and servers
- **Variable System**: Parameterize commands and files with variables
- **Security**: Vault-based secret storage, role-based access control
- **Logging**: Comprehensive execution logging and audit trails
- **SSH Key Management**: Manage SSH keys and authentication
- **Scheduling**: Automated task execution with cron integration

### Module Information

```python
# File: /home/user/cetmix-tower/cetmix_tower_server/__manifest__.py

{
    "name": "Cetmix Tower Server",
    "summary": "Manage servers and applications from Odoo",
    "version": "17.0.2.0.5",
    "category": "Productivity",
    "website": "https://tower.cetmix.com",
    "author": "Cetmix",
    "license": "AGPL-3",
    "application": False,
    "installable": True,
}
```

---

## Installation and Dependencies

### Odoo Module Dependencies

```python
"depends": [
    "mail",          # Messaging and activity tracking
    "rpc_helper",    # RPC utilities
]
```

### Python Dependencies

```python
"external_dependencies": {
    "python": [
        "paramiko<4",    # SSH connectivity (version < 4)
        "tldextract",    # Domain extraction and parsing
        "dnspython",     # DNS operations
    ],
}
```

### Installation Steps

1. **Install Python dependencies:**

   ```bash
   pip install 'paramiko<4' tldextract dnspython
   ```

2. **Install Odoo dependencies:**

   Ensure `mail` and `rpc_helper` modules are installed.

3. **Install cetmix_tower_server:**

   ```bash
   # Via Odoo UI
   Apps → Search "Cetmix Tower Server" → Install

   # Via command line
   odoo-bin -d database -i cetmix_tower_server
   ```

4. **Configure access:**

   Assign users to appropriate security groups (User/Manager/Root).

---

## Models

### Model Architecture

The module provides 30+ models organized into categories:

**Business Models** (19 models):
- Server management
- Command execution
- Flight plans
- File management
- Variables and values

**Security Models** (4 models):
- Vault storage
- SSH keys
- Access control

**Configuration Models** (3 models):
- Operating systems
- Tags
- Templates

**Logging Models** (3 models):
- Command logs
- Plan logs
- Server logs

**Mixin Models** (7 abstract models):
- Reference management
- Access control
- Template rendering
- Vault security

---

### Core Business Models

#### cx.tower.server

**Purpose:** Represents a physical or virtual server managed by Tower.

**File:** `/home/user/cetmix-tower/cetmix_tower_server/models/cx_tower_server.py`

**Inheritance:**
```python
_name = "cx.tower.server"
_inherit = [
    "cx.tower.access.role.mixin",
    "cx.tower.variable.mixin",
    "cx.tower.reference.mixin",
    "mail.thread",
    "mail.activity.mixin",
    "cx.tower.vault.mixin",
]
_order = "name asc"
```

**Key Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `name` | Char | Server name (required) |
| `reference` | Char | Unique identifier (auto-generated) |
| `active` | Boolean | Active/archived status |
| `ip_address` | Char | IP address or domain |
| `ssh_port` | Integer | SSH port (default: 22) |
| `ssh_username` | Char | SSH username |
| `ssh_password` | Char | SSH password (vaulted) |
| `ssh_auth_mode` | Selection | Authentication mode |
| `host_key` | Text | SSH host key (vaulted) |
| `os_id` | Many2one | Operating system |
| `partner_id` | Many2one | Related partner/contact |
| `tag_ids` | Many2many | Organization tags |
| `status` | Selection | Connection status |
| `user_ids` | Many2many | Assigned users |
| `manager_ids` | Many2many | Assigned managers |

**Secret Fields (Vaulted):**
```python
SECRET_FIELDS = ["ssh_password", "host_key"]
```

**Important Methods:**

```python
def _get_ssh_client(self, raise_on_error=False):
    """
    Get SSH connection for this server.

    Args:
        raise_on_error (bool): Raise exception on connection failure

    Returns:
        SSHConnection: Active SSH connection object

    File: cx_tower_server.py, line ~200
    """

@ensure_ssh_disconnect
def execute_command(self, command_code, timeout=None, path=None):
    """
    Execute command on server via SSH.

    Args:
        command_code (str): Command to execute
        timeout (int): Execution timeout in seconds
        path (str): Working directory for execution

    Returns:
        dict: {
            'stdout': str,
            'stderr': str,
            'exit_code': int,
            'status': str
        }

    File: cx_tower_server.py, line ~350
    """

def test_connection(self):
    """
    Test SSH connectivity to server.

    Returns:
        bool: True if connection successful

    File: cx_tower_server.py, line ~280
    """

def action_refresh_status(self):
    """Update server connection status."""
```

**Relationships:**
- **Commands**: Many2many via `command_ids`
- **Files**: One2many via `file_ids`
- **Variables**: One2many values via `variable_value_ids`
- **Logs**: One2many via `server_log_ids`
- **Template**: Many2one via `template_id`

---

#### cx.tower.command

**Purpose:** Defines commands that can be executed on servers.

**File:** `/home/user/cetmix-tower/cetmix_tower_server/models/cx_tower_command.py`

**Inheritance:**
```python
_name = "cx.tower.command"
_inherit = [
    "cx.tower.template.mixin",
    "cx.tower.reference.mixin",
    "cx.tower.access.mixin",
    "cx.tower.access.role.mixin",
    "cx.tower.key.mixin",
]
_order = "name"
```

**Key Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `name` | Char | Command name (required) |
| `reference` | Char | Unique identifier |
| `active` | Boolean | Active status |
| `code` | Text | Command code/script |
| `action` | Selection | Command type (Shell/Python/etc) |
| `timeout` | Integer | Execution timeout (seconds) |
| `allow_parallel_run` | Boolean | Allow concurrent execution |
| `server_ids` | Many2many | Allowed servers |
| `os_ids` | Many2many | Compatible operating systems |
| `tag_ids` | Many2many | Organization tags |
| `variable_ids` | Many2many | Used variables (computed) |
| `secret_ids` | Many2many | Used secrets (computed) |
| `access_level` | Selection | User/Manager/Root |

**Command Types:**

```python
def _selection_action(self):
    return [
        ('s', 'Shell'),           # Shell command
        ('p', 'Python'),          # Python script
        ('f', 'Flight Plan'),     # Run flight plan
        ('ft', 'File Template'),  # Deploy file template
    ]
```

**Important Methods:**

```python
def run(self, server_ids=None, variable_values=None):
    """
    Run command on specified servers.

    Args:
        server_ids (list): Server IDs to run on
        variable_values (dict): Variable value overrides

    Returns:
        cx.tower.command.log: Execution log recordset

    File: cx_tower_command.py, line ~450
    """

def _render_command_code(self, variable_values):
    """
    Render command code with variable substitution.

    Args:
        variable_values (dict): {variable_ref: value}

    Returns:
        str: Rendered command code

    File: cx_tower_command.py, line ~380
    """

def _check_compatibility(self, server):
    """
    Check if command can run on server.

    Args:
        server (cx.tower.server): Server to check

    Returns:
        bool: True if compatible

    Raises:
        ValidationError: If incompatible

    File: cx_tower_command.py, line ~320
    """
```

---

#### cx.tower.plan

**Purpose:** Orchestrates multiple commands in sequence (Flight Plans).

**File:** `/home/user/cetmix-tower/cetmix_tower_server/models/cx_tower_plan.py`

**Inheritance:**
```python
_name = "cx.tower.plan"
_inherit = [
    "cx.tower.reference.mixin",
    "cx.tower.access.mixin",
    "cx.tower.access.role.mixin",
]
_order = "name asc"
```

**Key Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `name` | Char | Plan name (required) |
| `reference` | Char | Unique identifier |
| `active` | Boolean | Active status |
| `allow_parallel_run` | Boolean | Allow concurrent execution |
| `on_error_action` | Selection | Error handling strategy |
| `custom_exit_code` | Integer | Custom error exit code |
| `line_ids` | One2many | Plan steps |
| `server_ids` | Many2many | Target servers |
| `tag_ids` | Many2many | Organization tags |
| `command_ids` | Many2many | Commands used (computed) |
| `access_level` | Selection | Access level |

**Error Actions:**

```python
on_error_action = fields.Selection([
    ('e', 'Exit with command exit code'),
    ('ec', 'Exit with custom exit code'),
    ('n', 'Run next command'),
])
```

**Important Methods:**

```python
def run(self, server_ids=None, variable_values=None):
    """
    Execute flight plan on servers.

    Args:
        server_ids (list): Target server IDs
        variable_values (dict): Variable overrides

    Returns:
        cx.tower.plan.log: Plan execution log

    File: cx_tower_plan.py, line ~200
    """

def _execute_plan_lines(self, server, log_id):
    """
    Execute plan lines sequentially on server.

    Args:
        server (cx.tower.server): Target server
        log_id (cx.tower.plan.log): Execution log

    Returns:
        dict: Execution results

    File: cx_tower_plan.py, line ~280
    """
```

---

#### cx.tower.plan.line

**Purpose:** Individual steps within a flight plan.

**File:** `/home/user/cetmix-tower/cetmix_tower_server/models/cx_tower_plan_line.py`

**Key Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `sequence` | Integer | Execution order |
| `name` | Char | Line name (from command) |
| `plan_id` | Many2one | Parent flight plan |
| `command_id` | Many2one | Command to execute |
| `path` | Char | Execution path override |
| `use_sudo` | Boolean | Force sudo usage |
| `condition` | Char | Conditional execution |
| `action_ids` | One2many | Conditional actions |

**Conditional Execution:**

```python
condition = fields.Char(
    help="Conditions under which this Flight Plan Line will be launched. "
         "e.g.: {{ odoo_version}} == '17.0'"
)
```

---

#### cx.tower.file

**Purpose:** Manages files synchronized between Tower and servers.

**File:** `/home/user/cetmix-tower/cetmix_tower_server/models/cx_tower_file.py`

**Key Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `name` | Char | Filename |
| `reference` | Char | Unique identifier |
| `server_dir` | Char | Target directory on server |
| `source` | Selection | Tower or Server |
| `file_type` | Selection | Code or Binary |
| `code` | Text | Text file content |
| `file_binary` | Binary | Binary file content |
| `auto_sync` | Boolean | Enable auto-sync |
| `auto_sync_interval` | Selection | Sync frequency |
| `sync_date_next` | Datetime | Next sync time |
| `sync_date_last` | Datetime | Last sync time |
| `template_id` | Many2one | File template |

**File Sources:**

```python
source = fields.Selection([
    ('tower', 'Tower'),    # Push from Tower to Server
    ('server', 'Server'),  # Pull from Server to Tower
])
```

**Important Methods:**

```python
def sync_to_server(self):
    """Push file from Tower to server(s)."""

def sync_from_server(self):
    """Pull file from server(s) to Tower."""

def _compute_render(self):
    """Render filename and path with variables."""
```

---

#### cx.tower.variable

**Purpose:** Defines variables used in commands and files.

**File:** `/home/user/cetmix-tower/cetmix_tower_server/models/cx_tower_variable.py`

**Key Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `name` | Char | Variable name |
| `reference` | Char | Variable identifier |
| `variable_type` | Selection | String or Options |
| `applied_expression` | Text | Value transformation |
| `validation_pattern` | Char | Regex validation |
| `validation_message` | Char | Error message |
| `option_ids` | One2many | Option values |
| `value_ids` | One2many | Assigned values |
| `command_ids` | Many2many | Commands using variable |

**Variable Types:**

```python
variable_type = fields.Selection([
    ('s', 'String'),   # Free text
    ('o', 'Options'),  # Dropdown options
])
```

---

### Logging Models

#### cx.tower.command.log

**Purpose:** Logs command execution history.

**File:** `/home/user/cetmix-tower/cetmix_tower_server/models/cx_tower_command_log.py`

**Key Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `command_id` | Many2one | Executed command |
| `server_id` | Many2one | Target server |
| `status` | Selection | Execution status |
| `stdout` | Text | Standard output |
| `stderr` | Text | Error output |
| `exit_code` | Integer | Command exit code |
| `start_date` | Datetime | Execution start |
| `end_date` | Datetime | Execution end |
| `duration` | Float | Execution duration |

---

#### cx.tower.plan.log

**Purpose:** Logs flight plan execution history.

**File:** `/home/user/cetmix-tower/cetmix_tower_server/models/cx_tower_plan_log.py`

**Key Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `plan_id` | Many2one | Executed plan |
| `server_id` | Many2one | Target server |
| `status` | Selection | Overall status |
| `start_date` | Datetime | Execution start |
| `end_date` | Datetime | Execution end |
| `command_log_ids` | One2many | Individual command logs |

---

### Abstract Mixin Models

#### cx.tower.reference.mixin

**Purpose:** Automatic unique reference generation.

**File:** `/home/user/cetmix-tower/cetmix_tower_server/models/cx_tower_reference_mixin.py`

**Provides:**
- `name` field (required, indexed)
- `reference` field (unique, auto-generated)
- Auto-generation from name
- Duplicate handling with suffixes
- Reference-based search

**Methods:**
```python
def get_by_reference(self, reference):
    """Get record by reference."""

def _generate_or_fix_reference(self, reference_source):
    """Generate or fix reference from source."""
```

---

#### cx.tower.vault.mixin

**Purpose:** Secure storage of sensitive fields.

**File:** `/home/user/cetmix-tower/cetmix_tower_server/models/cx_tower_vault_mixin.py`

**Usage:**
```python
class MyModel(models.Model):
    _inherit = ['cx.tower.vault.mixin']

    SECRET_FIELDS = ["password", "api_key"]

    password = fields.Char()  # Auto-vaulted
    api_key = fields.Char()   # Auto-vaulted
```

**Methods:**
```python
def _get_secret_value(self, field_name):
    """Get actual secret value."""

def _set_secret_values(self, vals):
    """Store secret values in vault."""
```

---

#### cx.tower.template.mixin

**Purpose:** Jinja2 template rendering.

**File:** `/home/user/cetmix-tower/cetmix_tower_server/models/cx_tower_template_mixin.py`

**Provides:**
- `code` field for template content
- `variable_ids` auto-detection
- Template rendering with variables

**Methods:**
```python
def render_code(self, pythonic_mode=False, **kwargs):
    """Render template with variables."""

def get_variables_from_code(self, code):
    """Extract variable names from template."""
```

---

## Views and Wizards

### Main Views

**View Files Location:** `/home/user/cetmix-tower/cetmix_tower_server/views/`

#### Server Views

- **Form View**: `cx_tower_server_view.xml`
  - Comprehensive server configuration
  - Connection settings
  - Variable values
  - Related commands and files

- **Tree View**: Server list with status indicators
- **Kanban View**: Visual server organization
- **Calendar View**: Scheduled tasks

#### Command Views

- **Form View**: `cx_tower_command_view.xml`
  - Command code editor with syntax highlighting
  - Variable management
  - Compatibility settings

- **Tree View**: Command list with quick run
- **Kanban View**: Command organization by status

#### Flight Plan Views

- **Form View**: `cx_tower_plan_view.xml`
  - Plan configuration
  - Line management with sequence
  - Error handling settings

- **Tree View**: Plan list
- **Kanban View**: Plan workflow visualization

---

### Wizards

**Wizard Files Location:** `/home/user/cetmix-tower/cetmix_tower_server/wizards/`

#### Run Command Wizard

**Model:** `cx.tower.command.run.wizard`
**File:** `cx_tower_command_run_wizard_view.xml`

**Purpose:** Execute commands with runtime configuration.

**Fields:**
- Server selection
- Variable values
- Custom options

---

#### Run Flight Plan Wizard

**Model:** `cx.tower.plan.run.wizard`
**File:** `cx_tower_plan_run_wizard_view.xml`

**Purpose:** Execute flight plans with configuration.

**Fields:**
- Server selection
- Plan variables
- Execution options

---

## Security

### Security Groups

**File:** `/home/user/cetmix-tower/cetmix_tower_server/security/cetmix_tower_server_groups.xml`

**Groups:**
1. **User** (`group_user`): Basic operations
2. **Manager** (`group_manager`): Create and manage
3. **Root** (`group_root`): Full administrative access

### Model Access Rights

**File:** `/home/user/cetmix-tower/cetmix_tower_server/security/ir.model.access.csv`

**Pattern:**
- **Users**: Read-only access to assigned records
- **Managers**: CRUD on assigned records
- **Root**: Full access to all records

### Record Rules

**Files:** `/home/user/cetmix-tower/cetmix_tower_server/security/*_security.xml`

**Server Rules:**
- Users: Read assigned servers (`user_ids`)
- Managers: Manage assigned servers (`manager_ids`)
- Root: All servers

**Command Rules:**
- Access level filtering
- User/Manager assignment
- Root unrestricted

**See:** [Security and Access Control Guide](../../07-technical-guides/03-security-and-access.md)

---

## Configuration

### System Parameters

**Access:** `Settings → Technical → Parameters → System Parameters`

**Key Parameters:**
- `cetmix_tower.default_timeout`: Default command timeout
- `cetmix_tower.ssh_timeout`: SSH connection timeout
- `cetmix_tower.max_parallel_commands`: Concurrent command limit

### Cron Jobs

**File:** `/home/user/cetmix-tower/cetmix_tower_server/data/ir_cron.xml`

**Scheduled Jobs:**
- **Auto Sync Files**: Synchronize files based on schedule
- **Update Server Status**: Check server connectivity
- **Clean Old Logs**: Archive or delete old execution logs

---

## API and Integration

### Public API Methods

#### Server Management

```python
# Get server by reference
server = env['cx.tower.server'].get_by_reference('prod_web_01')

# Test connection
server.test_connection()

# Execute command
result = server.execute_command('uptime')
```

#### Command Execution

```python
# Run command
command = env['cx.tower.command'].get_by_reference('restart_nginx')
logs = command.run(
    server_ids=[server.id],
    variable_values={'service_name': 'nginx'}
)
```

#### Flight Plan Execution

```python
# Run flight plan
plan = env['cx.tower.plan'].get_by_reference('deploy_app')
plan_log = plan.run(
    server_ids=[server.id],
    variable_values={'version': 'v2.0.0'}
)
```

#### File Synchronization

```python
# Sync file to server
file = env['cx.tower.file'].get_by_reference('nginx_conf')
file.sync_to_server()

# Sync file from server
log_file = env['cx.tower.file'].get_by_reference('app_log')
log_file.sync_from_server()
```

---

## Usage Examples

### Example 1: Configure Server and Run Command

```python
# Create server
server = env['cx.tower.server'].create({
    'name': 'Production Web Server',
    'reference': 'prod_web',
    'ip_address': '192.168.1.100',
    'ssh_port': 22,
    'ssh_username': 'admin',
    'ssh_password': 'secret_password',  # Auto-vaulted
    'os_id': ubuntu_os.id,
})

# Create command
command = env['cx.tower.command'].create({
    'name': 'System Update',
    'reference': 'sys_update',
    'action': 's',  # Shell
    'code': '''
sudo apt update
sudo apt upgrade -y
    ''',
    'timeout': 600,
})

# Run command
command.run(server_ids=[server.id])
```

### Example 2: Create Flight Plan for Deployment

```python
# Create deployment plan
plan = env['cx.tower.plan'].create({
    'name': 'Deploy Application',
    'reference': 'deploy_app',
    'on_error_action': 'e',  # Exit on error
    'line_ids': [
        (0, 0, {
            'sequence': 10,
            'command_id': stop_app_cmd.id,
        }),
        (0, 0, {
            'sequence': 20,
            'command_id': backup_db_cmd.id,
        }),
        (0, 0, {
            'sequence': 30,
            'command_id': pull_code_cmd.id,
        }),
        (0, 0, {
            'sequence': 40,
            'command_id': migrate_db_cmd.id,
        }),
        (0, 0, {
            'sequence': 50,
            'command_id': start_app_cmd.id,
        }),
    ],
})

# Run plan
plan.run(server_ids=[server.id])
```

### Example 3: Manage Configuration Files

```python
# Create configuration file with variables
config_file = env['cx.tower.file'].create({
    'name': 'app.conf',
    'reference': 'app_config',
    'source': 'tower',
    'file_type': 'code',
    'server_dir': '/etc/myapp',
    'code': '''
[app]
environment = {{ environment }}
database_host = {{ db_host }}
database_port = {{ db_port }}
    ''',
    'server_ids': [(6, 0, [server.id])],
    'auto_sync': True,
    'auto_sync_interval': '1-days',
})

# Set variable values for server
env['cx.tower.variable.value'].create({
    'variable_id': env_var.id,
    'value': 'production',
    'server_id': server.id,
})

# Sync file
config_file.sync_to_server()
```

---

## Related Documentation

### Technical Guides

- [Models and ORM](../../07-technical-guides/01-models-and-orm.md): In-depth model architecture
- [Views and UI](../../07-technical-guides/02-views-and-ui.md): UI components and customization
- [Security and Access](../../07-technical-guides/03-security-and-access.md): Complete security guide

### Feature Guides

- [Server Management](../../04-feature-guides/01-servers/README.md)
- [Command Execution](../../04-feature-guides/02-commands/README.md)
- [Flight Plans](../../04-feature-guides/03-flight-plans/README.md)
- [File Management](../../04-feature-guides/04-files/README.md)
- [Variables](../../04-feature-guides/05-variables/README.md)

### User Guides

- [Navigation](../../03-user-guides/01-navigation.md)
- [Basic Operations](../../03-user-guides/02-basic-operations.md)
- [Common Tasks](../../03-user-guides/03-common-tasks.md)

---

## Support and Contributing

**Official Website:** https://tower.cetmix.com
**Documentation:** https://tower.cetmix.com/documentation
**Source Code:** https://github.com/cetmix/cetmix-tower
**Issue Tracker:** https://github.com/cetmix/cetmix-tower/issues

---

**Navigation:**
- [← Module References Home](../README.md)
- [↑ Documentation Home](../../README.md)

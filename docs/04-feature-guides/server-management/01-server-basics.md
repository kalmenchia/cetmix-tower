---
title: Server Basics
audience: [administrators, end-users, developers]
topics: [server-creation, ssh-configuration, server-organization, server-lifecycle]
---

# Server Basics

This guide covers the fundamental concepts and operations for managing servers in Cetmix Tower.

## Table of Contents

- [For Administrators](#for-administrators)
  - [Server Model Overview](#server-model-overview)
  - [Server Creation Methods](#server-creation-methods)
  - [Server Configuration](#server-configuration)
  - [Server Status Lifecycle](#server-status-lifecycle)
  - [Server Templates](#server-templates)
  - [Server Organization](#server-organization)
- [For End Users](#for-end-users)
  - [Viewing Servers](#viewing-servers)
  - [Server Information](#server-information)
  - [Running Commands on Servers](#running-commands-on-servers)
- [For Developers](#for-developers)
  - [Server Model Technical Details](#server-model-technical-details)
  - [Key Fields and Their Purposes](#key-fields-and-their-purposes)
  - [Server Methods](#server-methods)
  - [SSH Client Implementation](#ssh-client-implementation)
  - [Code Examples](#code-examples)

---

## For Administrators

### Server Model Overview

The server model (`cx.tower.server`) is the core component of Cetmix Tower, representing remote servers that you can manage and execute operations on.

**Location**: `cetmix_tower_server/models/cx_tower_server.py`

Each server record contains:
- Connection credentials (SSH)
- Authentication configuration
- Organizational metadata (tags, partner)
- Associated resources (commands, flight plans, files)
- Access control settings

### Server Creation Methods

#### Method 1: Manual Creation via UI

1. Navigate to **Cetmix Tower > Servers**
2. Click **Create**
3. Fill in the required fields:
   - **Name**: Descriptive server name
   - **IPv4 Address** or **IPv6 Address**: Server IP
   - **SSH Port**: Default is 22
   - **SSH Username**: User account for SSH
   - **SSH Auth Mode**: Choose password or key-based
4. Configure authentication:
   - For password: Enter SSH password
   - For key: Select SSH private key
5. Click **Save**

#### Method 2: Using Server Templates

1. Create a server template with default settings
2. Navigate to **Cetmix Tower > Servers**
3. Click **Create**
4. Select the template from **Server Template** field
5. Template variables and settings are auto-populated
6. Adjust specific values as needed
7. Click **Save**

#### Method 3: Programmatic Creation

See [For Developers](#for-developers) section for code examples.

### Server Configuration

#### SSH Connection Settings

**Basic Configuration**:
- `ip_v4_address` or `ip_v6_address`: Server IP address (at least one required)
- `ssh_port`: SSH port (default: 22)
- `ssh_username`: SSH user account (required)

**Authentication Methods**:

1. **Password Authentication** (`ssh_auth_mode='p'`):
   - Set `ssh_password`: SSH account password
   - Password is stored securely using vault mixin

2. **Key-Based Authentication** (`ssh_auth_mode='k'`):
   - Set `ssh_key_id`: Reference to stored SSH private key
   - More secure than password authentication
   - Recommended for production environments

**Host Key Verification**:
- `host_key`: Server's SSH host key for verification
- `skip_host_key`: Bypass host key verification (use with caution)

To get and save the host key:
1. Open the server record
2. Click **Show Host Key** action
3. The system retrieves the actual host key from the server
4. Click **Save** to store it

**Sudo Configuration**:

The `use_sudo` field controls whether commands run with sudo:
- **Empty**: No sudo used
- **Without password** (`'n'`): Uses `sudo -S -p ''` without password
- **With password** (`'p'`): Uses `sudo -S -p ''` and sends password

Note: When `ssh_username='root'`, sudo is automatically disabled.

#### Operating System Configuration

Set the `os_id` field to specify the server's operating system. This helps with:
- Command compatibility
- OS-specific operations
- Filtering and organization

#### URL Configuration

The `url` field stores the server's web interface URL (e.g., `https://server.example.com`). Useful for quick access to admin panels.

### Server Status Lifecycle

Servers track their current state using the `status` field:

| Status | Description | When Set |
|--------|-------------|----------|
| `None` | Undefined | Initial state, server not categorized |
| `stopped` | Server is stopped | Set manually or by command |
| `starting` | Server is starting | Set manually or by command |
| `running` | Server is running | Set manually or by command |
| `stopping` | Server is stopping | Set manually or by command |
| `restarting` | Server is restarting | Set manually or by command |
| `deleting` | Deletion in progress | Set when delete flight plan is running |
| `delete_error` | Deletion failed | Set when delete flight plan fails |

**Status Transitions**:
- Status can be set manually from the server form
- Commands can update status via `server_status` field
- Delete operations trigger special status handling

**Defined in**: `cetmix_tower_server/models/cx_tower_server.py:264-279`

### Server Templates

Server templates (`cx.tower.server.template`) allow you to create reusable server configurations.

**Benefits**:
- Standardize server settings across multiple instances
- Pre-configure variables for specific server types
- Define default server logs
- Faster server creation

**Creating a Template**:
1. Navigate to **Cetmix Tower > Configuration > Server Templates**
2. Click **Create**
3. Configure template fields
4. Set default variable values
5. Save template

**Using a Template**:
1. When creating a server, select the template
2. Template variables are automatically applied
3. Override specific values as needed

**Template Fields Copied**:
- Variable values (`variable_value_ids`)
- Server logs configuration (`server_log_ids`)
- Other configuration settings

### Server Organization

#### Tags

Use tags (`tag_ids`) to categorize servers:
- Production vs. Development
- By project or customer
- By function (web server, database, etc.)
- By location or data center

Tags are many-to-many, so servers can have multiple tags.

#### Partners

Link servers to partners (`partner_id`) to:
- Associate with customers or organizations
- Group servers by client
- Track ownership

#### Shortcuts

Assign shortcuts (`shortcut_ids`) for quick access to frequently used operations.

#### Notes

Use the `note` field for:
- Server documentation
- Special instructions
- Maintenance notes
- Contact information

### Access Control

**Access Levels**:
Servers inherit access control from `cx.tower.access.role.mixin`:
- Define who can view the server
- Control who can execute commands
- Set manager permissions

**User Assignment**:
- `user_ids`: Users who can view and use the server
- `manager_ids`: Users who can manage the server

**Access in Commands**:
Custom variable values in `run_command()` are only applied if the user has write access to the server.

**Defined in**: `cetmix_tower_server/models/cx_tower_server.py:221-228`

### Testing SSH Connection

After configuring a server, test the connection:

1. Open the server record
2. Click **Test SSH Connection** action
3. The system performs:
   - SSH connection establishment
   - Command execution test (`uname -a`)
   - File upload/download test
4. Success notification appears if all tests pass

**Technical Details**:
- Method: `test_ssh_connection()`
- Location: `cetmix_tower_server/models/cx_tower_server.py:581-712`
- Default test command: `uname -a`

**Parameters**:
```python
test_ssh_connection(
    raise_on_error=True,      # Raise exception on error
    return_notification=True, # Show notification
    try_command=True,         # Test command execution
    try_file=True,            # Test file operations
    timeout=60                # Timeout in seconds
)
```

### Server Deletion

When deleting a server, you can run a cleanup flight plan:

1. Set `plan_delete_id` field to a flight plan
2. When server is deleted, the flight plan runs automatically
3. Server status becomes `deleting`
4. Upon success, server is deleted
5. On failure, status becomes `delete_error`

To force delete without running the plan:
- Use context key `server_force_delete=True`

**Deletion Process** (defined in `cx_tower_server.py:333-369`):
```python
def unlink(self):
    """Run post-delete flight plan"""
    # Check if forced, no plan, or already deleting
    # Run delete plan if configured
    # Delete server on success or set error status
```

---

## For End Users

### Viewing Servers

Access your available servers:

1. Navigate to **Cetmix Tower > Servers**
2. You see servers you have access to
3. Use filters to find specific servers:
   - By name or IP
   - By tags
   - By partner
   - By status

**List View Information**:
- Server name
- IP address
- Status
- Partner
- Tags
- Operating system

### Server Information

Open a server to view:

**Connection Tab**:
- IP addresses (IPv4/IPv6)
- SSH port and username
- Authentication method
- Connection status

**Details Tab**:
- Operating system
- Tags
- Partner
- URL
- Notes

**Tabs**:
- **Commands**: Available commands
- **Flight Plans**: Available flight plans
- **Files**: Server files
- **Variables**: Server-specific variable values
- **Logs**: Execution history

### Running Commands on Servers

To execute a command on a server:

1. Open the server record
2. Click **Run Command** button
3. Select a command from the list
4. Provide any required variable values
5. Click **Run**
6. Monitor execution in the **Command Logs** tab

**Available Commands**:
- Commands you have access to
- Commands compatible with the server
- Commands not limited to specific servers

See [Command Execution](../command-execution/01-command-types.md) for more details.

### Running Flight Plans

To execute a flight plan:

1. Open the server record
2. Click **Run Flight Plan** button
3. Select a flight plan
4. Provide any required variable values
5. Click **Run**
6. Monitor execution in the **Plan Logs** tab

See [Flight Plans](../flight-plans/01-flight-plan-basics.md) for more details.

### Viewing Execution History

**Command Logs**:
- Click **Command Logs** button or tab
- View all commands executed on this server
- Filter by date, status, or command
- Check execution details and results

**Flight Plan Logs**:
- Click **Flight Plan Logs** button or tab
- View all flight plans executed on this server
- Monitor currently running plans
- Review execution results

### Server Files

View and manage files on the server:

1. Open the server record
2. Click **Files** tab or **Files** button
3. View all files associated with this server
4. Upload or download files
5. Sync files between Tower and server

See [File Management](../file-management/README.md) for more details.

---

## For Developers

### Server Model Technical Details

**Model Name**: `cx.tower.server`

**Location**: `cetmix_tower_server/models/cx_tower_server.py`

**Inheritance**:
```python
_inherit = [
    "cx.tower.access.role.mixin",  # Access control
    "cx.tower.variable.mixin",      # Variable management
    "cx.tower.reference.mixin",     # Reference tracking
    "mail.thread",                  # Messaging
    "mail.activity.mixin",          # Activities
    "cx.tower.vault.mixin",         # Secure storage
]
```

**Description**: "Cetmix Tower Server"

**Order**: `name asc`

### Key Fields and Their Purposes

#### Connection Fields

```python
# IP addresses (at least one required)
ip_v4_address = fields.Char(string="IPv4 Address", groups="...")
ip_v6_address = fields.Char(string="IPv6 Address", groups="...")

# SSH configuration
ssh_port = fields.Integer(required=True, default=22)
ssh_username = fields.Char(required=True)
ssh_auth_mode = fields.Selection([('p', 'Password'), ('k', 'Key')], default='p')

# Authentication credentials
ssh_password = fields.Char()  # Stored securely
ssh_key_id = fields.Many2one('cx.tower.key', domain=[('key_type', '=', 'k')])

# Host key verification
skip_host_key = fields.Boolean(default=False)
host_key = fields.Char()  # Server's SSH host key

# Sudo configuration
use_sudo = fields.Selection([('n', 'Without password'), ('p', 'With password')])
```

**Defined**: Lines 103-152

#### Organization Fields

```python
# Basic info
active = fields.Boolean(default=True)
color = fields.Integer()
partner_id = fields.Many2one('res.partner')
status = fields.Selection(selection=lambda self: self._selection_status())

# Categorization
os_id = fields.Many2one('cx.tower.os')
tag_ids = fields.Many2many('cx.tower.tag')
note = fields.Text()
url = fields.Char()  # Web interface URL

# Template
server_template_id = fields.Many2one('cx.tower.server.template', readonly=True)
```

**Defined**: Lines 93-211

#### Relational Fields

```python
# Variables
variable_value_ids = fields.One2many('cx.tower.variable.value', 'server_id')

# Secrets
secret_ids = fields.One2many('cx.tower.key.value', 'server_id')

# Resources
command_log_ids = fields.One2many('cx.tower.command.log', 'server_id')
plan_log_ids = fields.One2many('cx.tower.plan.log', 'server_id')
file_ids = fields.One2many('cx.tower.file', 'server_id')
server_log_ids = fields.One2many('cx.tower.server.log', 'server_id')

# Associations
shortcut_ids = fields.Many2many('cx.tower.shortcut')
scheduled_task_ids = fields.Many2many('cx.tower.scheduled.task')
command_ids = fields.Many2many('cx.tower.command')
plan_ids = fields.Many2many('cx.tower.plan')

# Deletion
plan_delete_id = fields.Many2one('cx.tower.plan')  # On-delete flight plan
```

**Defined**: Lines 156-262

#### Computed Fields

```python
file_count = fields.Integer(compute='_compute_file_count')
```

**Defined**: Line 194-197

### Server Methods

#### Command Execution

**`run_command(command, path=None, sudo=None, ssh_connection=None, **kwargs)`**

Main method for executing commands on the server.

**Location**: `cetmix_tower_server/models/cx_tower_server.py:804-938`

**Parameters**:
- `command` (cx.tower.command): Command record to execute
- `path` (str, optional): Override default command path
- `sudo` (selection, optional): Sudo mode override
- `ssh_connection` (SSH instance, optional): Reuse existing connection
- `kwargs` (dict): Additional arguments
  - `log` (dict): Values for command log
  - `key` (dict): Values for key parser
  - `variable_values` (dict): Custom variable values

**Context Keys**:
- `no_command_log`: If True, skip log creation and return results directly

**Returns**: `dict` (if `no_command_log=True`) or `None`

**Example**:
```python
# Basic command execution
server.run_command(command)

# With custom path and sudo
server.run_command(command, path='/opt/myapp', sudo='n')

# With custom variables (requires write access)
server.run_command(
    command,
    variable_values={'version': '16.0', 'env': 'prod'}
)

# Without logging (for programmatic use)
result = server.with_context(no_command_log=True).run_command(command)
# result = {'status': 0, 'response': '...', 'error': None}
```

#### Flight Plan Execution

**`run_flight_plan(flight_plan, **kwargs)`**

Execute a flight plan on the server.

**Location**: `cetmix_tower_server/models/cx_tower_server.py:1033-1053`

**Parameters**:
- `flight_plan` (cx.tower.plan): Flight plan record
- `kwargs` (dict): Optional arguments (same as run_command)

**Returns**: `cx.tower.plan.log` record

**Example**:
```python
# Execute flight plan
plan_log = server.run_flight_plan(flight_plan)

# With custom variables
plan_log = server.run_flight_plan(
    flight_plan,
    variable_values={'env': 'staging'}
)

# With custom label for tracking
plan_log = server.run_flight_plan(
    flight_plan,
    plan_log={'label': 'deployment_v2.0'}
)
```

#### SSH Connection

**`_get_ssh_client(raise_on_error=False, timeout=5000, skip_host_key=False)`**

Create and return an SSH client instance.

**Location**: `cetmix_tower_server/models/cx_tower_server.py:529-579`

**Parameters**:
- `raise_on_error` (bool): Raise exception on error (default: False)
- `timeout` (int): Connection timeout in milliseconds (default: 5000)
- `skip_host_key` (bool): Skip host key verification (default: False)

**Returns**: `SSHManager` instance or `(False, exception)`

**Example**:
```python
# Get SSH client
client = server._get_ssh_client(raise_on_error=True)

# Use client for operations
status, response, error = client.command_executor.exec_command('ls -la')

# Always disconnect when done
client.disconnect()
```

**Important**: Use `@ensure_ssh_disconnect` decorator to ensure cleanup:

```python
from cetmix_tower_server.models.cx_tower_server import ensure_ssh_disconnect

@ensure_ssh_disconnect
def my_custom_operation(self):
    client = self._get_ssh_client(raise_on_error=True)
    # Perform operations
    # Connection auto-disconnects after transaction
```

#### File Operations

**`upload_file(data, remote_path, from_path=False)`**

Upload a file to the remote server.

**Location**: `cetmix_tower_server/models/cx_tower_server.py:1839-1868`

**Parameters**:
- `data` (str/bytes): File content or local path
- `remote_path` (str): Remote file path (e.g., `/opt/myapp/config.ini`)
- `from_path` (bool): If True, `data` is treated as local file path

**Returns**: `SFTPAttributes` (file metadata)

**Example**:
```python
# Upload string content
server.upload_file('Hello World', '/tmp/test.txt')

# Upload from local file
server.upload_file('/local/path/file.txt', '/remote/path/file.txt', from_path=True)
```

**`download_file(remote_path)`**

Download a file from the remote server.

**Location**: `cetmix_tower_server/models/cx_tower_server.py:1871-1894`

**Parameters**:
- `remote_path` (str): Remote file path

**Returns**: `bytes` (file content)

**Example**:
```python
# Download file
content = server.download_file('/etc/hostname')
print(content.decode())
```

**`delete_file(remote_path)`**

Delete a file from the remote server.

**Location**: `cetmix_tower_server/models/cx_tower_server.py:1826-1836`

**Example**:
```python
server.delete_file('/tmp/old_file.txt')
```

#### SSH Testing

**`test_ssh_connection(raise_on_error=True, return_notification=True, try_command=True, try_file=True, timeout=60)`**

Test SSH connection and operations.

**Location**: `cetmix_tower_server/models/cx_tower_server.py:581-712`

**Returns**: `dict` with test results or notification action

**Example**:
```python
# Test with all checks
result = server.test_ssh_connection()

# Test connection only (no command/file tests)
result = server.test_ssh_connection(
    try_command=False,
    try_file=False,
    return_notification=False
)
# result = {'status': 0, 'response': 'Connection successful.', 'error': ''}
```

### SSH Client Implementation

The SSH functionality is implemented using Paramiko and consists of two main classes:

#### SSHConnection

Represents a single SSH connection with its parameters.

**Location**: `cetmix_tower_server/ssh/ssh.py`

**Attributes**:
- `host`: Server IP address
- `port`: SSH port
- `username`: SSH username
- `password`: SSH password (optional)
- `ssh_key`: SSH private key (optional)
- `host_key`: Expected host key for verification
- `mode`: Auth mode ('p' for password, 'k' for key)
- `timeout`: Connection timeout

#### SSHManager

Manages SSH connections and provides command execution interface.

**Location**: `cetmix_tower_server/ssh/ssh.py`

**Methods**:
- `connection.connect()`: Establish SSH connection
- `connection.disconnect()`: Close connection
- `command_executor.exec_command(cmd, sudo)`: Execute command
- `sftp_service.upload_file(local, remote)`: Upload file
- `sftp_service.download_file(remote)`: Download file
- `sftp_service.delete_file(remote)`: Delete file

#### @ensure_ssh_disconnect Decorator

Ensures SSH connections are properly closed after operations.

**Location**: `cetmix_tower_server/models/cx_tower_server.py:32-68`

**How it works**:
1. Obtains SSH connection before function execution
2. Registers disconnect hooks for commit and rollback
3. Executes the wrapped function
4. Connection is disconnected regardless of outcome

**Example**:
```python
@ensure_ssh_disconnect
def my_operation(self):
    client = self._get_ssh_client(raise_on_error=True)
    # Do work with client
    # Automatic cleanup happens after transaction
```

### Code Examples

#### Example 1: Create a Server Programmatically

```python
# File: custom_module/models/server_creator.py

def create_production_server(self, name, ip_address, ssh_key_id):
    """Create a production server with standard settings"""
    server_obj = self.env['cx.tower.server']

    server = server_obj.create({
        'name': name,
        'ip_v4_address': ip_address,
        'ssh_port': 22,
        'ssh_username': 'deploy',
        'ssh_auth_mode': 'k',
        'ssh_key_id': ssh_key_id,
        'use_sudo': 'n',  # Sudo without password
        'tag_ids': [(6, 0, [self.env.ref('my_module.tag_production').id])],
    })

    # Test connection
    server.test_ssh_connection()

    return server
```

#### Example 2: Run Multiple Commands with Shared Connection

```python
# File: custom_module/models/batch_executor.py

@ensure_ssh_disconnect
def run_deployment_commands(self, server):
    """Run multiple deployment commands reusing SSH connection"""
    # Get SSH client once
    ssh_client = server._get_ssh_client(raise_on_error=True)

    # Command objects
    cmd_stop = self.env.ref('my_module.command_stop_service')
    cmd_update = self.env.ref('my_module.command_update_code')
    cmd_start = self.env.ref('my_module.command_start_service')

    # Execute commands with shared connection
    server.run_command(cmd_stop, ssh_connection=ssh_client)
    server.run_command(cmd_update, ssh_connection=ssh_client)
    server.run_command(cmd_start, ssh_connection=ssh_client)

    # Connection automatically cleaned up by decorator
```

#### Example 3: Programmatic Server with Variables

```python
# File: custom_module/models/server_with_vars.py

def setup_odoo_server(self, name, ip, version):
    """Create Odoo server with version variable"""
    server = self.env['cx.tower.server'].create({
        'name': f'{name} (Odoo {version})',
        'ip_v4_address': ip,
        'ssh_username': 'odoo',
        'ssh_auth_mode': 'k',
        'ssh_key_id': self.env.ref('my_module.odoo_deploy_key').id,
    })

    # Set version variable
    version_var = self.env.ref('my_module.var_odoo_version')
    self.env['cx.tower.variable.value'].create({
        'server_id': server.id,
        'variable_id': version_var.id,
        'value_char': version,
    })

    return server
```

#### Example 4: Custom Command Runner

```python
# File: custom_module/models/custom_server.py

class CustomServer(models.Model):
    _inherit = 'cx.tower.server'

    def run_health_check(self):
        """Run custom health check command"""
        self.ensure_one()

        # Get health check command
        health_cmd = self.env.ref('my_module.cmd_health_check')

        # Run without logging for quick check
        result = self.with_context(no_command_log=True).run_command(health_cmd)

        if result['status'] == 0:
            return {
                'type': 'ir.actions.client',
                'tag': 'display_notification',
                'params': {
                    'message': f"Server {self.name} is healthy",
                    'type': 'success',
                }
            }
        else:
            raise UserError(f"Health check failed: {result['error']}")
```

#### Example 5: Server Lifecycle Hook

```python
# File: custom_module/models/server_hooks.py

class ServerWithHooks(models.Model):
    _inherit = 'cx.tower.server'

    @api.model_create_multi
    def create(self, vals_list):
        """Override create to run initialization after server creation"""
        servers = super().create(vals_list)

        # Run initialization for each server
        for server in servers:
            if server.tag_ids.filtered(lambda t: t.name == 'Auto-Init'):
                self._run_initialization(server)

        return servers

    def _run_initialization(self, server):
        """Run initialization flight plan"""
        init_plan = self.env.ref('my_module.plan_server_init')
        server.run_flight_plan(init_plan, plan_log={'label': 'auto_init'})
```

#### Example 6: Direct SSH Operations

```python
# File: custom_module/models/direct_ssh.py

@ensure_ssh_disconnect
def get_disk_usage(self, server):
    """Get disk usage without creating command log"""
    client = server._get_ssh_client(raise_on_error=True)

    # Execute df command directly
    status, response, error = client.command_executor.exec_command('df -h')

    if status == 0:
        return {
            'disk_info': ''.join(response),
            'status': 'success'
        }
    else:
        return {
            'error': ''.join(error),
            'status': 'error'
        }
    # Connection auto-cleaned by decorator
```

### Additional Technical Notes

#### Constraints

**SSH Settings Validation** (`_constraint_ssh_settings`):
- At least one IP address (v4 or v6) required
- SSH key required when `ssh_auth_mode='k'`
- SSH password required when `ssh_auth_mode='p'` (on create)

**Location**: `cetmix_tower_server/models/cx_tower_server.py:286-330`

**Skip validation**: Set context key `skip_ssh_settings_check=True`

#### Secret Fields

Fields stored securely via vault mixin:
- `ssh_password`
- `host_key`

Access via `_get_secret_value(field_name)` method.

#### Copy Behavior

When copying a server (`copy()` method):
- Secrets are copied (password, host key, secret_ids)
- Files are duplicated with `auto_sync=False`
- Variable values are duplicated
- Server logs are duplicated
- Status is reset to `None`

**Location**: `cetmix_tower_server/models/cx_tower_server.py:371-408`

#### Zombie Command Cleanup

The `_check_zombie_commands()` method identifies and terminates commands running longer than configured timeout.

**Location**: `cetmix_tower_server/models/cx_tower_server.py:1793-1819`

**Configuration**: System parameter `cetmix_tower_server.command_timeout` (seconds)

## Related Documentation

- [Command Execution](../command-execution/README.md)
- [Flight Plans](../flight-plans/README.md)
- [Variable Management](../variable-management/README.md)
- [Secret Management](../secret-management/README.md)
- [File Management](../file-management/README.md)

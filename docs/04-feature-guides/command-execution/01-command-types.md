---
title: Command Types
audience: [administrators, end-users, developers]
topics: [ssh-commands, python-code, file-templates, flight-plans]
---

# Command Types

This guide provides a comprehensive overview of the four command types available in Cetmix Tower, explaining when to use each type, how to configure them, and technical implementation details.

## Table of Contents

- [Overview of Command Types](#overview-of-command-types)
- [For Administrators](#for-administrators)
  - [SSH Commands](#ssh-commands-ssh_command)
  - [Python Code Commands](#python-code-commands-python_code)
  - [File Template Commands](#file-template-commands-file_using_template)
  - [Flight Plan Commands](#flight-plan-commands-plan)
  - [Command Configuration](#command-configuration)
  - [Access Control](#access-control)
- [For End Users](#for-end-users)
  - [Running Different Command Types](#running-different-command-types)
  - [Understanding Command Results](#understanding-command-results)
- [For Developers](#for-developers)
  - [Command Model](#command-model)
  - [Command Execution Flow](#command-execution-flow)
  - [Python Code Execution Context](#python-code-execution-context)
  - [Code Examples](#code-examples)

---

## Overview of Command Types

Cetmix Tower supports four distinct command types, each designed for specific use cases:

| Type | Action Value | Purpose | Typical Use Case |
|------|-------------|---------|------------------|
| SSH Command | `ssh_command` | Execute shell commands | System administration, deployment scripts |
| Python Code | `python_code` | Run Python code in Odoo | Data manipulation, conditional logic |
| File Template | `file_using_template` | Create/update files | Configuration management |
| Flight Plan | `plan` | Execute orchestrated operations | Complex multi-step processes |

**Defined**: `cetmix_tower_server/models/cx_tower_command.py:217-228`

```python
def _selection_action(self):
    """Actions that can be run by a command."""
    return [
        ("ssh_command", "SSH command"),
        ("python_code", "Run Python code"),
        ("file_using_template", "Create file using template"),
        ("plan", "Run flight plan"),
    ]
```

---

## For Administrators

### SSH Commands (ssh_command)

SSH commands execute shell scripts on remote servers via SSH connection.

#### When to Use

- System administration tasks (restart services, check status)
- Package management (apt update, yum install)
- File operations (copy, move, delete)
- Deployment scripts
- Database operations
- Any shell-based automation

#### Configuration

**Required Fields**:
- `name`: Command name
- `action`: Set to "SSH command"
- `code`: Shell command(s) to execute

**Optional Fields**:
- `path`: Default working directory
- `use_sudo`: Can be overridden per server
- `no_split_for_sudo`: Don't split commands on `&&` when using sudo

#### Code Examples

**Simple Command**:
```bash
systemctl status nginx
```

**Multiple Commands**:
```bash
cd /opt/myapp && \
git pull && \
pip install -r requirements.txt && \
systemctl restart myapp
```

**With Variables**:
```bash
cd {{ project_path }} && \
git checkout {{ branch_name }} && \
git pull && \
systemctl restart {{ service_name }}
```

**With Sudo Splitting** (default behavior):
```bash
# This command with sudo='p' becomes:
cd /opt/app && ls -la

# Executed as:
# sudo -S -p '' cd /opt/app
# sudo -S -p '' ls -la
```

**Without Sudo Splitting** (set `no_split_for_sudo=True`):
```bash
# This command with sudo='p' and no_split_for_sudo=True:
cd /opt/app && ls -la

# Executed as:
# sudo -S -p '' cd /opt/app && ls -la
```

#### Execution Details

**Runner Method**: `_command_runner_ssh()`
**Location**: `cetmix_tower_server/models/cx_tower_server.py:1318-1369`

**Process**:
1. Get or create SSH connection
2. Parse inline secrets from command code
3. Prepare SSH command (add sudo if needed)
4. Execute via `client.command_executor.exec_command()`
5. Parse results and update log

**Sudo Handling**:
- Without sudo: Command executed as-is
- With sudo (no password): `sudo -S -p '' <command>`
- With sudo (password): Commands split on `&&` (unless `no_split_for_sudo=True`)

**Implementation**: `_prepare_ssh_command()`
**Location**: `cetmix_tower_server/models/cx_tower_server.py:1643-1712`

### Python Code Commands (python_code)

Python code commands execute Python code within the Odoo environment, without SSH connection.

#### When to Use

- Data manipulation and calculations
- Conditional logic based on server state
- Integration with Odoo models
- API calls and external integrations
- Variable updates during flight plan execution
- Complex decision-making

#### Configuration

**Required Fields**:
- `name`: Command name
- `action`: Set to "Run Python code"
- `code`: Python code to execute (auto-populated with template)

**Code Field**:
The `code` field is auto-computed when action is set to "python_code" with a default template.

**Default Template**:
```python
# Please refer to the 'Help' tab and documentation for more information.
#
# You can return command result in the 'result' variable which is a dictionary:
#   result = {"exit_code": 0, "message": "Some message"}
#   default value is {"exit_code": 0, "message": None}
```

**Defined**: `cetmix_tower_server/models/constants.py:80-87`

#### Available Objects and Libraries

Python commands have access to the following:

**Odoo Objects**:
- `uid`: Current user ID
- `user`: Current user record
- `env`: Odoo environment
- `server`: Server record where command runs
- `tower`: Cetmix Tower helper class

**Python Libraries**:
- `time`: Time module
- `datetime`: Datetime module
- `dateutil`: Date utilities
- `timezone`: Timezone support
- `requests`: HTTP requests (post, get, delete, request methods)
- `json`: JSON operations (dumps method)
- `float_compare`: Odoo float comparison
- `UserError`: Raise user errors
- `hashlib`: Hashing (sha1, sha256, md5, etc.)
- `hmac`: HMAC operations
- `tldextract`: Domain parsing
- `dns`: DNS operations (resolver, reversename, exception)

**Custom Values**:
- `custom_values`: Dictionary for passing data between commands in flight plans

**Defined**: `cetmix_tower_server/models/cx_tower_command.py:322-432`

#### Code Examples

**Simple Example**:
```python
# Get server name
server_name = server.name

# Set result
result = {
    "exit_code": 0,
    "message": f"Running on server: {server_name}"
}
```

**Using Custom Values**:
```python
# Get value from previous command
version = custom_values.get('app_version', '1.0.0')

# Process
if version.startswith('2.'):
    message = "Version 2.x detected"
    exit_code = 0
else:
    message = "Old version, upgrade needed"
    exit_code = 1

# Set custom value for next commands
custom_values['needs_upgrade'] = exit_code == 1

result = {"exit_code": exit_code, "message": message}
```

**External API Call**:
```python
# Make HTTP request
response = requests.get('https://api.example.com/status')

if response.status_code == 200:
    data = json.loads(response.text)
    custom_values['api_status'] = data['status']
    result = {"exit_code": 0, "message": "API check successful"}
else:
    result = {"exit_code": 1, "message": f"API error: {response.status_code}"}
```

**Working with Odoo Models**:
```python
# Find related servers
same_partner_servers = env['cx.tower.server'].search([
    ('partner_id', '=', server.partner_id.id),
    ('id', '!=', server.id)
])

count = len(same_partner_servers)

result = {
    "exit_code": 0,
    "message": f"Found {count} other servers for partner {server.partner_id.name}"
}
```

**Hash Generation**:
```python
# Generate SHA256 hash
import hashlib

data = f"{server.name}:{server.ip_v4_address}"
hash_value = hashlib.sha256(data.encode()).hexdigest()

custom_values['server_hash'] = hash_value

result = {
    "exit_code": 0,
    "message": f"Generated hash: {hash_value[:16]}..."
}
```

#### Execution Details

**Runner Method**: `_command_runner_python_code()`
**Location**: `cetmix_tower_server/models/cx_tower_server.py:1427-1466`

**Process**:
1. Parse inline secrets from code
2. Get evaluation context with available objects/libraries
3. Execute code using `safe_eval()` in exec mode
4. Extract `result` dictionary from context
5. Parse and log results

**Low-Level Method**: `_run_python_code()`
**Location**: `cetmix_tower_server/models/cx_tower_server.py:1562-1641`

**Evaluation Context**: `_get_python_command_eval_context()`
**Location**: `cetmix_tower_server/models/cx_tower_command.py:519-539`

### File Template Commands (file_using_template)

File template commands create or update files on servers using pre-defined templates.

#### When to Use

- Deploy configuration files
- Generate server-specific configs
- Create dynamic scripts
- Template-based file management
- Automated configuration updates

#### Configuration

**Required Fields**:
- `name`: Command name
- `action`: Set to "Create file using template"
- `file_template_id`: Select file template to use
- `path`: Directory where file will be created

**Optional Fields**:
- `if_file_exists`: Action when file exists ('skip', 'overwrite', 'raise')
- `disconnect_file`: Disconnect file from template after creation

**File Existence Handling**:

| Option | Behavior |
|--------|----------|
| `skip` | Don't create/update if file exists (default) |
| `overwrite` | Replace existing file with new content |
| `raise` | Raise error if file exists |

**Defined**: `cetmix_tower_server/models/cx_tower_command.py:170-181`

#### Template Example

File template code can include variables:

```ini
# config.ini template
[server]
name = {{ server_name }}
host = {{ server_host }}
port = {{ server_port }}

[application]
version = {{ app_version }}
environment = {{ env_type }}
debug = {{ debug_mode }}
```

#### Execution Details

**Runner Method**: `_command_runner_file_using_template()`
**Location**: `cetmix_tower_server/models/cx_tower_server.py:1215-1316`

**Process**:
1. Get file template and plan line (if from flight plan)
2. Create file using `file_template_id.create_file()`
3. Check if file creation was skipped
4. Push file to server (if source is 'tower') or pull from server (if source is 'server')
5. Optionally disconnect file from template
6. Log success or error

**File Creation Helper**: `_command_runner_file_using_template_create_file()`
**Location**: `cetmix_tower_server/models/cx_tower_server.py:1188-1213`

### Flight Plan Commands (plan)

Flight plan commands execute entire flight plans as a single command.

#### When to Use

- Reuse flight plans within other flight plans
- Modular orchestration
- Nested automation workflows
- Complex multi-stage operations

#### Configuration

**Required Fields**:
- `name`: Command name
- `action`: Set to "Run flight plan"
- `flight_plan_id`: Select flight plan to execute

**Context Key**:
When executed from a flight plan, context key `from_command=True` is set to skip compatibility checks.

#### Execution Details

**Runner Method**: `_command_runner_flight_plan()`
**Location**: `cetmix_tower_server/models/cx_tower_server.py:1371-1425`

**Process**:
1. Generate custom label for tracking
2. Set parent flight plan log reference
3. Execute flight plan with `flight_plan_id._run_single()`
4. Check plan execution status
5. Log results

**Important**: Context key `prevent_plan_recursion=True` prevents infinite loops when flight plans call themselves.

### Command Configuration

#### Server Compatibility

Restrict commands to specific servers or operating systems:

**Fields**:
- `server_ids`: Limit to specific servers (empty = all servers)
- `os_ids`: Limit to specific operating systems (empty = all OSes)

**Check Method**: `_check_server_compatibility()`
**Location**: `cetmix_tower_server/models/cx_tower_command.py:309-318`

```python
def _check_server_compatibility(self, server):
    """Check if command is compatible with server"""
    self.ensure_one()
    return not self.server_ids or server.id in self.server_ids.ids
```

#### Parallel Execution

Control concurrent execution:

**Field**: `allow_parallel_run` (Boolean)

- `False`: Only one instance can run at a time per server
- `True`: Multiple instances can run concurrently

If blocked, status code `-201` (ANOTHER_COMMAND_RUNNING) is returned.

**Check Location**: `cetmix_tower_server/models/cx_tower_server.py:877-903`

#### Server Status Update

Automatically update server status on successful execution:

**Field**: `server_status` (Selection)

Options: stopped, starting, running, stopping, restarting, deleting, delete_error

When command finishes with status 0 and `server_status` is set, server's status field is updated.

**Update Location**: `cetmix_tower_server/models/cx_tower_server.py:1161-1169`

#### Variables

Commands can use variables in code and paths:

**Field**: `variable_ids` (computed, Many2many)

Variables are auto-detected from:
- Command code
- Command path

**Variable Detection**: Inherited from `cx.tower.template.mixin`

Variables are rendered when command runs using `_render_command()` method.

#### Tags

Organize commands using tags:

**Field**: `tag_ids` (Many2many to cx.tower.tag)

Use tags to:
- Group related commands
- Filter in UI
- Categorize by purpose (deployment, backup, monitoring, etc.)

### Access Control

#### Access Levels

Commands inherit access control from `cx.tower.access.mixin`:

- **Public**: Anyone can execute
- **User Level**: Specific users can execute
- **Manager Level**: Only managers can execute

#### User Assignment

**Fields**:
- `user_ids`: Users who can execute command
- `manager_ids`: Users who can manage command

**Relations**:
- `cx_tower_command_user_rel`
- `cx_tower_command_manager_rel`

**Defined**: `cetmix_tower_server/models/cx_tower_command.py:190-196`

---

## For End Users

### Running Different Command Types

All command types are executed the same way from a user perspective:

1. Navigate to server record
2. Click **Run Command** button
3. Select command from list
4. Provide any required variable values
5. Click **Run**

The system automatically handles execution based on command type.

### Understanding Command Results

#### Command Log Fields

After execution, view the command log to see:

**Timing**:
- **Start Date**: When command started
- **Finish Date**: When command completed
- **Duration**: Time taken (seconds)

**Status**:
- **Status Code**:
  - `0`: Success
  - Negative: Error (see error codes)
  - Positive: Custom status

**Results**:
- **Response**: Command output
- **Error**: Error messages (if any)
- **Code**: Actual code that was executed

**Context**:
- **Server**: Where it ran
- **Command**: What command was executed
- **Variables**: Variable values used

#### Interpreting Results by Type

**SSH Commands**:
- Response contains command output (stdout)
- Error contains error output (stderr)
- Status is shell exit code

**Python Code**:
- Response contains `result['message']`
- Error contains exception messages
- Status is `result['exit_code']`

**File Template**:
- Response indicates file creation success
- Shows skip message if file existed
- Error shows file creation failures

**Flight Plan**:
- Links to plan log record
- Status reflects plan execution result
- Error shows plan failure reason

---

## For Developers

### Command Model

**Model Name**: `cx.tower.command`

**Location**: `cetmix_tower_server/models/cx_tower_command.py`

**Inheritance**:
```python
_inherit = [
    "cx.tower.template.mixin",      # Variable detection
    "cx.tower.reference.mixin",     # Reference tracking
    "cx.tower.access.mixin",        # Access control (users)
    "cx.tower.access.role.mixin",   # Access control (roles)
    "cx.tower.key.mixin",           # Secret parsing
]
```

**Description**: "Cetmix Tower Command"

**Order**: `name`

### Command Execution Flow

#### Main Entry Point

**Method**: `server.run_command(command, path=None, sudo=None, ssh_connection=None, **kwargs)`

**Location**: `cetmix_tower_server/models/cx_tower_server.py:804-938`

**Flow**:
```
1. Determine sudo value
   ↓
2. Prepare log values
   ↓
3. Check command-server compatibility
   ↓
4. Check parallel run restriction
   ↓
5. Render command code and path
   ↓
6. Prepare key values
   ↓
7. Create log record (unless no_command_log=True)
   ↓
8. Call _command_runner_wrapper()
```

#### Command Runner Wrapper

**Method**: `_command_runner_wrapper()`
**Location**: `cetmix_tower_server/models/cx_tower_server.py:1055-1096`

**Purpose**: Hook for implementing custom runner mechanisms (e.g., queue jobs)

Default implementation calls `_command_runner()`

#### Main Command Router

**Method**: `_command_runner()`
**Location**: `cetmix_tower_server/models/cx_tower_server.py:1098-1186`

**Routes to type-specific runners**:

```python
if command.action == "ssh_command":
    response = self._command_runner_ssh(...)
elif command.action == "file_using_template":
    response = self._command_runner_file_using_template(...)
elif command.action == "python_code":
    response = self._command_runner_python_code(...)
elif command.action == "plan":
    response = self._command_runner_flight_plan(...)
else:
    # No runner found
    status = NO_COMMAND_RUNNER_FOUND
```

After execution, updates server status if configured.

### Python Code Execution Context

#### Evaluation Context

**Method**: `_get_python_command_eval_context(server=None, **kwargs)`
**Location**: `cetmix_tower_server/models/cx_tower_command.py:519-539`

**Returns**: Dictionary with available objects and libraries

```python
def _get_python_command_eval_context(self, server=None, **kwargs):
    """Get evaluation context for python command"""
    # Get Odoo objects
    imports = self._get_python_command_odoo_objects(server=server)

    # Add libraries
    imports.update(self._get_python_command_libraries())

    # Build context
    eval_context = {key: value["import"] for key, value in imports.items()}
    eval_context["custom_values"] = kwargs.get("variable_values", {})

    return eval_context
```

#### Odoo Objects

**Method**: `_get_python_command_odoo_objects(server=None)`
**Location**: `cetmix_tower_server/models/cx_tower_command.py:434-468`

**Returns**:
```python
{
    "uid": {"import": self._uid, "help": "Current Odoo user ID"},
    "user": {"import": self.env.user, "help": "Current Odoo user"},
    "env": {"import": self.env, "help": "Odoo Environment"},
    "server": {"import": server, "help": "Current server"},
    "tower": {"import": self.env["cetmix.tower"], "help": "Tower helper"},
}
```

#### Python Libraries

**Method**: `_get_python_command_libraries()` (cached)
**Location**: `cetmix_tower_server/models/cx_tower_command.py:322-432`

**Custom Libraries**:
Extend via `_custom_python_libraries()` method (line 470-517)

**Example Extension**:
```python
def _custom_python_libraries(self):
    """Add custom libraries"""
    custom = super()._custom_python_libraries()
    custom.update({
        "my_module": {
            "boto3": {
                "import": boto3,
                "help": "AWS SDK for Python"
            }
        }
    })
    return custom
```

### Code Examples

#### Example 1: Create SSH Command Programmatically

```python
# File: custom_module/models/command_creator.py

def create_restart_command(self, service_name):
    """Create a restart service command"""
    command_obj = self.env['cx.tower.command']

    command = command_obj.create({
        'name': f'Restart {service_name}',
        'action': 'ssh_command',
        'code': f'systemctl restart {service_name}',
        'allow_parallel_run': False,
        'server_status': 'running',
        'tag_ids': [(6, 0, [self.env.ref('my_module.tag_maintenance').id])],
    })

    return command
```

#### Example 2: Execute Command Without Logging

```python
# File: custom_module/models/quick_check.py

def quick_disk_check(self, server):
    """Quick disk usage check without logging"""
    check_cmd = self.env.ref('my_module.cmd_disk_usage')

    # Execute without creating log
    result = server.with_context(no_command_log=True).run_command(check_cmd)

    if result['status'] == 0:
        usage = result['response']
        return {'success': True, 'usage': usage}
    else:
        return {'success': False, 'error': result['error']}
```

#### Example 3: Python Command with Custom Libraries

```python
# File: custom_module/models/cx_tower_command.py

class CxTowerCommand(models.Model):
    _inherit = 'cx.tower.command'

    def _custom_python_libraries(self):
        """Add boto3 for AWS operations"""
        custom = super()._custom_python_libraries()

        import boto3
        from odoo.tools.safe_eval import wrap_module

        custom.update({
            "cetmix_tower_aws": {
                "boto3": {
                    "import": wrap_module(boto3, ['client', 'resource']),
                    "help": "AWS SDK - Available: client(), resource()"
                }
            }
        })
        return custom
```

Then in Python command code:
```python
# Use boto3 in command
s3 = boto3.client('s3',
    aws_access_key_id='...',
    aws_secret_access_key='...'
)

buckets = s3.list_buckets()

result = {
    "exit_code": 0,
    "message": f"Found {len(buckets['Buckets'])} S3 buckets"
}
```

#### Example 4: File Template Command

```python
# File: custom_module/models/config_deployer.py

def deploy_nginx_config(self, server, domain):
    """Deploy nginx config for domain"""

    # Get template
    template = self.env.ref('my_module.template_nginx_vhost')

    # Create command
    cmd = self.env['cx.tower.command'].create({
        'name': f'Deploy nginx config for {domain}',
        'action': 'file_using_template',
        'file_template_id': template.id,
        'path': '/etc/nginx/sites-available',
        'if_file_exists': 'overwrite',
        'disconnect_file': True,
    })

    # Set domain variable
    self.env['cx.tower.variable.value'].create({
        'server_id': server.id,
        'variable_id': self.env.ref('my_module.var_domain').id,
        'value_char': domain,
    })

    # Execute
    server.run_command(cmd)

    return cmd
```

#### Example 5: Custom Command Runner

```python
# File: custom_module/models/custom_server.py

class CustomServer(models.Model):
    _inherit = 'cx.tower.server'

    def _command_runner_wrapper(self, command, log_record,
                                 rendered_command_code, **kwargs):
        """Custom runner for async execution"""

        # Check if should run in background
        if command.tag_ids.filtered(lambda t: t.name == 'Background'):
            # Queue the job
            log_record.with_delay()._execute_command(
                command.id, rendered_command_code, **kwargs
            )
            return

        # Use default runner
        return super()._command_runner_wrapper(
            command, log_record, rendered_command_code, **kwargs
        )
```

#### Example 6: Advanced Python Command

```python
# Command code in UI
# File: Can be entered in command form

# Get all servers with same tags
current_tags = server.tag_ids.ids
similar_servers = env['cx.tower.server'].search([
    ('tag_ids', 'in', current_tags),
    ('id', '!=', server.id),
    ('active', '=', True)
])

# Check their status
running_count = similar_servers.filtered(lambda s: s.status == 'running')

# Make API call to monitoring service
try:
    response = requests.post(
        'https://monitor.example.com/api/report',
        json={
            'server': server.name,
            'similar_servers': len(similar_servers),
            'running_servers': len(running_count),
            'timestamp': datetime.datetime.now().isoformat()
        }
    )

    if response.status_code == 200:
        custom_values['monitoring_updated'] = True
        result = {
            "exit_code": 0,
            "message": f"Reported {len(similar_servers)} similar servers"
        }
    else:
        result = {
            "exit_code": 1,
            "message": f"Monitoring API error: {response.status_code}"
        }
except Exception as e:
    result = {
        "exit_code": 2,
        "message": f"Exception: {str(e)}"
    }
```

## Related Documentation

- [Server Management](../server-management/README.md)
- [Flight Plans](../flight-plans/README.md)
- [Variable Management](../variable-management/README.md)
- [Secret Management](../secret-management/README.md)
- [File Management](../file-management/README.md)

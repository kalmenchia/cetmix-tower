---
title: Odoo ORM API
description: Complete reference for accessing Cetmix Tower via Odoo ORM and RPC
category: api-references
order: 2
---

# Odoo ORM API Reference

Access Cetmix Tower programmatically through Odoo's ORM (Object-Relational Mapping) and RPC (Remote Procedure Call) interfaces.

## Table of Contents

- [Overview](#overview)
- [Authentication](#authentication)
- [Core Models](#core-models)
  - [cx.tower.server](#cxtowerserver)
  - [cx.tower.command](#cxtowercommand)
  - [cx.tower.plan](#cxtowerplan)
  - [cx.tower.file](#cxtowerfile)
  - [cx.tower.variable](#cxtowervariable)
  - [cx.tower.key](#cxtowerkey)
- [Access Methods](#access-methods)
- [Code Examples](#code-examples)
- [Error Handling](#error-handling)

## Overview

The Odoo ORM API provides three primary access methods:

1. **Direct ORM** - From Python code within Odoo (controllers, models, scheduled actions)
2. **XML-RPC** - From external applications
3. **Odoo Shell** - For administrative and debugging tasks

All methods provide access to the same Tower models and operations.

## Authentication

### Internal Access (Direct ORM)

```python
# Within Odoo (controllers, models, etc.)
servers = self.env['cx.tower.server'].search([])
```

### External Access (XML-RPC)

```python
import xmlrpc.client

# Configuration
url = 'https://your-odoo.com'
db = 'your-database'
username = 'admin'
password = 'admin'

# Authenticate
common = xmlrpc.client.ServerProxy(f'{url}/xmlrpc/2/common')
uid = common.authenticate(db, username, password, {})

# Access models
models = xmlrpc.client.ServerProxy(f'{url}/xmlrpc/2/object')
```

### Odoo Shell Access

```bash
# Start Odoo shell
odoo-bin shell -d your-database -c /etc/odoo/odoo.conf

# Access models
servers = env['cx.tower.server'].search([])
```

## Core Models

### cx.tower.server

**Model**: `cx.tower.server`
**File**: `/home/user/cetmix-tower/cetmix_tower_server/models/cx_tower_server.py`

Server entity for managing remote systems.

#### Key Fields

```python
{
    'name': 'Server Name',
    'active': True,
    'ip_v4_address': '192.168.1.100',
    'ip_v6_address': None,
    'ssh_port': 22,
    'ssh_username': 'ubuntu',
    'ssh_auth_mode': 'k',  # 'p' for password, 'k' for key
    'ssh_password': 'encrypted_password',  # If auth_mode='p'
    'ssh_key_id': key_record_id,  # If auth_mode='k'
    'skip_host_key': False,
    'host_key': 'host_key_string',
    'use_sudo': 'n',  # 'n' = without password, 'p' = with password
    'partner_id': partner_id,
    'os_id': os_id,
    'tag_ids': [(6, 0, [tag_id1, tag_id2])],
    'status': 'running',  # stopped, starting, running, stopping, etc.
    'url': 'https://example.com',
}
```

#### Key Methods

##### `run_command(command, path=None, sudo=None, ssh_connection=None, **kwargs)`

Run a command on the server.

**Parameters**:
- `command` (recordset): `cx.tower.command` record
- `path` (str, optional): Override default command path
- `sudo` (bool, optional): Use sudo
- `ssh_connection` (SSHManager, optional): Reuse existing SSH connection
- `kwargs` (dict): Additional arguments
  - `log` (dict): Values for command log
  - `key` (dict): Values for key parser
  - `variable_values` (dict): Custom variable values `{variable_reference: value}`

**Returns**: `None` (creates log record) or `dict` if `no_command_log` context is set

**Example**:
```python
# Internal
server = env['cx.tower.server'].browse(1)
command = env['cx.tower.command'].browse(1)
server.run_command(command)

# With custom variables
server.run_command(
    command,
    variable_values={'odoo_version': '17.0'}
)

# XML-RPC
models.execute_kw(db, uid, password,
    'cx.tower.server', 'run_command',
    [server_id, command_id],
    {'variable_values': {'odoo_version': '17.0'}}
)
```

##### `run_flight_plan(flight_plan, **kwargs)`

Execute a flight plan on the server.

**Parameters**:
- `flight_plan` (recordset): `cx.tower.plan` record
- `kwargs` (dict): Additional arguments
  - `plan_log` (dict): Values for flight plan log
  - `log` (dict): Values for command logs
  - `variable_values` (dict): Custom variable values

**Returns**: `cx.tower.plan.log` record

**Example**:
```python
# Internal
server = env['cx.tower.server'].browse(1)
plan = env['cx.tower.plan'].search([('reference', '=', 'deploy_app')])
plan_log = server.run_flight_plan(plan)

# XML-RPC
plan_log_id = models.execute_kw(db, uid, password,
    'cx.tower.server', 'run_flight_plan',
    [server_id, plan_id]
)
```

##### `test_ssh_connection(raise_on_error=True, return_notification=True, try_command=True, try_file=True, timeout=60)`

Test SSH connectivity to the server.

**Parameters**:
- `raise_on_error` (bool): Raise exception on error
- `return_notification` (bool): Return notification action
- `try_command` (bool): Test command execution
- `try_file` (bool): Test file operations
- `timeout` (int): Connection timeout in seconds

**Returns**: `dict` with status, response, and error

**Example**:
```python
# Internal
server = env['cx.tower.server'].browse(1)
result = server.test_ssh_connection(
    raise_on_error=False,
    return_notification=False
)
# result = {'status': 0, 'response': 'Connection successful.', 'error': ''}
```

##### `upload_file(data, remote_path, from_path=False)`

Upload file to remote server.

**Parameters**:
- `data` (str/bytes): File content or local file path
- `remote_path` (str): Full remote path (e.g., `/var/www/file.txt`)
- `from_path` (bool): If `True`, `data` is treated as local file path

**Returns**: `paramiko.sftp_attr.SFTPAttributes`

**Example**:
```python
# Upload string content
server.upload_file('Hello World', '/tmp/hello.txt')

# Upload from local file
server.upload_file('/local/path/file.txt', '/remote/path/file.txt', from_path=True)
```

##### `download_file(remote_path)`

Download file from remote server.

**Parameters**:
- `remote_path` (str): Full remote path (e.g., `/var/log/app.log`)

**Returns**: `bytes` - File content

**Example**:
```python
content = server.download_file('/var/log/app.log')
# content is bytes
```

##### `delete_file(remote_path)`

Delete file from remote server.

**Parameters**:
- `remote_path` (str): Full remote path

**Example**:
```python
server.delete_file('/tmp/old_file.txt')
```

#### Context Keys

- `no_command_log` (bool): Skip command log creation, return results directly
- `skip_ssh_settings_check` (bool): Skip SSH settings validation

---

### cx.tower.command

**Model**: `cx.tower.command`
**File**: `/home/user/cetmix-tower/cetmix_tower_server/models/cx_tower_command.py`

Commands that can be executed on servers.

#### Key Fields

```python
{
    'name': 'Command Name',
    'reference': 'command_ref',  # Unique identifier
    'active': True,
    'action': 'ssh_command',  # ssh_command, python_code, file_using_template, plan
    'code': 'echo "Hello"',  # Command code
    'path': '/home/user',  # Execution path
    'allow_parallel_run': False,
    'server_ids': [(6, 0, [server_id1])],  # Specific servers or empty for all
    'tag_ids': [(6, 0, [tag_id1])],
    'os_ids': [(6, 0, [os_id1])],
    'server_status': 'running',  # Update server status on success
    'no_split_for_sudo': False,  # Don't split on && when using sudo
}
```

#### Action Types

1. **ssh_command** - Execute SSH command
   ```python
   {
       'action': 'ssh_command',
       'code': 'systemctl restart nginx',
       'path': '/etc/nginx'
   }
   ```

2. **python_code** - Execute Python code in Tower
   ```python
   {
       'action': 'python_code',
       'code': '''
# Access Odoo environment
servers = env['cx.tower.server'].search([])
result = {'exit_code': 0, 'message': f'Found {len(servers)} servers'}
       '''
   }
   ```

3. **file_using_template** - Create/update file from template
   ```python
   {
       'action': 'file_using_template',
       'file_template_id': template_id,
       'path': '/etc/config',
       'if_file_exists': 'skip'  # skip, overwrite, raise
   }
   ```

4. **plan** - Run another flight plan
   ```python
   {
       'action': 'plan',
       'flight_plan_id': plan_id
   }
   ```

#### Key Methods

##### `get_variables_from_code(code)`

Extract variables from code (variables in `{{ variable_name }}` format).

**Returns**: `list` of variable references

**Example**:
```python
command = env['cx.tower.command'].browse(1)
variables = command.get_variables_from_code('echo {{ app_version }}')
# ['app_version']
```

##### `render_code_custom(code, **kwargs)`

Render code with variable values.

**Parameters**:
- `code` (str): Code with variables
- `kwargs`: Variable values as keyword arguments

**Returns**: `str` - Rendered code

**Example**:
```python
rendered = command.render_code_custom(
    'echo {{ app_version }}',
    app_version='1.2.3'
)
# 'echo 1.2.3'
```

---

### cx.tower.plan

**Model**: `cx.tower.plan`
**File**: `/home/user/cetmix-tower/cetmix_tower_server/models/cx_tower_plan.py`

Flight plans (sequences of commands).

#### Key Fields

```python
{
    'name': 'Deploy Application',
    'reference': 'deploy_app',
    'active': True,
    'allow_parallel_run': False,
    'server_ids': [(6, 0, [server_id1])],
    'tag_ids': [(6, 0, [tag_id1])],
    'line_ids': [(0, 0, {
        'sequence': 10,
        'command_id': command_id,
        'path': '/custom/path',
        'condition': '{{ deploy_env }} == "production"',
        'action_ids': [(0, 0, {
            'condition': '==',
            'value_char': '0',
            'action': 'n'  # n=next, e=exit, ec=exit with custom code
        })]
    })],
    'on_error_action': 'e',  # e=exit, ec=exit custom code, n=next
    'custom_exit_code': 1,
}
```

#### Key Methods

##### `_run_single(server, **kwargs)`

Run plan on a single server.

**Parameters**:
- `server` (recordset): `cx.tower.server` record
- `kwargs` (dict): Additional arguments

**Returns**: `cx.tower.plan.log` record

**Example**:
```python
plan = env['cx.tower.plan'].browse(1)
server = env['cx.tower.server'].browse(1)
log = plan._run_single(server, variable_values={'env': 'production'})
```

---

### cx.tower.file

**Model**: `cx.tower.file`
**File**: `/home/user/cetmix-tower/cetmix_tower_server/models/cx_tower_file.py`

File management on servers.

#### Key Fields

```python
{
    'name': 'config.conf',  # File name without path
    'server_dir': '/etc/app',  # Directory on server
    'source': 'tower',  # 'tower' or 'server'
    'file_type': 'text',  # 'text' or 'binary'
    'code': 'configuration content',  # For text files
    'file': base64_content,  # For binary files
    'auto_sync': True,
    'auto_sync_interval': '1-hours',  # Format: number-type
    'keep_when_deleted': False,
    'template_id': template_id,
    'server_id': server_id,
}
```

#### Key Methods

##### `action_push_to_server()`

Push file from Tower to server.

**Example**:
```python
file = env['cx.tower.file'].browse(1)
file.action_push_to_server()
```

##### `action_pull_from_server()`

Pull file from server to Tower.

**Example**:
```python
file.action_pull_from_server()
```

##### `action_delete_from_server()`

Delete file from server.

**Example**:
```python
file.action_delete_from_server()
```

##### `upload(raise_error=False)`

Low-level upload method.

##### `download(raise_error=False)`

Low-level download method.

##### `delete(raise_error=False)`

Low-level delete method.

---

### cx.tower.variable

**Model**: `cx.tower.variable`
**File**: `/home/user/cetmix-tower/cetmix_tower_server/models/cx_tower_variable.py`

Variables for use in commands and files.

#### Key Fields

```python
{
    'name': 'Application Version',
    'reference': 'app_version',  # Used in {{ app_version }}
    'variable_type': 's',  # 's' = string, 'o' = options
    'value_ids': [  # Server-specific values
        (0, 0, {
            'server_id': server_id,
            'value_char': '1.2.3'
        })
    ],
    'option_ids': [  # For type 'o'
        (0, 0, {'value': 'option1', 'label': 'Option 1'})
    ],
    'applied_expression': 'result = value.lower()',  # Transform value
    'validation_pattern': '^[0-9.]+$',  # Regex validation
    'validation_message': 'Must be version format',
}
```

#### Key Methods

##### `_validate_value(value_char)`

Validate variable value.

**Returns**: `(bool, str)` - (is_valid, error_message)

**Example**:
```python
variable = env['cx.tower.variable'].browse(1)
is_valid, message = variable._validate_value('1.2.3')
```

---

### cx.tower.key

**Model**: `cx.tower.key`
**File**: `/home/user/cetmix-tower/cetmix_tower_server/models/cx_tower_key.py`

SSH keys and secrets storage.

#### Key Fields

```python
{
    'name': 'GitHub Token',
    'reference': 'github_token',
    'key_type': 's',  # 'k' = SSH key, 's' = secret
    'secret_value': 'encrypted_secret',  # Stored in vault
    'value_ids': [  # Server/partner-specific values
        (0, 0, {
            'server_id': server_id,
            'partner_id': partner_id,
            'secret_value': 'encrypted_value'
        })
    ],
}
```

#### Usage in Code

Reference secrets using the format: `#!cxtower.secret.REFERENCE!#`

```bash
# In command code
git clone https://#!cxtower.secret.github_token!#@github.com/user/repo.git
```

#### Key Methods

##### `_parse_code(code, **kwargs)`

Replace secret placeholders with actual values.

**Parameters**:
- `code` (str): Code with secret placeholders
- `kwargs`: Optional `server_id` and `partner_id`

**Returns**: `str` - Code with secrets replaced

**Example**:
```python
key_model = env['cx.tower.key']
parsed = key_model._parse_code(
    'curl -H "Authorization: #!cxtower.secret.api_key!#"',
    server_id=1
)
```

## Access Methods

### 1. Direct ORM (Internal)

```python
# In Odoo controller
from odoo import http

class TowerController(http.Controller):
    @http.route('/tower/deploy', auth='user')
    def deploy(self):
        server = request.env['cx.tower.server'].browse(1)
        command = request.env['cx.tower.command'].search([
            ('reference', '=', 'deploy')
        ])
        server.run_command(command)
        return "Deployed"
```

### 2. XML-RPC (External)

```python
import xmlrpc.client

# Setup
url = 'https://your-odoo.com'
db = 'database'
username = 'admin'
password = 'password'

# Authenticate
common = xmlrpc.client.ServerProxy(f'{url}/xmlrpc/2/common')
uid = common.authenticate(db, username, password, {})

models = xmlrpc.client.ServerProxy(f'{url}/xmlrpc/2/object')

# Search
server_ids = models.execute_kw(
    db, uid, password,
    'cx.tower.server', 'search',
    [[['name', 'like', 'prod']]]
)

# Read
servers = models.execute_kw(
    db, uid, password,
    'cx.tower.server', 'read',
    [server_ids],
    {'fields': ['name', 'ip_v4_address', 'status']}
)

# Create
server_id = models.execute_kw(
    db, uid, password,
    'cx.tower.server', 'create',
    [{
        'name': 'New Server',
        'ip_v4_address': '10.0.0.5',
        'ssh_username': 'ubuntu',
        'ssh_port': 22,
        'ssh_auth_mode': 'p',
        'ssh_password': 'secret'
    }]
)

# Write
models.execute_kw(
    db, uid, password,
    'cx.tower.server', 'write',
    [[server_id], {'status': 'running'}]
)

# Execute method
models.execute_kw(
    db, uid, password,
    'cx.tower.server', 'run_command',
    [server_id, command_id]
)
```

### 3. odoo-rpc-client Library

```python
from odoo_rpc_client import Client

# Connect
client = Client(
    host='your-odoo.com',
    dbname='database',
    user='admin',
    pwd='password',
    protocol='jsonrpc+ssl'
)

# Access models
Server = client['cx.tower.server']
Command = client['cx.tower.command']

# Search
servers = Server.search([('active', '=', True)])

# Read
for server in servers:
    print(f"Server: {server.name}, IP: {server.ip_v4_address}")

# Execute methods
server = Server.browse(1)
command = Command.search([('reference', '=', 'restart_nginx')])[0]
server.run_command(command)
```

### 4. Odoo Shell

```bash
# Start shell
odoo-bin shell -d your-database

# In shell
>>> servers = env['cx.tower.server'].search([])
>>> server = servers[0]
>>> server.name
'Production Server'

>>> command = env['cx.tower.command'].search([('reference', '=', 'update_system')])
>>> server.run_command(command[0])

>>> # Check logs
>>> logs = env['cx.tower.command.log'].search([('server_id', '=', server.id)], limit=5)
>>> for log in logs:
...     print(f"{log.command_id.name}: {log.command_status}")
```

## Code Examples

### Example 1: Automated Deployment via XML-RPC

```python
#!/usr/bin/env python3
import xmlrpc.client
import sys

class TowerClient:
    def __init__(self, url, db, username, password):
        self.url = url
        self.db = db
        self.username = username
        self.password = password

        # Authenticate
        common = xmlrpc.client.ServerProxy(f'{url}/xmlrpc/2/common')
        self.uid = common.authenticate(db, username, password, {})

        if not self.uid:
            raise Exception("Authentication failed")

        self.models = xmlrpc.client.ServerProxy(f'{url}/xmlrpc/2/object')

    def execute(self, model, method, *args, **kwargs):
        return self.models.execute_kw(
            self.db, self.uid, self.password,
            model, method, args, kwargs
        )

    def deploy_application(self, server_ref, version):
        # Find server
        server_ids = self.execute(
            'cx.tower.server', 'search',
            [('reference', '=', server_ref)]
        )

        if not server_ids:
            raise Exception(f"Server {server_ref} not found")

        # Find deployment plan
        plan_ids = self.execute(
            'cx.tower.plan', 'search',
            [('reference', '=', 'deploy_app')]
        )

        if not plan_ids:
            raise Exception("Deployment plan not found")

        # Run deployment
        plan_log_id = self.execute(
            'cx.tower.server', 'run_flight_plan',
            server_ids[0], plan_ids[0],
            variable_values={'app_version': version}
        )

        # Wait for completion and check status
        import time
        max_wait = 300  # 5 minutes
        waited = 0

        while waited < max_wait:
            log = self.execute(
                'cx.tower.plan.log', 'read',
                [plan_log_id],
                {'fields': ['is_running', 'plan_status']}
            )[0]

            if not log['is_running']:
                if log['plan_status'] == 0:
                    print(f"✓ Deployment successful!")
                    return True
                else:
                    print(f"✗ Deployment failed with status {log['plan_status']}")
                    return False

            time.sleep(5)
            waited += 5
            print(f"Waiting... ({waited}s)")

        print("✗ Deployment timeout")
        return False

if __name__ == '__main__':
    client = TowerClient(
        url='https://your-odoo.com',
        db='production',
        username='api_user',
        password='api_password'
    )

    success = client.deploy_application('prod_server_01', '1.2.3')
    sys.exit(0 if success else 1)
```

### Example 2: Server Monitoring via ORM

```python
# As Odoo scheduled action
def check_server_health():
    """Check all production servers and alert if any issues"""
    Server = env['cx.tower.server']
    Command = env['cx.tower.command']

    # Get health check command
    health_check = Command.search([('reference', '=', 'health_check')], limit=1)

    if not health_check:
        _logger.warning("Health check command not found")
        return

    # Get all production servers
    servers = Server.search([
        ('active', '=', True),
        ('tag_ids.reference', '=', 'production')
    ])

    failed_servers = []

    for server in servers:
        # Run health check
        result = server.with_context(no_command_log=True).run_command(health_check)

        if result['status'] != 0:
            failed_servers.append({
                'name': server.name,
                'error': result.get('error', 'Unknown error')
            })

    # Send alert if any servers failed
    if failed_servers:
        _send_health_alert(failed_servers)

def _send_health_alert(failed_servers):
    """Send email alert for failed servers"""
    template = env.ref('custom_module.health_check_failed_email')
    admin_users = env['res.users'].search([('groups_id.name', '=', 'Tower / Manager')])

    for user in admin_users:
        template.send_mail(
            user.id,
            email_values={
                'email_to': user.email,
                'subject': f'Tower Health Check Failed: {len(failed_servers)} servers',
                'body_html': render_template(failed_servers)
            }
        )
```

### Example 3: Batch Server Operations

```python
# Via Odoo shell or controller
def update_all_production_servers():
    """Update system packages on all production servers"""
    Server = env['cx.tower.server']
    Command = env['cx.tower.command']

    # Get update command
    update_cmd = Command.search([('reference', '=', 'system_update')], limit=1)

    # Get production servers
    servers = Server.search([
        ('active', '=', True),
        ('tag_ids.reference', '=', 'production')
    ])

    results = {}

    for server in servers:
        try:
            # Run update
            server.run_command(update_cmd)
            results[server.name] = 'Started'
        except Exception as e:
            results[server.name] = f'Error: {str(e)}'

    return results

# Usage
results = update_all_production_servers()
for server_name, status in results.items():
    print(f"{server_name}: {status}")
```

## Error Handling

### Common Exceptions

```python
from odoo.exceptions import UserError, ValidationError, AccessError

try:
    server.run_command(command)
except AccessError:
    # User doesn't have permission
    print("Access denied")
except ValidationError as e:
    # Validation failed (e.g., SSH connection error)
    print(f"Validation error: {e}")
except UserError as e:
    # User-facing error
    print(f"Error: {e}")
```

### Checking Command Results

```python
# With no_command_log context
result = server.with_context(no_command_log=True).run_command(command)

if result['status'] == 0:
    print(f"Success: {result['response']}")
else:
    print(f"Failed with code {result['status']}: {result['error']}")
```

### Handling SSH Errors

```python
# Test connection first
try:
    test_result = server.test_ssh_connection(
        raise_on_error=True,
        return_notification=False
    )
except ValidationError as e:
    print(f"SSH connection failed: {e}")
    # Handle connection error
```

## Best Practices

1. **Use References**: Use `reference` field instead of IDs for stability across environments
   ```python
   # Good
   command = env['cx.tower.command'].search([('reference', '=', 'deploy')])

   # Avoid
   command = env['cx.tower.command'].browse(42)  # ID may differ
   ```

2. **Handle Errors**: Always wrap API calls in try-except blocks
   ```python
   try:
       server.run_command(command)
   except Exception as e:
       _logger.error(f"Command failed: {e}")
       # Handle error
   ```

3. **Use Context Keys**: Leverage context for special behavior
   ```python
   # Skip logging for monitoring checks
   result = server.with_context(no_command_log=True).run_command(check_command)
   ```

4. **Reuse SSH Connections**: For multiple commands on same server
   ```python
   client = server._get_ssh_client(raise_on_error=True)
   for command in commands:
       server.run_command(command, ssh_connection=client)
   ```

5. **Use sudo()**: For vault-encrypted fields
   ```python
   password = server.sudo()._get_secret_value('ssh_password')
   ```

6. **Check Access Rights**: Before operations
   ```python
   if server.check_access_rights('write', raise_exception=False):
       # User can modify server
       pass
   ```

## Related Documentation

- [Python Command API](02-python-command-api.md)
- [Webhook API](03-webhook-api.md)
- [User Guides](../03-user-guides/README.md)
- [Feature Guides](../04-feature-guides/README.md)

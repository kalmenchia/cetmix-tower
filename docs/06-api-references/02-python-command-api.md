---
title: Python Command API
description: Execute Python code within Tower commands with access to Odoo environment
category: api-references
order: 3
---

# Python Command API Reference

Execute Python code within Cetmix Tower commands with access to Odoo environment, current server context, and specialized libraries.

## Table of Contents

- [Overview](#overview)
- [Execution Environment](#execution-environment)
- [Available Objects](#available-objects)
- [Available Libraries](#available-libraries)
- [Return Values](#return-values)
- [Code Examples](#code-examples)
- [Best Practices](#best-practices)

## Overview

Python commands in Tower execute within a sandboxed environment using Odoo's `safe_eval`. This provides:

- Full access to Odoo ORM (`env`)
- Current server context (`server`)
- Pre-imported Python libraries
- Variable management (`custom_values`)
- Safe execution with security restrictions

**Model**: `cx.tower.command` with `action='python_code'`
**Implementation**: `/home/user/cetmix-tower/cetmix_tower_server/models/cx_tower_command.py`

## Execution Environment

### Creating Python Commands

```python
# Via ORM
command = env['cx.tower.command'].create({
    'name': 'Python Example',
    'reference': 'python_example',
    'action': 'python_code',
    'code': '''
# Your Python code here
result = {
    'exit_code': 0,
    'message': 'Success'
}
'''
})
```

### Default Template

When creating a Python command, Tower provides this template:

```python
# -*- coding: utf-8 -*-
# Use 'result' dictionary to return execution status:
# result = {
#     'exit_code': 0,  # 0 for success, non-zero for failure
#     'message': 'Success message'
# }
#
# Available objects:
# - env: Odoo Environment
# - user: Current User
# - uid: Current User ID
# - server: Current Tower Server
# - tower: Tower helper class
# - custom_values: Flight plan custom values (dict)
#
# Available libraries:
# - time, datetime, dateutil, timezone
# - requests (methods: post, get, delete, request)
# - json (methods: dumps)
# - hashlib, hmac, tldextract, dns
# - float_compare, UserError
#
# Example:
result = {
    'exit_code': 0,
    'message': f'Running on server: {server.name if server else "Unknown"}'
}
```

## Available Objects

### 1. env - Odoo Environment

Full Odoo environment for accessing all models and data.

```python
# Access any Odoo model
servers = env['cx.tower.server'].search([('active', '=', True)])

# Create records
partner = env['res.partner'].create({
    'name': 'New Partner',
    'email': 'partner@example.com'
})

# Search and read
users = env['res.users'].search([('login', 'like', 'admin')])

# Execute any Odoo operation
env.cr.execute("SELECT COUNT(*) FROM cx_tower_server")
count = env.cr.fetchone()[0]
```

**Type**: `odoo.api.Environment`
**Documentation**: [Odoo ORM Documentation](https://www.odoo.com/documentation/17.0/developer/reference/backend/orm.html)

### 2. user - Current User

The user executing the command.

```python
# Get user information
user_name = user.name
user_email = user.email
user_company = user.company_id.name

# Check user groups
is_admin = user.has_group('base.group_system')
is_tower_manager = user.has_group('cetmix_tower_server.group_manager')

# Use in logic
if not is_tower_manager:
    result = {
        'exit_code': 1,
        'message': 'Insufficient permissions'
    }
```

**Type**: `res.users` record
**Model**: `res.users`

### 3. uid - Current User ID

Integer ID of current user.

```python
# User ID
current_uid = uid  # Same as user.id

# Use in searches
my_servers = env['cx.tower.server'].search([
    ('manager_ids', 'in', [uid])
])
```

**Type**: `int`

### 4. server - Current Server

The server on which the command is executing (if applicable).

```python
# Server information
server_name = server.name
server_ip = server.ip_v4_address or server.ip_v6_address

# Access server relationships
server_tags = server.tag_ids.mapped('name')
server_os = server.os_id.name

# Get server variables
variables = server.variable_value_ids
for var_value in variables:
    print(f"{var_value.variable_id.reference}: {var_value.value_char}")

# Execute operations on server
# Note: Be careful with recursive operations
other_command = env['cx.tower.command'].search([('reference', '=', 'other_cmd')])
server.run_command(other_command)
```

**Type**: `cx.tower.server` record or `None`
**Note**: Available only when command is executed on a specific server

### 5. tower - Tower Helper Class

Convenience class for Tower operations.

```python
# Access Tower utilities
tower_servers = tower.server  # Same as env['cx.tower.server']
tower_commands = tower.command  # Same as env['cx.tower.command']

# Use in searches
production_servers = tower.server.search([
    ('tag_ids.reference', '=', 'production')
])
```

**Type**: `cetmix.tower` singleton
**Model**: `cetmix.tower`
**File**: `/home/user/cetmix-tower/cetmix_tower_server/models/cetmix_tower.py`

### 6. custom_values - Custom Variables Dictionary

Dictionary for storing and retrieving flight plan variables.

```python
# Read custom values (from flight plan or previous commands)
app_version = custom_values.get('app_version', '1.0.0')
environment = custom_values.get('environment', 'development')

# Set custom values (available to subsequent commands in flight plan)
custom_values['deployment_time'] = datetime.datetime.now().isoformat()
custom_values['deployed_by'] = user.name
custom_values['server_count'] = len(env['cx.tower.server'].search([]))

# Use in conditional logic
if custom_values.get('skip_validation'):
    result = {'exit_code': 0, 'message': 'Validation skipped'}
else:
    # Perform validation
    pass
```

**Type**: `dict`
**Scope**: Available across all commands in same flight plan execution

## Available Libraries

Python commands have access to these pre-imported libraries:

### 1. time

Standard Python time library.

```python
import time

# Wait
time.sleep(2)

# Timestamps
timestamp = time.time()
```

**Documentation**: [Python time](https://docs.python.org/3/library/time.html)

### 2. datetime

Date and time operations.

```python
from datetime import datetime, timedelta

# Current time
now = datetime.now()

# Date arithmetic
tomorrow = now + timedelta(days=1)

# Formatting
formatted = now.strftime('%Y-%m-%d %H:%M:%S')
```

**Documentation**: [Python datetime](https://docs.python.org/3/library/datetime.html)

### 3. dateutil

Advanced date handling.

```python
from dateutil import parser, relativedelta

# Parse dates
date = parser.parse('2024-01-15')

# Relative dates
next_month = datetime.now() + relativedelta.relativedelta(months=1)
```

**Documentation**: [dateutil](https://dateutil.readthedocs.io/)

### 4. timezone

Timezone support (from pytz).

```python
from pytz import timezone

# Get timezone
utc = timezone('UTC')
eastern = timezone('US/Eastern')

# Convert
utc_time = datetime.now(utc)
eastern_time = utc_time.astimezone(eastern)
```

**Documentation**: [pytz](https://pythonhosted.org/pytz/)

### 5. requests

HTTP requests (limited methods for security).

**Available methods**: `post`, `get`, `delete`, `request`

```python
import requests

# GET request
response = requests.get('https://api.example.com/status')
data = response.json()

# POST request
response = requests.post(
    'https://api.example.com/webhook',
    json={'event': 'deploy', 'version': '1.2.3'},
    headers={'Authorization': 'Bearer TOKEN'}
)

# Check response
if response.status_code == 200:
    result = {'exit_code': 0, 'message': 'Webhook sent'}
else:
    result = {'exit_code': 1, 'message': f'Request failed: {response.status_code}'}
```

**Documentation**: [Requests](https://requests.readthedocs.io/)

### 6. json

JSON encoding/decoding (limited to `dumps` for security).

```python
import json

# Encode to JSON
data = {'name': 'Server', 'status': 'running'}
json_string = json.dumps(data)

# Use with requests
response = requests.post(
    'https://api.example.com/endpoint',
    data=json.dumps({'key': 'value'}),
    headers={'Content-Type': 'application/json'}
)
```

**Note**: Use `response.json()` to decode responses

### 7. hashlib

Cryptographic hashing.

**Available methods**: `sha1`, `sha224`, `sha256`, `sha384`, `sha512`, `sha3_224`, `sha3_256`, `sha3_384`, `sha3_512`, `shake_128`, `shake_256`, `blake2b`, `blake2s`, `md5`, `new`

```python
import hashlib

# SHA256 hash
data = "Hello World"
hash_object = hashlib.sha256(data.encode())
hex_hash = hash_object.hexdigest()

# MD5 hash
md5_hash = hashlib.md5(data.encode()).hexdigest()
```

**Documentation**: [Python hashlib](https://docs.python.org/3/library/hashlib.html)

### 8. hmac

HMAC authentication.

**Available methods**: `new`, `compare_digest`

```python
import hmac
import hashlib

# Generate HMAC
secret = b'secret-key'
message = b'message to authenticate'
signature = hmac.new(secret, message, hashlib.sha256).hexdigest()

# Verify HMAC
expected = 'expected_signature'
is_valid = hmac.compare_digest(signature, expected)
```

**Documentation**: [Python hmac](https://docs.python.org/3/library/hmac.html)

### 9. tldextract

Extract domain components from URLs.

```python
import tldextract

# Extract domain parts
extracted = tldextract.extract('http://www.example.com')
# extracted.domain = 'example'
# extracted.suffix = 'com'
# extracted.subdomain = 'www'

# Use in validation
url = 'https://api.github.com/repos'
parts = tldextract.extract(url)
if parts.domain == 'github':
    # Process GitHub URL
    pass
```

**Documentation**: [tldextract](https://github.com/john-kurkowski/tldextract)

### 10. dns

DNS operations using dnspython.

```python
# DNS lookup
import dns.resolver
import dns.reversename

# Forward lookup
answers = dns.resolver.resolve('example.com', 'A')
for rdata in answers:
    print(f'IP: {rdata.address}')

# Reverse lookup
addr = dns.reversename.from_address('8.8.8.8')
answers = dns.resolver.resolve(addr, 'PTR')
for rdata in answers:
    print(f'Hostname: {rdata}')

# Handle DNS errors
try:
    answers = dns.resolver.resolve('nonexistent.example.com', 'A')
except dns.exception.DNSException as e:
    result = {'exit_code': 1, 'message': f'DNS error: {e}'}
```

**Documentation**: [dnspython](https://dnspython.readthedocs.io/)

### 11. float_compare

Odoo float comparison utility.

```python
from odoo.tools.float_utils import float_compare

# Compare floats with precision
value1 = 1.123456
value2 = 1.123457
precision = 0.0001

comparison = float_compare(value1, value2, precision_rounding=precision)
# Returns: -1, 0, or 1

if comparison == 0:
    # Values are equal within precision
    pass
```

### 12. UserError

Raise user-friendly errors.

```python
from odoo.exceptions import UserError

# Validate input
version = custom_values.get('version')
if not version:
    raise UserError('Version is required')

# Conditional error
if server.status == 'stopped':
    raise UserError(f'Server {server.name} is stopped. Start it first.')
```

## Return Values

Python commands should set the `result` dictionary to return execution status.

### Success Response

```python
result = {
    'exit_code': 0,
    'message': 'Operation completed successfully'
}
```

### Error Response

```python
result = {
    'exit_code': 1,  # Non-zero indicates error
    'message': 'Error description here'
}
```

### Custom Exit Codes

```python
# Use meaningful exit codes
EXIT_CODE_SUCCESS = 0
EXIT_CODE_VALIDATION_ERROR = 10
EXIT_CODE_CONNECTION_ERROR = 20
EXIT_CODE_PERMISSION_ERROR = 30

# Example usage
if not server.ip_v4_address:
    result = {
        'exit_code': EXIT_CODE_VALIDATION_ERROR,
        'message': 'Server IP address not configured'
    }
```

### Omitting Result

If `result` is not set, command completes successfully with no message:

```python
# This is valid - implicit success
servers = env['cx.tower.server'].search([])
custom_values['server_count'] = len(servers)
# result not set = exit_code 0
```

## Code Examples

### Example 1: Server Health Check

```python
# Check if server is responsive
import requests

server_url = server.url
if not server_url:
    result = {
        'exit_code': 1,
        'message': 'Server URL not configured'
    }
else:
    try:
        response = requests.get(
            f'{server_url}/health',
            timeout=10
        )
        if response.status_code == 200:
            result = {
                'exit_code': 0,
                'message': f'Server healthy: {response.text}'
            }
        else:
            result = {
                'exit_code': 1,
                'message': f'Server returned {response.status_code}'
            }
    except requests.RequestException as e:
        result = {
            'exit_code': 2,
            'message': f'Connection failed: {str(e)}'
        }
```

### Example 2: Conditional Deployment

```python
# Deploy only if all conditions are met
from datetime import datetime

# Check deployment window
now = datetime.now()
deploy_hour = custom_values.get('deploy_hour', 22)  # Default 10 PM

if now.hour != deploy_hour:
    result = {
        'exit_code': 1,
        'message': f'Deployment only allowed at {deploy_hour}:00'
    }
else:
    # Check if backup exists
    backup_command = env['cx.tower.command'].search([
        ('reference', '=', 'check_backup')
    ])

    if backup_command:
        backup_result = server.with_context(no_command_log=True).run_command(
            backup_command[0]
        )

        if backup_result['status'] != 0:
            result = {
                'exit_code': 2,
                'message': 'Backup verification failed'
            }
        else:
            # All checks passed
            custom_values['deployment_approved'] = True
            result = {
                'exit_code': 0,
                'message': 'Pre-deployment checks passed'
            }
```

### Example 3: Batch Operations

```python
# Update multiple servers based on tags
import time

# Get target tag from custom values
target_tag = custom_values.get('target_tag', 'production')

# Find all servers with this tag
servers = env['cx.tower.server'].search([
    ('active', '=', True),
    ('tag_ids.reference', '=', target_tag)
])

if not servers:
    result = {
        'exit_code': 1,
        'message': f'No servers found with tag: {target_tag}'
    }
else:
    # Get update command
    update_cmd = env['cx.tower.command'].search([
        ('reference', '=', 'system_update')
    ], limit=1)

    success_count = 0
    failed_servers = []

    for srv in servers:
        try:
            # Run update on each server
            srv.run_command(update_cmd)
            success_count += 1
            time.sleep(2)  # Delay between servers
        except Exception as e:
            failed_servers.append(f'{srv.name}: {str(e)}')

    # Store results in custom_values
    custom_values['updated_servers'] = success_count
    custom_values['failed_servers'] = len(failed_servers)

    if failed_servers:
        result = {
            'exit_code': 1,
            'message': f'Updated {success_count}/{len(servers)}. Failures:\n' +
                      '\n'.join(failed_servers)
        }
    else:
        result = {
            'exit_code': 0,
            'message': f'Successfully updated all {success_count} servers'
        }
```

### Example 4: API Integration

```python
# Notify external system about deployment
import requests
import json
import hashlib

# Get deployment details
app_name = custom_values.get('app_name', 'unknown')
app_version = custom_values.get('app_version', 'unknown')
environment = custom_values.get('environment', 'development')

# Prepare notification payload
payload = {
    'application': app_name,
    'version': app_version,
    'environment': environment,
    'server': server.name,
    'timestamp': datetime.datetime.now().isoformat(),
    'deployed_by': user.name
}

# Get API credentials from Tower secrets
api_url = 'https://api.example.com/deployments'
# Assume API key is stored as Tower secret

try:
    response = requests.post(
        api_url,
        json=payload,
        headers={
            'Content-Type': 'application/json'
        },
        timeout=30
    )

    if response.status_code == 201:
        custom_values['notification_id'] = response.json().get('id')
        result = {
            'exit_code': 0,
            'message': f'Deployment notification sent: {response.json()}'
        }
    else:
        result = {
            'exit_code': 1,
            'message': f'Notification failed: {response.status_code} - {response.text}'
        }

except requests.RequestException as e:
    result = {
        'exit_code': 2,
        'message': f'API request failed: {str(e)}'
    }
```

### Example 5: Database Operations

```python
# Query and report on Tower data
from datetime import datetime, timedelta

# Get statistics for the last 24 hours
yesterday = datetime.now() - timedelta(hours=24)

# Count successful commands
successful_logs = env['cx.tower.command.log'].search_count([
    ('create_date', '>=', yesterday),
    ('command_status', '=', 0)
])

# Count failed commands
failed_logs = env['cx.tower.command.log'].search_count([
    ('create_date', '>=', yesterday),
    ('command_status', '!=', 0)
])

# Get most active servers
env.cr.execute("""
    SELECT server_id, COUNT(*) as cmd_count
    FROM cx_tower_command_log
    WHERE create_date >= %s
    GROUP BY server_id
    ORDER BY cmd_count DESC
    LIMIT 5
""", (yesterday,))

top_servers = env.cr.fetchall()
server_stats = []
for server_id, count in top_servers:
    srv = env['cx.tower.server'].browse(server_id)
    server_stats.append(f'{srv.name}: {count} commands')

# Prepare report
report = f"""
Tower Activity Report (Last 24 Hours)
=====================================
Successful Commands: {successful_logs}
Failed Commands: {failed_logs}
Success Rate: {successful_logs/(successful_logs+failed_logs)*100:.1f}%

Most Active Servers:
{chr(10).join(server_stats)}
"""

# Store in custom_values for use in notification
custom_values['activity_report'] = report

result = {
    'exit_code': 0,
    'message': report
}
```

### Example 6: Dynamic Server Selection

```python
# Select servers based on load or other metrics
import random

# Get available servers with specific tags
available_servers = env['cx.tower.server'].search([
    ('active', '=', True),
    ('status', '=', 'running'),
    ('tag_ids.reference', 'in', ['web', 'app'])
])

if not available_servers:
    result = {
        'exit_code': 1,
        'message': 'No available servers found'
    }
else:
    # Simple round-robin or random selection
    # In real scenario, could check actual load metrics
    selected_server = random.choice(available_servers)

    # Store selected server for subsequent commands
    custom_values['target_server_id'] = selected_server.id
    custom_values['target_server_name'] = selected_server.name

    result = {
        'exit_code': 0,
        'message': f'Selected server: {selected_server.name} ({selected_server.ip_v4_address})'
    }
```

## Best Practices

### 1. Always Set Result

```python
# Good
result = {'exit_code': 0, 'message': 'Success'}

# Also good (implicit success)
custom_values['status'] = 'completed'

# Avoid unclear outcomes
# (no result, no custom_values updates)
```

### 2. Handle Exceptions

```python
# Good
try:
    servers = env['cx.tower.server'].search([])
    # ... operations
    result = {'exit_code': 0, 'message': f'Processed {len(servers)} servers'}
except Exception as e:
    result = {'exit_code': 1, 'message': f'Error: {str(e)}'}

# Also good - let UserError propagate
if not server:
    raise UserError('Server context required')
```

### 3. Use Custom Values for State

```python
# Good - Share data between commands in flight plan
custom_values['database_backup_path'] = '/backups/db_20240115.sql'
custom_values['deployment_started'] = datetime.now().isoformat()

# Later command can use these values
backup_path = custom_values.get('database_backup_path')
if backup_path:
    # Restore from backup
    pass
```

### 4. Validate Inputs

```python
# Good
required_vars = ['app_version', 'environment', 'deploy_user']
missing = [v for v in required_vars if not custom_values.get(v)]

if missing:
    result = {
        'exit_code': 1,
        'message': f'Missing required variables: {", ".join(missing)}'
    }
else:
    # Proceed with deployment
    pass
```

### 5. Use Meaningful Exit Codes

```python
# Good - Clear error categories
EXIT_SUCCESS = 0
EXIT_VALIDATION_ERROR = 1
EXIT_CONNECTION_ERROR = 2
EXIT_PERMISSION_ERROR = 3
EXIT_TIMEOUT = 4

if not server.ip_v4_address:
    result = {
        'exit_code': EXIT_VALIDATION_ERROR,
        'message': 'Server IP not configured'
    }
```

### 6. Log Important Operations

```python
# Good
import logging
_logger = logging.getLogger(__name__)

_logger.info(f'Starting deployment on {server.name}')
# ... operations
_logger.info(f'Deployment completed successfully')

result = {'exit_code': 0, 'message': 'Deployed'}
```

### 7. Limit External API Calls

```python
# Good - Use timeout
response = requests.get(url, timeout=30)

# Good - Handle failures gracefully
try:
    response = requests.post(api_url, json=data, timeout=10)
    response.raise_for_status()
except requests.Timeout:
    result = {'exit_code': 1, 'message': 'API request timed out'}
except requests.RequestException as e:
    result = {'exit_code': 1, 'message': f'API error: {e}'}
```

### 8. Don't Modify Server in Recursive Calls

```python
# Avoid - Can cause issues
if server:
    server.run_command(some_command)  # Executing on same server

# Better - Use different server or flag
other_server = env['cx.tower.server'].browse(other_id)
other_server.run_command(command)
```

## Security Considerations

1. **Sandboxed Execution** - Code runs in `safe_eval` with limited imports
2. **No File System Access** - Cannot directly read/write local files
3. **Limited Imports** - Only pre-approved libraries available
4. **User Context** - Executes with current user's permissions
5. **Audit Trail** - All executions logged in command logs

## Performance Tips

1. **Avoid Heavy Loops** - Use Odoo's search/read efficiently
2. **Batch Operations** - Use `create()`, `write()` with multiple records
3. **Use SQL for Aggregates** - `env.cr.execute()` for complex queries
4. **Limit External Calls** - Cache results when possible
5. **Set Timeouts** - Always use timeouts for external requests

## Troubleshooting

### Common Issues

**NameError: name 'xyz' is not defined**
- Only pre-imported objects/libraries are available
- Check available objects list

**ValidationError during execution**
- Check user permissions
- Verify server configuration
- Review error in command log

**Timeout errors**
- Reduce operation complexity
- Use job queue module for long operations
- Increase command timeout in settings

## Related Documentation

- [Odoo ORM API](01-odoo-orm-api.md)
- [Webhook API](03-webhook-api.md)
- [Variables Guide](../04-feature-guides/02-variables-management.md)
- [Command Management](../04-feature-guides/04-command-management.md)

---
title: Webhook API
description: Trigger Tower operations via HTTP webhooks
category: api-references
order: 4
---

# Webhook API Reference

Trigger Cetmix Tower commands and flight plans via HTTP webhooks with customizable authentication and payload processing.

## Table of Contents

- [Overview](#overview)
- [Webhook Configuration](#webhook-configuration)
- [Authentication Methods](#authentication-methods)
- [Request Format](#request-format)
- [Response Format](#response-format)
- [Code Examples](#code-examples)
- [Security Considerations](#security-considerations)

## Overview

The Webhook API allows external systems to trigger Tower operations via HTTP requests. Webhooks can:

- Execute custom Python code
- Trigger Tower commands
- Run flight plans
- Access payload data
- Use authentication for security
- Return custom responses

**Module**: `cetmix_tower_webhook`
**Model**: `cx.tower.webhook`
**File**: `/home/user/cetmix-tower/cetmix_tower_webhook/models/cx_tower_webhook.py`

## Webhook Configuration

### Creating a Webhook

```python
# Via Odoo ORM
webhook = env['cx.tower.webhook'].create({
    'name': 'Deploy Application',
    'endpoint': 'deploy/app',
    'method': 'post',  # 'post' or 'get'
    'content_type': 'json',  # 'json' or 'form'
    'authenticator_id': authenticator_id,
    'user_id': user_id,  # User context for execution
    'active': True,
    'code': '''
# Process webhook payload
app_name = payload.get('application')
version = payload.get('version')

if not app_name or not version:
    result = {
        'exit_code': 1,
        'message': 'Missing application or version'
    }
else:
    # Find server and deploy
    server = env['cx.tower.server'].search([
        ('reference', '=', payload.get('server', 'production'))
    ], limit=1)

    if not server:
        result = {
            'exit_code': 1,
            'message': 'Server not found'
        }
    else:
        # Run deployment command
        deploy_cmd = env['cx.tower.command'].search([
            ('reference', '=', 'deploy_app')
        ], limit=1)

        server.run_command(
            deploy_cmd,
            variable_values={
                'app_name': app_name,
                'version': version
            }
        )

        result = {
            'exit_code': 0,
            'message': f'Deployment started for {app_name} v{version}'
        }
    '''
})
```

### Webhook Fields

```python
{
    'name': 'Webhook Name',
    'endpoint': 'my/webhook/path',  # URL path component
    'full_url': 'https://odoo.example.com/cetmix_tower_webhooks/my/webhook/path',  # Auto-computed
    'method': 'post',  # 'post' or 'get'
    'content_type': 'json',  # 'json' or 'form'
    'authenticator_id': auth_id,  # Required
    'user_id': user_id,  # User context (default: admin)
    'active': True,
    'code': '# Python code to execute',
    'variable_ids': [(6, 0, [var_id1, var_id2])],  # Used variables
}
```

### Endpoint Format

Endpoint must:
- Start and end with letter or digit
- May contain `_`, `-`, `/` in between
- No spaces or special characters

**Valid**: `deploy`, `github/webhook`, `ci-cd/deploy`
**Invalid**: `/deploy`, `deploy/`, `deploy webhook`, `déploy`

### Full URL

Webhooks are accessed at:
```
https://<your-odoo-url>/cetmix_tower_webhooks/<endpoint>
```

Example:
```
https://tower.example.com/cetmix_tower_webhooks/deploy/app
```

## Authentication Methods

Webhooks require authentication. Create authenticators via `cx.tower.webhook.authenticator`.

### 1. Bearer Token

Simple token-based authentication.

**Configuration**:
```python
auth = env['cx.tower.webhook.authenticator'].create({
    'name': 'Deploy Token',
    'auth_type': 'bearer',
    'bearer_token': 'your-secret-token-here'
})
```

**Usage**:
```bash
curl -X POST https://tower.example.com/cetmix_tower_webhooks/deploy \
  -H "Authorization: Bearer your-secret-token-here" \
  -H "Content-Type: application/json" \
  -d '{"app": "myapp", "version": "1.2.3"}'
```

### 2. HMAC Signature

Message authentication using shared secret.

**Configuration**:
```python
auth = env['cx.tower.webhook.authenticator'].create({
    'name': 'HMAC Auth',
    'auth_type': 'hmac',
    'hmac_secret': 'shared-secret-key',
    'hmac_header': 'X-Hub-Signature-256',  # Header name
    'hmac_algorithm': 'sha256'  # sha1, sha256, etc.
})
```

**Usage**:
```python
import hmac
import hashlib
import requests

secret = b'shared-secret-key'
payload = '{"app":"myapp","version":"1.2.3"}'

# Generate signature
signature = 'sha256=' + hmac.new(
    secret,
    payload.encode(),
    hashlib.sha256
).hexdigest()

# Send request
requests.post(
    'https://tower.example.com/cetmix_tower_webhooks/deploy',
    data=payload,
    headers={
        'Content-Type': 'application/json',
        'X-Hub-Signature-256': signature
    }
)
```

### 3. API Key

Header-based API key authentication.

**Configuration**:
```python
auth = env['cx.tower.webhook.authenticator'].create({
    'name': 'API Key',
    'auth_type': 'api_key',
    'api_key': 'your-api-key',
    'api_key_header': 'X-API-Key'  # Header name
})
```

**Usage**:
```bash
curl -X POST https://tower.example.com/cetmix_tower_webhooks/deploy \
  -H "X-API-Key: your-api-key" \
  -H "Content-Type: application/json" \
  -d '{"app": "myapp"}'
```

### 4. No Authentication (Development Only)

**⚠️ Warning**: Only for development/testing!

```python
auth = env['cx.tower.webhook.authenticator'].create({
    'name': 'No Auth',
    'auth_type': 'none'
})
```

## Request Format

### POST Request with JSON

```bash
curl -X POST https://tower.example.com/cetmix_tower_webhooks/deploy \
  -H "Authorization: Bearer TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "application": "web-app",
    "version": "2.1.0",
    "environment": "production",
    "notify": true
  }'
```

**Webhook code access**:
```python
# Access payload
app = payload.get('application')
version = payload.get('version')
env_name = payload.get('environment', 'development')
notify = payload.get('notify', False)
```

### POST Request with Form Data

```bash
curl -X POST https://tower.example.com/cetmix_tower_webhooks/deploy \
  -H "Authorization: Bearer TOKEN" \
  -d "application=web-app" \
  -d "version=2.1.0" \
  -d "environment=production"
```

**Webhook code access** (same as JSON):
```python
app = payload.get('application')
```

### GET Request

```bash
curl -X GET "https://tower.example.com/cetmix_tower_webhooks/health?server=web01&check=full" \
  -H "Authorization: Bearer TOKEN"
```

**Webhook code access**:
```python
# Access query parameters
server_ref = payload.get('server')
check_type = payload.get('check', 'basic')
```

### Headers Access

```python
# Access request headers
user_agent = headers.get('User-Agent')
content_type = headers.get('Content-Type')
custom_header = headers.get('X-Custom-Header')

# Common use case - verify source
github_event = headers.get('X-GitHub-Event')
if github_event == 'push':
    # Process GitHub push event
    pass
```

## Response Format

Webhooks return JSON responses.

### Success Response

**Code**:
```python
result = {
    'exit_code': 0,
    'message': 'Deployment started successfully'
}
```

**HTTP Response**:
```json
{
    "exit_code": 0,
    "message": "Deployment started successfully"
}
```

**Status Code**: 200 OK

### Error Response

**Code**:
```python
result = {
    'exit_code': 1,
    'message': 'Invalid application name'
}
```

**HTTP Response**:
```json
{
    "exit_code": 1,
    "message": "Invalid application name"
}
```

**Status Code**: 200 OK (still returns 200, check exit_code)

### Authentication Failure

**HTTP Response**:
```json
{
    "error": "Authentication failed"
}
```

**Status Code**: 401 Unauthorized

### Invalid Endpoint

**HTTP Response**:
```json
{
    "error": "Webhook not found"
}
```

**Status Code**: 404 Not Found

## Webhook Code Environment

Webhook Python code has access to:

### Available Objects

```python
# env - Odoo Environment
servers = env['cx.tower.server'].search([])

# user - Current User (from webhook.user_id)
user_name = user.name

# uid - User ID
current_uid = uid

# payload - Request payload (dict)
data = payload.get('key')

# headers - Request headers (dict)
auth = headers.get('Authorization')

# result - Return value (set this)
result = {'exit_code': 0, 'message': 'Success'}

# custom_values - Shared state dict
custom_values['processed'] = True
```

### Available Libraries

Same as Python Command API:
- `time`, `datetime`, `dateutil`, `timezone`
- `requests`, `json`
- `hashlib`, `hmac`
- `tldextract`, `dns`
- `float_compare`, `UserError`

See [Python Command API](02-python-command-api.md#available-libraries) for details.

## Code Examples

### Example 1: Simple Command Trigger

```python
# Webhook: trigger-backup
# Method: POST
# Content-Type: JSON

server_ref = payload.get('server')
backup_type = payload.get('type', 'full')

if not server_ref:
    result = {
        'exit_code': 1,
        'message': 'Server reference required'
    }
else:
    # Find server
    server = env['cx.tower.server'].search([
        ('reference', '=', server_ref)
    ], limit=1)

    if not server:
        result = {
            'exit_code': 1,
            'message': f'Server {server_ref} not found'
        }
    else:
        # Find backup command
        backup_cmd = env['cx.tower.command'].search([
            ('reference', '=', f'backup_{backup_type}')
        ], limit=1)

        if not backup_cmd:
            result = {
                'exit_code': 1,
                'message': f'Backup command for type {backup_type} not found'
            }
        else:
            # Execute backup
            server.run_command(backup_cmd)
            result = {
                'exit_code': 0,
                'message': f'{backup_type.capitalize()} backup started on {server.name}'
            }
```

**Usage**:
```bash
curl -X POST https://tower.example.com/cetmix_tower_webhooks/trigger-backup \
  -H "Authorization: Bearer TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"server": "prod_db_01", "type": "incremental"}'
```

### Example 2: GitHub Webhook Integration

```python
# Webhook: github/deploy
# Method: POST
# Content-Type: JSON
# Handles GitHub push events

# Verify GitHub event
github_event = headers.get('X-GitHub-Event')

if github_event != 'push':
    result = {
        'exit_code': 0,
        'message': f'Ignored event: {github_event}'
    }
else:
    # Extract push information
    ref = payload.get('ref', '')
    repository = payload.get('repository', {})
    repo_name = repository.get('name', 'unknown')
    pusher = payload.get('pusher', {}).get('name', 'unknown')

    # Only deploy main branch
    if ref != 'refs/heads/main':
        result = {
            'exit_code': 0,
            'message': f'Ignored branch: {ref}'
        }
    else:
        # Find deployment plan
        deploy_plan = env['cx.tower.plan'].search([
            ('reference', '=', 'deploy_from_github')
        ], limit=1)

        if not deploy_plan:
            result = {
                'exit_code': 1,
                'message': 'Deployment plan not found'
            }
        else:
            # Find target servers
            servers = env['cx.tower.server'].search([
                ('tag_ids.reference', '=', 'github_auto_deploy')
            ])

            if not servers:
                result = {
                    'exit_code': 1,
                    'message': 'No auto-deploy servers found'
                }
            else:
                # Trigger deployment on all servers
                for server in servers:
                    server.run_flight_plan(
                        deploy_plan,
                        variable_values={
                            'repository': repo_name,
                            'branch': 'main',
                            'deployed_by': pusher
                        }
                    )

                result = {
                    'exit_code': 0,
                    'message': f'Deployment started on {len(servers)} server(s) by {pusher}'
                }
```

**GitHub Configuration**:
```
Payload URL: https://tower.example.com/cetmix_tower_webhooks/github/deploy
Content type: application/json
Events: Just the push event
```

### Example 3: Health Check Endpoint

```python
# Webhook: health/check
# Method: GET
# Simple health check endpoint

server_ref = payload.get('server')

if not server_ref:
    # Overall Tower health
    active_servers = env['cx.tower.server'].search_count([
        ('active', '=', True)
    ])

    running_commands = env['cx.tower.command.log'].search_count([
        ('is_running', '=', True)
    ])

    result = {
        'exit_code': 0,
        'message': f'Tower healthy: {active_servers} servers, {running_commands} running commands'
    }
else:
    # Specific server health
    server = env['cx.tower.server'].search([
        ('reference', '=', server_ref)
    ], limit=1)

    if not server:
        result = {
            'exit_code': 1,
            'message': f'Server {server_ref} not found'
        }
    else:
        # Run health check command
        health_cmd = env['cx.tower.command'].search([
            ('reference', '=', 'health_check')
        ], limit=1)

        if health_cmd:
            check_result = server.with_context(no_command_log=True).run_command(
                health_cmd
            )

            if check_result['status'] == 0:
                result = {
                    'exit_code': 0,
                    'message': f'Server {server.name} is healthy'
                }
            else:
                result = {
                    'exit_code': 1,
                    'message': f'Server {server.name} health check failed: {check_result["error"]}'
                }
        else:
            result = {
                'exit_code': 0,
                'message': f'Server {server.name} exists (no health check available)'
            }
```

**Usage**:
```bash
# Overall health
curl "https://tower.example.com/cetmix_tower_webhooks/health/check?token=TOKEN"

# Specific server
curl "https://tower.example.com/cetmix_tower_webhooks/health/check?server=web01&token=TOKEN"
```

### Example 4: Slack Command Integration

```python
# Webhook: slack/command
# Method: POST
# Content-Type: form
# Handles Slack slash commands

# Slack sends form data
command_text = payload.get('text', '')
user_name = payload.get('user_name', 'unknown')
channel_name = payload.get('channel_name', 'unknown')

# Parse command
parts = command_text.split()
if not parts:
    result = {
        'exit_code': 1,
        'message': 'Usage: /tower <action> <server> [args]'
    }
else:
    action = parts[0]
    server_ref = parts[1] if len(parts) > 1 else None

    if action == 'status' and server_ref:
        # Get server status
        server = env['cx.tower.server'].search([
            ('reference', '=', server_ref)
        ], limit=1)

        if not server:
            result = {
                'exit_code': 1,
                'message': f'Server `{server_ref}` not found'
            }
        else:
            # Get recent logs
            logs = env['cx.tower.command.log'].search([
                ('server_id', '=', server.id)
            ], limit=5, order='create_date desc')

            log_summary = '\n'.join([
                f'• {log.command_id.name}: {"✓" if log.command_status == 0 else "✗"}'
                for log in logs
            ])

            result = {
                'exit_code': 0,
                'message': f'*{server.name}*\nStatus: {server.status}\nIP: {server.ip_v4_address}\n\nRecent commands:\n{log_summary}'
            }

    elif action == 'restart' and server_ref:
        # Restart server service
        server = env['cx.tower.server'].search([
            ('reference', '=', server_ref)
        ], limit=1)

        restart_cmd = env['cx.tower.command'].search([
            ('reference', '=', 'restart_service')
        ], limit=1)

        if server and restart_cmd:
            server.run_command(restart_cmd)
            result = {
                'exit_code': 0,
                'message': f'Restart initiated on `{server.name}` by {user_name}'
            }
        else:
            result = {
                'exit_code': 1,
                'message': 'Server or command not found'
            }

    else:
        result = {
            'exit_code': 1,
            'message': 'Unknown action. Available: status, restart'
        }
```

**Slack Configuration**:
```
Command: /tower
Request URL: https://tower.example.com/cetmix_tower_webhooks/slack/command
Method: POST
```

### Example 5: CI/CD Pipeline Trigger

```python
# Webhook: ci/pipeline
# Method: POST
# Content-Type: JSON
# Trigger deployment pipeline from CI/CD

import requests

# Extract CI/CD variables
pipeline_id = payload.get('pipeline_id')
project = payload.get('project')
branch = payload.get('branch')
commit_sha = payload.get('commit_sha')
environment = payload.get('environment', 'staging')

# Validate required fields
if not all([pipeline_id, project, branch, commit_sha]):
    result = {
        'exit_code': 1,
        'message': 'Missing required fields: pipeline_id, project, branch, commit_sha'
    }
else:
    # Find deployment plan for environment
    plan = env['cx.tower.plan'].search([
        ('reference', '=', f'deploy_{environment}')
    ], limit=1)

    if not plan:
        result = {
            'exit_code': 1,
            'message': f'Deployment plan for {environment} not found'
        }
    else:
        # Find servers for environment
        servers = env['cx.tower.server'].search([
            ('tag_ids.reference', '=', environment)
        ])

        if not servers:
            result = {
                'exit_code': 1,
                'message': f'No servers found for environment: {environment}'
            }
        else:
            # Store deployment metadata
            custom_values['pipeline_id'] = pipeline_id
            custom_values['commit_sha'] = commit_sha
            custom_values['project'] = project

            # Trigger deployment
            deployment_count = 0
            for server in servers:
                try:
                    server.run_flight_plan(
                        plan,
                        variable_values={
                            'branch': branch,
                            'commit': commit_sha,
                            'project': project
                        }
                    )
                    deployment_count += 1
                except Exception as e:
                    # Log but continue
                    pass

            result = {
                'exit_code': 0,
                'message': f'Deployment triggered on {deployment_count} server(s)\nPipeline: {pipeline_id}\nCommit: {commit_sha[:8]}'
            }
```

**GitLab CI Usage**:
```yaml
deploy:
  stage: deploy
  script:
    - |
      curl -X POST https://tower.example.com/cetmix_tower_webhooks/ci/pipeline \
        -H "Authorization: Bearer $TOWER_TOKEN" \
        -H "Content-Type: application/json" \
        -d "{
          \"pipeline_id\": \"$CI_PIPELINE_ID\",
          \"project\": \"$CI_PROJECT_NAME\",
          \"branch\": \"$CI_COMMIT_BRANCH\",
          \"commit_sha\": \"$CI_COMMIT_SHA\",
          \"environment\": \"production\"
        }"
```

## Security Considerations

### 1. Always Use Authentication

**Never** create webhooks without authentication in production:
```python
# ❌ BAD - No auth
auth = env['cx.tower.webhook.authenticator'].create({
    'auth_type': 'none'
})

# ✅ GOOD - Use bearer token or stronger
auth = env['cx.tower.webhook.authenticator'].create({
    'auth_type': 'bearer',
    'bearer_token': generate_secure_token()
})
```

### 2. Use HTTPS

Always use HTTPS for webhook URLs:
- Protects tokens/secrets in transit
- Prevents man-in-the-middle attacks
- Required for production use

### 3. Validate Payload

Always validate payload data:
```python
# Validate required fields
required = ['app', 'version']
if not all(payload.get(field) for field in required):
    result = {'exit_code': 1, 'message': 'Missing required fields'}
    # Stop execution

# Validate values
version = payload.get('version')
if not version.match(r'^\d+\.\d+\.\d+$'):
    result = {'exit_code': 1, 'message': 'Invalid version format'}
```

### 4. Use Specific User Context

Set appropriate `user_id` for webhooks:
```python
# Get user with limited permissions
webhook_user = env['res.users'].search([
    ('login', '=', 'webhook_executor')
])

webhook = env['cx.tower.webhook'].create({
    'user_id': webhook_user.id,  # Not admin
    # ...
})
```

### 5. Rate Limiting

Consider implementing rate limiting:
```python
# Check recent webhook calls
recent_calls = env['cx.tower.webhook.log'].search_count([
    ('webhook_id', '=', env.context.get('webhook_id')),
    ('create_date', '>=', datetime.now() - timedelta(minutes=1))
])

if recent_calls > 10:  # Max 10 per minute
    result = {
        'exit_code': 1,
        'message': 'Rate limit exceeded'
    }
```

### 6. IP Whitelisting

Implement IP restrictions:
```python
# Check source IP
allowed_ips = ['192.168.1.0/24', '10.0.0.5']
source_ip = headers.get('X-Real-IP') or headers.get('X-Forwarded-For')

if source_ip not in allowed_ips:
    result = {
        'exit_code': 1,
        'message': 'IP not allowed'
    }
```

### 7. Verify Webhook Source

For GitHub, Slack, etc., verify signatures:
```python
# GitHub signature verification
import hmac
import hashlib

github_signature = headers.get('X-Hub-Signature-256', '')
secret = b'your-webhook-secret'

# Compute expected signature
payload_body = # Raw request body
expected = 'sha256=' + hmac.new(
    secret,
    payload_body.encode(),
    hashlib.sha256
).hexdigest()

if not hmac.compare_digest(github_signature, expected):
    result = {
        'exit_code': 1,
        'message': 'Invalid signature'
    }
```

### 8. Log Webhook Calls

All webhook calls are automatically logged in `cx.tower.webhook.log`:
```python
# View webhook logs
logs = env['cx.tower.webhook.log'].search([
    ('webhook_id', '=', webhook_id)
], limit=100, order='create_date desc')

for log in logs:
    print(f"{log.create_date}: {log.exit_code} - {log.message}")
```

## Testing Webhooks

### Test in Odoo

```python
# Test webhook execution
webhook = env['cx.tower.webhook'].browse(webhook_id)

result = webhook.execute(
    payload={'app': 'test', 'version': '1.0.0'},
    headers={'X-Test': 'true'}
)

print(result)
# {'exit_code': 0, 'message': 'Success'}
```

### Test with curl

```bash
# POST JSON
curl -v -X POST https://tower.example.com/cetmix_tower_webhooks/test \
  -H "Authorization: Bearer TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"test": true}'

# POST form data
curl -v -X POST https://tower.example.com/cetmix_tower_webhooks/test \
  -H "Authorization: Bearer TOKEN" \
  -d "field1=value1" \
  -d "field2=value2"

# GET
curl -v "https://tower.example.com/cetmix_tower_webhooks/test?param=value" \
  -H "Authorization: Bearer TOKEN"
```

### Test with Python

```python
import requests

response = requests.post(
    'https://tower.example.com/cetmix_tower_webhooks/deploy',
    json={'app': 'myapp', 'version': '1.0.0'},
    headers={'Authorization': 'Bearer YOUR_TOKEN'}
)

print(f"Status: {response.status_code}")
print(f"Response: {response.json()}")
```

## Best Practices

1. **Use descriptive endpoint names** - `deploy/production` not `webhook1`
2. **Always set result** - Provide meaningful exit codes and messages
3. **Validate inputs** - Check payload before executing operations
4. **Handle errors gracefully** - Use try-except blocks
5. **Log important operations** - Use custom_values or logging
6. **Use HMAC for external services** - More secure than bearer tokens
7. **Set appropriate timeouts** - For commands that might run long
8. **Document your webhooks** - Include usage examples
9. **Test thoroughly** - Test with actual payload formats
10. **Monitor webhook logs** - Check for failures and abuse

## Related Documentation

- [Odoo ORM API](01-odoo-orm-api.md)
- [Python Command API](02-python-command-api.md)
- [Command Management](../04-feature-guides/04-command-management.md)
- [Flight Plans](../04-feature-guides/05-flight-plans.md)

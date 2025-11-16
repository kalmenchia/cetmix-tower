---
title: API References
description: Complete API documentation for Cetmix Tower
category: api-references
order: 1
---

# API References

This section provides comprehensive API documentation for integrating with and extending Cetmix Tower.

## Overview

Cetmix Tower provides multiple APIs for different use cases:

- **Odoo ORM/RPC API**: Access Tower functionality programmatically through Odoo's standard ORM and RPC mechanisms
- **Python Command API**: Execute Python code within Tower commands with access to special objects and libraries
- **Webhook API**: Trigger Tower operations via HTTP webhooks

## Available APIs

### 1. Odoo ORM/RPC API

Access and control Tower resources using Odoo's ORM from:
- Python code within Odoo
- External applications using XML-RPC or JSON-RPC
- Odoo shell for administrative tasks

**Key Models**:
- `cx.tower.server` - Server management
- `cx.tower.command` - Command operations
- `cx.tower.plan` - Flight plan execution
- `cx.tower.file` - File management
- `cx.tower.variable` - Variable management
- `cx.tower.key` - Secret/key management

[View ORM API Documentation →](01-odoo-orm-api.md)

### 2. Python Command API

Execute Python code within Tower commands with access to:
- Odoo environment (`env`)
- Current user (`user`)
- Current server (`server`)
- Tower helper class (`tower`)
- Python libraries (requests, json, hashlib, etc.)

[View Python Command API Documentation →](02-python-command-api.md)

### 3. Webhook API

Trigger Tower commands and flight plans via HTTP webhooks:
- POST/GET endpoints
- JSON or form-urlencoded payloads
- Authentication via Bearer tokens, HMAC, or API keys
- Custom Python code execution

[View Webhook API Documentation →](03-webhook-api.md)

## Authentication

All API access requires proper authentication:

### Internal/ORM Access
- Odoo session authentication
- Access rights based on Tower security groups
- Record-level access rules apply

### External/RPC Access
- Database name
- Username and password
- Or API keys (if configured)

### Webhook Access
- Bearer token
- HMAC signature
- API key
- Custom authentication

## Common Use Cases

### Remote Server Management
```python
# Via XML-RPC
import xmlrpc.client

url = 'https://your-odoo.com'
db = 'your-database'
username = 'admin'
password = 'admin'

common = xmlrpc.client.ServerProxy(f'{url}/xmlrpc/2/common')
uid = common.authenticate(db, username, password, {})

models = xmlrpc.client.ServerProxy(f'{url}/xmlrpc/2/object')

# Get servers
server_ids = models.execute_kw(db, uid, password,
    'cx.tower.server', 'search',
    [[['active', '=', True]]])

# Run command
models.execute_kw(db, uid, password,
    'cx.tower.server', 'run_command',
    [server_id, command_id])
```

### Automated Deployments
```python
# Via Odoo shell
server = env['cx.tower.server'].browse(server_id)
deploy_plan = env['cx.tower.plan'].search([('reference', '=', 'deploy_production')])
server.run_flight_plan(deploy_plan)
```

### Webhook Triggers
```bash
# Via curl
curl -X POST https://your-odoo.com/cetmix_tower_webhooks/deploy \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"branch": "main", "environment": "production"}'
```

## Error Handling

All APIs return consistent error information:

### ORM/RPC
- Standard Odoo exceptions
- `UserError` for user-facing errors
- `ValidationError` for validation failures
- `AccessError` for permission issues

### Python Commands
Return dict with:
```python
{
    'status': 0,  # 0 = success, non-zero = error
    'response': 'Success message',
    'error': None  # Error message if status != 0
}
```

### Webhooks
Return JSON with:
```json
{
    "exit_code": 0,
    "message": "Success message"
}
```

## Rate Limiting

Be mindful of:
- Odoo's configured worker limits
- Database connection pool
- Long-running operations should use job queues

## Best Practices

1. **Use appropriate API** - Choose the right API for your use case
2. **Handle errors gracefully** - Always check return codes and handle exceptions
3. **Use job queues** - For long-running operations, use `cetmix_tower_server_queue` module
4. **Secure credentials** - Never hardcode passwords or API keys
5. **Test thoroughly** - Test API calls in non-production environment first
6. **Monitor logs** - Check Tower command logs for debugging

## API Versioning

Tower API follows Odoo module versioning:
- Major version changes may break compatibility
- Minor version changes maintain backward compatibility
- Check `__manifest__.py` for current version

## Getting Help

- [User Guides](../03-user-guides/README.md) - End-user documentation
- [Feature Guides](../04-feature-guides/README.md) - Feature-specific guides
- [Technical Guides](../07-technical-guides/README.md) - Advanced technical documentation
- [GitHub Issues](https://github.com/cetmix/cetmix-tower/issues) - Report bugs or request features

## Next Steps

- [Odoo ORM API Documentation](01-odoo-orm-api.md)
- [Python Command API Documentation](02-python-command-api.md)
- [Webhook API Documentation](03-webhook-api.md)

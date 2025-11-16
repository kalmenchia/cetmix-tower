---
title: Creating Custom Commands
description: Guide to creating custom command types and extending command functionality
author: Cetmix
date: 2025-11-16
version: 1.0
module: cetmix_tower_server
category: customization
tags: [custom-commands, python-libraries, command-runners, integrations]
---

# Creating Custom Commands

This guide covers how to extend Cetmix Tower's command system by adding custom command types, Python libraries, and command runners through inheritance.

## Table of Contents

- [Overview](#overview)
- [Adding Python Libraries](#adding-python-libraries)
- [Adding Odoo Objects](#adding-odoo-objects)
- [Creating Custom Command Types](#creating-custom-command-types)
- [Custom Command Runners](#custom-command-runners)
- [Real-World Examples](#real-world-examples)
- [Best Practices](#best-practices)

## Overview

Tower's command system allows extensive customization:

- **Add Python Libraries**: Make external libraries available in Python commands
- **Add Odoo Objects**: Expose custom Odoo models/methods to commands
- **Custom Command Types**: Create entirely new command action types
- **Custom Runners**: Implement custom execution logic for commands

### Command Action Types

Tower includes these built-in action types:

- `ssh_command` - Run shell commands via SSH
- `python_code` - Execute Python code
- `file_using_template` - Create files from templates
- `plan` - Execute flight plans

You can add your own custom action types!

## Adding Python Libraries

### Basic Python Library Addition

Use `_custom_python_libraries()` to add Python libraries to command context:

```python
# custom_tower_extension/models/cx_tower_command.py
from odoo import _, models
from odoo.tools.safe_eval import wrap_module

# Wrap your library safely
requests = wrap_module(__import__("requests"), ["get", "post", "put", "delete"])
yaml = wrap_module(__import__("yaml"), ["safe_load", "safe_dump"])

class CxTowerCommand(models.Model):
    _inherit = "cx.tower.command"

    def _custom_python_libraries(self):
        """Add custom Python libraries"""
        libraries = super()._custom_python_libraries()

        libraries.update({
            'custom_tower_extension': {
                'requests': {
                    'import': requests,
                    'help': _(
                        "Python 'requests' library for HTTP calls. "
                        "Methods: get, post, put, delete"
                    ),
                },
                'yaml': {
                    'import': yaml,
                    'help': _(
                        "Python 'yaml' library for YAML processing. "
                        "Methods: safe_load, safe_dump"
                    ),
                },
            }
        })

        return libraries
```

### AWS Integration Example

Real example from `cetmix_tower_aws`:

```python
# cetmix_tower_aws/models/cx_tower_command.py
from odoo import _, models
from odoo.tools.safe_eval import wrap_module

# Wrap boto3 safely
boto3 = wrap_module(__import__("boto3"), ["client", "resource", "Session"])

class CxTowerCommand(models.Model):
    _inherit = "cx.tower.command"

    def _custom_python_libraries(self):
        """Add boto3 library for AWS integration"""
        libraries = super()._custom_python_libraries()

        libraries.update({
            'cetmix_tower_aws': {
                'boto3': {
                    'import': boto3,
                    'help': _(
                        "Python 'boto3' library for AWS services. "
                        "Available methods: 'client', 'resource', 'Session'<br/>"
                        "Supports AWS services like EC2, S3, RDS, Lambda, "
                        "CloudWatch, etc.<br/>"
                        "Please check the <a "
                        "href='https://boto3.amazonaws.com/v1/documentation/api/latest/index.html'"
                        " target='_blank'>Boto3 Documentation</a> for detailed "
                        "information about services and methods."
                    ),
                },
            }
        })

        return libraries
```

### Usage in Python Commands

Once added, libraries are available in Python command context:

```python
# Example Python command code using custom libraries

# Use requests library
response = requests.get('https://api.example.com/status')
if response.status_code == 200:
    data = response.json()
    result = {
        'exit_code': 0,
        'message': f"API Status: {data.get('status')}"
    }

# Use YAML library
config = yaml.safe_load("""
    server:
      host: localhost
      port: 8069
""")

# Use boto3 for AWS
ec2 = boto3.client('ec2', region_name='us-east-1')
instances = ec2.describe_instances()
```

### Library Security

**IMPORTANT**: Always use `wrap_module` to safely wrap external libraries:

```python
# ✅ CORRECT: Safe wrapping
boto3 = wrap_module(__import__("boto3"), ["client", "resource", "Session"])

# ❌ WRONG: Direct import (security risk)
import boto3  # Don't do this!
```

`wrap_module` ensures only specified methods are accessible, preventing malicious code execution.

### Advanced Library Examples

#### Custom API Client

```python
# custom_tower_extension/models/cx_tower_command.py
from odoo import _, models
from odoo.tools.safe_eval import wrap_module

# Custom API client
class CustomAPIClient:
    """Custom API client for integration"""

    def __init__(self, api_key=None):
        self.api_key = api_key
        self.base_url = "https://api.example.com"

    def get_status(self):
        """Get API status"""
        # Implementation
        return {"status": "ok"}

    def create_resource(self, data):
        """Create a resource"""
        # Implementation
        return {"id": "12345"}

# Wrap the client
custom_api = wrap_module(
    CustomAPIClient,
    ["get_status", "create_resource"]
)

class CxTowerCommand(models.Model):
    _inherit = "cx.tower.command"

    def _custom_python_libraries(self):
        libraries = super()._custom_python_libraries()

        libraries.update({
            'custom_tower_extension': {
                'custom_api': {
                    'import': CustomAPIClient,
                    'help': _("Custom API client for integration"),
                },
            }
        })

        return libraries
```

## Adding Odoo Objects

### Custom Odoo Models in Commands

Expose custom Odoo models to command context:

```python
# custom_tower_extension/models/cx_tower_command.py
from odoo import _, models

class CxTowerCommand(models.Model):
    _inherit = "cx.tower.command"

    def _get_python_command_odoo_objects(self, server=None):
        """Add custom Odoo objects to command context"""
        objects = super()._get_python_command_odoo_objects(server=server)

        # Add custom models
        objects.update({
            'CustomDatacenter': {
                'import': self.env['custom.datacenter'],
                'help': _("Custom datacenter model"),
            },
            'CustomBackup': {
                'import': self.env['custom.server.backup'],
                'help': _("Server backup model"),
            },
        })

        return objects
```

### Usage in Commands

```python
# Python command code using custom Odoo objects

# Search for datacenter
datacenter = CustomDatacenter.search([('name', '=', 'US-East-1')], limit=1)

# Create backup record
backup = CustomBackup.create({
    'name': 'Backup ' + tower.tools.now,
    'server_id': server.id,
    'backup_type': 'full',
    'status': 'completed',
})

result = {
    'exit_code': 0,
    'message': f'Backup created: {backup.name}'
}
```

### Helper Methods

Expose helper methods to commands:

```python
class CxTowerCommand(models.Model):
    _inherit = "cx.tower.command"

    def _get_python_command_odoo_objects(self, server=None):
        objects = super()._get_python_command_odoo_objects(server=server)

        # Add helper class
        objects.update({
            'monitoring_helper': {
                'import': self.env['custom.monitoring.helper'],
                'help': _("Monitoring helper methods"),
            },
        })

        return objects
```

```python
# custom_tower_extension/models/custom_monitoring_helper.py
from odoo import models

class CustomMonitoringHelper(models.AbstractModel):
    _name = 'custom.monitoring.helper'
    _description = 'Monitoring Helper'

    def check_server_health(self, server):
        """Check server health"""
        # Implementation
        return {
            'cpu': 45.2,
            'memory': 62.8,
            'disk': 78.3,
        }

    def send_alert(self, server, message):
        """Send monitoring alert"""
        # Implementation
        return True
```

## Creating Custom Command Types

### Adding a New Command Action

Add a new action type to the selection:

```python
# custom_tower_extension/models/cx_tower_command.py
from odoo import models

class CxTowerCommand(models.Model):
    _inherit = "cx.tower.command"

    def _selection_action(self):
        """Add custom action type"""
        actions = super()._selection_action()

        # Add your custom action
        actions.append(('http_request', 'HTTP Request'))
        actions.append(('database_query', 'Database Query'))

        return actions
```

### Custom Action Fields

Add fields specific to your action type:

```python
from odoo import api, fields, models

class CxTowerCommand(models.Model):
    _inherit = "cx.tower.command"

    # HTTP Request fields
    http_url = fields.Char(
        string="URL",
        help="HTTP request URL",
    )

    http_method = fields.Selection(
        selection=[
            ('GET', 'GET'),
            ('POST', 'POST'),
            ('PUT', 'PUT'),
            ('DELETE', 'DELETE'),
        ],
        string="HTTP Method",
        default='GET',
    )

    http_headers = fields.Text(
        string="Headers",
        help="HTTP headers (JSON format)",
    )

    http_body = fields.Text(
        string="Request Body",
        help="HTTP request body",
    )
```

### Action Visibility

Control field visibility based on action type:

```xml
<!-- custom_tower_extension/views/cx_tower_command_views.xml -->
<odoo>
    <record id="view_cx_tower_command_form_custom" model="ir.ui.view">
        <field name="name">cx.tower.command.form.custom</field>
        <field name="model">cx.tower.command</field>
        <field name="inherit_id" ref="cetmix_tower_server.view_cx_tower_command_form"/>
        <field name="arch" type="xml">

            <!-- Add custom fields -->
            <xpath expr="//field[@name='code']" position="after">
                <group attrs="{'invisible': [('action', '!=', 'http_request')]}">
                    <field name="http_url"/>
                    <field name="http_method"/>
                    <field name="http_headers"/>
                    <field name="http_body"/>
                </group>
            </xpath>

        </field>
    </record>
</odoo>
```

## Custom Command Runners

### Implementing a Command Runner

Create a runner for your custom action type:

```python
# custom_tower_extension/models/cx_tower_server.py
from odoo import _, models, fields
from odoo.exceptions import ValidationError
import json
import requests

class CxTowerServer(models.Model):
    _inherit = "cx.tower.server"

    def _command_runner(self, command, log_record, rendered_command_code,
                       sudo=None, rendered_command_path=None,
                       ssh_connection=None, **kwargs):
        """Add custom command runner"""

        # Handle HTTP request action
        if command.action == 'http_request':
            return self._command_runner_http_request(
                command, log_record, **kwargs
            )

        # Handle database query action
        if command.action == 'database_query':
            return self._command_runner_database_query(
                command, log_record, rendered_command_code, **kwargs
            )

        # Call parent for other actions
        return super()._command_runner(
            command, log_record, rendered_command_code,
            sudo, rendered_command_path, ssh_connection, **kwargs
        )

    def _command_runner_http_request(self, command, log_record, **kwargs):
        """Run HTTP request command"""
        self.ensure_one()

        try:
            # Parse headers
            headers = {}
            if command.http_headers:
                headers = json.loads(command.http_headers)

            # Make HTTP request
            method = command.http_method.lower()
            request_func = getattr(requests, method)

            response = request_func(
                command.http_url,
                headers=headers,
                data=command.http_body,
                timeout=30
            )

            # Prepare result
            status = 0 if response.status_code < 400 else 1
            response_text = response.text[:1000]  # Limit size

            result = {
                'status': status,
                'response': f"HTTP {response.status_code}: {response_text}",
                'error': None if status == 0 else f"HTTP Error {response.status_code}",
            }

        except Exception as e:
            result = {
                'status': 1,
                'response': None,
                'error': str(e),
            }

        # Log result
        if log_record:
            log_record.finish(
                finish_date=fields.Datetime.now(),
                status=result['status'],
                response=result['response'],
                error=result['error'],
            )
        else:
            return result

    def _command_runner_database_query(self, command, log_record, query, **kwargs):
        """Run database query command (read-only)"""
        self.ensure_one()

        try:
            # Execute query (read-only)
            self.env.cr.execute(query)
            results = self.env.cr.fetchall()

            # Format results
            response = json.dumps(results, indent=2)

            result = {
                'status': 0,
                'response': response,
                'error': None,
            }

        except Exception as e:
            result = {
                'status': 1,
                'response': None,
                'error': str(e),
            }

        # Log result
        if log_record:
            log_record.finish(
                finish_date=fields.Datetime.now(),
                status=result['status'],
                response=result['response'],
                error=result['error'],
            )
        else:
            return result
```

### Asynchronous Command Execution

For long-running commands, use queue jobs:

```python
# custom_tower_extension/models/cx_tower_server.py
from odoo import models

class CxTowerServer(models.Model):
    _inherit = "cx.tower.server"

    def _command_runner_wrapper(self, command, log_record, rendered_command_code,
                                sudo=None, rendered_command_path=None,
                                ssh_connection=None, **kwargs):
        """Wrapper for custom async execution"""

        # Run custom actions asynchronously
        if command.action in ['http_request', 'long_running_task']:
            # Use queue_job if available
            if hasattr(self, 'with_delay'):
                self.with_delay()._command_runner(
                    command, log_record, rendered_command_code,
                    sudo, rendered_command_path, ssh_connection, **kwargs
                )
                return

        # Call parent for other actions
        return super()._command_runner_wrapper(
            command, log_record, rendered_command_code,
            sudo, rendered_command_path, ssh_connection, **kwargs
        )
```

## Real-World Examples

### Example 1: Kubernetes Integration

```python
# tower_kubernetes/models/cx_tower_command.py
from odoo import _, models
from odoo.tools.safe_eval import wrap_module
from kubernetes import client

# Wrap Kubernetes client
k8s_client = wrap_module(client, ["CoreV1Api", "AppsV1Api"])

class CxTowerCommand(models.Model):
    _inherit = "cx.tower.command"

    def _custom_python_libraries(self):
        libraries = super()._custom_python_libraries()

        libraries.update({
            'tower_kubernetes': {
                'k8s_client': {
                    'import': k8s_client,
                    'help': _(
                        "Kubernetes Python client. "
                        "Access Kubernetes API from commands."
                    ),
                },
            }
        })

        return libraries

    def _selection_action(self):
        actions = super()._selection_action()
        actions.append(('k8s_apply', 'Apply Kubernetes Manifest'))
        return actions
```

```python
# tower_kubernetes/models/cx_tower_server.py
from odoo import _, models, fields
from kubernetes import client, config
import yaml

class CxTowerServer(models.Model):
    _inherit = "cx.tower.server"

    k8s_config = fields.Text(
        string="Kubernetes Config",
        help="Kubernetes configuration (kubeconfig)",
    )

    def _command_runner(self, command, log_record, rendered_command_code,
                       sudo=None, rendered_command_path=None,
                       ssh_connection=None, **kwargs):
        """Add Kubernetes command runner"""

        if command.action == 'k8s_apply':
            return self._command_runner_k8s_apply(
                log_record, rendered_command_code, **kwargs
            )

        return super()._command_runner(
            command, log_record, rendered_command_code,
            sudo, rendered_command_path, ssh_connection, **kwargs
        )

    def _command_runner_k8s_apply(self, log_record, manifest_code, **kwargs):
        """Apply Kubernetes manifest"""
        self.ensure_one()

        try:
            # Load kubeconfig
            config.load_kube_config_from_dict(
                yaml.safe_load(self.k8s_config)
            )

            # Parse manifest
            manifest = yaml.safe_load(manifest_code)

            # Apply manifest
            api = client.AppsV1Api()
            if manifest['kind'] == 'Deployment':
                api.create_namespaced_deployment(
                    namespace=manifest['metadata']['namespace'],
                    body=manifest
                )

            result = {
                'status': 0,
                'response': f"Applied {manifest['kind']}: {manifest['metadata']['name']}",
                'error': None,
            }

        except Exception as e:
            result = {
                'status': 1,
                'response': None,
                'error': str(e),
            }

        if log_record:
            log_record.finish(
                finish_date=fields.Datetime.now(),
                status=result['status'],
                response=result['response'],
                error=result['error'],
            )
        else:
            return result
```

### Example 2: Monitoring Integration

```python
# tower_monitoring/models/cx_tower_command.py
from odoo import _, models
from odoo.tools.safe_eval import wrap_module
import prometheus_client

# Wrap Prometheus client
prometheus = wrap_module(
    prometheus_client,
    ["Counter", "Gauge", "Histogram", "Summary"]
)

class CxTowerCommand(models.Model):
    _inherit = "cx.tower.command"

    def _custom_python_libraries(self):
        libraries = super()._custom_python_libraries()

        libraries.update({
            'tower_monitoring': {
                'prometheus': {
                    'import': prometheus,
                    'help': _(
                        "Prometheus metrics client. "
                        "Create and update Prometheus metrics."
                    ),
                },
            }
        })

        return libraries

    def _get_python_command_odoo_objects(self, server=None):
        objects = super()._get_python_command_odoo_objects(server=server)

        objects.update({
            'monitoring': {
                'import': self.env['custom.monitoring.helper'],
                'help': _("Monitoring helper methods"),
            },
        })

        return objects
```

## Best Practices

### ✅ DO:

1. **Always wrap external libraries safely**
   ```python
   lib = wrap_module(__import__("library"), ["method1", "method2"])
   ```

2. **Provide clear help text**
   ```python
   'help': _("Clear description with usage examples")
   ```

3. **Handle errors gracefully**
   ```python
   try:
       # Your code
       result = {'status': 0, 'response': 'Success', 'error': None}
   except Exception as e:
       result = {'status': 1, 'response': None, 'error': str(e)}
   ```

4. **Use module namespacing**
   ```python
   libraries.update({
       'your_module_name': {  # Module namespace
           'library_name': {...},
       }
   })
   ```

5. **Document return formats**
   ```python
   result = {
       'status': 0,       # 0 = success, non-zero = error
       'response': '...',  # Command output
       'error': None,     # Error message if any
   }
   ```

### ❌ DON'T:

1. **Don't expose unsafe methods**
2. **Don't hardcode credentials**
3. **Don't skip error handling**
4. **Don't expose entire modules unwrapped**
5. **Don't forget to call super()**

### Security Checklist

- ✅ Use `wrap_module` for all external libraries
- ✅ Validate all inputs
- ✅ Limit exposed methods to necessary ones only
- ✅ Use appropriate access controls
- ✅ Sanitize output before logging
- ✅ Handle sensitive data appropriately

## Next Steps

- Review [Extending Mixins](04-extending-mixins.md)
- Check [Deployment Guide](../09-deployment-operations/01-deployment-guide.md)
- Explore [Version Upgrades](../13-potential-upgrade/01-odoo-version-upgrades.md)

---

**Remember**: Always inherit, never modify. Use `wrap_module` for security!

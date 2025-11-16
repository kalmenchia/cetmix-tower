---
title: Inheritance Approach for Tower Customization
description: Comprehensive guide to creating custom modules using inheritance
author: Cetmix
date: 2025-11-16
version: 1.0
module: cetmix_tower_server
category: customization
tags: [inheritance, custom-modules, best-practices, odoo-17]
---

# Inheritance Approach for Tower Customization

## Core Principle

**NEVER modify the original Cetmix Tower module code. ALWAYS use inheritance.**

This guide shows you how to create custom modules that extend Tower functionality through inheritance, ensuring your customizations survive upgrades and remain maintainable.

## Table of Contents

- [Why Inheritance?](#why-inheritance)
- [Creating a Custom Module](#creating-a-custom-module)
- [Module Structure](#module-structure)
- [Inheriting Models](#inheriting-models)
- [Inheriting Views](#inheriting-views)
- [Adding New Models](#adding-new-models)
- [Version Compatibility](#version-compatibility)
- [Complete Examples](#complete-examples)

## Why Inheritance?

### The Problem with Direct Modification

❌ **Wrong Approach**:
```bash
# DON'T DO THIS!
cd cetmix_tower_server/models/
nano cx_tower_server.py  # Editing original file
# Add your custom field here...
```

**Issues**:
- Lost on module update
- No upgrade path
- Breaks other customizations
- Impossible to disable
- Violates Odoo best practices

### The Solution: Inheritance

✅ **Correct Approach**:
```python
# custom_tower_extension/models/cx_tower_server.py
class CxTowerServer(models.Model):
    _inherit = "cx.tower.server"

    # Your customizations here
    custom_field = fields.Char("Custom Field")
```

**Benefits**:
- ✅ Survives module updates
- ✅ Can be enabled/disabled
- ✅ Modular and organized
- ✅ Follows Odoo standards
- ✅ Easy to maintain

## Creating a Custom Module

### Step 1: Module Naming Convention

Choose a clear, descriptive name:

- `custom_tower_[feature]` - For specific features
- `[company]_tower_[feature]` - For company-specific extensions
- `tower_[integration]` - For third-party integrations

Examples:
- `custom_tower_monitoring` - Custom monitoring features
- `acme_tower_extension` - Acme Corp's Tower customizations
- `tower_zabbix` - Zabbix integration

### Step 2: Create Module Directory

```bash
cd /path/to/odoo/addons
mkdir custom_tower_extension
cd custom_tower_extension
```

### Step 3: Module Structure

Create the following structure:

```
custom_tower_extension/
├── __init__.py
├── __manifest__.py
├── models/
│   ├── __init__.py
│   ├── cx_tower_server.py
│   ├── cx_tower_command.py
│   └── cx_tower_plan.py
├── views/
│   ├── cx_tower_server_views.xml
│   ├── cx_tower_command_views.xml
│   └── menus.xml
├── security/
│   ├── ir.model.access.csv
│   └── security.xml
├── data/
│   └── initial_data.xml
├── static/
│   └── description/
│       ├── icon.png
│       └── index.html
└── README.md
```

## Module Structure

### `__manifest__.py`

```python
# custom_tower_extension/__manifest__.py
{
    'name': 'Custom Tower Extension',
    'version': '17.0.1.0.0',
    'category': 'Tools',
    'summary': 'Custom extensions for Cetmix Tower',
    'description': """
        Custom Tower Extension
        ======================

        This module extends Cetmix Tower with custom functionality:
        * Custom fields for servers
        * Custom command types
        * Additional monitoring features
    """,
    'author': 'Your Company',
    'website': 'https://www.yourcompany.com',
    'license': 'AGPL-3',
    'depends': [
        'cetmix_tower_server',  # Core Tower module
        # Add other Tower modules as needed:
        # 'cetmix_tower_yaml',
        # 'cetmix_tower_git',
        # 'cetmix_tower_webhook',
    ],
    'data': [
        'security/ir.model.access.csv',
        'views/cx_tower_server_views.xml',
        'views/cx_tower_command_views.xml',
        'data/initial_data.xml',
    ],
    'installable': True,
    'application': False,
    'auto_install': False,
}
```

### Root `__init__.py`

```python
# custom_tower_extension/__init__.py
from . import models
```

### Models `__init__.py`

```python
# custom_tower_extension/models/__init__.py
from . import cx_tower_server
from . import cx_tower_command
from . import cx_tower_plan
```

## Inheriting Models

### Basic Inheritance Pattern

Use `_inherit` to extend existing models:

```python
# custom_tower_extension/models/cx_tower_server.py
from odoo import api, fields, models

class CxTowerServer(models.Model):
    _inherit = "cx.tower.server"

    # Add your customizations here
```

### Adding Custom Fields

#### Simple Fields

```python
from odoo import fields, models

class CxTowerServer(models.Model):
    _inherit = "cx.tower.server"

    # Text fields
    custom_notes = fields.Text(
        string="Custom Notes",
        help="Additional notes for this server"
    )

    # Char fields
    monitoring_id = fields.Char(
        string="Monitoring ID",
        help="External monitoring system ID"
    )

    # Selection fields
    environment_type = fields.Selection(
        selection=[
            ('dev', 'Development'),
            ('staging', 'Staging'),
            ('prod', 'Production'),
        ],
        string="Environment",
        default='dev',
        help="Server environment type"
    )

    # Boolean fields
    is_critical = fields.Boolean(
        string="Critical Server",
        default=False,
        help="Mark if this is a critical production server"
    )

    # Integer/Float fields
    cpu_cores = fields.Integer(
        string="CPU Cores",
        help="Number of CPU cores"
    )

    memory_gb = fields.Float(
        string="Memory (GB)",
        digits=(10, 2),
        help="Total memory in GB"
    )

    # Date/Datetime fields
    maintenance_date = fields.Date(
        string="Next Maintenance",
        help="Scheduled maintenance date"
    )

    last_health_check = fields.Datetime(
        string="Last Health Check",
        help="Last time health check was performed"
    )
```

#### Relational Fields

```python
from odoo import fields, models

class CxTowerServer(models.Model):
    _inherit = "cx.tower.server"

    # Many2one (belongs to one)
    datacenter_id = fields.Many2one(
        comodel_name='custom.datacenter',
        string="Data Center",
        ondelete='restrict',
        help="Physical datacenter location"
    )

    # One2many (has many)
    backup_ids = fields.One2many(
        comodel_name='custom.server.backup',
        inverse_name='server_id',
        string="Backups",
        help="Server backup records"
    )

    # Many2many (has many, belongs to many)
    application_ids = fields.Many2many(
        comodel_name='custom.application',
        relation='custom_server_application_rel',
        column1='server_id',
        column2='application_id',
        string="Applications",
        help="Applications running on this server"
    )
```

#### Computed Fields

```python
from odoo import api, fields, models

class CxTowerServer(models.Model):
    _inherit = "cx.tower.server"

    backup_count = fields.Integer(
        string="Backup Count",
        compute='_compute_backup_count',
        store=True,
        help="Total number of backups"
    )

    health_status = fields.Selection(
        selection=[
            ('healthy', 'Healthy'),
            ('warning', 'Warning'),
            ('critical', 'Critical'),
        ],
        string="Health Status",
        compute='_compute_health_status',
        help="Overall server health"
    )

    @api.depends('backup_ids')
    def _compute_backup_count(self):
        for server in self:
            server.backup_count = len(server.backup_ids)

    @api.depends('last_health_check', 'cpu_cores', 'memory_gb')
    def _compute_health_status(self):
        from datetime import datetime, timedelta

        for server in self:
            # Example health logic
            if not server.last_health_check:
                server.health_status = 'critical'
            elif server.last_health_check < datetime.now() - timedelta(days=1):
                server.health_status = 'warning'
            else:
                server.health_status = 'healthy'
```

### Adding Custom Methods

```python
from odoo import _, api, models
from odoo.exceptions import UserError

class CxTowerServer(models.Model):
    _inherit = "cx.tower.server"

    def action_run_health_check(self):
        """Run health check on server"""
        self.ensure_one()

        # Your custom health check logic
        result = self._perform_health_check()

        # Update health check timestamp
        self.write({
            'last_health_check': fields.Datetime.now(),
        })

        return {
            'type': 'ir.actions.client',
            'tag': 'display_notification',
            'params': {
                'type': 'success',
                'title': _('Health Check Complete'),
                'message': _('Server health check completed successfully.'),
                'sticky': False,
            },
        }

    def _perform_health_check(self):
        """Internal method for health check logic"""
        self.ensure_one()
        # Your implementation here
        return True
```

### Overriding Existing Methods

**IMPORTANT**: When overriding methods, always call `super()`:

```python
from odoo import models

class CxTowerServer(models.Model):
    _inherit = "cx.tower.server"

    def run_command(self, command, path=None, sudo=None, ssh_connection=None, **kwargs):
        """Override run_command to add custom logging"""

        # Custom logic BEFORE original method
        self._log_command_execution(command)

        # Call original method
        result = super().run_command(
            command=command,
            path=path,
            sudo=sudo,
            ssh_connection=ssh_connection,
            **kwargs
        )

        # Custom logic AFTER original method
        self._update_command_statistics(command, result)

        return result

    def _log_command_execution(self, command):
        """Log command execution to custom table"""
        # Your logging logic
        pass

    def _update_command_statistics(self, command, result):
        """Update command statistics"""
        # Your statistics logic
        pass
```

## Inheriting Views

### Basic XPath Pattern

Use XPath to extend existing views without copying them:

```xml
<!-- custom_tower_extension/views/cx_tower_server_views.xml -->
<odoo>
    <record id="view_cx_tower_server_form_custom" model="ir.ui.view">
        <field name="name">cx.tower.server.form.custom</field>
        <field name="model">cx.tower.server</field>
        <field name="inherit_id" ref="cetmix_tower_server.view_cx_tower_server_form"/>
        <field name="arch" type="xml">
            <!-- Add field after existing field -->
            <xpath expr="//field[@name='note']" position="after">
                <field name="custom_notes"/>
                <field name="environment_type"/>
            </xpath>
        </field>
    </record>
</odoo>
```

### XPath Positions

```xml
<odoo>
    <record id="view_cx_tower_server_form_custom" model="ir.ui.view">
        <field name="name">cx.tower.server.form.custom</field>
        <field name="model">cx.tower.server</field>
        <field name="inherit_id" ref="cetmix_tower_server.view_cx_tower_server_form"/>
        <field name="arch" type="xml">

            <!-- Add AFTER an element -->
            <xpath expr="//field[@name='partner_id']" position="after">
                <field name="datacenter_id"/>
            </xpath>

            <!-- Add BEFORE an element -->
            <xpath expr="//field[@name='note']" position="before">
                <separator string="Custom Information"/>
            </xpath>

            <!-- Add INSIDE an element (at the end) -->
            <xpath expr="//group[@name='connection']" position="inside">
                <field name="monitoring_id"/>
            </xpath>

            <!-- REPLACE an element -->
            <xpath expr="//field[@name='color']" position="replace">
                <field name="color" widget="color_picker"/>
            </xpath>

            <!-- Set ATTRIBUTES on an element -->
            <xpath expr="//field[@name='url']" position="attributes">
                <attribute name="placeholder">https://example.com</attribute>
            </xpath>

        </field>
    </record>
</odoo>
```

### Adding a New Page/Group

```xml
<odoo>
    <record id="view_cx_tower_server_form_custom" model="ir.ui.view">
        <field name="name">cx.tower.server.form.custom</field>
        <field name="model">cx.tower.server</field>
        <field name="inherit_id" ref="cetmix_tower_server.view_cx_tower_server_form"/>
        <field name="arch" type="xml">

            <!-- Add a new page in notebook -->
            <xpath expr="//notebook" position="inside">
                <page string="Monitoring" name="monitoring">
                    <group>
                        <group>
                            <field name="is_critical"/>
                            <field name="last_health_check"/>
                            <field name="health_status"/>
                        </group>
                        <group>
                            <field name="cpu_cores"/>
                            <field name="memory_gb"/>
                            <field name="maintenance_date"/>
                        </group>
                    </group>
                    <separator string="Health Checks"/>
                    <button name="action_run_health_check"
                            type="object"
                            string="Run Health Check"
                            class="btn-primary"/>
                </page>
            </xpath>

        </field>
    </record>
</odoo>
```

### Extending Tree/List Views

```xml
<odoo>
    <record id="view_cx_tower_server_tree_custom" model="ir.ui.view">
        <field name="name">cx.tower.server.tree.custom</field>
        <field name="model">cx.tower.server</field>
        <field name="inherit_id" ref="cetmix_tower_server.view_cx_tower_server_tree"/>
        <field name="arch" type="xml">

            <!-- Add column after existing column -->
            <xpath expr="//field[@name='partner_id']" position="after">
                <field name="environment_type"/>
                <field name="health_status" widget="badge"
                       decoration-success="health_status == 'healthy'"
                       decoration-warning="health_status == 'warning'"
                       decoration-danger="health_status == 'critical'"/>
            </xpath>

        </field>
    </record>
</odoo>
```

### Extending Search Views

```xml
<odoo>
    <record id="view_cx_tower_server_search_custom" model="ir.ui.view">
        <field name="name">cx.tower.server.search.custom</field>
        <field name="model">cx.tower.server</field>
        <field name="inherit_id" ref="cetmix_tower_server.view_cx_tower_server_search"/>
        <field name="arch" type="xml">

            <!-- Add search fields -->
            <xpath expr="//field[@name='name']" position="after">
                <field name="monitoring_id"/>
                <field name="environment_type"/>
            </xpath>

            <!-- Add filters -->
            <xpath expr="//filter[@name='inactive']" position="after">
                <separator/>
                <filter name="critical"
                        string="Critical Servers"
                        domain="[('is_critical', '=', True)]"/>
                <filter name="production"
                        string="Production"
                        domain="[('environment_type', '=', 'prod')]"/>
                <separator/>
                <filter name="unhealthy"
                        string="Unhealthy"
                        domain="[('health_status', '!=', 'healthy')]"/>
            </xpath>

            <!-- Add group by -->
            <xpath expr="//group" position="inside">
                <filter name="group_environment"
                        string="Environment"
                        context="{'group_by': 'environment_type'}"/>
                <filter name="group_datacenter"
                        string="Data Center"
                        context="{'group_by': 'datacenter_id'}"/>
            </xpath>

        </field>
    </record>
</odoo>
```

## Adding New Models

### Creating Complementary Models

Create new models that reference Tower models:

```python
# custom_tower_extension/models/custom_datacenter.py
from odoo import fields, models

class CustomDatacenter(models.Model):
    _name = "custom.datacenter"
    _description = "Data Center"
    _order = "name"

    name = fields.Char(
        string="Name",
        required=True,
        help="Data center name"
    )

    location = fields.Char(
        string="Location",
        help="Physical location"
    )

    # Reference to Tower servers
    server_ids = fields.One2many(
        comodel_name='cx.tower.server',
        inverse_name='datacenter_id',
        string="Servers",
        help="Servers in this datacenter"
    )

    server_count = fields.Integer(
        string="Server Count",
        compute='_compute_server_count',
        store=True,
    )

    @api.depends('server_ids')
    def _compute_server_count(self):
        for datacenter in self:
            datacenter.server_count = len(datacenter.server_ids)
```

```python
# custom_tower_extension/models/custom_server_backup.py
from odoo import fields, models

class CustomServerBackup(models.Model):
    _name = "custom.server.backup"
    _description = "Server Backup"
    _order = "backup_date desc"

    name = fields.Char(
        string="Backup Name",
        required=True,
    )

    # Reference to Tower server
    server_id = fields.Many2one(
        comodel_name='cx.tower.server',
        string="Server",
        required=True,
        ondelete='cascade',
    )

    backup_date = fields.Datetime(
        string="Backup Date",
        default=fields.Datetime.now,
        required=True,
    )

    backup_type = fields.Selection(
        selection=[
            ('full', 'Full'),
            ('incremental', 'Incremental'),
            ('differential', 'Differential'),
        ],
        string="Type",
        default='full',
        required=True,
    )

    size_mb = fields.Float(
        string="Size (MB)",
        digits=(12, 2),
    )

    status = fields.Selection(
        selection=[
            ('pending', 'Pending'),
            ('running', 'Running'),
            ('completed', 'Completed'),
            ('failed', 'Failed'),
        ],
        string="Status",
        default='pending',
    )

    file_path = fields.Char(
        string="File Path",
        help="Backup file location"
    )
```

### Security for New Models

```csv
# custom_tower_extension/security/ir.model.access.csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_custom_datacenter_user,custom.datacenter.user,model_custom_datacenter,cetmix_tower_server.group_user,1,0,0,0
access_custom_datacenter_manager,custom.datacenter.manager,model_custom_datacenter,cetmix_tower_server.group_manager,1,1,1,1
access_custom_server_backup_user,custom.server.backup.user,model_custom_server_backup,cetmix_tower_server.group_user,1,0,0,0
access_custom_server_backup_manager,custom.server.backup.manager,model_custom_server_backup,cetmix_tower_server.group_manager,1,1,1,1
```

## Version Compatibility

### For All Odoo Versions

Write code that works across versions when possible:

```python
# Good: Version-independent code
from odoo import fields, models

class CxTowerServer(models.Model):
    _inherit = "cx.tower.server"

    custom_field = fields.Char("Custom Field")
```

### Odoo 17.0 Specific

Current version considerations:

```python
# Odoo 17.0 features
from odoo import fields, models

class CxTowerServer(models.Model):
    _inherit = "cx.tower.server"

    # Use modern field attributes
    custom_json = fields.Json(
        string="Custom JSON Data",
        help="JSON field available in Odoo 17+"
    )
```

### Odoo 18.0 Preparation

Prepare for future versions:

```python
# Write upgrade-friendly code
from odoo import api, fields, models

class CxTowerServer(models.Model):
    _inherit = "cx.tower.server"

    # Avoid deprecated patterns
    # Use @api.depends instead of old-style compute
    backup_count = fields.Integer(
        compute='_compute_backup_count',
        store=True,
    )

    @api.depends('backup_ids')
    def _compute_backup_count(self):
        for server in self:
            server.backup_count = len(server.backup_ids)
```

### Version Detection

```python
from odoo import release

class CxTowerServer(models.Model):
    _inherit = "cx.tower.server"

    def custom_method(self):
        # Version-specific logic
        version_info = release.version_info

        if version_info[0] >= 18:
            # Odoo 18+ specific code
            pass
        elif version_info[0] == 17:
            # Odoo 17 specific code
            pass
```

## Complete Examples

### Example 1: Server Monitoring Extension

```python
# custom_tower_monitoring/__manifest__.py
{
    'name': 'Tower Server Monitoring',
    'version': '17.0.1.0.0',
    'depends': ['cetmix_tower_server'],
    'data': [
        'security/ir.model.access.csv',
        'views/cx_tower_server_views.xml',
        'data/monitoring_cron.xml',
    ],
}
```

```python
# custom_tower_monitoring/models/cx_tower_server.py
from odoo import api, fields, models
from datetime import datetime, timedelta

class CxTowerServer(models.Model):
    _inherit = "cx.tower.server"

    # Monitoring fields
    monitoring_enabled = fields.Boolean(
        string="Enable Monitoring",
        default=False,
    )

    monitoring_interval = fields.Integer(
        string="Check Interval (minutes)",
        default=5,
    )

    last_monitoring_check = fields.Datetime(
        string="Last Check",
        readonly=True,
    )

    monitoring_status = fields.Selection([
        ('ok', 'OK'),
        ('warning', 'Warning'),
        ('error', 'Error'),
        ('unknown', 'Unknown'),
    ], string="Status", default='unknown', readonly=True)

    cpu_usage = fields.Float(
        string="CPU Usage (%)",
        readonly=True,
    )

    memory_usage = fields.Float(
        string="Memory Usage (%)",
        readonly=True,
    )

    disk_usage = fields.Float(
        string="Disk Usage (%)",
        readonly=True,
    )

    def action_check_monitoring(self):
        """Run monitoring check"""
        self.ensure_one()

        if not self.monitoring_enabled:
            return

        # Run monitoring command
        command = self.env.ref('custom_tower_monitoring.command_system_check')
        result = self.run_command(command)

        # Parse results and update fields
        self._parse_monitoring_results(result)

    def _parse_monitoring_results(self, result):
        """Parse monitoring command results"""
        # Implementation here
        pass

    @api.model
    def cron_run_monitoring(self):
        """Cron job to run monitoring checks"""
        servers = self.search([
            ('monitoring_enabled', '=', True),
            ('active', '=', True),
        ])

        for server in servers:
            # Check if it's time to run
            if server.last_monitoring_check:
                next_check = server.last_monitoring_check + \
                    timedelta(minutes=server.monitoring_interval)
                if datetime.now() < next_check:
                    continue

            server.action_check_monitoring()
```

### Example 2: Custom Command Type

```python
# custom_tower_commands/models/cx_tower_command.py
from odoo import models, _
from odoo.tools.safe_eval import wrap_module

# Import your custom library
custom_lib = wrap_module(__import__("your_custom_library"), ["method1", "method2"])

class CxTowerCommand(models.Model):
    _inherit = "cx.tower.command"

    def _selection_action(self):
        """Add custom command type"""
        actions = super()._selection_action()
        actions.append(('custom_action', 'Custom Action Type'))
        return actions

    def _custom_python_libraries(self):
        """Add custom Python library"""
        libraries = super()._custom_python_libraries()
        libraries.update({
            'custom_tower_commands': {
                'custom_lib': {
                    'import': custom_lib,
                    'help': _("Your custom library description"),
                },
            }
        })
        return libraries
```

```python
# custom_tower_commands/models/cx_tower_server.py
from odoo import models

class CxTowerServer(models.Model):
    _inherit = "cx.tower.server"

    def _command_runner(self, command, log_record, rendered_command_code,
                       sudo=None, rendered_command_path=None,
                       ssh_connection=None, **kwargs):
        """Add custom command runner"""

        if command.action == 'custom_action':
            return self._command_runner_custom_action(
                log_record, rendered_command_code, **kwargs
            )

        return super()._command_runner(
            command, log_record, rendered_command_code,
            sudo, rendered_command_path, ssh_connection, **kwargs
        )

    def _command_runner_custom_action(self, log_record, code, **kwargs):
        """Custom command runner implementation"""
        # Your custom logic here
        pass
```

## Next Steps

- Learn about [Custom Fields](02-custom-fields.md) in detail
- Explore [Custom Commands](03-custom-commands.md) patterns
- Review [Extending Mixins](04-extending-mixins.md)
- Check [Deployment Guide](../09-deployment-operations/01-deployment-guide.md)

## Best Practices Summary

✅ **Always use inheritance**
✅ **Call super() when overriding methods**
✅ **Use XPath for view extensions**
✅ **Follow Odoo naming conventions**
✅ **Add proper security rules**
✅ **Document your code**
✅ **Test before deploying**

❌ **Never modify original files**
❌ **Don't copy entire files**
❌ **Don't skip super() calls**
❌ **Don't hardcode values**

---

**Remember**: The inheritance approach ensures your customizations are upgrade-safe, maintainable, and follow Odoo best practices!

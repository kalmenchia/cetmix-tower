---
title: Adding Custom Fields via Inheritance
description: Guide to adding custom fields to Tower models using inheritance
author: Cetmix
date: 2025-11-16
version: 1.0
module: cetmix_tower_server
category: customization
tags: [custom-fields, inheritance, models, odoo-17]
---

# Adding Custom Fields via Inheritance

This guide covers how to add custom fields to existing Cetmix Tower models using inheritance, including field types, computed fields, relational fields, and security considerations.

## Table of Contents

- [Basic Principles](#basic-principles)
- [Field Types](#field-types)
- [Computed Fields](#computed-fields)
- [Relational Fields](#relational-fields)
- [Field Attributes](#field-attributes)
- [Security Considerations](#security-considerations)
- [Migration Scripts](#migration-scripts)
- [Best Practices](#best-practices)

## Basic Principles

### Always Use Inheritance

```python
# custom_tower_extension/models/cx_tower_server.py
from odoo import fields, models

class CxTowerServer(models.Model):
    _inherit = "cx.tower.server"

    # Your custom fields here
    custom_field = fields.Char("Custom Field")
```

### Field Naming Conventions

- Use clear, descriptive names
- Prefix with module name for clarity (optional but recommended)
- Use snake_case for field names
- Avoid conflicts with existing field names

```python
# Good examples
monitoring_url = fields.Char("Monitoring URL")
backup_retention_days = fields.Integer("Backup Retention (days)")
is_production_server = fields.Boolean("Production Server")

# Avoid generic names that might conflict
# custom = fields.Char()  # Too generic
# data = fields.Text()     # Too generic
```

## Field Types

### Character Fields (Char)

For short text strings:

```python
from odoo import fields, models

class CxTowerServer(models.Model):
    _inherit = "cx.tower.server"

    external_id = fields.Char(
        string="External ID",
        size=64,
        help="ID in external system",
        copy=False,
        index=True,
    )

    monitoring_url = fields.Char(
        string="Monitoring URL",
        help="URL to monitoring dashboard",
    )

    serial_number = fields.Char(
        string="Serial Number",
        readonly=True,
        copy=False,
    )
```

### Text Fields

For longer text content:

```python
class CxTowerServer(models.Model):
    _inherit = "cx.tower.server"

    maintenance_notes = fields.Text(
        string="Maintenance Notes",
        help="Detailed maintenance history and notes",
    )

    configuration_details = fields.Text(
        string="Configuration",
        help="Server configuration details",
    )
```

### HTML Fields

For rich text content:

```python
class CxTowerServer(models.Model):
    _inherit = "cx.tower.server"

    documentation = fields.Html(
        string="Documentation",
        help="Server documentation with formatting",
        sanitize=True,  # Sanitize HTML for security
    )
```

### Integer Fields

For whole numbers:

```python
class CxTowerServer(models.Model):
    _inherit = "cx.tower.server"

    cpu_cores = fields.Integer(
        string="CPU Cores",
        default=1,
        help="Number of CPU cores",
    )

    backup_retention_days = fields.Integer(
        string="Backup Retention",
        default=30,
        help="Days to retain backups",
    )

    max_connections = fields.Integer(
        string="Max Connections",
        help="Maximum allowed connections",
    )
```

### Float Fields

For decimal numbers:

```python
class CxTowerServer(models.Model):
    _inherit = "cx.tower.server"

    memory_gb = fields.Float(
        string="Memory (GB)",
        digits=(10, 2),  # Total 10 digits, 2 after decimal
        help="Total RAM in GB",
    )

    cpu_usage_threshold = fields.Float(
        string="CPU Threshold (%)",
        digits=(5, 2),  # e.g., 99.99%
        default=80.0,
        help="Alert threshold for CPU usage",
    )

    disk_size_tb = fields.Float(
        string="Disk Size (TB)",
        digits=(12, 3),
        help="Total disk space in TB",
    )
```

### Monetary Fields

For currency amounts:

```python
class CxTowerServer(models.Model):
    _inherit = "cx.tower.server"

    monthly_cost = fields.Monetary(
        string="Monthly Cost",
        currency_field='currency_id',
        help="Monthly hosting cost",
    )

    currency_id = fields.Many2one(
        'res.currency',
        string="Currency",
        default=lambda self: self.env.company.currency_id,
    )
```

### Boolean Fields

For yes/no values:

```python
class CxTowerServer(models.Model):
    _inherit = "cx.tower.server"

    is_monitored = fields.Boolean(
        string="Monitored",
        default=False,
        help="Server is actively monitored",
    )

    auto_backup_enabled = fields.Boolean(
        string="Auto Backup",
        default=True,
        help="Enable automatic backups",
    )

    requires_vpn = fields.Boolean(
        string="Requires VPN",
        default=False,
        help="VPN required to access server",
    )
```

### Selection Fields

For predefined choices:

```python
class CxTowerServer(models.Model):
    _inherit = "cx.tower.server"

    environment = fields.Selection(
        selection=[
            ('dev', 'Development'),
            ('test', 'Testing'),
            ('staging', 'Staging'),
            ('prod', 'Production'),
        ],
        string="Environment",
        default='dev',
        required=True,
        help="Server environment type",
        index=True,
    )

    priority = fields.Selection(
        selection=[
            ('low', 'Low'),
            ('normal', 'Normal'),
            ('high', 'High'),
            ('critical', 'Critical'),
        ],
        string="Priority",
        default='normal',
        help="Server priority level",
    )

    # Dynamic selection using method
    server_tier = fields.Selection(
        selection='_selection_server_tier',
        string="Server Tier",
        help="Server tier/plan",
    )

    def _selection_server_tier(self):
        """Dynamic selection values"""
        return [
            ('basic', 'Basic'),
            ('standard', 'Standard'),
            ('premium', 'Premium'),
            ('enterprise', 'Enterprise'),
        ]
```

### Date and DateTime Fields

```python
class CxTowerServer(models.Model):
    _inherit = "cx.tower.server"

    # Date only
    warranty_expiry = fields.Date(
        string="Warranty Expiry",
        help="Hardware warranty expiration date",
    )

    next_maintenance = fields.Date(
        string="Next Maintenance",
        help="Scheduled maintenance date",
        index=True,
    )

    # Date and time
    last_backup = fields.Datetime(
        string="Last Backup",
        readonly=True,
        help="Last successful backup timestamp",
    )

    created_at = fields.Datetime(
        string="Created At",
        default=fields.Datetime.now,
        readonly=True,
    )
```

### Binary Fields

For file attachments:

```python
class CxTowerServer(models.Model):
    _inherit = "cx.tower.server"

    license_file = fields.Binary(
        string="License File",
        attachment=True,
        help="Server license file",
    )

    license_filename = fields.Char(
        string="License Filename",
    )

    server_image = fields.Image(
        string="Server Image",
        max_width=1024,
        max_height=1024,
        help="Server/rack photo",
    )
```

### JSON Fields (Odoo 17+)

For structured data:

```python
class CxTowerServer(models.Model):
    _inherit = "cx.tower.server"

    custom_metadata = fields.Json(
        string="Custom Metadata",
        help="Custom JSON data for integrations",
    )

    monitoring_config = fields.Json(
        string="Monitoring Config",
        default={},
        help="Monitoring configuration in JSON format",
    )
```

## Computed Fields

### Basic Computed Fields

```python
from odoo import api, fields, models

class CxTowerServer(models.Model):
    _inherit = "cx.tower.server"

    backup_count = fields.Integer(
        string="Backup Count",
        compute='_compute_backup_count',
        store=True,  # Store in database for better performance
        help="Total number of backups",
    )

    @api.depends('backup_ids')
    def _compute_backup_count(self):
        """Compute total backups"""
        for server in self:
            server.backup_count = len(server.backup_ids)
```

### Computed Fields with Multiple Dependencies

```python
class CxTowerServer(models.Model):
    _inherit = "cx.tower.server"

    resource_utilization = fields.Float(
        string="Resource Utilization (%)",
        compute='_compute_resource_utilization',
        store=True,
        help="Overall resource utilization",
    )

    @api.depends('cpu_usage', 'memory_usage', 'disk_usage')
    def _compute_resource_utilization(self):
        """Compute average resource utilization"""
        for server in self:
            if server.cpu_usage or server.memory_usage or server.disk_usage:
                total = sum(filter(None, [
                    server.cpu_usage,
                    server.memory_usage,
                    server.disk_usage
                ]))
                count = sum(1 for x in [
                    server.cpu_usage,
                    server.memory_usage,
                    server.disk_usage
                ] if x)
                server.resource_utilization = total / count if count else 0.0
            else:
                server.resource_utilization = 0.0
```

### Computed Selection Fields

```python
class CxTowerServer(models.Model):
    _inherit = "cx.tower.server"

    health_status = fields.Selection(
        selection=[
            ('healthy', 'Healthy'),
            ('warning', 'Warning'),
            ('critical', 'Critical'),
            ('unknown', 'Unknown'),
        ],
        string="Health Status",
        compute='_compute_health_status',
        store=True,
        help="Overall server health",
    )

    @api.depends('cpu_usage', 'memory_usage', 'disk_usage', 'last_backup')
    def _compute_health_status(self):
        """Compute health status based on metrics"""
        from datetime import datetime, timedelta

        for server in self:
            # Critical conditions
            if (server.cpu_usage and server.cpu_usage > 95) or \
               (server.memory_usage and server.memory_usage > 95) or \
               (server.disk_usage and server.disk_usage > 95):
                server.health_status = 'critical'
                continue

            # Warning conditions
            if (server.cpu_usage and server.cpu_usage > 80) or \
               (server.memory_usage and server.memory_usage > 80) or \
               (server.disk_usage and server.disk_usage > 80):
                server.health_status = 'warning'
                continue

            # Check last backup
            if server.last_backup:
                days_since_backup = (datetime.now() - server.last_backup).days
                if days_since_backup > 7:
                    server.health_status = 'warning'
                    continue

            # Default to healthy
            server.health_status = 'healthy' if server.active else 'unknown'
```

### Inverse Methods for Computed Fields

```python
class CxTowerServer(models.Model):
    _inherit = "cx.tower.server"

    full_address = fields.Char(
        string="Full Address",
        compute='_compute_full_address',
        inverse='_inverse_full_address',
        store=True,
        help="IP address with port",
    )

    @api.depends('ip_v4_address', 'ssh_port')
    def _compute_full_address(self):
        """Compute full address from IP and port"""
        for server in self:
            if server.ip_v4_address and server.ssh_port:
                server.full_address = f"{server.ip_v4_address}:{server.ssh_port}"
            else:
                server.full_address = server.ip_v4_address or ''

    def _inverse_full_address(self):
        """Parse full address to IP and port"""
        for server in self:
            if server.full_address and ':' in server.full_address:
                parts = server.full_address.split(':')
                server.ip_v4_address = parts[0]
                try:
                    server.ssh_port = int(parts[1])
                except (ValueError, IndexError):
                    pass
```

## Relational Fields

### Many2one (Belongs To)

```python
class CxTowerServer(models.Model):
    _inherit = "cx.tower.server"

    datacenter_id = fields.Many2one(
        comodel_name='custom.datacenter',
        string="Data Center",
        ondelete='restrict',  # Prevent deletion of datacenter with servers
        help="Physical datacenter location",
        index=True,
    )

    cluster_id = fields.Many2one(
        comodel_name='custom.server.cluster',
        string="Cluster",
        ondelete='set null',  # Remove reference when cluster deleted
        help="Server cluster membership",
    )

    primary_contact_id = fields.Many2one(
        comodel_name='res.partner',
        string="Primary Contact",
        domain=[('is_company', '=', False)],  # Only individuals
        help="Primary technical contact",
    )
```

### One2many (Has Many)

```python
class CxTowerServer(models.Model):
    _inherit = "cx.tower.server"

    backup_ids = fields.One2many(
        comodel_name='custom.server.backup',
        inverse_name='server_id',
        string="Backups",
        help="Server backup records",
    )

    monitoring_log_ids = fields.One2many(
        comodel_name='custom.monitoring.log',
        inverse_name='server_id',
        string="Monitoring Logs",
        help="Historical monitoring data",
    )

    alert_ids = fields.One2many(
        comodel_name='custom.server.alert',
        inverse_name='server_id',
        string="Alerts",
        domain=[('status', '=', 'active')],  # Only active alerts
        help="Server alerts",
    )
```

### Many2many (Has Many, Belongs To Many)

```python
class CxTowerServer(models.Model):
    _inherit = "cx.tower.server"

    application_ids = fields.Many2many(
        comodel_name='custom.application',
        relation='custom_server_application_rel',
        column1='server_id',
        column2='application_id',
        string="Applications",
        help="Applications running on this server",
    )

    maintenance_team_ids = fields.Many2many(
        comodel_name='res.users',
        relation='custom_server_maintenance_team_rel',
        column1='server_id',
        column2='user_id',
        string="Maintenance Team",
        help="Team responsible for server maintenance",
    )

    compliance_tag_ids = fields.Many2many(
        comodel_name='custom.compliance.tag',
        relation='custom_server_compliance_tag_rel',
        column1='server_id',
        column2='tag_id',
        string="Compliance Tags",
        help="Compliance and certification tags",
    )
```

### Related Fields

```python
class CxTowerServer(models.Model):
    _inherit = "cx.tower.server"

    # Related field from Many2one
    datacenter_location = fields.Char(
        related='datacenter_id.location',
        string="Datacenter Location",
        store=True,  # Store for search/filter performance
        readonly=True,
    )

    datacenter_country_id = fields.Many2one(
        related='datacenter_id.country_id',
        string="Datacenter Country",
        store=True,
    )

    # Related field with multiple levels
    partner_email = fields.Char(
        related='partner_id.email',
        string="Partner Email",
        readonly=True,
    )
```

## Field Attributes

### Common Attributes

```python
class CxTowerServer(models.Model):
    _inherit = "cx.tower.server"

    custom_field = fields.Char(
        # Display
        string="Display Name",
        help="Help text shown on hover",

        # Requirements
        required=True,
        readonly=True,
        copy=False,  # Don't copy when duplicating record

        # Database
        index=True,  # Create database index
        store=True,  # Store computed field in database

        # Default
        default="default value",
        default=lambda self: self._default_value(),

        # Groups (security)
        groups="cetmix_tower_server.group_manager",

        # Tracking (for mail.thread models)
        tracking=True,

        # Search
        search='_search_custom_field',  # Custom search method
    )
```

### Field Groups (Conditional Visibility)

```python
class CxTowerServer(models.Model):
    _inherit = "cx.tower.server"

    # Only managers can see/edit
    secret_config = fields.Text(
        string="Secret Configuration",
        groups="cetmix_tower_server.group_manager",
    )

    # Multiple groups (OR logic)
    admin_notes = fields.Text(
        string="Admin Notes",
        groups="base.group_system,cetmix_tower_server.group_manager",
    )
```

### Tracking Changes (for mail.thread models)

```python
class CxTowerServer(models.Model):
    _inherit = "cx.tower.server"
    # Note: cx.tower.server already inherits mail.thread

    environment = fields.Selection(
        selection=[
            ('dev', 'Development'),
            ('prod', 'Production'),
        ],
        tracking=True,  # Track changes in chatter
        string="Environment",
    )

    is_critical = fields.Boolean(
        string="Critical Server",
        tracking=True,
    )
```

## Security Considerations

### Access Rights

Define who can read/write your custom fields:

```csv
# custom_tower_extension/security/ir.model.access.csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
# Tower User can read, Manager can write
access_custom_model_user,custom.model.user,model_custom_model,cetmix_tower_server.group_user,1,0,0,0
access_custom_model_manager,custom.model.manager,model_custom_model,cetmix_tower_server.group_manager,1,1,1,1
```

### Record Rules

```xml
<!-- custom_tower_extension/security/security.xml -->
<odoo>
    <data noupdate="1">
        <!-- Users can only see servers they have access to -->
        <record id="custom_server_backup_rule_user" model="ir.rule">
            <field name="name">Custom Backup: User Access</field>
            <field name="model_id" ref="model_custom_server_backup"/>
            <field name="domain_force">[
                '|',
                ('server_id.user_ids', 'in', [user.id]),
                ('server_id.manager_ids', 'in', [user.id])
            ]</field>
            <field name="groups" eval="[(4, ref('cetmix_tower_server.group_user'))]"/>
        </record>

        <!-- Managers can see all -->
        <record id="custom_server_backup_rule_manager" model="ir.rule">
            <field name="name">Custom Backup: Manager Access</field>
            <field name="model_id" ref="model_custom_server_backup"/>
            <field name="domain_force">[(1, '=', 1)]</field>
            <field name="groups" eval="[(4, ref('cetmix_tower_server.group_manager'))]"/>
        </record>
    </data>
</odoo>
```

### Field-Level Security

```python
from odoo import api, fields, models
from odoo.exceptions import AccessError

class CxTowerServer(models.Model):
    _inherit = "cx.tower.server"

    sensitive_data = fields.Text(
        string="Sensitive Data",
        groups="base.group_system",  # Only system admins
    )

    @api.constrains('sensitive_data')
    def _check_sensitive_data_access(self):
        """Additional access check"""
        for server in self:
            if server.sensitive_data and not self.env.user.has_group('base.group_system'):
                raise AccessError("Only system administrators can modify sensitive data")
```

## Migration Scripts

### Adding Fields (Automatic)

Most fields are added automatically when the module is updated:

```bash
# Upgrade module
odoo-bin -u custom_tower_extension -d database_name
```

### Data Migration Scripts

For complex data migrations:

```python
# custom_tower_extension/migrations/17.0.1.1.0/pre-migrate.py
def migrate(cr, version):
    """Pre-migration script"""
    if not version:
        return

    # Add column with default value
    cr.execute("""
        ALTER TABLE cx_tower_server
        ADD COLUMN IF NOT EXISTS environment VARCHAR(20) DEFAULT 'dev'
    """)
```

```python
# custom_tower_extension/migrations/17.0.1.1.0/post-migrate.py
def migrate(cr, version):
    """Post-migration script"""
    from odoo import api, SUPERUSER_ID

    if not version:
        return

    env = api.Environment(cr, SUPERUSER_ID, {})

    # Update environment based on tags
    servers = env['cx.tower.server'].search([])
    for server in servers:
        if 'production' in server.tag_ids.mapped('name'):
            server.environment = 'prod'
        elif 'staging' in server.tag_ids.mapped('name'):
            server.environment = 'staging'
```

### Renaming Fields

```python
# custom_tower_extension/migrations/17.0.1.2.0/pre-migrate.py
def migrate(cr, version):
    """Rename field"""
    if not version:
        return

    # Rename column
    cr.execute("""
        ALTER TABLE cx_tower_server
        RENAME COLUMN old_field_name TO new_field_name
    """)
```

## Best Practices

### ✅ DO:

1. **Use clear, descriptive field names**
   ```python
   backup_retention_days = fields.Integer()  # Good
   days = fields.Integer()  # Too generic
   ```

2. **Add help text**
   ```python
   custom_field = fields.Char(
       help="Clear description of what this field does"
   )
   ```

3. **Use appropriate field types**
   ```python
   # Use Selection for predefined choices
   environment = fields.Selection([...])

   # Not Char with validation
   environment = fields.Char()  # Avoid this
   ```

4. **Store computed fields when appropriate**
   ```python
   # Frequently accessed/searched
   total_backups = fields.Integer(compute='...', store=True)
   ```

5. **Use indexes for frequently searched fields**
   ```python
   external_id = fields.Char(index=True)
   ```

### ❌ DON'T:

1. **Don't modify core model files**
2. **Don't use generic field names**
3. **Don't forget security rules**
4. **Don't skip help text**
5. **Don't ignore migration scripts for complex changes**

### Performance Tips

```python
# Store computed fields that are frequently accessed
backup_count = fields.Integer(compute='...', store=True)

# Use related fields sparingly (can impact performance)
# Consider copying data if it rarely changes
datacenter_location = fields.Char(related='datacenter_id.location', store=True)

# Index fields used in search/filter
external_id = fields.Char(index=True)
environment = fields.Selection(index=True)
```

## Complete Example

```python
# custom_tower_monitoring/models/cx_tower_server.py
from odoo import api, fields, models
from datetime import datetime, timedelta

class CxTowerServer(models.Model):
    _inherit = "cx.tower.server"

    # Basic fields
    environment = fields.Selection(
        selection=[
            ('dev', 'Development'),
            ('test', 'Testing'),
            ('staging', 'Staging'),
            ('prod', 'Production'),
        ],
        string="Environment",
        default='dev',
        required=True,
        index=True,
        tracking=True,
        help="Server environment type",
    )

    monitoring_enabled = fields.Boolean(
        string="Enable Monitoring",
        default=False,
        tracking=True,
        help="Enable automatic monitoring for this server",
    )

    # Monitoring data
    cpu_usage = fields.Float(
        string="CPU Usage (%)",
        digits=(5, 2),
        readonly=True,
        groups="cetmix_tower_server.group_user",
    )

    memory_usage = fields.Float(
        string="Memory Usage (%)",
        digits=(5, 2),
        readonly=True,
    )

    last_monitoring_check = fields.Datetime(
        string="Last Check",
        readonly=True,
        index=True,
    )

    # Computed fields
    health_status = fields.Selection(
        selection=[
            ('healthy', 'Healthy'),
            ('warning', 'Warning'),
            ('critical', 'Critical'),
        ],
        string="Health",
        compute='_compute_health_status',
        store=True,
    )

    days_since_check = fields.Integer(
        string="Days Since Check",
        compute='_compute_days_since_check',
    )

    # Relational fields
    datacenter_id = fields.Many2one(
        'custom.datacenter',
        string="Data Center",
        ondelete='restrict',
        help="Physical datacenter location",
    )

    monitoring_log_ids = fields.One2many(
        'custom.monitoring.log',
        'server_id',
        string="Monitoring Logs",
    )

    # Related fields
    datacenter_location = fields.Char(
        related='datacenter_id.location',
        string="Location",
        store=True,
        readonly=True,
    )

    @api.depends('cpu_usage', 'memory_usage')
    def _compute_health_status(self):
        for server in self:
            if server.cpu_usage > 90 or server.memory_usage > 90:
                server.health_status = 'critical'
            elif server.cpu_usage > 75 or server.memory_usage > 75:
                server.health_status = 'warning'
            else:
                server.health_status = 'healthy'

    @api.depends('last_monitoring_check')
    def _compute_days_since_check(self):
        for server in self:
            if server.last_monitoring_check:
                delta = datetime.now() - server.last_monitoring_check
                server.days_since_check = delta.days
            else:
                server.days_since_check = -1
```

## Next Steps

- Explore [Custom Commands](03-custom-commands.md)
- Learn about [Extending Mixins](04-extending-mixins.md)
- Review [Deployment Guide](../09-deployment-operations/01-deployment-guide.md)

---

**Remember**: Always use inheritance to add fields, never modify original files!

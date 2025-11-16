---
title: Extending Tower Mixins
description: Guide to understanding and extending Tower's mixin architecture
author: Cetmix
date: 2025-11-16
version: 1.0
module: cetmix_tower_server
category: customization
tags: [mixins, architecture, inheritance, patterns]
---

# Extending Tower Mixins

This guide explains Tower's mixin architecture and how to create and extend mixins for reusable functionality across models.

## Table of Contents

- [Understanding Mixins](#understanding-mixins)
- [Tower's Built-in Mixins](#towers-built-in-mixins)
- [Extending Existing Mixins](#extending-existing-mixins)
- [Creating Custom Mixins](#creating-custom-mixins)
- [Best Practices](#best-practices)

## Understanding Mixins

### What are Mixins?

Mixins are Abstract Models that provide reusable functionality to multiple models through inheritance. They don't create database tables themselves but add fields and methods to models that inherit them.

### Why Use Mixins?

- **Code Reusability**: Define functionality once, use in multiple models
- **Consistency**: Same behavior across different models
- **Maintainability**: Update in one place, changes apply everywhere
- **Modularity**: Separate concerns into focused components

### Basic Mixin Pattern

```python
# Abstract model (mixin)
class MyMixin(models.AbstractModel):
    _name = "my.mixin"
    _description = "My Custom Mixin"

    # Fields added to inheriting models
    my_field = fields.Char("My Field")

    # Methods available to inheriting models
    def my_method(self):
        """Method available to all inheriting models"""
        pass

# Using the mixin
class MyModel(models.Model):
    _name = "my.model"
    _inherit = ["my.mixin"]  # Inherits mixin functionality

    # Now has my_field and my_method available
```

## Tower's Built-in Mixins

### Access Control Mixins

#### 1. `cx.tower.access.mixin`

Provides access level management:

```python
# Location: cetmix_tower_server/models/cx_tower_access_mixin.py
class CxTowerAccessMixin(models.AbstractModel):
    _name = "cx.tower.access.mixin"

    access_level = fields.Selection(
        selection=[
            ('1', 'User'),
            ('2', 'Manager'),
            ('3', 'Root'),
        ],
        default='2',
        required=True,
    )
```

**Usage**: Used by templates, commands, plans, etc.

#### 2. `cx.tower.access.role.mixin`

Provides role-based access control:

```python
# Fields added by mixin
user_ids = fields.Many2many('res.users')      # Users with access
manager_ids = fields.Many2many('res.users')   # Managers with access
```

**Usage**: Used by servers, commands, plans.

### Data Management Mixins

#### 3. `cx.tower.variable.mixin`

Provides variable management:

```python
# Location: cetmix_tower_server/models/cx_tower_variable_mixin.py
class TowerVariableMixin(models.AbstractModel):
    _name = "cx.tower.variable.mixin"

    variable_value_ids = fields.One2many(
        'cx.tower.variable.value',
        string="Variable Values",
    )

    def get_variable_values(self, variable_references, apply_modifiers=True):
        """Get variable values for selected records"""
        pass
```

**Usage**: Used by servers and other models that need variables.

#### 4. `cx.tower.key.mixin`

Provides secret key management:

```python
# Fields and methods for managing secrets
def _parse_code_and_return_key_values(self, code, **kwargs):
    """Parse code and extract key values"""
    pass
```

**Usage**: Used by commands and models that handle secrets.

#### 5. `cx.tower.vault.mixin`

Provides secure vault integration for secrets:

```python
# Methods for vault operations
def _get_secret_value(self, field_name):
    """Get secret value from vault"""
    pass

def _set_secret_value(self, field_name, value):
    """Set secret value in vault"""
    pass
```

**Usage**: Used by servers and models storing sensitive data.

### Template and Reference Mixins

#### 6. `cx.tower.template.mixin`

Provides template functionality:

```python
# Methods for template rendering
def render_code_custom(self, code, **kwargs):
    """Render template code with variables"""
    pass

def get_variables_from_code(self, code):
    """Extract variable references from code"""
    pass
```

**Usage**: Used by commands, files, plans that support template rendering.

#### 7. `cx.tower.reference.mixin`

Provides reference field management:

```python
# Fields
reference = fields.Char("Reference", required=True, index=True)

# Methods
def _generate_reference(self):
    """Generate unique reference"""
    pass
```

**Usage**: Used by most Tower models for unique references.

### Custom Variable Mixins

#### 8. `cx.tower.custom.variable.value.mixin`

For models that need custom variable values:

```python
# Provides custom variable value management
```

**Usage**: Used by scheduled tasks and other models with custom variables.

## Extending Existing Mixins

### Adding Fields to a Mixin

```python
# custom_tower_extension/models/cx_tower_access_mixin.py
from odoo import fields, models

class CxTowerAccessMixin(models.AbstractModel):
    _inherit = "cx.tower.access.mixin"

    # Add custom access levels
    def _selection_access_level(self):
        """Extend access levels"""
        levels = super()._selection_access_level()

        # Add custom levels
        levels.extend([
            ('4', 'Super Root'),
            ('5', 'System Admin'),
        ])

        return levels

    # Add related field
    access_level_name = fields.Char(
        string="Access Level Name",
        compute='_compute_access_level_name',
    )

    def _compute_access_level_name(self):
        """Compute access level name"""
        levels = dict(self._selection_access_level())
        for record in self:
            record.access_level_name = levels.get(record.access_level, 'Unknown')
```

### Adding Methods to a Mixin

```python
# custom_tower_extension/models/cx_tower_variable_mixin.py
from odoo import models

class TowerVariableMixin(models.AbstractModel):
    _inherit = "cx.tower.variable.mixin"

    def get_variable_value_by_reference(self, reference):
        """Get a single variable value by reference

        Args:
            reference (str): Variable reference

        Returns:
            str: Variable value
        """
        self.ensure_one()

        values = self.get_variable_values([reference])
        return values.get(self.id, {}).get(reference)

    def set_variable_value(self, reference, value):
        """Set variable value

        Args:
            reference (str): Variable reference
            value (str): New value
        """
        self.ensure_one()

        # Find or create variable value
        var_value = self.variable_value_ids.filtered(
            lambda v: v.variable_reference == reference
        )

        if var_value:
            var_value.value_char = value
        else:
            variable = self.env['cx.tower.variable'].search([
                ('reference', '=', reference)
            ], limit=1)

            if variable:
                self.env['cx.tower.variable.value'].create({
                    'variable_id': variable.id,
                    'server_id': self.id if self._name == 'cx.tower.server' else False,
                    'value_char': value,
                })
```

### Overriding Mixin Methods

```python
# custom_tower_extension/models/cx_tower_vault_mixin.py
from odoo import models
import logging

_logger = logging.getLogger(__name__)

class CxTowerVaultMixin(models.AbstractModel):
    _inherit = "cx.tower.vault.mixin"

    def _get_secret_value(self, field_name):
        """Override to add custom logging"""

        # Custom logic before
        _logger.info(f"Accessing secret: {field_name}")

        # Call original method
        result = super()._get_secret_value(field_name)

        # Custom logic after
        if result:
            _logger.info(f"Secret {field_name} retrieved successfully")

        return result

    def _set_secret_value(self, field_name, value):
        """Override to add validation"""

        # Custom validation
        if not value:
            raise ValueError(f"Secret value for {field_name} cannot be empty")

        # Call original method
        return super()._set_secret_value(field_name, value)
```

## Creating Custom Mixins

### Basic Custom Mixin

```python
# custom_tower_extension/models/custom_audit_mixin.py
from odoo import api, fields, models

class CustomAuditMixin(models.AbstractModel):
    """Mixin for audit trail functionality"""

    _name = "custom.audit.mixin"
    _description = "Custom Audit Mixin"

    # Audit fields
    created_by_id = fields.Many2one(
        'res.users',
        string="Created By",
        default=lambda self: self.env.user,
        readonly=True,
    )

    created_date = fields.Datetime(
        string="Created Date",
        default=fields.Datetime.now,
        readonly=True,
    )

    modified_by_id = fields.Many2one(
        'res.users',
        string="Last Modified By",
        readonly=True,
    )

    modified_date = fields.Datetime(
        string="Last Modified Date",
        readonly=True,
    )

    modification_count = fields.Integer(
        string="Modifications",
        default=0,
        readonly=True,
        help="Number of times record has been modified",
    )

    def write(self, vals):
        """Track modifications"""
        vals.update({
            'modified_by_id': self.env.user.id,
            'modified_date': fields.Datetime.now(),
            'modification_count': self.modification_count + 1,
        })
        return super().write(vals)
```

### Using Custom Mixin

```python
# custom_tower_extension/models/cx_tower_server.py
from odoo import models

class CxTowerServer(models.Model):
    _inherit = ["cx.tower.server", "custom.audit.mixin"]
```

### Advanced Custom Mixin

```python
# custom_tower_extension/models/custom_notification_mixin.py
from odoo import api, fields, models
import logging

_logger = logging.getLogger(__name__)

class CustomNotificationMixin(models.AbstractModel):
    """Mixin for notification functionality"""

    _name = "custom.notification.mixin"
    _description = "Custom Notification Mixin"

    # Notification settings
    notification_enabled = fields.Boolean(
        string="Enable Notifications",
        default=True,
    )

    notification_channel_ids = fields.Many2many(
        'custom.notification.channel',
        string="Notification Channels",
        help="Channels to send notifications to",
    )

    notification_template_id = fields.Many2one(
        'mail.template',
        string="Notification Template",
        help="Email template for notifications",
    )

    last_notification_date = fields.Datetime(
        string="Last Notification",
        readonly=True,
    )

    # Notification methods
    def send_notification(self, subject, message, notification_type='info'):
        """Send notification through configured channels

        Args:
            subject (str): Notification subject
            message (str): Notification message
            notification_type (str): Type (info, warning, error)
        """
        self.ensure_one()

        if not self.notification_enabled:
            return

        for channel in self.notification_channel_ids:
            try:
                channel.send_notification(
                    record=self,
                    subject=subject,
                    message=message,
                    notification_type=notification_type,
                )
            except Exception as e:
                _logger.error(f"Failed to send notification via {channel.name}: {e}")

        # Update last notification date
        self.last_notification_date = fields.Datetime.now()

    def send_template_notification(self, template_id=None):
        """Send notification using email template

        Args:
            template_id (int): Mail template ID (uses default if not provided)
        """
        self.ensure_one()

        template = template_id or self.notification_template_id
        if not template:
            return

        template.send_mail(self.id, force_send=True)
        self.last_notification_date = fields.Datetime.now()

    @api.model
    def _get_notification_context(self):
        """Get context for notification templates

        Returns:
            dict: Template context
        """
        return {
            'user': self.env.user,
            'company': self.env.company,
            'timestamp': fields.Datetime.now(),
        }
```

### Mixin with Computed Fields

```python
# custom_tower_extension/models/custom_monitoring_mixin.py
from odoo import api, fields, models
from datetime import datetime, timedelta

class CustomMonitoringMixin(models.AbstractModel):
    """Mixin for monitoring functionality"""

    _name = "custom.monitoring.mixin"
    _description = "Custom Monitoring Mixin"

    # Monitoring fields
    monitoring_enabled = fields.Boolean(
        string="Monitoring Enabled",
        default=False,
    )

    monitoring_interval = fields.Integer(
        string="Check Interval (minutes)",
        default=5,
        help="How often to run monitoring checks",
    )

    last_monitoring_check = fields.Datetime(
        string="Last Check",
        readonly=True,
    )

    next_monitoring_check = fields.Datetime(
        string="Next Check",
        compute='_compute_next_monitoring_check',
        store=True,
    )

    monitoring_status = fields.Selection([
        ('ok', 'OK'),
        ('warning', 'Warning'),
        ('error', 'Error'),
        ('unknown', 'Unknown'),
    ], string="Status", default='unknown')

    is_monitoring_overdue = fields.Boolean(
        string="Monitoring Overdue",
        compute='_compute_is_monitoring_overdue',
    )

    @api.depends('last_monitoring_check', 'monitoring_interval')
    def _compute_next_monitoring_check(self):
        """Compute next monitoring check time"""
        for record in self:
            if record.last_monitoring_check and record.monitoring_interval:
                record.next_monitoring_check = \
                    record.last_monitoring_check + \
                    timedelta(minutes=record.monitoring_interval)
            else:
                record.next_monitoring_check = False

    @api.depends('next_monitoring_check')
    def _compute_is_monitoring_overdue(self):
        """Check if monitoring is overdue"""
        now = datetime.now()
        for record in self:
            if record.next_monitoring_check:
                record.is_monitoring_overdue = record.next_monitoring_check < now
            else:
                record.is_monitoring_overdue = False

    def run_monitoring_check(self):
        """Run monitoring check - implement in inheriting models"""
        raise NotImplementedError("Implement run_monitoring_check in your model")

    @api.model
    def cron_run_monitoring(self):
        """Cron job to run monitoring checks"""
        records = self.search([
            ('monitoring_enabled', '=', True),
            '|',
            ('next_monitoring_check', '<=', fields.Datetime.now()),
            ('next_monitoring_check', '=', False),
        ])

        for record in records:
            try:
                record.run_monitoring_check()
            except Exception as e:
                _logger.error(f"Monitoring check failed for {record.name}: {e}")
```

### Mixin Dependencies

When a mixin depends on fields/methods from another mixin:

```python
# custom_tower_extension/models/custom_advanced_mixin.py
from odoo import api, fields, models

class CustomAdvancedMixin(models.AbstractModel):
    """Advanced mixin that depends on notification mixin"""

    _name = "custom.advanced.mixin"
    _description = "Custom Advanced Mixin"
    # Requires custom.notification.mixin to be inherited too

    auto_notify_on_error = fields.Boolean(
        string="Auto Notify on Error",
        default=True,
        help="Automatically send notification when error occurs",
    )

    def handle_error(self, error_message):
        """Handle error with automatic notification

        Args:
            error_message (str): Error message

        Note:
            Requires custom.notification.mixin to be inherited
        """
        self.ensure_one()

        # Send notification if enabled
        if self.auto_notify_on_error and hasattr(self, 'send_notification'):
            self.send_notification(
                subject='Error Occurred',
                message=error_message,
                notification_type='error',
            )

        return {
            'type': 'ir.actions.client',
            'tag': 'display_notification',
            'params': {
                'type': 'danger',
                'title': 'Error',
                'message': error_message,
                'sticky': True,
            },
        }
```

## Best Practices

### ✅ DO:

1. **Use Abstract Models for Mixins**
   ```python
   class MyMixin(models.AbstractModel):
       _name = "my.mixin"
       _description = "My Mixin"
   ```

2. **Provide Clear Documentation**
   ```python
   class MyMixin(models.AbstractModel):
       """Mixin for XYZ functionality

       Provides:
           - Fields: field1, field2
           - Methods: method1(), method2()

       Usage:
           class MyModel(models.Model):
               _inherit = ["my.model", "my.mixin"]

       Requirements:
           - Model must have 'name' field
           - Model should inherit mail.thread for notifications
       """
       _name = "my.mixin"
   ```

3. **Use Meaningful Names**
   ```python
   # Good
   "custom.audit.mixin"
   "custom.monitoring.mixin"
   "custom.notification.mixin"

   # Avoid
   "my.mixin"
   "custom.mixin"
   ```

4. **Call super() When Overriding**
   ```python
   def write(self, vals):
       # Custom logic
       result = super().write(vals)
       # More custom logic
       return result
   ```

5. **Keep Mixins Focused**
   - One mixin = one concern
   - Don't create "god mixins" with everything

6. **Document Dependencies**
   ```python
   class MyMixin(models.AbstractModel):
       """Requires models to have:
       - 'name' field
       - 'active' field
       - Inherit mail.thread
       """
   ```

### ❌ DON'T:

1. **Don't use regular Models**
   ```python
   # Wrong
   class MyMixin(models.Model):  # Creates database table
       _name = "my.mixin"

   # Correct
   class MyMixin(models.AbstractModel):  # No database table
       _name = "my.mixin"
   ```

2. **Don't put business logic in mixins**
   - Mixins = reusable functionality
   - Business logic = in actual models

3. **Don't create circular dependencies**

4. **Don't skip super() calls**

5. **Don't make mixins too complex**

### Mixin Checklist

Before creating a mixin, ask:

- [ ] Is this functionality needed by multiple models?
- [ ] Is it truly reusable or model-specific?
- [ ] Have I documented requirements/dependencies?
- [ ] Have I tested with multiple inheriting models?
- [ ] Is the mixin name clear and descriptive?
- [ ] Does it follow single responsibility principle?

## Complete Example

Here's a complete custom mixin implementation:

```python
# custom_tower_extension/models/__init__.py
from . import custom_lifecycle_mixin
from . import cx_tower_server

# custom_tower_extension/models/custom_lifecycle_mixin.py
from odoo import api, fields, models
import logging

_logger = logging.getLogger(__name__)

class CustomLifecycleMixin(models.AbstractModel):
    """Mixin for lifecycle management

    Provides fields and methods for tracking record lifecycle:
    - Creation, activation, archival
    - State transitions
    - Lifecycle notifications

    Usage:
        class MyModel(models.Model):
            _inherit = ["my.model", "custom.lifecycle.mixin"]

            # Optionally override lifecycle methods
            def _on_lifecycle_created(self):
                super()._on_lifecycle_created()
                # Custom logic

    Requirements:
        - Model must have 'active' field
        - Model should inherit mail.thread for tracking
    """

    _name = "custom.lifecycle.mixin"
    _description = "Custom Lifecycle Mixin"

    # Lifecycle fields
    lifecycle_state = fields.Selection([
        ('draft', 'Draft'),
        ('active', 'Active'),
        ('inactive', 'Inactive'),
        ('archived', 'Archived'),
    ], string="Lifecycle State", default='draft', tracking=True)

    lifecycle_created_date = fields.Datetime(
        string="Lifecycle Created",
        default=fields.Datetime.now,
        readonly=True,
    )

    lifecycle_activated_date = fields.Datetime(
        string="Activated Date",
        readonly=True,
    )

    lifecycle_archived_date = fields.Datetime(
        string="Archived Date",
        readonly=True,
    )

    lifecycle_days_active = fields.Integer(
        string="Days Active",
        compute='_compute_lifecycle_days_active',
    )

    @api.depends('lifecycle_activated_date', 'lifecycle_archived_date')
    def _compute_lifecycle_days_active(self):
        """Compute how many days record has been active"""
        for record in self:
            if not record.lifecycle_activated_date:
                record.lifecycle_days_active = 0
                continue

            end_date = record.lifecycle_archived_date or fields.Datetime.now()
            delta = end_date - record.lifecycle_activated_date
            record.lifecycle_days_active = delta.days

    @api.model_create_multi
    def create(self, vals_list):
        """Override create to track lifecycle"""
        records = super().create(vals_list)
        for record in records:
            record._on_lifecycle_created()
        return records

    def action_activate(self):
        """Activate record"""
        self.write({
            'lifecycle_state': 'active',
            'lifecycle_activated_date': fields.Datetime.now(),
            'active': True,
        })
        self._on_lifecycle_activated()

    def action_deactivate(self):
        """Deactivate record"""
        self.write({
            'lifecycle_state': 'inactive',
            'active': False,
        })
        self._on_lifecycle_deactivated()

    def toggle_active(self):
        """Override toggle_active to track lifecycle"""
        for record in self:
            if record.active:
                record.action_deactivate()
            else:
                record.action_activate()

    # Lifecycle hooks (override in inheriting models)
    def _on_lifecycle_created(self):
        """Called when record is created"""
        _logger.info(f"Lifecycle: {self._name} {self.id} created")

    def _on_lifecycle_activated(self):
        """Called when record is activated"""
        _logger.info(f"Lifecycle: {self._name} {self.id} activated")

    def _on_lifecycle_deactivated(self):
        """Called when record is deactivated"""
        _logger.info(f"Lifecycle: {self._name} {self.id} deactivated")
```

```python
# custom_tower_extension/models/cx_tower_server.py
from odoo import models
import logging

_logger = logging.getLogger(__name__)

class CxTowerServer(models.Model):
    _inherit = ["cx.tower.server", "custom.lifecycle.mixin"]

    def _on_lifecycle_activated(self):
        """Override to add custom notification"""
        super()._on_lifecycle_activated()

        # Send notification (if notification mixin is also inherited)
        if hasattr(self, 'send_notification'):
            self.send_notification(
                subject=f"Server {self.name} Activated",
                message=f"Server {self.name} has been activated",
                notification_type='info',
            )

        _logger.info(f"Server {self.name} activated successfully")
```

## Next Steps

- Review [Deployment Guide](../09-deployment-operations/01-deployment-guide.md)
- Explore [Version Upgrades](../13-potential-upgrade/01-odoo-version-upgrades.md)
- Check [Development Workflows](../08-development-workflows/)

---

**Remember**: Mixins are for reusable functionality. Keep them focused, documented, and tested!

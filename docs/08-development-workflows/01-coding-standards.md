---
title: Coding Standards
description: Coding standards and conventions for Cetmix Tower development
category: development-workflows
order: 2
---

# Coding Standards

Cetmix Tower follows Odoo Community Association (OCA) coding standards with project-specific conventions to ensure code quality and consistency.

## Table of Contents

- [Python Standards](#python-standards)
- [XML Standards](#xml-standards)
- [JavaScript Standards](#javascript-standards)
- [Naming Conventions](#naming-conventions)
- [Documentation Standards](#documentation-standards)
- [Pre-commit Hooks](#pre-commit-hooks)
- [Linting Tools](#linting-tools)

## Python Standards

### PEP 8 Compliance

Follow [PEP 8](https://pep8.org/) Python style guide with Odoo-specific adaptations.

#### Line Length

- **Maximum 88 characters** (Ruff/Black default)
- **Maximum 79 characters** for docstrings and comments (PEP 8)

```python
# Good
result = some_function(
    parameter1, parameter2, parameter3, parameter4
)

# Bad - too long
result = some_function(parameter1, parameter2, parameter3, parameter4, parameter5, parameter6)
```

#### Imports

**Order** (enforced by Ruff):
1. Future imports
2. Standard library
3. Third-party libraries
4. Odoo core (`odoo`)
5. Odoo addons (`odoo.addons`)
6. Local application/library specific

**Example**:
```python
# Copyright statement
# License

# Future (if needed)
from __future__ import annotations

# Standard library
import logging
from datetime import datetime, timedelta

# Third-party
import requests
from lxml import etree

# Odoo
from odoo import _, api, fields, models
from odoo.exceptions import UserError, ValidationError
from odoo.tools import config

# Odoo addons
from odoo.addons.base.models.res_partner import Partner

# Local
from .constants import COMMAND_STATUS_CODES
from .tools import generate_random_id

_logger = logging.getLogger(__name__)
```

**Rules**:
- One import per line
- No wildcard imports (`from module import *`)
- Absolute imports preferred
- Group by type, alphabetically within groups

#### Indentation

- **4 spaces** (never tabs)
- Continuation lines should be aligned

```python
# Good - aligned
result = some_long_function_name(
    arg1, arg2,
    arg3, arg4
)

# Good - hanging indent
def long_function_name(
        var_one, var_two, var_three,
        var_four):
    print(var_one)

# Bad - not aligned
result = some_long_function_name(arg1, arg2,
    arg3, arg4)
```

#### Whitespace

```python
# Good
spam(ham[1], {eggs: 2})
foo = (0,)
if x == 4:
    print(x, y)
    x, y = y, x

# Bad
spam( ham[ 1 ], { eggs: 2 } )
foo = (0, )
if x == 4 :
    print(x , y)
    x , y = y , x
```

#### Quotes

- **Single quotes** for strings by default
- **Double quotes** for messages/user-facing strings
- **Triple quotes** for docstrings

```python
# Good
name = 'example'
message = _("This is a user message")
"""This is a docstring."""

# Acceptable
name = "example"  # Consistency within file matters most
```

### Odoo-Specific Conventions

#### Model Definitions

```python
class CxTowerServer(models.Model):
    """Server entity for managing remote systems.

    This model represents a server that can be managed via SSH.
    It includes connection settings, authentication, and operations.
    """

    _name = "cx.tower.server"
    _inherit = ["mail.thread", "mail.activity.mixin"]
    _description = "Cetmix Tower Server"
    _order = "name asc"

    # Private attributes first
    _check_company_auto = True

    # Then fields in logical order:
    # 1. Basic fields
    name = fields.Char(required=True, index=True)
    active = fields.Boolean(default=True)

    # 2. Relational fields
    partner_id = fields.Many2one(
        comodel_name="res.partner",
        string="Related Partner",
        ondelete="restrict",
    )

    # 3. Computed fields
    server_status = fields.Selection(
        selection="_selection_status",
        compute="_compute_status",
        store=True,
    )

    # Then methods in order:
    # 1. Default methods
    @api.model
    def _selection_status(self):
        """Return status selection options."""
        return [
            ('running', 'Running'),
            ('stopped', 'Stopped'),
        ]

    # 2. Compute methods
    @api.depends('command_log_ids', 'command_log_ids.is_running')
    def _compute_status(self):
        """Compute server status based on running commands."""
        for record in self:
            record.server_status = # ...

    # 3. Constraint methods
    @api.constrains('ip_v4_address', 'ip_v6_address')
    def _constraint_ip_address(self):
        """Ensure at least one IP address is provided."""
        for record in self:
            if not record.ip_v4_address and not record.ip_v6_address:
                raise ValidationError(_("Provide IPv4 or IPv6 address"))

    # 4. CRUD methods
    @api.model_create_multi
    def create(self, vals_list):
        """Override create to validate SSH settings."""
        # Implementation
        return super().create(vals_list)

    def write(self, vals):
        """Override write to handle field updates."""
        # Implementation
        return super().write(vals)

    def unlink(self):
        """Override unlink to run cleanup."""
        # Implementation
        return super().unlink()

    # 5. Action methods
    def action_open_command_logs(self):
        """Open command logs for this server."""
        return {
            'type': 'ir.actions.act_window',
            'name': _('Command Logs'),
            'res_model': 'cx.tower.command.log',
            'view_mode': 'tree,form',
            'domain': [('server_id', '=', self.id)],
        }

    # 6. Business logic methods
    def run_command(self, command, **kwargs):
        """Run a command on this server.

        Args:
            command: cx.tower.command record
            **kwargs: Additional arguments

        Returns:
            None or dict depending on context
        """
        self.ensure_one()
        # Implementation

    # 7. Private methods
    def _get_ssh_client(self, raise_on_error=False):
        """Get SSH client for this server.

        Private method indicated by leading underscore.

        Args:
            raise_on_error (bool): Raise exception on error

        Returns:
            SSHManager or False
        """
        # Implementation
```

#### Field Definitions

```python
# Basic field with all common attributes
name = fields.Char(
    string="Server Name",  # Display name
    required=True,          # Validation
    index=True,             # Database index
    copy=False,             # Don't copy on duplicate
    tracking=True,          # Track changes
    help="Name of the server"  # Tooltip
)

# Many2one with full specification
partner_id = fields.Many2one(
    comodel_name="res.partner",
    string="Partner",
    ondelete="restrict",  # Database constraint
    domain=[('is_company', '=', True)],
    context={'default_is_company': True},
    help="Related partner"
)

# Many2many with explicit relation
tag_ids = fields.Many2many(
    comodel_name="cx.tower.tag",
    relation="cx_tower_server_tag_rel",
    column1="server_id",
    column2="tag_id",
    string="Tags",
)

# Computed field with dependencies
server_count = fields.Integer(
    string="Server Count",
    compute="_compute_server_count",
    store=True,  # Store in database
    readonly=True,
)

# Selection field
status = fields.Selection(
    selection=[
        ('running', 'Running'),
        ('stopped', 'Stopped'),
    ],
    default='stopped',
    required=True,
    string="Status",
)

# Selection with method
status = fields.Selection(
    selection=lambda self: self._selection_status(),
    string="Status",
)
```

#### Method Decorators

```python
# Model method (no record)
@api.model
def default_get(self, fields_list):
    """Override default_get."""
    res = super().default_get(fields_list)
    return res

# Multi-record method (works on recordsets)
def action_confirm(self):
    """Confirm multiple records."""
    for record in self:
        record.state = 'confirmed'

# Single-record method (use ensure_one)
def get_report_data(self):
    """Get report data for single record."""
    self.ensure_one()
    return {'name': self.name}

# Depends decorator for compute
@api.depends('line_ids', 'line_ids.amount')
def _compute_total(self):
    """Compute total from lines."""
    for record in self:
        record.total = sum(record.line_ids.mapped('amount'))

# Constrains decorator
@api.constrains('start_date', 'end_date')
def _check_dates(self):
    """Ensure start date is before end date."""
    for record in self:
        if record.start_date > record.end_date:
            raise ValidationError(_("Start date must be before end date"))

# Onchange decorator
@api.onchange('partner_id')
def _onchange_partner(self):
    """Update address when partner changes."""
    if self.partner_id:
        self.address = self.partner_id.address
```

### Ruff Configuration

**File**: `/home/user/cetmix-tower/.ruff.toml`

```toml
target-version = "py310"
fix = true

[lint]
extend-select = [
    "B",      # flake8-bugbear
    "C90",    # mccabe complexity
    "E501",   # line too long
    "I",      # isort
    "UP",     # pyupgrade
]
extend-safe-fixes = ["UP008"]
exclude = ["setup/*"]

[format]
exclude = ["setup/*"]

[per-file-ignores]
"__init__.py" = ["F401", "I001"]  # unused imports OK
"__manifest__.py" = ["B018"]       # useless expression OK

[isort]
section-order = [
    "future",
    "standard-library",
    "third-party",
    "odoo",
    "odoo-addons",
    "first-party",
    "local-folder"
]

[isort.sections]
"odoo" = ["odoo"]
"odoo-addons" = ["odoo.addons"]

[mccabe]
max-complexity = 16
```

## XML Standards

### View Definitions

```xml
<?xml version="1.0" encoding="utf-8" ?>
<odoo>
    <!-- Form View -->
    <record id="view_cx_tower_server_form" model="ir.ui.view">
        <field name="name">cx.tower.server.form</field>
        <field name="model">cx.tower.server</field>
        <field name="arch" type="xml">
            <form string="Server">
                <header>
                    <button
                        name="action_test_connection"
                        string="Test Connection"
                        type="object"
                        class="oe_highlight"
                    />
                    <field name="status" widget="statusbar" />
                </header>
                <sheet>
                    <div class="oe_title">
                        <label for="name" string="Server Name" />
                        <h1>
                            <field name="name" placeholder="e.g. Production Web Server" />
                        </h1>
                    </div>
                    <group>
                        <group name="connection" string="Connection">
                            <field name="ip_v4_address" />
                            <field name="ssh_port" />
                            <field name="ssh_username" />
                        </group>
                        <group name="authentication" string="Authentication">
                            <field name="ssh_auth_mode" widget="radio" />
                            <field
                                name="ssh_password"
                                password="True"
                                invisible="ssh_auth_mode != 'p'"
                            />
                            <field
                                name="ssh_key_id"
                                invisible="ssh_auth_mode != 'k'"
                            />
                        </group>
                    </group>
                    <notebook>
                        <page name="commands" string="Commands">
                            <field name="command_log_ids">
                                <tree>
                                    <field name="create_date" />
                                    <field name="command_id" />
                                    <field name="command_status" />
                                </tree>
                            </field>
                        </page>
                    </notebook>
                </sheet>
                <div class="oe_chatter">
                    <field name="message_follower_ids" />
                    <field name="activity_ids" />
                    <field name="message_ids" />
                </div>
            </form>
        </field>
    </record>

    <!-- Tree View -->
    <record id="view_cx_tower_server_tree" model="ir.ui.view">
        <field name="name">cx.tower.server.tree</field>
        <field name="model">cx.tower.server</field>
        <field name="arch" type="xml">
            <tree string="Servers">
                <field name="name" />
                <field name="ip_v4_address" />
                <field name="status" />
                <field name="partner_id" optional="show" />
                <field name="tag_ids" widget="many2many_tags" optional="hide" />
            </tree>
        </field>
    </record>

    <!-- Search View -->
    <record id="view_cx_tower_server_search" model="ir.ui.view">
        <field name="name">cx.tower.server.search</field>
        <field name="model">cx.tower.server</field>
        <field name="arch" type="xml">
            <search string="Servers">
                <field name="name" />
                <field name="ip_v4_address" />
                <field name="partner_id" />
                <field name="tag_ids" />
                <filter
                    name="active_servers"
                    string="Active"
                    domain="[('active', '=', True)]"
                />
                <filter
                    name="running"
                    string="Running"
                    domain="[('status', '=', 'running')]"
                />
                <group expand="0" string="Group By">
                    <filter
                        name="group_partner"
                        string="Partner"
                        context="{'group_by': 'partner_id'}"
                    />
                    <filter
                        name="group_status"
                        string="Status"
                        context="{'group_by': 'status'}"
                    />
                </group>
            </search>
        </field>
    </record>

    <!-- Action -->
    <record id="action_cx_tower_server" model="ir.actions.act_window">
        <field name="name">Servers</field>
        <field name="res_model">cx.tower.server</field>
        <field name="view_mode">tree,form</field>
        <field name="context">{}</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create your first server
            </p>
        </field>
    </record>

    <!-- Menu -->
    <menuitem
        id="menu_cx_tower_server"
        name="Servers"
        parent="menu_cx_tower_root"
        action="action_cx_tower_server"
        sequence="10"
    />
</odoo>
```

### XML Formatting

- **Indent**: 4 spaces
- **Attribute order**: id, name, model, other attributes
- **One attribute per line** for readability
- **Close tags** on same line for short content
- **Empty elements**: Use self-closing tags `<tag />`

### Security Rules

**File**: `security/ir.model.access.csv`

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_cx_tower_server_user,cx.tower.server.user,model_cx_tower_server,group_user,1,0,0,0
access_cx_tower_server_manager,cx.tower.server.manager,model_cx_tower_server,group_manager,1,1,1,1
```

**Record rules**:
```xml
<record id="cx_tower_server_user_rule" model="ir.rule">
    <field name="name">Server: User Access</field>
    <field name="model_id" ref="model_cx_tower_server" />
    <field name="domain_force">
        ['|',
            ('user_ids', 'in', [user.id]),
            ('manager_ids', 'in', [user.id])
        ]
    </field>
    <field name="groups" eval="[(4, ref('group_user'))]" />
</record>
```

## JavaScript Standards

### ESLint Configuration

**File**: `/home/user/cetmix-tower/.eslintrc.yml`

```yaml
env:
  browser: true
  es6: true
extends: "eslint:recommended"
rules:
  indent: ["error", 4]
  linebreak-style: ["error", "unix"]
  quotes: ["error", "double"]
  semi: ["error", "always"]
```

### OWL Components

```javascript
/** @odoo-module **/

import { Component } from "@odoo/owl";
import { registry } from "@web/core/registry";

export class ServerStatusWidget extends Component {
    static template = "cetmix_tower.ServerStatusWidget";
    static props = {
        record: Object,
        name: String,
    };

    /**
     * Get status color based on server status.
     * @returns {string} CSS class for status color
     */
    get statusColor() {
        const status = this.props.record.data[this.props.name];
        const colors = {
            running: "text-success",
            stopped: "text-danger",
            starting: "text-warning",
        };
        return colors[status] || "text-muted";
    }

    /**
     * Handle status click event.
     */
    onStatusClick() {
        this.env.model.notification.add(
            `Server status: ${this.props.record.data[this.props.name]}`,
            {type: "info"}
        );
    }
}

registry.category("fields").add("server_status_widget", ServerStatusWidget);
```

## Naming Conventions

### Python

```python
# Classes: PascalCase
class CxTowerServer(models.Model):
    pass

# Functions/Methods: snake_case
def run_command(self):
    pass

# Constants: UPPER_SNAKE_CASE
MAX_RETRY_COUNT = 3
DEFAULT_TIMEOUT = 30

# Private methods: _leading_underscore
def _compute_status(self):
    pass

# Module-level private: _leading_underscore
_logger = logging.getLogger(__name__)

# Protected (inherit but don't override): no prefix
# Public: no prefix
```

### Models

```python
# Technical name: lowercase with dots
_name = "cx.tower.server"
_name = "cx.tower.command.log"

# Inherit: match original or extend
_inherit = "mail.thread"
_inherit = ["mail.thread", "mail.activity.mixin"]

# Table name (auto-generated): underscores
# cx.tower.server → cx_tower_server
```

### XML IDs

```xml
<!-- Views: view_<model>_<type> -->
<record id="view_cx_tower_server_form" model="ir.ui.view">

<!-- Actions: action_<model> or action_<purpose> -->
<record id="action_cx_tower_server" model="ir.actions.act_window">

<!-- Menus: menu_<name> -->
<menuitem id="menu_cx_tower_server" />

<!-- Security groups: group_<name> -->
<record id="group_manager" model="res.groups">

<!-- Data: <module>.<identifier> -->
<record id="cetmix_tower_server.default_os" model="cx.tower.os">
```

### Files and Directories

```
# Modules: lowercase_with_underscores
cetmix_tower_server/
cetmix_tower_webhook/

# Python files: lowercase_with_underscores
cx_tower_server.py
cx_tower_command.py

# XML files: lowercase_with_underscores
server_views.xml
command_views.xml

# Directories: lowercase
models/
views/
static/
```

## Documentation Standards

### Module Documentation

**README.md** (auto-generated from readme/ directory):
```markdown
# Cetmix Tower Server

Server management and SSH automation for Odoo.

## Features

* SSH connection management
* Command execution
* Flight plans (command sequences)
* File synchronization
* Variable management

## Configuration

1. Install the module
2. Go to Tower > Configuration > Servers
3. Create a new server
4. Configure SSH connection

## Usage

### Running a Command

...
```

### Docstrings

```python
def run_command(self, command, path=None, sudo=None, **kwargs):
    """Run a command on the server.

    This is the main function to use for running commands.
    It renders command code, creates log record and calls command runner.

    Args:
        command (cx.tower.command): Command record to execute
        path (str, optional): Directory where command is run.
            Defaults to command's default path.
        sudo (bool, optional): Use sudo for execution.
            Defaults to server's sudo setting.
        **kwargs: Additional arguments:
            - log (dict): Values passed to command logger
            - key (dict): Values passed to key parser
            - variable_values (dict): Custom variable values

    Returns:
        None: When command log is created
        dict: When no_command_log context is set:
            {
                'status': int,      # 0 for success
                'response': str,    # Command output
                'error': str,       # Error message if any
            }

    Raises:
        ValidationError: If command is not compatible with server
        UserError: If configuration is invalid

    Example:
        >>> server = env['cx.tower.server'].browse(1)
        >>> command = env['cx.tower.command'].browse(1)
        >>> server.run_command(command)
        >>>
        >>> # With custom variables
        >>> server.run_command(
        ...     command,
        ...     variable_values={'version': '1.2.3'}
        ... )

    Note:
        Set 'no_command_log' context key to True to disable
        log creation and return results directly.
    """
    self.ensure_one()
    # Implementation...
```

### Inline Comments

```python
# Good - Explain WHY, not WHAT
# Skip validation if importing from YAML
if not self.env.context.get('skip_validation'):
    self._validate_settings()

# Bad - States obvious
# Increment counter
counter += 1

# Good - Complex logic explanation
# Commands run with sudo are split on '&&' to ensure
# each part runs with sudo privileges separately.
# This prevents privilege escalation issues.
if sudo and '&&' in command_code:
    commands = command_code.split('&&')
```

## Pre-commit Hooks

**File**: `/home/user/cetmix-tower/.pre-commit-config.yaml`

### Configured Hooks

1. **Prettier** - Format XML, JSON, YAML
2. **ESLint** - Lint JavaScript
3. **Ruff** - Lint and format Python
4. **Pylint-Odoo** - Odoo-specific checks
5. **OCA Checks** - Module validation
6. **Whool** - Setup tools

### Running Hooks

```bash
# Install hooks
pre-commit install

# Run on all files
pre-commit run --all-files

# Run specific hook
pre-commit run prettier --all-files

# Skip hooks (avoid if possible)
git commit --no-verify

# Update hooks
pre-commit autoupdate
```

## Linting Tools

### Ruff

```bash
# Check all Python files
ruff check .

# Auto-fix issues
ruff check --fix .

# Format code
ruff format .

# Check specific file
ruff check cetmix_tower_server/models/cx_tower_server.py
```

### Pylint

```bash
# Run with optional checks
pylint --rcfile=.pylintrc cetmix_tower_server/

# Run mandatory checks only
pylint --rcfile=.pylintrc-mandatory cetmix_tower_server/

# Check specific file
pylint --rcfile=.pylintrc cetmix_tower_server/models/cx_tower_server.py
```

### ESLint

```bash
# Check JavaScript
eslint static/src/**/*.js

# Auto-fix
eslint --fix static/src/**/*.js
```

### Prettier

```bash
# Format XML
prettier --write "**/*.xml"

# Check formatting
prettier --check "**/*.{xml,json,yaml}"
```

## Common Issues and Solutions

### Import Order

**Problem**: Ruff/isort complaining about import order

**Solution**: Ensure imports follow the configured order:
```python
# Standard library
import logging

# Odoo
from odoo import models, fields

# Local
from .constants import STATUS_CODES
```

### Line Too Long

**Problem**: Line exceeds 88 characters

**Solution**: Break into multiple lines:
```python
# Before
result = some_function(param1, param2, param3, param4, param5)

# After
result = some_function(
    param1, param2, param3,
    param4, param5
)
```

### Trailing Whitespace

**Problem**: Pre-commit fails on trailing whitespace

**Solution**: Configure editor to remove on save or run:
```bash
pre-commit run trailing-whitespace --all-files
```

## Best Practices Summary

1. ✅ Run pre-commit hooks before committing
2. ✅ Write docstrings for all public methods
3. ✅ Follow PEP 8 and Odoo conventions
4. ✅ Use meaningful variable names
5. ✅ Keep functions small and focused
6. ✅ Add comments for complex logic
7. ✅ Write tests for new code
8. ✅ Update README when adding features
9. ✅ Use type hints where helpful
10. ✅ Review your own code before PR

## Related Documentation

- [Testing Guidelines](02-testing-guidelines.md)
- [Git Workflow](03-git-workflow.md)
- [Development Workflows](README.md)
- [API References](../06-api-references/README.md)

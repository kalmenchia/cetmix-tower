---
title: "Models and ORM"
section: "07-technical-guides"
document_id: "TG-001"
version: "17.0"
audience: "Developers"
last_updated: "2025-11-16"
tags: ["models", "orm", "mixins", "database", "inheritance"]
---

# Models and ORM

This guide provides comprehensive documentation of Cetmix Tower's data model architecture, including all major models, relationships, mixins, and ORM patterns.

## Table of Contents

1. [Model Architecture Overview](#model-architecture-overview)
2. [Core Models](#core-models)
3. [Mixin System](#mixin-system)
4. [Model Relationships](#model-relationships)
5. [ORM Patterns](#orm-patterns)
6. [Database Schema](#database-schema)
7. [Performance Optimization](#performance-optimization)

---

## Model Architecture Overview

Cetmix Tower uses a sophisticated model architecture built on Odoo's ORM, leveraging multiple inheritance patterns and mixins to provide flexible, secure, and maintainable code.

### Design Principles

**1. Separation of Concerns**
- Business logic separated into focused models
- Cross-cutting concerns handled by mixins
- Clear responsibility boundaries

**2. Reusability**
- Common functionality extracted into mixins
- Template-based record creation
- Inheritance for extensibility

**3. Security First**
- Sensitive data stored in vault
- Access control at multiple levels
- Field-level security enforcement

**4. Performance**
- Efficient database queries
- Strategic use of caching
- Optimized relationship definitions

### Module Location

All core models are located in:
```
/home/user/cetmix-tower/cetmix_tower_server/models/
```

### Model List Overview

**Core Business Models:**
- `cx.tower.server` - Server management
- `cx.tower.command` - Command definitions
- `cx.tower.plan` - Flight plan orchestration
- `cx.tower.plan.line` - Flight plan steps
- `cx.tower.file` - File management
- `cx.tower.variable` - Variable definitions

**Configuration Models:**
- `cx.tower.os` - Operating system definitions
- `cx.tower.tag` - Organization tags
- `cx.tower.server.template` - Server templates
- `cx.tower.file.template` - File templates
- `cx.tower.variable.option` - Variable options

**Execution & Logging:**
- `cx.tower.command.log` - Command execution logs
- `cx.tower.plan.log` - Flight plan execution logs
- `cx.tower.server.log` - Server activity logs

**Security & Access:**
- `cx.tower.key` - SSH keys and secrets
- `cx.tower.key.value` - Key-value pairs
- `cx.tower.vault` - Secure secret storage
- `cx.tower.variable.value` - Variable value assignments

**Utilities:**
- `cx.tower.shortcut` - Quick access shortcuts
- `cx.tower.scheduled.task` - Scheduled operations
- `cx.tower.plan.line.action` - Conditional actions

**Mixins (Abstract Models):**
- `cx.tower.reference.mixin`
- `cx.tower.access.mixin`
- `cx.tower.access.role.mixin`
- `cx.tower.vault.mixin`
- `cx.tower.template.mixin`
- `cx.tower.variable.mixin`
- `cx.tower.key.mixin`
- `cx.tower.custom.variable.value.mixin`

---

## Core Models

### cx.tower.server

**Purpose:** Represents a physical or virtual server managed by Tower.

**File:** `/home/user/cetmix-tower/cetmix_tower_server/models/cx_tower_server.py`

**Inheritance:**
```python
_name = "cx.tower.server"
_inherit = [
    "cx.tower.access.role.mixin",    # Role-based access control
    "cx.tower.variable.mixin",       # Variable management
    "cx.tower.reference.mixin",      # Unique reference generation
    "mail.thread",                    # Messaging/activity tracking
    "mail.activity.mixin",           # Activity management
    "cx.tower.vault.mixin",          # Secure secret storage
]
```

**Key Fields:**

```python
# Basic Information
name = fields.Char(required=True)              # Server name
reference = fields.Char(index=True)            # Unique identifier
active = fields.Boolean(default=True)          # Archive status
color = fields.Integer()                       # UI color coding
partner_id = fields.Many2one('res.partner')   # Related partner

# Connection Details
ip_address = fields.Char()                     # IP or domain
ssh_port = fields.Integer(default=22)          # SSH port
ssh_username = fields.Char()                   # SSH user
ssh_password = fields.Char()                   # SSH password (vaulted)
ssh_auth_mode = fields.Selection([...])        # Auth method
host_key = fields.Text()                       # SSH host key (vaulted)

# Server Properties
os_id = fields.Many2one('cx.tower.os')        # Operating system
tag_ids = fields.Many2many('cx.tower.tag')    # Organization tags
status = fields.Selection([...])               # Connection status

# Relationships
command_ids = fields.Many2many('cx.tower.command')
file_ids = fields.One2many('cx.tower.file')
template_id = fields.Many2one('cx.tower.server.template')
```

**Secret Fields (Stored in Vault):**
```python
SECRET_FIELDS = ["ssh_password", "host_key"]
```

**Important Methods:**

```python
def _get_ssh_client(self, raise_on_error=False):
    """
    Get SSH connection for this server.

    Returns:
        SSHConnection: Active SSH connection object

    Raises:
        UserError: If connection fails and raise_on_error=True
    """

@ensure_ssh_disconnect
def execute_command(self, command_code, timeout=None):
    """
    Execute command on server via SSH.
    Decorator ensures SSH cleanup after transaction.

    Args:
        command_code (str): Command to execute
        timeout (int): Execution timeout in seconds

    Returns:
        dict: {
            'stdout': str,
            'stderr': str,
            'exit_code': int,
            'status': str
        }
    """

def test_connection(self):
    """Test SSH connectivity to server."""
```

**Usage Example:**

```python
# Create server
server = env['cx.tower.server'].create({
    'name': 'Production Web Server',
    'ip_address': '192.168.1.100',
    'ssh_port': 22,
    'ssh_username': 'admin',
    'ssh_password': 'secret_password',  # Automatically vaulted
    'os_id': os_ubuntu.id,
    'tag_ids': [(6, 0, [prod_tag.id, web_tag.id])],
})

# Test connection
server.test_connection()

# Execute command
result = server.execute_command('uptime', timeout=30)
print(result['stdout'])
```

---

### cx.tower.command

**Purpose:** Defines commands that can be executed on servers.

**File:** `/home/user/cetmix-tower/cetmix_tower_server/models/cx_tower_command.py`

**Inheritance:**
```python
_name = "cx.tower.command"
_inherit = [
    "cx.tower.template.mixin",       # Template rendering
    "cx.tower.reference.mixin",      # Reference management
    "cx.tower.access.mixin",         # Access level control
    "cx.tower.access.role.mixin",    # Role-based access
    "cx.tower.key.mixin",            # Secret/key management
]
```

**Key Fields:**

```python
# Basic Information
name = fields.Char(required=True)              # Command name
reference = fields.Char(index=True)            # Unique identifier
active = fields.Boolean(default=True)
code = fields.Text()                           # Command code/script

# Command Configuration
action = fields.Selection([...])               # Command type
timeout = fields.Integer()                     # Execution timeout
allow_parallel_run = fields.Boolean()          # Parallel execution

# Compatibility
server_ids = fields.Many2many('cx.tower.server')  # Allowed servers
os_ids = fields.Many2many('cx.tower.os')          # Compatible OS
tag_ids = fields.Many2many('cx.tower.tag')        # Organization tags

# Template Rendering (from mixin)
variable_ids = fields.Many2many('cx.tower.variable')  # Used variables
secret_ids = fields.Many2many('cx.tower.key')         # Used secrets

# Access Control (from mixin)
access_level = fields.Selection([...])         # User/Manager/Root
```

**Command Types (action field):**

```python
def _selection_action(self):
    return [
        ('s', 'Shell'),              # Shell command (bash, sh)
        ('p', 'Python'),             # Python script
        ('f', 'Flight Plan'),        # Run another flight plan
        ('ft', 'File Template'),     # Deploy file template
        # Extended by other modules (Git, etc.)
    ]
```

**Important Methods:**

```python
def run(self, server_ids=None, variable_values=None):
    """
    Run command on specified servers.

    Args:
        server_ids (list): Server IDs to run on
        variable_values (dict): Variable value overrides

    Returns:
        cx.tower.command.log recordset
    """

def _render_command_code(self, variable_values):
    """
    Render command code with variable substitution.

    Args:
        variable_values (dict): {variable_ref: value}

    Returns:
        str: Rendered command code
    """

def _check_compatibility(self, server):
    """
    Check if command can run on server.

    Args:
        server (cx.tower.server): Server to check

    Returns:
        bool: True if compatible

    Raises:
        ValidationError: If incompatible
    """
```

**Usage Example:**

```python
# Create command
command = env['cx.tower.command'].create({
    'name': 'Restart Nginx',
    'reference': 'restart_nginx',
    'action': 's',  # Shell
    'code': 'sudo systemctl restart nginx',
    'timeout': 300,
    'os_ids': [(6, 0, [ubuntu_os.id, debian_os.id])],
    'tag_ids': [(6, 0, [web_tag.id])],
    'access_level': '2',  # Manager
})

# Run command
command.run(server_ids=[server.id])

# Command with variables
deploy_cmd = env['cx.tower.command'].create({
    'name': 'Deploy Application',
    'code': '''
        cd {{ app_path }}
        git checkout {{ version }}
        docker-compose up -d
    ''',
    'variable_ids': [(6, 0, [app_path_var.id, version_var.id])],
})

# Run with variable values
deploy_cmd.run(
    server_ids=[server.id],
    variable_values={
        'app_path': '/opt/myapp',
        'version': 'v2.5.0',
    }
)
```

---

### cx.tower.plan

**Purpose:** Orchestrates multiple commands in sequence (Flight Plans).

**File:** `/home/user/cetmix-tower/cetmix_tower_server/models/cx_tower_plan.py`

**Inheritance:**
```python
_name = "cx.tower.plan"
_inherit = [
    "cx.tower.reference.mixin",
    "cx.tower.access.mixin",
    "cx.tower.access.role.mixin",
]
```

**Key Fields:**

```python
# Basic Information
name = fields.Char(required=True)
reference = fields.Char(index=True)
active = fields.Boolean(default=True)
color = fields.Integer()

# Plan Configuration
allow_parallel_run = fields.Boolean()
on_error_action = fields.Selection([
    ('e', 'Exit with command exit code'),
    ('ec', 'Exit with custom exit code'),
    ('n', 'Run next command'),
])
custom_exit_code = fields.Integer()

# Plan Structure
line_ids = fields.One2many(
    'cx.tower.plan.line',
    'plan_id',
    copy=True
)

# Relationships
server_ids = fields.Many2many('cx.tower.server')
tag_ids = fields.Many2many('cx.tower.tag')
command_ids = fields.Many2many(
    'cx.tower.command',
    compute='_compute_command_ids',
    store=True
)
```

**Important Methods:**

```python
def run(self, server_ids=None, variable_values=None):
    """
    Execute flight plan on servers.

    Args:
        server_ids (list): Target servers
        variable_values (dict): Variable overrides

    Returns:
        cx.tower.plan.log: Plan execution log
    """

def _execute_plan_lines(self, server, log_id):
    """
    Execute plan lines sequentially.

    Args:
        server (cx.tower.server): Target server
        log_id (cx.tower.plan.log): Execution log

    Returns:
        dict: Execution results
    """
```

**Usage Example:**

```python
# Create flight plan
plan = env['cx.tower.plan'].create({
    'name': 'Deploy Web Application',
    'reference': 'deploy_web_app',
    'on_error_action': 'e',  # Exit on error
    'line_ids': [
        (0, 0, {
            'sequence': 10,
            'command_id': stop_app_cmd.id,
        }),
        (0, 0, {
            'sequence': 20,
            'command_id': pull_code_cmd.id,
        }),
        (0, 0, {
            'sequence': 30,
            'command_id': start_app_cmd.id,
        }),
    ],
})

# Run plan
plan.run(server_ids=[server.id])
```

---

### cx.tower.plan.line

**Purpose:** Individual steps within a flight plan.

**File:** `/home/user/cetmix-tower/cetmix_tower_server/models/cx_tower_plan_line.py`

**Inheritance:**
```python
_name = "cx.tower.plan.line"
_inherit = ["cx.tower.reference.mixin"]
_order = "sequence, plan_id"
```

**Key Fields:**

```python
sequence = fields.Integer(default=10)          # Execution order
name = fields.Char(related='command_id.name')  # Line name
plan_id = fields.Many2one('cx.tower.plan', ondelete='cascade')
command_id = fields.Many2one('cx.tower.command', required=True)

# Execution Configuration
path = fields.Char()                           # Execution path override
use_sudo = fields.Boolean()                    # Force sudo usage
condition = fields.Char()                      # Conditional execution

# Actions on Results
action_ids = fields.One2many(
    'cx.tower.plan.line.action',
    'line_id'
)
```

**Conditional Execution:**

Lines can have conditions that determine if they should execute:

```python
# Example condition
condition = "{{ odoo_version }} == '17.0'"
```

**Usage Example:**

```python
# Plan line with condition
line = env['cx.tower.plan.line'].create({
    'plan_id': plan.id,
    'sequence': 25,
    'command_id': migrate_db_cmd.id,
    'condition': "{{ upgrade_mode }} == 'full'",
    'action_ids': [
        (0, 0, {
            'action_type': 's',  # On success
            'next_action': 'n',  # Run next
        }),
        (0, 0, {
            'action_type': 'f',  # On failure
            'next_action': 'e',  # Exit
            'exit_code': 1,
        }),
    ],
})
```

---

### cx.tower.file

**Purpose:** Manages files synchronized between Tower and servers.

**File:** `/home/user/cetmix-tower/cetmix_tower_server/models/cx_tower_file.py`

**Inheritance:**
```python
_name = "cx.tower.file"
_inherit = [
    "cx.tower.template.mixin",
    "cx.tower.reference.mixin",
    "mail.thread",
    "mail.activity.mixin",
    "cx.tower.key.mixin",
]
```

**Key Fields:**

```python
# File Identity
name = fields.Char()                           # Filename
reference = fields.Char(index=True)
active = fields.Boolean(default=True)

# File Location
server_dir = fields.Char(required=True)        # Target directory
source = fields.Selection([
    ('tower', 'Tower'),      # Push from Tower to Server
    ('server', 'Server'),    # Pull from Server to Tower
])
file_type = fields.Selection([
    ('code', 'Code'),        # Text file
    ('binary', 'Binary'),    # Binary file
])

# File Content
code = fields.Text()                           # Text content
file_binary = fields.Binary()                  # Binary content

# Synchronization
auto_sync = fields.Boolean()
auto_sync_interval = fields.Selection([...])
sync_date_next = fields.Datetime()
sync_date_last = fields.Datetime(readonly=True)

# Template Support
template_id = fields.Many2one('cx.tower.file.template')
rendered_name = fields.Char(compute='_compute_render')
rendered_server_dir = fields.Char(compute='_compute_render')
```

**Important Methods:**

```python
def sync_to_server(self):
    """Push file from Tower to server(s)."""

def sync_from_server(self):
    """Pull file from server(s) to Tower."""

def _compute_render(self):
    """Render filename and path with variables."""
```

**Usage Example:**

```python
# Create configuration file
config_file = env['cx.tower.file'].create({
    'name': 'app.conf',
    'reference': 'nginx_app_conf',
    'source': 'tower',
    'file_type': 'code',
    'server_dir': '/etc/nginx/conf.d',
    'code': '''
server {
    listen 80;
    server_name {{ domain_name }};
    root {{ web_root }};
}
    ''',
    'server_ids': [(6, 0, [server.id])],
    'auto_sync': True,
    'auto_sync_interval': '1-days',
})

# Sync file
config_file.sync_to_server()
```

---

### cx.tower.variable

**Purpose:** Defines variables used in commands and files.

**File:** `/home/user/cetmix-tower/cetmix_tower_server/models/cx_tower_variable.py`

**Inheritance:**
```python
_name = "cx.tower.variable"
_inherit = [
    "cx.tower.reference.mixin",
    "cx.tower.access.mixin"
]
```

**Key Fields:**

```python
name = fields.Char(required=True)
reference = fields.Char(index=True, required=True)

# Variable Type
variable_type = fields.Selection([
    ('s', 'String'),         # Free text
    ('o', 'Options'),        # Dropdown options
])

# String Variable Configuration
applied_expression = fields.Text()             # Transformation expression
validation_pattern = fields.Char()             # Regex validation
validation_message = fields.Char()             # Error message

# Option Variable Configuration
option_ids = fields.One2many(
    'cx.tower.variable.option',
    'variable_id'
)

# Usage Tracking
value_ids = fields.One2many(
    'cx.tower.variable.value',
    'variable_id'
)
command_ids = fields.Many2many('cx.tower.command')
plan_line_ids = fields.Many2many('cx.tower.plan.line')
```

**Variable Types:**

**String Variables:**
```python
# Simple string
env['cx.tower.variable'].create({
    'name': 'Application Path',
    'reference': 'app_path',
    'variable_type': 's',
})

# With validation
env['cx.tower.variable'].create({
    'name': 'Version Number',
    'reference': 'version',
    'variable_type': 's',
    'validation_pattern': r'^v\d+\.\d+\.\d+$',
    'validation_message': 'Version must be in format: vX.Y.Z',
})

# With transformation
env['cx.tower.variable'].create({
    'name': 'Database Name',
    'reference': 'db_name',
    'variable_type': 's',
    'applied_expression': "result = value.lower().replace(' ', '_')",
})
```

**Option Variables:**
```python
env_var = env['cx.tower.variable'].create({
    'name': 'Environment',
    'reference': 'environment',
    'variable_type': 'o',
    'option_ids': [
        (0, 0, {'reference': 'dev', 'name': 'Development'}),
        (0, 0, {'reference': 'staging', 'name': 'Staging'}),
        (0, 0, {'reference': 'prod', 'name': 'Production'}),
    ],
})
```

---

### cx.tower.variable.value

**Purpose:** Stores variable values for specific contexts (server, template, plan).

**File:** `/home/user/cetmix-tower/cetmix_tower_server/models/cx_tower_variable_value.py`

**Key Fields:**

```python
variable_id = fields.Many2one('cx.tower.variable', required=True)
value = fields.Char()                          # String value
option_id = fields.Many2one('cx.tower.variable.option')  # Option value

# Context (only one should be set)
server_id = fields.Many2one('cx.tower.server')
server_template_id = fields.Many2one('cx.tower.server.template')
plan_line_action_id = fields.Many2one('cx.tower.plan.line.action')
```

**Usage Example:**

```python
# Set variable value for server
env['cx.tower.variable.value'].create({
    'variable_id': app_path_var.id,
    'value': '/opt/production/myapp',
    'server_id': prod_server.id,
})

# Set option variable
env['cx.tower.variable.value'].create({
    'variable_id': environment_var.id,
    'option_id': prod_option.id,
    'server_id': prod_server.id,
})
```

---

## Mixin System

Cetmix Tower extensively uses mixins to provide reusable functionality across models. All mixins are abstract models (not stored in database).

### cx.tower.reference.mixin

**Purpose:** Automatic unique reference generation and management.

**File:** `/home/user/cetmix-tower/cetmix_tower_server/models/cx_tower_reference_mixin.py`

**Provides:**

```python
name = fields.Char(required=True, index='trigram')
reference = fields.Char(
    index=True,
    unaccent=False,
    help="Can contain English letters, digits and '_'"
)

_sql_constraints = [
    ('reference_unique', 'UNIQUE(reference)', 'Reference must be unique')
]
```

**Functionality:**

1. **Auto-generation from name:**
   ```python
   # Name: "Production Web Server"
   # Generated reference: "production_web_server"
   ```

2. **Duplicate handling:**
   ```python
   # If "production_web_server" exists
   # New reference: "production_web_server_2"
   ```

3. **Custom reference patterns:**
   ```python
   def _get_reference_pattern(self):
       """Returns [a-z0-9_] - customize per model"""
       return "[a-z0-9_]"
   ```

4. **Search by reference:**
   ```python
   record = model.get_by_reference('production_web_server')
   ```

**Important Methods:**

```python
def _generate_or_fix_reference(self, reference_source):
    """Generate or fix reference from source string."""

def _get_id_by_reference(self, reference):
    """Get record ID by reference (cached)."""

def get_by_reference(self, reference):
    """Get record by reference."""
```

**Usage in Models:**

```python
class MyModel(models.Model):
    _name = 'my.model'
    _inherit = ['cx.tower.reference.mixin']

    # name and reference fields provided by mixin
    # Auto-generation on create/write
```

---

### cx.tower.access.mixin

**Purpose:** Access level control (User, Manager, Root).

**File:** `/home/user/cetmix-tower/cetmix_tower_server/models/cx_tower_access_mixin.py`

**Provides:**

```python
access_level = fields.Selection([
    ('1', 'User'),
    ('2', 'Manager'),
    ('3', 'Root'),
], default='2', required=True, index=True)
```

**Access Levels:**

- **User (1)**: Basic operations, restricted access
- **Manager (2)**: Create and modify, standard access
- **Root (3)**: Full control, administrative access

**Usage in Security Rules:**

```xml
<!-- Only users with sufficient access level can see records -->
<record id="rule_access_level" model="ir.rule">
    <field name="domain_force">
        [('access_level', '&lt;=', user.tower_access_level)]
    </field>
</record>
```

---

### cx.tower.access.role.mixin

**Purpose:** User and manager assignment for fine-grained access.

**File:** `/home/user/cetmix-tower/cetmix_tower_server/models/cx_tower_access_role_mixin.py`

**Provides:**

```python
user_ids = fields.Many2many(
    'res.users',
    string='Users',
    help='Users who can access this record'
)
manager_ids = fields.Many2many(
    'res.users',
    string='Managers',
    help='Users who can manage this record'
)
```

**Usage:**

```python
# Assign users to server
server.user_ids = [(6, 0, [user1.id, user2.id])]
server.manager_ids = [(6, 0, [manager1.id])]

# Check access
if self.env.user in server.user_ids:
    # User has access
    pass
```

**Security Rules:**

```xml
<!-- Users can only see records they're assigned to -->
<record id="rule_user_access" model="ir.rule">
    <field name="domain_force">
        ['|', ('user_ids', 'in', user.id),
              ('manager_ids', 'in', user.id)]
    </field>
</record>
```

---

### cx.tower.vault.mixin

**Purpose:** Secure storage of sensitive fields (passwords, keys).

**File:** `/home/user/cetmix-tower/cetmix_tower_server/models/cx_tower_vault_mixin.py`

**How It Works:**

1. **Field Definition:**
   ```python
   class MyModel(models.Model):
       _inherit = ['cx.tower.vault.mixin']

       SECRET_FIELDS = ["password", "api_key"]

       password = fields.Char()
       api_key = fields.Char()
   ```

2. **Automatic Vaulting:**
   - On `create()`: Secrets moved to `cx.tower.vault`
   - On `write()`: Secrets updated in vault
   - On `read()`: Placeholder returned
   - On `unlink()`: Vault records deleted

3. **Retrieving Secrets:**
   ```python
   # Read returns placeholder
   print(server.ssh_password)  # Output: "*****"

   # Get actual secret
   actual_password = server._get_secret_value('ssh_password')
   print(actual_password)  # Output: "real_password"
   ```

**Important Methods:**

```python
def _get_secret_value(self, field_name):
    """Get actual secret value for single field."""

def _get_secret_values(self, fields_list=None):
    """Get secret values for multiple fields/records."""

def _set_secret_values(self, vals):
    """Store secret values in vault."""
```

**Vault Storage Model:**

```python
# cx.tower.vault
res_model = fields.Char()      # Model name
res_id = fields.Integer()      # Record ID
field_name = fields.Char()     # Field name
data = fields.Binary()         # Encrypted data
```

---

### cx.tower.template.mixin

**Purpose:** Jinja2 template rendering with variable substitution.

**File:** `/home/user/cetmix-tower/cetmix_tower_server/models/cx_tower_template_mixin.py`

**Provides:**

```python
code = fields.Text()  # Template content
variable_ids = fields.Many2many(
    'cx.tower.variable',
    compute='_compute_variable_ids',
    store=True
)
```

**Functionality:**

1. **Variable Detection:**
   ```python
   code = "Hello {{ name }}, version {{ version }}"
   # Auto-detects: ['name', 'version']
   # Links to variable records
   ```

2. **Template Rendering:**
   ```python
   rendered = record.render_code(
       name="World",
       version="2.0"
   )
   # Output: "Hello World, version 2.0"
   ```

3. **Python Mode:**
   ```python
   code = "result = {{ value }}"
   rendered = record.render_code(
       pythonic_mode=True,
       value="test"
   )
   # Output: result = "test"  (quoted)
   ```

**Important Methods:**

```python
def render_code(self, pythonic_mode=False, **kwargs):
    """Render code field with variables."""

def get_variables_from_code(self, code):
    """Extract variable names from code."""

def _compute_variable_ids(self):
    """Automatically link to variable records."""
```

---

### cx.tower.variable.mixin

**Purpose:** Manage variable values for records.

**File:** `/home/user/cetmix-tower/cetmix_tower_server/models/cx_tower_variable_mixin.py`

**Provides:**

```python
variable_value_ids = fields.One2many(
    'cx.tower.variable.value',
    inverse_name='server_id',  # Or template_id, etc.
)
```

**Usage:**

```python
# Get variable values for record
values = server.get_variable_values()
# Returns: {'app_path': '/opt/myapp', 'environment': 'production'}

# Set variable value
server.set_variable_value('app_path', '/opt/newapp')
```

---

### cx.tower.key.mixin

**Purpose:** Manage SSH keys and secrets in commands/files.

**File:** `/home/user/cetmix-tower/cetmix_tower_server/models/cx_tower_key_mixin.py`

**Provides:**

```python
secret_ids = fields.Many2many(
    'cx.tower.key',
    compute='_compute_secret_ids',
    store=True
)
```

**Secret References in Code:**

```python
# SSH Key reference syntax
code = """
ssh-add {{ secret:my_ssh_key }}
export API_KEY={{ secret:api_key }}
"""
# Auto-detects and links to cx.tower.key records
```

---

## Model Relationships

### Relationship Diagram

```
cx.tower.server
├── Many2many → cx.tower.command
├── One2many → cx.tower.file
├── One2many → cx.tower.variable.value
├── One2many → cx.tower.server.log
├── Many2one → cx.tower.server.template
├── Many2many → cx.tower.tag
└── Many2one → cx.tower.os

cx.tower.command
├── Many2many → cx.tower.server
├── Many2many → cx.tower.variable
├── Many2many → cx.tower.key
├── Many2many → cx.tower.os
├── Many2many → cx.tower.tag
└── One2many → cx.tower.command.log

cx.tower.plan
├── One2many → cx.tower.plan.line
│   ├── Many2one → cx.tower.command
│   └── One2many → cx.tower.plan.line.action
├── Many2many → cx.tower.server
├── Many2many → cx.tower.tag
└── One2many → cx.tower.plan.log

cx.tower.file
├── Many2many → cx.tower.server
├── Many2one → cx.tower.file.template
└── Many2many → cx.tower.variable

cx.tower.variable
├── One2many → cx.tower.variable.value
├── One2many → cx.tower.variable.option
├── Many2many → cx.tower.command
└── Many2many → cx.tower.plan.line
```

### Key Relationship Patterns

**1. Server-Command (Many2many)**
```python
# Commands available for server
server.command_ids
# Servers where command can run
command.server_ids
```

**2. Plan-Line (One2many with cascade)**
```python
plan.line_ids  # Deleting plan deletes lines
line.plan_id
```

**3. Variable-Value (One2many)**
```python
variable.value_ids  # All values for this variable
value.variable_id   # Parent variable
```

---

## ORM Patterns

### Pattern 1: Decorator for SSH Connection Management

```python
from functools import wraps

def ensure_ssh_disconnect(func):
    """Ensures SSH connection cleanup after transaction."""
    @wraps(func)
    def wrapped(self, *args, **kwargs):
        try:
            connection = self._get_ssh_client(raise_on_error=True)
        except Exception as e:
            _logger.error(f"Error obtaining SSH connection: {e}")
            connection = None

        def disconnect_connection():
            if connection:
                try:
                    connection.disconnect()
                except Exception as e:
                    _logger.error(f"Error disconnecting: {e}")

        self.env.cr.postcommit.add(disconnect_connection)
        self.env.cr.postrollback.add(disconnect_connection)

        result = func(self, *args, **kwargs)
        return result

    return wrapped
```

**Usage:**
```python
@ensure_ssh_disconnect
def execute_command(self, command_code):
    connection = self._get_ssh_client()
    # Connection auto-cleanup on commit/rollback
    return connection.execute(command_code)
```

---

### Pattern 2: Mixin-based Feature Composition

```python
class CxTowerServer(models.Model):
    _name = "cx.tower.server"
    _inherit = [
        "cx.tower.reference.mixin",      # References
        "cx.tower.access.role.mixin",    # User assignments
        "cx.tower.vault.mixin",          # Secret storage
        "cx.tower.variable.mixin",       # Variable values
        "mail.thread",                    # Messaging
    ]
    # Combines functionality from multiple mixins
```

---

### Pattern 3: Computed Fields with Dependencies

```python
@api.depends('line_ids.command_id')
def _compute_command_ids(self):
    """Compute all commands used in plan."""
    for record in self:
        commands = record.line_ids.mapped('command_id')
        record.command_ids = [(6, 0, commands.ids)]
```

---

### Pattern 4: Safe Eval for User Expressions

```python
from odoo.tools.safe_eval import safe_eval, wrap_module

# Wrap safe modules
re = wrap_module(__import__("re"), [
    "match", "search", "sub", "compile"
])

# Use in context
eval_context = {
    're': re,
    'value': user_input,
}
safe_eval(user_expression, eval_context)
```

---

## Database Schema

### Table Naming Convention

```
Model: cx.tower.server
Table: cx_tower_server

Model: cx.tower.command.log
Table: cx_tower_command_log
```

### Index Strategy

**Indexed Fields:**
- `reference` (unique, unaccent=False)
- `name` (trigram index for search)
- `access_level` (for filtering)
- Foreign keys (automatic)

**Example:**
```python
reference = fields.Char(
    index=True,
    unaccent=False  # Case-sensitive search
)
name = fields.Char(
    index='trigram'  # Full-text search
)
```

---

## Performance Optimization

### 1. Batch Operations

**Good:**
```python
# Single query for multiple records
servers = env['cx.tower.server'].search([
    ('tag_ids', 'in', [prod_tag.id])
])
for server in servers:
    server.test_connection()
```

**Bad:**
```python
# Multiple queries
for tag in tags:
    servers = env['cx.tower.server'].search([
        ('tag_ids', 'in', [tag.id])
    ])
```

---

### 2. Use read() for Data Extraction

```python
# Efficient
data = servers.read(['name', 'ip_address', 'status'])

# Less efficient
data = [
    {'name': s.name, 'ip': s.ip_address, 'status': s.status}
    for s in servers
]
```

---

### 3. Leverage Cache

```python
from odoo.tools import ormcache

@ormcache('reference')
def _get_id_by_reference(self, reference):
    """Cached reference lookup."""
    records = self.search([('reference', '=', reference)])
    return records and records[0].id
```

---

### 4. Prefetch Related Records

```python
# Auto-prefetch with auto_join
line_ids = fields.One2many(
    'cx.tower.plan.line',
    'plan_id',
    auto_join=True  # JOIN in SQL query
)
```

---

## Summary

Key takeaways:

✅ **Mixin Architecture**: Reusable functionality through abstract models
✅ **Vault Security**: Sensitive data never stored in plain text
✅ **Reference System**: Unique, human-readable identifiers
✅ **Access Control**: Multi-level security (access levels + role assignment)
✅ **Template Rendering**: Jinja2-based variable substitution
✅ **ORM Patterns**: Decorators, computed fields, safe evaluation

---

**Next Steps:**
- [Views and UI →](./02-views-and-ui.md)
- [Security and Access →](./03-security-and-access.md)
- [Module Reference →](../05-module-references/core-modules/cetmix_tower_server.md)

---

**Navigation:**
- [← Technical Guides Home](./README.md)
- [↑ Documentation Home](../README.md)

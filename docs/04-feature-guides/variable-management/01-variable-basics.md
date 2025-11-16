---
title: Variable Basics
audience: [administrators, end-users, developers]
topics: [variables, variable-types, variable-scope, variable-validation, applied-expressions]
---

# Variable Basics

This guide provides comprehensive coverage of variables in Cetmix Tower, including variable types, scope, usage, validation, and applied expressions.

## Table of Contents

- [What are Variables](#what-are-variables)
- [Variable Types](#variable-types)
- [Variable Scope](#variable-scope)
- [Variable Usage](#variable-usage)
- [Variable Validation](#variable-validation)
- [Applied Expressions](#applied-expressions)
- [For Developers](#for-developers)

---

## What are Variables

Variables in Cetmix Tower are dynamic placeholders that can be used in:
- Command code
- File paths
- File template content
- Plan line conditions
- Variable values (nested variables)

**Benefits**:
- Reusable commands and templates
- Server-specific configurations
- Dynamic command execution
- Environment-agnostic code

**Syntax**: Variables use Jinja-style syntax
```
{{ variable_name }}
```

**Model**: `cx.tower.variable`
**Location**: `cetmix_tower_server/models/cx_tower_variable.py`

---

## Variable Types

Cetmix Tower supports two variable types.

### String Variables (variable_type='s')

Free-form text variables that can contain any string value.

**Use Cases**:
- File paths
- Server names
- Version numbers
- Custom configuration values

**Example Variables**:
- `project_path`: `/opt/myapp`
- `odoo_version`: `16.0`
- `server_name`: `production-web-01`
- `database_name`: `my_database`

**Configuration**:
```python
variable_type = 's'  # String
```

### Option Variables (variable_type='o')

Variables with predefined options (like a selection field).

**Use Cases**:
- Environment types (dev, staging, production)
- Boolean flags (True, False)
- Predefined configurations
- Standardized values

**Example Variables**:
- `env_type` with options: development, staging, production
- `backup_enabled` with options: True, False
- `log_level` with options: DEBUG, INFO, WARNING, ERROR

**Configuration**:
```python
variable_type = 'o'  # Options
option_ids = [
    ('development', 'Development Environment'),
    ('staging', 'Staging Environment'),
    ('production', 'Production Environment'),
]
```

**Model**: `cx.tower.variable.option`
**Location**: `cetmix_tower_server/models/cx_tower_variable_option.py`

**Defined**: `cetmix_tower_server/models/cx_tower_variable.py:54-59`

```python
variable_type = fields.Selection(
    selection=[('s', 'String'), ('o', 'Options')],
    default='s',
    required=True,
    string='Type',
)
```

---

## Variable Scope

Variables can exist at different scopes, determining where they apply.

### Global Variables (is_global=True)

Variables that apply across the entire Tower system.

**Characteristics**:
- Not linked to any specific record
- Provide default values
- Can be overridden by server-specific values
- Useful for system-wide settings

**Use Cases**:
- Default application versions
- Common paths
- Standard configuration values
- System-wide settings

**Example**:
```
Global Variable: default_python_version = '3.11'
```

**Model Field**: `cx.tower.variable.value.is_global`
**Defined**: `cetmix_tower_server/models/cx_tower_variable_value.py:45-50`

### Server-Specific Variables

Variables assigned to specific servers.

**Characteristics**:
- Override global values
- Server-specific configuration
- Highest priority in value resolution

**Use Cases**:
- Server-specific paths
- Server names and IPs
- Environment-specific settings
- Custom server configuration

**Example**:
```
Server: web-server-01
Variable: app_path = '/opt/production/myapp'
```

**Model Field**: `cx.tower.variable.value.server_id`
**Defined**: `cetmix_tower_server/models/cx_tower_variable_value.py:75-77`

### Server Template Variables

Variables assigned to server templates.

**Characteristics**:
- Applied to servers created from template
- Template-level defaults
- Can be overridden by server-specific values

**Use Cases**:
- Template-specific configurations
- Default values for server type
- Standardized settings per template

**Example**:
```
Template: Production Web Server
Variable: max_workers = '4'
```

**Model Field**: `cx.tower.variable.value.server_template_id`
**Defined**: `cetmix_tower_server/models/cx_tower_variable_value.py:81-83`

### Action Variables

Variables set by plan line actions based on command results.

**Characteristics**:
- Set conditionally during flight plan execution
- Trigger based on exit codes
- Update server variables dynamically

**Use Cases**:
- Setting flags based on command results
- Dynamic configuration during execution
- Conditional workflow control

**Example**:
```
Plan Line Action: If exit code == 0, set needs_restart = 'True'
```

**Model Field**: `cx.tower.variable.value.plan_line_action_id`
**Defined**: `cetmix_tower_server/models/cx_tower_variable_value.py:78-80`

### Variable Resolution Order

When resolving a variable value for a server:

1. **Server-specific value** (highest priority)
2. **Server template value** (if server has template)
3. **Global value** (lowest priority, fallback)

**Method**: `get_by_variable_reference()`
**Location**: `cetmix_tower_server/models/cx_tower_variable_value.py:362-425`

```python
def get_by_variable_reference(self, variable_reference, server_id=None,
                              server_template_id=None, check_global=True):
    """Get variable value with scope resolution"""
    # Returns dict with:
    # {'server': value, 'server_template': value, 'global': value}
    # Server value takes precedence over template over global
```

---

## Variable Usage

### In Command Code

Variables are rendered when commands execute.

**SSH Command Example**:
```bash
cd {{ project_path }}
git checkout {{ branch_name }}
systemctl restart {{ service_name }}
```

**Python Command Example**:
```python
version = "{{ odoo_version }}"
env = "{{ env_type }}"

if env == 'production':
    result = {"exit_code": 0, "message": f"Deploying version {version} to production"}
```

**Rendering**: Variables are replaced with actual values before execution

### In File Paths

Use variables in command paths and file paths.

**Command Path**:
```
{{ project_path }}/logs
```

**File Template Path**:
```
{{ config_dir }}/{{ app_name }}.conf
```

### In File Templates

Variables can be used in file template content.

**Template Example** (nginx.conf):
```nginx
server {
    listen 80;
    server_name {{ domain_name }};
    root {{ web_root }};

    location / {
        proxy_pass http://{{ backend_host }}:{{ backend_port }};
    }
}
```

**Rendering**: Template is rendered with server's variable values before deployment

### In Plan Line Conditions

Variables enable conditional plan line execution.

**Condition Examples**:
```python
{{ odoo_version }} >= '16.0'
{{ env_type }} == 'production'
{{ needs_migration }} == True
{{ backup_enabled }} == 'True'
```

### In Variable Values (Nested Variables)

Variable values can reference other variables.

**Example**:
```
Variable: project_path = '/opt/{{ app_name }}'
Variable: app_name = 'myapp'
Result: project_path = '/opt/myapp'
```

**Compute Field**: `variable_ids` tracks variable dependencies
**Location**: `cetmix_tower_server/models/cx_tower_variable_value.py:84-93`

### Variable Rendering Modes

**Standard Mode** (default):
- Uses `{{ variable }}` syntax
- For shell commands and file paths

**Pythonic Mode**:
- Variables rendered without quotes
- Used in Python commands and conditions
- Example: `{{ version }}` becomes `16.0` not `"16.0"`

**Method**: `render_code_custom(code, pythonic_mode=False, **variables)`
**Location**: Inherited from `cx.tower.template.mixin`

---

## Variable Validation

Variables can have validation rules to ensure values are correct.

### Validation Pattern

**Field**: `validation_pattern` (Char)

Uses regular expressions with Python's `re.match()` function.

**Examples**:

**Alphanumeric**:
```
^[a-z0-9]+$
```

**Version Number**:
```
^\d+\.\d+$
```

**IP Address**:
```
^\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}$
```

**Email**:
```
^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$
```

**Path**:
```
^/[a-zA-Z0-9/_-]+$
```

**Defined**: `cetmix_tower_server/models/cx_tower_variable.py:68-72`

```python
validation_pattern = fields.Char(
    help="Regex pattern to validate variable values using 're.match'. "
         "If empty, values will not be validated.",
)
```

### Validation Message

**Field**: `validation_message` (Char, translatable)

Custom message shown when validation fails.

**Default Message**: "Invalid value!"

**Example Messages**:
- "Version must be in X.Y format (e.g., 16.0)"
- "Path must start with / and contain only alphanumeric characters"
- "Email address is not valid"

**Defined**: `cetmix_tower_server/models/cx_tower_variable.py:73-79`

### Validation Method

**Method**: `_validate_value(value_char)`
**Location**: `cetmix_tower_server/models/cx_tower_variable.py:338-364`

**Returns**: `(is_valid: bool, error_message: str)`

```python
def _validate_value(self, value_char=None):
    """Validate variable value against pattern"""
    self.ensure_one()

    if not self.validation_pattern or not value_char:
        return True, None

    if re.match(self.validation_pattern, value_char):
        return True, None

    message = self.validation_message or self.DEFAULT_VALIDATION_MESSAGE
    return False, f"Variable: {self.name}, Value: {value_char}\n{message}"
```

### Validation Constraints

Validation is enforced when creating or updating variable values.

**Constraint**: `_check_value_char_and_option_id`
**Location**: `cetmix_tower_server/models/cx_tower_variable_value.py:234-253`

**Raises**: `ValidationError` if value doesn't match pattern

---

## Applied Expressions

Applied expressions automatically transform variable values using Python code.

### Applied Expression Field

**Field**: `applied_expression` (Text)

Python code that processes the variable value and returns the transformed result.

**Available Objects**:
- `value`: The input variable value
- `result`: Variable to assign the final value
- `re`: Python regex module (wrapped)

**Defined**: `cetmix_tower_server/models/cx_tower_variable.py:60-67`

```python
applied_expression = fields.Text(
    help="Python expression to apply to the variable value. "
         "Use 'value' to refer to variable value, use 'result' "
         "to assign final result.\n"
         "Eg: result = value.lower().replace(' ', '_')",
)
```

### Expression Examples

**Convert to Lowercase**:
```python
result = value.lower()
```

**Remove Spaces**:
```python
result = value.replace(' ', '_')
```

**Extract Domain from Email**:
```python
result = value.split('@')[1] if '@' in value else value
```

**Normalize Path**:
```python
result = value.rstrip('/').lower()
```

**Extract Version Number**:
```python
match = re.match(r'(\d+\.\d+)', value)
result = match.group(1) if match else value
```

**Complex Transformation**:
```python
# Convert "My App Name" to "my_app_name"
result = value.lower().replace(' ', '_').replace('-', '_')
result = re.sub(r'[^a-z0-9_]', '', result)
```

### Evaluation Context

**Method**: `_get_eval_context(value_char)`
**Location**: `cetmix_tower_server/models/cx_tower_variable.py:243-258`

**Context Provided**:
```python
{
    're': re,          # Wrapped regex module
    'value': value_char,  # Input value
}
```

**Expression Execution**:
- Executed using `safe_eval()`
- Result extracted from `result` variable
- Applied before value is saved

### Use Cases

**Standardize Names**:
```python
# Variable: server_name
# Applied Expression:
result = value.lower().replace(' ', '-')

# Input: "Production Server 01"
# Output: "production-server-01"
```

**Extract Information**:
```python
# Variable: odoo_git_url
# Applied Expression:
match = re.search(r'/([^/]+?)\.git$', value)
result = match.group(1) if match else 'unknown'

# Input: "https://github.com/odoo/odoo.git"
# Output: "odoo"
```

**Format Paths**:
```python
# Variable: log_directory
# Applied Expression:
result = value.rstrip('/').lower()
if not value.startswith('/'):
    result = '/' + result

# Input: "Var/Log/MyApp/"
# Output: "/var/log/myapp"
```

**Validate and Transform**:
```python
# Variable: port_number
# Applied Expression:
try:
    port = int(value)
    if 1 <= port <= 65535:
        result = str(port)
    else:
        result = '8069'  # Default
except ValueError:
    result = '8069'  # Default

# Input: "abc" or "99999"
# Output: "8069"
```

---

## For Developers

### Variable Model

**Model Name**: `cx.tower.variable`
**Location**: `cetmix_tower_server/models/cx_tower_variable.py`

**Inheritance**:
```python
_inherit = [
    "cx.tower.reference.mixin",  # Reference tracking
    "cx.tower.access.mixin",     # Access control
]
```

**Order**: `name`

### Variable Value Model

**Model Name**: `cx.tower.variable.value`
**Location**: `cetmix_tower_server/models/cx_tower_variable_value.py`

**Inheritance**:
```python
_inherit = [
    "cx.tower.reference.mixin",  # Reference tracking
    "cx.tower.access.mixin",     # Access control
]
```

**Record Name**: `variable_reference`
**Order**: `sequence, variable_reference`

### Key Methods

#### Get Variable Values for Server

**Method**: `server.get_variable_values(variable_references)`
**Location**: Inherited from `cx.tower.variable.mixin`

**Returns**: Dict mapping server_id to variable values
```python
{
    server_id: {
        'variable_ref1': 'value1',
        'variable_ref2': 'value2',
    }
}
```

#### Get Variable by Reference

**Method**: `variable_value.get_by_variable_reference(reference, server_id, server_template_id, check_global)`
**Location**: `cetmix_tower_server/models/cx_tower_variable_value.py:362-425`

**Returns**: Dict with scope-specific values
```python
{
    'server': 'server_specific_value',
    'server_template': 'template_value',
    'global': 'global_value'
}
```

### Variable Tracking

Variables are auto-tracked in models that use them.

**Tracked By**:
- Commands (`command_ids`)
- Plan lines (`plan_line_ids`)
- Files (`file_ids`)
- File templates (`file_template_ids`)
- Variable values (for nested variables)

**Compute Method**: Auto-computed from code/path/condition fields
**Location**: Uses `cx.tower.template.mixin._prepare_variable_commands()`

### Variable Reference Propagation

When a variable's reference changes, all usages are updated.

**Method**: `_propagate_reference_change(old_ref, new_ref)`
**Location**: `cetmix_tower_server/models/cx_tower_variable.py:279-317`

**Updates**:
- Command code and paths
- File code, paths, and names
- File template code, paths, and names
- Variable values (nested variables)
- Plan line conditions

**Pattern Replacement**:
```python
# Pattern: {{ old_ref }} → {{ new_ref }}
pattern = re.compile(r"(\{\{\s*)" + re.escape(old_ref) + r"(\s*\}\})")
pattern.sub(lambda m: f"{m.group(1)}{new_ref}{m.group(2)}", text)
```

### Code Examples

#### Example 1: Create Variable Programmatically

```python
# File: custom_module/models/variable_creator.py

def create_odoo_version_variable(self):
    """Create Odoo version variable with options"""
    variable_obj = self.env['cx.tower.variable']
    option_obj = self.env['cx.tower.variable.option']

    # Create variable
    variable = variable_obj.create({
        'name': 'Odoo Version',
        'reference': 'odoo_version',
        'variable_type': 'o',  # Options
        'validation_pattern': r'^\d+\.\d+$',
        'validation_message': 'Version must be in X.Y format (e.g., 16.0)',
        'note': 'Odoo version to deploy',
    })

    # Create options
    for version in ['14.0', '15.0', '16.0', '17.0']:
        option_obj.create({
            'variable_id': variable.id,
            'value_char': version,
            'name': f'Odoo {version}',
        })

    return variable
```

#### Example 2: Set Variable Values

```python
# File: custom_module/models/variable_setter.py

def set_server_variables(self, server):
    """Set standard variables for a server"""
    value_obj = self.env['cx.tower.variable.value']

    # Set Odoo version
    value_obj.create({
        'server_id': server.id,
        'variable_id': self.env.ref('my_module.var_odoo_version').id,
        'value_char': '16.0',
    })

    # Set project path
    value_obj.create({
        'server_id': server.id,
        'variable_id': self.env.ref('my_module.var_project_path').id,
        'value_char': f'/opt/{server.name}',
    })

    # Set environment type
    value_obj.create({
        'server_id': server.id,
        'variable_id': self.env.ref('my_module.var_env_type').id,
        'option_id': self.env.ref('my_module.option_production').id,
    })
```

#### Example 3: Create Variable with Applied Expression

```python
# File: custom_module/models/smart_variable.py

def create_normalized_name_variable(self):
    """Create variable that normalizes input"""
    return self.env['cx.tower.variable'].create({
        'name': 'Application Name',
        'reference': 'app_name',
        'variable_type': 's',
        'applied_expression': """
# Convert to lowercase and replace spaces with underscores
result = value.lower().replace(' ', '_')
# Remove any non-alphanumeric characters except underscore
result = re.sub(r'[^a-z0-9_]', '', result)
""",
        'validation_pattern': r'^[a-z0-9_]+$',
        'validation_message': 'Must contain only lowercase letters, numbers, and underscores',
    })
```

#### Example 4: Get Variable Value with Fallback

```python
# File: custom_module/models/variable_getter.py

def get_variable_with_fallback(self, server, variable_ref, default=None):
    """Get variable value for server with fallback"""
    value_obj = self.env['cx.tower.variable.value']

    # Get value with scope resolution
    values = value_obj.get_by_variable_reference(
        variable_ref,
        server_id=server.id,
        check_global=True
    )

    # Return server-specific, then global, then default
    return values.get('server') or values.get('global') or default
```

#### Example 5: Validate Variable Before Use

```python
# File: custom_module/models/variable_validator.py

def validate_and_use_variable(self, variable, value):
    """Validate variable value before using it"""
    is_valid, error_msg = variable._validate_value(value)

    if not is_valid:
        raise UserError(f"Variable validation failed: {error_msg}")

    # Value is valid, use it
    return value
```

#### Example 6: Custom Variable Resolution

```python
# File: custom_module/models/custom_resolution.py

def resolve_all_variables(self, server, code):
    """Resolve all variables in code for a server"""
    # Extract variable references from code
    import re
    pattern = r'\{\{\s*([a-zA-Z_][a-zA-Z0-9_]*)\s*\}\}'
    variables = re.findall(pattern, code)

    # Get values
    value_obj = self.env['cx.tower.variable.value']
    resolved = {}

    for var_ref in variables:
        values = value_obj.get_by_variable_reference(
            var_ref,
            server_id=server.id,
            check_global=True
        )
        resolved[var_ref] = values.get('server') or values.get('global') or ''

    # Replace in code
    for var_ref, value in resolved.items():
        code = code.replace(f'{{{{ {var_ref} }}}}', value)

    return code
```

### SQL Constraints

**Unique Variable Names**:
```sql
CONSTRAINT name_uniq UNIQUE (name)
```

**Unique Variable Value per Scope**:
```sql
CONSTRAINT tower_variable_value_uniq
UNIQUE (variable_id, server_id, server_template_id, plan_line_action_id, is_global)
```

**Defined**: `cetmix_tower_server/models/cx_tower_variable_value.py:96-123`

### Access Control

Variables and variable values inherit access control:

**Variable**:
- `access_level`: Control who can view/edit variable definition

**Variable Value**:
- `access_level`: Automatically set from variable's access level
- Cannot be lower than variable's access level
- Validated by `_check_access_level_consistency` constraint

**Location**: `cetmix_tower_server/models/cx_tower_variable_value.py:174-207`

## Related Documentation

- [Command Execution](../command-execution/README.md)
- [Flight Plans](../flight-plans/README.md)
- [Server Management](../server-management/README.md)
- [File Management](../file-management/README.md)

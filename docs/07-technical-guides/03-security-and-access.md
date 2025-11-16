---
title: "Security and Access Control"
section: "07-technical-guides"
document_id: "TG-003"
version: "17.0"
audience: "Developers, Security Specialists"
last_updated: "2025-11-16"
tags: ["security", "access-control", "rbac", "vault", "ssh", "permissions"]
---

# Security and Access Control

Comprehensive guide to Cetmix Tower's security architecture, access control mechanisms, and best practices for securing your Tower installation.

## Table of Contents

1. [Security Architecture Overview](#security-architecture-overview)
2. [Role-Based Access Control](#role-based-access-control)
3. [Access Control Implementation](#access-control-implementation)
4. [Record Rules](#record-rules)
5. [Field-Level Security](#field-level-security)
6. [Vault System](#vault-system)
7. [SSH Security](#ssh-security)
8. [Security Best Practices](#security-best-practices)

---

## Security Architecture Overview

Cetmix Tower implements a multi-layered security architecture:

```
┌─────────────────────────────────────────────────┐
│         User Authentication (Odoo)              │
└──────────────────┬──────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────┐
│     Group-Based Access (User/Manager/Root)      │
└──────────────────┬──────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────┐
│      Record-Level Rules (ir.rule)               │
│  • User assignment (user_ids, manager_ids)      │
│  • Access level filtering                       │
└──────────────────┬──────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────┐
│      Field-Level Security                        │
│  • Secret fields → Vault                        │
│  • Computed fields with sudo                    │
└──────────────────┬──────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────┐
│         Data Encryption (Vault)                  │
│  • SSH passwords, host keys                     │
│  • API keys, secrets                            │
└─────────────────────────────────────────────────┘
```

### Security Principles

**1. Defense in Depth**
- Multiple security layers
- Redundant controls
- Fail-safe defaults

**2. Least Privilege**
- Minimal necessary permissions
- Role-based access
- Explicit grants only

**3. Separation of Duties**
- Clear role boundaries
- Manager approval workflows
- Audit trails

**4. Data Protection**
- Secrets never in plain text
- Vault-based encryption
- Secure communication (SSH)

---

## Role-Based Access Control

### Security Groups

Cetmix Tower defines three security groups with hierarchical permissions.

**File:** `/home/user/cetmix-tower/cetmix_tower_server/security/cetmix_tower_server_groups.xml`

```xml
<!-- Module Category -->
<record id="ir_module_category_tower" model="ir.module.category">
    <field name="name">Cetmix Tower</field>
    <field name="sequence">199</field>
</record>

<record id="ir_module_category_tower_server" model="ir.module.category">
    <field name="parent_id" ref="ir_module_category_tower"/>
    <field name="name">Access Level</field>
</record>

<!-- User Group (Level 1) -->
<record id="group_user" model="res.groups">
    <field name="name">User</field>
    <field name="category_id" ref="ir_module_category_tower_server"/>
    <field name="comment">
        Basic actions for selected servers.
    </field>
</record>

<!-- Manager Group (Level 2) -->
<record id="group_manager" model="res.groups">
    <field name="name">Manager</field>
    <field name="category_id" ref="ir_module_category_tower_server"/>
    <field name="implied_ids" eval="[(4, ref('group_user'))]"/>
    <field name="comment">
        Create and modify selected servers.
    </field>
</record>

<!-- Root Group (Level 3) -->
<record id="group_root" model="res.groups">
    <field name="name">Root</field>
    <field name="category_id" ref="ir_module_category_tower_server"/>
    <field name="implied_ids" eval="[(4, ref('group_manager'))]"/>
    <field name="comment">
        Full control over all servers.
    </field>
</record>
```

### Group Capabilities

| Capability | User | Manager | Root |
|------------|------|---------|------|
| **View assigned records** | ✅ | ✅ | ✅ |
| **Run commands** | ✅ | ✅ | ✅ |
| **View logs** | ✅ | ✅ | ✅ |
| **Create records** | ❌ | ✅ | ✅ |
| **Edit records** | ❌ | ✅ (if manager) | ✅ |
| **Delete records** | ❌ | ✅ (own records) | ✅ |
| **View all records** | ❌ | ❌ | ✅ |
| **Manage SSH keys** | ❌ | ✅ | ✅ |
| **Access vault secrets** | ❌ | ✅ | ✅ |
| **System configuration** | ❌ | ❌ | ✅ |

### Group Hierarchy

```
group_root (Level 3)
    ↓ implies
group_manager (Level 2)
    ↓ implies
group_user (Level 1)
```

Users in Root group automatically have Manager and User permissions.

---

## Access Control Implementation

### Model Access Rights

**File:** `/home/user/cetmix-tower/cetmix_tower_server/security/ir.model.access.csv`

Access rights are defined per model and group:

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink

# Servers
access_server_user,Server->User,model_cx_tower_server,group_user,1,0,0,0
access_server_manager,Server->Manager,model_cx_tower_server,group_manager,1,1,1,1
access_server_root,Server->Root,model_cx_tower_server,group_root,1,1,1,1

# Commands
access_command_user,Command->User,model_cx_tower_command,group_user,1,0,0,0
access_command_manager,Command->Manager,model_cx_tower_command,group_manager,1,1,1,1
access_command_root,Command->Root,model_cx_tower_command,group_root,1,1,1,1

# Variables
access_variable_user,Variable->User,model_cx_tower_variable,group_user,1,0,0,0
access_variable_manager,Variable->Manager,model_cx_tower_variable,group_manager,1,1,1,0
access_variable_root,Variable->Root,model_cx_tower_variable,group_root,1,1,1,1

# SSH Keys (restricted)
access_key_user,Key->User,model_cx_tower_key,group_user,0,0,0,0
access_key_manager,Key->Manager,model_cx_tower_key,group_manager,1,1,1,1
access_key_root,Key->Root,model_cx_tower_key,group_root,1,1,1,1
```

**Permission Columns:**
- `perm_read`: Can view records
- `perm_write`: Can modify records
- `perm_create`: Can create records
- `perm_unlink`: Can delete records

### Access Mixins

**cx.tower.access.mixin**

Provides `access_level` field:

```python
# File: cx_tower_access_mixin.py
class CxTowerAccessMixin(models.AbstractModel):
    _name = "cx.tower.access.mixin"

    access_level = fields.Selection([
        ('1', 'User'),
        ('2', 'Manager'),
        ('3', 'Root'),
    ], default='2', required=True, index=True)
```

**cx.tower.access.role.mixin**

Provides user/manager assignment:

```python
# File: cx_tower_access_role_mixin.py
class CxTowerAccessRoleMixin(models.AbstractModel):
    _name = "cx.tower.access.role.mixin"

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

---

## Record Rules

Record rules (ir.rule) filter records based on user context.

### Server Record Rules

**File:** `/home/user/cetmix-tower/cetmix_tower_server/security/cx_tower_server_security.xml`

**Rule 1: User Read Access**

```xml
<record id="rule_cx_tower_server_group_user_read" model="ir.rule">
    <field name="name">Tower Server: user visibility rule</field>
    <field name="model_id" ref="model_cx_tower_server"/>
    <field name="groups" eval="[(4, ref('group_user'))]"/>
    <field name="domain_force">[('user_ids', 'in', [user.id])]</field>
    <field name="perm_read" eval="1"/>
    <field name="perm_write" eval="0"/>
    <field name="perm_create" eval="0"/>
    <field name="perm_unlink" eval="0"/>
</record>
```

**Explanation:**
- Users can only **read** servers where they're assigned in `user_ids`
- No write, create, or delete permissions

---

**Rule 2: Manager Read Access**

```xml
<record id="rule_cx_tower_server_group_manager_read" model="ir.rule">
    <field name="name">Tower Server: Manager Read</field>
    <field name="model_id" ref="model_cx_tower_server"/>
    <field name="groups" eval="[(4, ref('group_manager'))]"/>
    <field name="domain_force">
        ['|', ('user_ids', 'in', [user.id]),
              ('manager_ids', 'in', [user.id])]
    </field>
    <field name="perm_read" eval="1"/>
    <field name="perm_write" eval="0"/>
    <field name="perm_create" eval="0"/>
    <field name="perm_unlink" eval="0"/>
</record>
```

**Explanation:**
- Managers can **read** servers where they're in `user_ids` OR `manager_ids`
- Broader visibility than users

---

**Rule 3: Manager Write/Create Access**

```xml
<record id="rule_cx_tower_server_group_manager_write" model="ir.rule">
    <field name="name">Tower Server: Manager Write &amp; Create</field>
    <field name="model_id" ref="model_cx_tower_server"/>
    <field name="groups" eval="[(4, ref('group_manager'))]"/>
    <field name="domain_force">[('manager_ids', 'in', [user.id])]</field>
    <field name="perm_read" eval="0"/>
    <field name="perm_write" eval="1"/>
    <field name="perm_create" eval="1"/>
    <field name="perm_unlink" eval="0"/>
</record>
```

**Explanation:**
- Managers can **write/create** only if they're in `manager_ids`
- Ensures managers can only modify resources they manage

---

**Rule 4: Manager Delete Access**

```xml
<record id="rule_cx_tower_server_group_manager_unlink" model="ir.rule">
    <field name="name">Tower Server: Manager Delete</field>
    <field name="model_id" ref="model_cx_tower_server"/>
    <field name="groups" eval="[(4, ref('group_manager'))]"/>
    <field name="domain_force">
        [('manager_ids', 'in', [user.id]),
         ('create_uid', '=', user.id)]
    </field>
    <field name="perm_read" eval="0"/>
    <field name="perm_write" eval="0"/>
    <field name="perm_create" eval="0"/>
    <field name="perm_unlink" eval="1"/>
</record>
```

**Explanation:**
- Managers can **delete** only if:
  - They're in `manager_ids` AND
  - They created the record (`create_uid`)
- Prevents deleting other managers' resources

---

**Rule 5: Root Full Access**

```xml
<record id="rule_cx_tower_server_group_root_full" model="ir.rule">
    <field name="name">Tower Server: root visibility rule</field>
    <field name="model_id" ref="model_cx_tower_server"/>
    <field name="domain_force">[(1, '=', 1)]</field>
    <field name="groups" eval="[(4, ref('group_root'))]"/>
</record>
```

**Explanation:**
- Domain `[(1, '=', 1)]` means "all records"
- Root users have unrestricted access
- All permissions (CRUD) granted via model access rights

---

### Command Record Rules

Similar pattern for commands:

```xml
<!-- User: read commands they can access -->
<record id="rule_cx_tower_command_user" model="ir.rule">
    <field name="name">Tower Command: user</field>
    <field name="model_id" ref="model_cx_tower_command"/>
    <field name="groups" eval="[(4, ref('group_user'))]"/>
    <field name="domain_force">
        ['|', ('user_ids', 'in', [user.id]),
              ('access_level', '&lt;=', '1')]
    </field>
</record>

<!-- Manager: read commands with manager or lower access level -->
<record id="rule_cx_tower_command_manager" model="ir.rule">
    <field name="name">Tower Command: manager</field>
    <field name="model_id" ref="model_cx_tower_command"/>
    <field name="groups" eval="[(4, ref('group_manager'))]"/>
    <field name="domain_force">
        ['|', ('manager_ids', 'in', [user.id]),
              ('access_level', '&lt;=', '2')]
    </field>
</record>
```

---

## Field-Level Security

### Computed Fields with sudo()

Some fields require elevated permissions:

```python
rendered_code = fields.Text(
    compute='_compute_render',
    compute_sudo=True  # Bypass access checks
)

@api.depends('code', 'variable_ids')
def _compute_render(self):
    """Compute with sudo to access all variables."""
    for record in self:
        # Can access all variable values regardless of user permissions
        record.rendered_code = record.render_code(...)
```

**Use Case:** Rendering templates requires reading variable values that users may not directly access.

---

### Read-Only Fields

Critical fields are read-only for users:

```python
sync_date_last = fields.Datetime(
    readonly=True,
    tracking=True
)

exit_code = fields.Integer(
    readonly=True
)
```

---

### Groups Attribute

Restrict field visibility to specific groups:

```xml
<field name="ssh_password" password="True"
       groups="cetmix_tower_server.group_manager,cetmix_tower_server.group_root"/>
```

Only Managers and Root users see this field.

---

## Vault System

The vault provides secure storage for sensitive data.

### Architecture

**Vault Model:** `cx.tower.vault`

```python
class CxTowerVault(models.Model):
    _name = "cx.tower.vault"
    _description = "Secure Secret Storage"

    res_model = fields.Char(required=True)    # Model name
    res_id = fields.Integer(required=True)    # Record ID
    field_name = fields.Char(required=True)   # Field name
    data = fields.Binary(required=True)       # Encrypted data
```

**Storage Pattern:**
```
Model: cx.tower.server (ID: 123)
Field: ssh_password
Value: "secret_password"

Vault Record:
  res_model: "cx.tower.server"
  res_id: 123
  field_name: "ssh_password"
  data: <encrypted binary>
```

---

### Vault Mixin Implementation

**File:** `/home/user/cetmix-tower/cetmix_tower_server/models/cx_tower_vault_mixin.py`

**Usage in Models:**

```python
class CxTowerServer(models.Model):
    _name = "cx.tower.server"
    _inherit = ["cx.tower.vault.mixin"]

    SECRET_FIELDS = ["ssh_password", "host_key"]

    ssh_password = fields.Char()  # Never stored in main table
    host_key = fields.Text()      # Never stored in main table
```

---

### Vault Lifecycle

**1. Create:**

```python
# User creates server with password
server = env['cx.tower.server'].create({
    'name': 'My Server',
    'ssh_password': 'secret123',  # User provides
})

# Vault mixin intercepts:
# 1. Extracts ssh_password from vals
# 2. Creates record with placeholder
# 3. Stores actual value in vault
# 4. Clears placeholder from main table

# Main table: ssh_password = NULL
# Vault table: data = <encrypted "secret123">
```

**2. Read:**

```python
# Direct read returns placeholder
print(server.ssh_password)  # Output: "*****"

# Get actual secret
actual = server._get_secret_value('ssh_password')
print(actual)  # Output: "secret123"
```

**3. Write:**

```python
# Update password
server.write({'ssh_password': 'new_secret'})

# Vault mixin:
# 1. Updates vault record
# 2. Keeps main table NULL
```

**4. Delete:**

```python
# Delete server
server.unlink()

# Vault mixin:
# 1. Deletes main record
# 2. Automatically deletes vault records
# 3. No orphaned secrets
```

---

### Vault Security Features

**✅ Advantages:**

1. **Separation:** Secrets never in main table
2. **Encryption:** Binary storage (can be enhanced)
3. **Access Control:** Only specific methods retrieve secrets
4. **Audit Trail:** Can log vault access
5. **Cleanup:** Automatic deletion with parent record

**🔒 Security Guarantees:**

- ORM reads never expose secrets
- Export functions don't include secrets
- Database dumps don't contain plain secrets
- Secrets only accessible via `_get_secret_value()`

---

## SSH Security

### SSH Authentication Methods

Tower supports multiple SSH authentication methods:

```python
ssh_auth_mode = fields.Selection([
    ('p', 'Password'),           # Username + password
    ('k', 'SSH Key'),            # SSH key file
    ('pk', 'Password + Key'),    # Both
], default='p')
```

### SSH Connection Security

**File:** `/home/user/cetmix-tower/cetmix_tower_server/ssh/ssh.py`

**1. Host Key Verification:**

```python
def _verify_host_key(self, hostname, port, server_host_key):
    """Verify server host key to prevent MITM attacks."""
    if not server_host_key:
        # First connection - store host key
        return True

    # Subsequent connections - verify matches
    if current_key != server_host_key:
        raise SecurityError("Host key mismatch! Possible MITM attack.")

    return True
```

**2. Connection Timeout:**

```python
def connect(self, timeout=30):
    """Connect with timeout to prevent hanging."""
    self.client.connect(
        hostname=self.hostname,
        port=self.port,
        username=self.username,
        password=self.password,
        timeout=timeout,
        banner_timeout=timeout,
    )
```

**3. Automatic Disconnection:**

```python
@ensure_ssh_disconnect
def execute_command(self, command_code):
    """Decorator ensures cleanup even on errors."""
    connection = self._get_ssh_client()
    result = connection.execute(command_code)
    # Connection auto-closed on transaction commit/rollback
    return result
```

---

### SSH Best Practices

**✅ Recommended:**

1. **Use SSH Keys:** More secure than passwords
2. **Verify Host Keys:** Store and verify on each connection
3. **Limited User Accounts:** Use non-root accounts where possible
4. **Key Rotation:** Regularly update SSH keys
5. **Connection Timeouts:** Prevent resource exhaustion
6. **Audit Logging:** Log all SSH connections

**❌ Avoid:**

1. **Root SSH:** Don't use root account
2. **Shared Passwords:** One password per server
3. **Unverified Keys:** Always verify host keys
4. **Long Timeouts:** Keep timeouts reasonable
5. **Hardcoded Credentials:** Always use vault

---

## Security Best Practices

### For Administrators

**1. User Management:**

```python
# Principle of least privilege
user = env['res.users'].create({
    'name': 'John Operator',
    'login': 'joperator',
    'groups_id': [(4, env.ref('cetmix_tower_server.group_user').id)],
})

# Assign to specific servers only
server.user_ids = [(4, user.id)]
```

**2. Regular Audits:**

```python
# Review user assignments
servers = env['cx.tower.server'].search([])
for server in servers:
    print(f"{server.name}:")
    print(f"  Users: {server.user_ids.mapped('name')}")
    print(f"  Managers: {server.manager_ids.mapped('name')}")
```

**3. SSH Key Rotation:**

```python
# Update SSH key
server.write({
    'ssh_key_id': new_key.id,
    'ssh_password': False,  # Remove password auth
})
```

---

### For Developers

**1. Never Bypass Security:**

```python
# ❌ BAD: Bypassing access checks
servers = env['cx.tower.server'].sudo().search([])

# ✅ GOOD: Respect access rules
servers = env['cx.tower.server'].search([])
```

**2. Use Vault for Secrets:**

```python
# ❌ BAD: Plain text secret
class MyModel(models.Model):
    api_key = fields.Char()

# ✅ GOOD: Vaulted secret
class MyModel(models.Model):
    _inherit = ['cx.tower.vault.mixin']
    SECRET_FIELDS = ['api_key']
    api_key = fields.Char()
```

**3. Validate User Input:**

```python
def create(self, vals):
    # Validate before creating
    if 'ssh_password' in vals:
        self._validate_password_complexity(vals['ssh_password'])

    return super().create(vals)
```

---

### Security Checklist

**Installation:**
- [ ] Configure secure admin password
- [ ] Enable SSL/TLS for Odoo
- [ ] Restrict network access
- [ ] Configure firewall rules

**Configuration:**
- [ ] Create security groups
- [ ] Assign users to groups
- [ ] Configure record rules
- [ ] Test access restrictions

**Operation:**
- [ ] Regular security audits
- [ ] Monitor access logs
- [ ] Rotate SSH keys
- [ ] Update system regularly

**Development:**
- [ ] Use vault for secrets
- [ ] Respect access controls
- [ ] Validate user input
- [ ] Document security decisions

---

## Summary

Key Takeaways:

✅ **Multi-Layer Security:** Groups, record rules, field security, vault
✅ **Role-Based Access:** User, Manager, Root hierarchy
✅ **Vault System:** Encrypted secret storage
✅ **SSH Security:** Host key verification, timeouts, auto-disconnect
✅ **Best Practices:** Least privilege, regular audits, key rotation

---

**Next Steps:**
- [Models and ORM ←](./01-models-and-orm.md)
- [Views and UI ←](./02-views-and-ui.md)
- [Module Reference →](../05-module-references/core-modules/cetmix_tower_server.md)

---

**Navigation:**
- [← Technical Guides Home](./README.md)
- [↑ Documentation Home](../README.md)

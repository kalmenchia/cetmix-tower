---
title: Security Guidelines
description: Comprehensive security guidelines for Cetmix Tower deployment and usage
category: security-compliance
order: 2
---

# Security Guidelines

Comprehensive security guidelines for deploying, configuring, and using Cetmix Tower securely.

## Table of Contents

- [Authentication & Authorization](#authentication--authorization)
- [Vault Security](#vault-security)
- [SSH Key Management](#ssh-key-management)
- [Secret Handling](#secret-handling)
- [Network Security](#network-security)
- [Audit Logging](#audit-logging)
- [Security Checklist](#security-checklist)
- [Incident Response](#incident-response)

## Authentication & Authorization

### Odoo Authentication

**Password Policy**:
```python
# Recommended in res.config.settings or via Odoo config
- Minimum length: 12 characters
- Require uppercase, lowercase, numbers
- Require special characters
- Password expiration: 90 days
- Password history: 5 previous passwords
- Account lockout: 5 failed attempts
```

**Multi-Factor Authentication**:
```bash
# Install OCA module
pip install odoo-addon-auth_totp

# Enable for administrators
Settings > Users > [User] > Enable MFA
```

**Session Security**:
```ini
# /etc/odoo/odoo.conf
[options]
session_lifetime = 8  # Hours
session_timeout = 30  # Minutes
```

### Tower Access Control

**Security Groups**:

1. **Tower User** (`cetmix_tower_server.group_user`)
   ```xml
   - Read access to assigned servers
   - Execute commands on assigned servers
   - View command logs
   - Cannot modify servers or commands
   ```

2. **Tower Manager** (`cetmix_tower_server.group_manager`)
   ```xml
   - Full access to servers, commands, plans
   - Can assign servers to users
   - Manage SSH credentials
   - Create and modify configurations
   ```

3. **System Administrator** (`base.group_system`)
   ```xml
   - Full system access
   - Vault management
   - Security configuration
   - User management
   ```

**Assigning Roles**:
```python
# Via UI: Settings > Users > [User] > Access Rights > Tower

# Via code:
user = env['res.users'].browse(user_id)
manager_group = env.ref('cetmix_tower_server.group_manager')
user.write({'groups_id': [(4, manager_group.id)]})
```

### Record-Level Security

**Server Access Rules**:
```python
# Users see only assigned servers
<record id="cx_tower_server_user_rule" model="ir.rule">
    <field name="name">Server: User Access</field>
    <field name="model_id" ref="model_cx_tower_server"/>
    <field name="domain_force">
        ['|',
            ('user_ids', 'in', [user.id]),
            ('manager_ids', 'in', [user.id])
        ]
    </field>
    <field name="groups" eval="[(4, ref('group_user'))]"/>
</record>

# Managers see all servers they manage
<record id="cx_tower_server_manager_rule" model="ir.rule">
    <field name="name">Server: Manager Access</field>
    <field name="model_id" ref="model_cx_tower_server"/>
    <field name="domain_force">[(1, '=', 1)]</field>
    <field name="groups" eval="[(4, ref('group_manager'))]"/>
</record>
```

**Field-Level Security**:
```xml
<!-- SSH credentials only for managers -->
<field name="ssh_password"
       groups="cetmix_tower_server.group_manager"
       password="True"/>
<field name="ssh_key_id"
       groups="cetmix_tower_server.group_manager"/>
```

## Vault Security

### Encryption

**How It Works**:
```python
# Vault uses Fernet symmetric encryption (AES 128)
# Each field encrypted separately
# Encryption key stored in Odoo config or environment

# Fields protected by vault:
- server.ssh_password
- server.host_key
- key.secret_value
- key_value.secret_value
```

**Configuration**:
```ini
# /etc/odoo/odoo.conf
[options]
# Generate with: from cryptography.fernet import Fernet; Fernet.generate_key()
vault_encryption_key = YOUR_ENCRYPTION_KEY_HERE
```

**Key Generation**:
```python
from cryptography.fernet import Fernet

# Generate encryption key
key = Fernet.generate_key()
print(key.decode())  # Add to Odoo config
```

### Vault Best Practices

1. **Secure Key Storage**:
   ```bash
   # Store encryption key separately
   # Option 1: Environment variable
   export ODOO_VAULT_KEY="your-key-here"

   # Option 2: Separate config file (secured)
   chmod 600 /etc/odoo/vault.conf

   # Option 3: Key management service (AWS KMS, etc.)
   ```

2. **Key Rotation**:
   ```python
   # Rotate encryption key annually
   # 1. Generate new key
   # 2. Re-encrypt all vault data
   # 3. Update configuration
   # 4. Restart Odoo

   # Use migration script for re-encryption
   ```

3. **Backup Security**:
   ```bash
   # Encrypt backups
   pg_dump dbname | gpg --encrypt > backup.sql.gpg

   # Secure backup storage
   chmod 600 backup.sql.gpg
   ```

### Accessing Vault Data

**Always use sudo()**:
```python
# Good - Encrypted field access
password = server.sudo()._get_secret_value('ssh_password')

# Bad - Direct access (returns encrypted blob)
password = server.ssh_password  # Don't do this
```

## SSH Key Management

### SSH Key Generation

**Generate Strong Keys**:
```bash
# RSA 4096-bit (recommended)
ssh-keygen -t rsa -b 4096 -C "tower@example.com" -f tower_rsa

# Ed25519 (modern, smaller)
ssh-keygen -t ed25519 -C "tower@example.com" -f tower_ed25519

# Set passphrase when prompted
```

**Key Storage in Tower**:
```python
# Create SSH key record
ssh_key = env['cx.tower.key'].create({
    'name': 'Production Deploy Key',
    'reference': 'prod_deploy_key',
    'key_type': 'k',  # SSH Key
    'secret_value': open('tower_rsa').read(),  # Private key
    'note': 'Used for production deployments'
})
```

### SSH Host Key Verification

**Enable by Default**:
```python
# When creating server
server = env['cx.tower.server'].create({
    'name': 'Production Server',
    'skip_host_key': False,  # Verify host key (default)
    # ... other fields
})

# Get and store host key
host_key = server._get_host_key_from_host()
server.host_key = host_key
```

**Manual Host Key Verification**:
```bash
# Get host key manually
ssh-keyscan -H 192.168.1.100

# Or connect once and verify fingerprint
ssh user@192.168.1.100
# Verify fingerprint matches expected value
```

### SSH Security Best Practices

1. **Prefer Keys Over Passwords**:
   ```python
   server = env['cx.tower.server'].create({
       'ssh_auth_mode': 'k',  # Key-based (preferred)
       'ssh_key_id': key_id,
       # Not: 'ssh_auth_mode': 'p' (password)
   })
   ```

2. **Restrict SSH Access**:
   ```bash
   # On remote server: /etc/ssh/sshd_config
   PermitRootLogin no
   PasswordAuthentication no
   PubkeyAuthentication yes
   AllowUsers tower-user
   ```

3. **Use Dedicated Keys**:
   ```python
   # Separate keys for different purposes
   - Production deployments: prod_deploy_key
   - Staging: staging_deploy_key
   - Development: dev_access_key
   ```

4. **Key Rotation**:
   ```python
   # Rotate SSH keys annually
   # 1. Generate new key pair
   # 2. Add new public key to servers
   # 3. Update Tower SSH key record
   # 4. Test connections
   # 5. Remove old public key from servers
   ```

## Secret Handling

### Creating Secrets

```python
# Create secret
secret = env['cx.tower.key'].create({
    'name': 'GitHub Token',
    'reference': 'github_token',
    'key_type': 's',  # Secret
    'secret_value': 'ghp_xxxxxxxxxxxx',
})

# Server-specific secret value
secret_value = env['cx.tower.key.value'].create({
    'key_id': secret.id,
    'server_id': server.id,
    'secret_value': 'server-specific-token',
})
```

### Using Secrets in Commands

```bash
# In SSH command
git clone https://#!cxtower.secret.github_token!#@github.com/user/repo.git

# In Python command
import requests
token = "#!cxtower.secret.api_token!#"
headers = {'Authorization': f'Bearer {token}'}
```

### Secret Best Practices

1. **Never Log Secrets**:
   ```python
   # Good - Secrets hidden in logs
   _logger.info('Deployment started')

   # Bad - Secret exposed in log
   _logger.info(f'Using token: {token}')
   ```

2. **Scope Secrets Appropriately**:
   ```python
   # Server-specific when needed
   server_secret.server_id = server.id

   # Partner-specific for multi-tenancy
   secret.partner_id = partner.id

   # Global only when truly global
   secret.server_id = False
   secret.partner_id = False
   ```

3. **Rotate Regularly**:
   ```python
   # Quarterly rotation schedule
   # 1. Generate new secret
   # 2. Update Tower record
   # 3. Test operations
   # 4. Revoke old secret
   ```

4. **Audit Secret Usage**:
   ```python
   # Track secret usage in command logs
   logs = env['cx.tower.command.log'].search([
       ('code', 'ilike', 'github_token')
   ])
   ```

## Network Security

### Firewall Configuration

**Odoo Server**:
```bash
# Allow HTTPS
ufw allow 443/tcp

# Allow Odoo (if different port)
ufw allow 8069/tcp

# Allow SSH (from specific IPs)
ufw allow from 192.168.1.0/24 to any port 22

# Enable firewall
ufw enable
```

**Managed Servers**:
```bash
# Allow SSH from Tower server only
ufw allow from TOWER_IP to any port 22

# Block all other SSH
ufw deny 22/tcp

# Allow necessary application ports
ufw allow 80/tcp
ufw allow 443/tcp
```

### HTTPS Configuration

**Nginx Reverse Proxy**:
```nginx
server {
    listen 443 ssl http2;
    server_name tower.example.com;

    ssl_certificate /etc/letsencrypt/live/tower.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/tower.example.com/privkey.pem;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    # HSTS
    add_header Strict-Transport-Security "max-age=31536000" always;

    location / {
        proxy_pass http://127.0.0.1:8069;
        proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### VPN for Server Management

**Recommended**:
```bash
# Use VPN for accessing managed servers
# Options:
- WireGuard
- OpenVPN
- Tailscale

# Tower server on VPN
# Managed servers on VPN
# All SSH traffic over VPN
```

### IP Whitelisting

```python
# Webhook IP restrictions
webhook_allowed_ips = ['192.168.1.0/24', '10.0.0.5']

# In webhook code
source_ip = headers.get('X-Real-IP')
if source_ip not in webhook_allowed_ips:
    result = {'exit_code': 1, 'message': 'IP not allowed'}
```

## Audit Logging

### What Gets Logged

**Command Logs** (`cx.tower.command.log`):
```python
- Who ran the command (create_uid)
- When it was run (create_date)
- Which server (server_id)
- Command status (command_status)
- Full output (response)
- Any errors (error)
```

**Flight Plan Logs** (`cx.tower.plan.log`):
```python
- Plan execution details
- All command logs in plan
- Variables used
- Success/failure status
```

**Webhook Logs** (`cx.tower.webhook.log`):
```python
- Webhook called
- Payload received
- Result returned
- Source IP (in headers)
```

### Log Retention

```python
# Configure in Settings > Technical > Scheduled Actions
# Or create custom action

@api.model
def _cleanup_old_logs(self):
    """Delete logs older than retention period."""
    retention_days = 90  # Configurable
    cutoff_date = fields.Datetime.now() - timedelta(days=retention_days)

    # Delete old command logs
    old_logs = self.env['cx.tower.command.log'].search([
        ('create_date', '<', cutoff_date)
    ])
    old_logs.unlink()
```

### Monitoring Logs

```python
# Failed commands
failed_commands = env['cx.tower.command.log'].search([
    ('command_status', '!=', 0),
    ('create_date', '>', fields.Datetime.now() - timedelta(days=1))
])

# Failed authentications (in Odoo logs)
# Failed SSH connections
failed_ssh = env['cx.tower.command.log'].search([
    ('error', 'ilike', 'SSH connection error'),
    ('create_date', '>', fields.Datetime.now() - timedelta(hours=1))
])
```

### Exporting Logs

```python
# Export for compliance/analysis
import csv

logs = env['cx.tower.command.log'].search([
    ('create_date', '>', start_date),
    ('create_date', '<', end_date)
])

with open('tower_audit_log.csv', 'w') as f:
    writer = csv.writer(f)
    writer.writerow(['Date', 'User', 'Server', 'Command', 'Status'])
    for log in logs:
        writer.writerow([
            log.create_date,
            log.create_uid.name,
            log.server_id.name,
            log.command_id.name,
            log.command_status
        ])
```

## Security Checklist

### Pre-Production

- [ ] **Authentication**
  - [ ] Strong password policy enabled
  - [ ] MFA enabled for administrators
  - [ ] Default passwords changed
  - [ ] Session timeout configured

- [ ] **Authorization**
  - [ ] User roles properly assigned
  - [ ] Record rules reviewed and tested
  - [ ] Field-level permissions verified
  - [ ] Server assignments configured

- [ ] **Vault**
  - [ ] Encryption key generated and secured
  - [ ] Vault key backed up separately
  - [ ] All credentials encrypted
  - [ ] Key rotation procedure documented

- [ ] **SSH**
  - [ ] SSH keys generated (not passwords)
  - [ ] Host key verification enabled
  - [ ] Keys properly distributed
  - [ ] SSH hardening on all servers

- [ ] **Network**
  - [ ] Firewall rules configured
  - [ ] HTTPS enabled and working
  - [ ] VPN configured (if used)
  - [ ] Unnecessary ports closed

- [ ] **Logging**
  - [ ] Audit logging enabled
  - [ ] Log retention policy set
  - [ ] Log monitoring configured
  - [ ] Log backup procedure ready

### Post-Production

- [ ] **Regular Tasks** (Weekly)
  - [ ] Review audit logs
  - [ ] Check failed logins
  - [ ] Monitor SSH failures
  - [ ] Review user access

- [ ] **Regular Tasks** (Monthly)
  - [ ] User access audit
  - [ ] Review server assignments
  - [ ] Check for security updates
  - [ ] Test backup restoration

- [ ] **Regular Tasks** (Quarterly)
  - [ ] Rotate secrets
  - [ ] Security training
  - [ ] Penetration testing
  - [ ] Policy review

- [ ] **Regular Tasks** (Annually)
  - [ ] Rotate SSH keys
  - [ ] Rotate vault key
  - [ ] Full security audit
  - [ ] DR test

## Incident Response

### Preparation

1. **Incident Response Plan**:
   ```markdown
   - Contact list (security team)
   - Escalation procedures
   - Communication templates
   - Evidence preservation steps
   ```

2. **Monitoring**:
   ```python
   # Set up alerts for:
   - Failed authentication attempts (>5 in 1 hour)
   - Unusual command patterns
   - Unexpected config changes
   - Large data transfers
   ```

### Detection

**Indicators of Compromise**:
- Multiple failed login attempts
- Commands run outside business hours
- Unexpected server additions/deletions
- Configuration changes by unexpected users
- Unusual network traffic

### Response Steps

1. **Contain**:
   ```python
   # Disable compromised accounts
   user.active = False

   # Disable compromised servers
   server.active = False

   # Block suspicious IPs (firewall)
   ```

2. **Investigate**:
   ```python
   # Review audit logs
   logs = env['cx.tower.command.log'].search([
       ('create_uid', '=', suspicious_user.id),
       ('create_date', '>', incident_start)
   ])

   # Check command patterns
   # Review configuration changes
   # Analyze network logs
   ```

3. **Eradicate**:
   ```python
   # Remove threat
   # Patch vulnerabilities
   # Update security rules
   ```

4. **Recover**:
   ```python
   # Restore from clean backup
   # Rotate all credentials
   # Re-enable services
   # Monitor closely
   ```

5. **Document**:
   ```markdown
   - Timeline of events
   - Actions taken
   - Root cause
   - Lessons learned
   - Preventive measures
   ```

## Security Contacts

### Report Security Issues

**DO NOT create public GitHub issues for security vulnerabilities!**

**Contact**:
- Email: security@cetmix.com
- Subject: [SECURITY] Brief description
- Include: Details, steps to reproduce, impact assessment

**Response Time**:
- Initial response: 48 hours
- Fix timeline: Depends on severity
- Public disclosure: After fix is available

## Related Documentation

- [Security Overview](README.md)
- [Deployment Guide](../09-deployment-operations/README.md)
- [API Security](../06-api-references/README.md)
- [Development Security](../08-development-workflows/01-coding-standards.md)

---
title: Odoo Version Upgrades
description: Version-specific upgrade guide for Odoo and Cetmix Tower
author: Cetmix
date: 2025-11-16
version: 1.0
module: cetmix_tower_server
category: upgrades
tags: [odoo-17, odoo-18, odoo-19, migration, version-upgrade]
---

# Odoo Version Upgrades

This guide provides version-specific information for upgrading Cetmix Tower across different Odoo versions, including current features, future migration paths, and best practices.

## Table of Contents

- [Version Overview](#version-overview)
- [Odoo 17.0 (Current)](#odoo-170-current)
- [Odoo 18.0 (Future)](#odoo-180-future)
- [Odoo 19.0+ (Long-term)](#odoo-190-long-term)
- [Custom Module Migration](#custom-module-migration)
- [Testing Strategies](#testing-strategies)
- [Upgrade Checklists](#upgrade-checklists)

## Version Overview

### Support Timeline

| Version | Release Date | Support Until | Tower Status |
|---------|-------------|---------------|--------------|
| Odoo 17.0 | Oct 2023 | Oct 2026 | ✅ Current, Full Support |
| Odoo 18.0 | Oct 2024 (est.) | Oct 2027 (est.) | ⏳ Planned Support |
| Odoo 19.0 | Oct 2025 (est.) | Oct 2028 (est.) | 📋 Future Planning |

### Version Numbering

Odoo uses semantic versioning:
- **Major**: 17.0, 18.0, 19.0 (annual releases)
- **Community**: Free, open-source
- **Enterprise**: Paid, additional features

## Odoo 17.0 (Current)

### Current Features and Capabilities

#### Platform Features

**Python Support**:
- Python 3.10, 3.11, 3.12
- Improved asyncio support
- Better type hints

**PostgreSQL Support**:
- PostgreSQL 12, 13, 14, 15
- Improved query performance
- Better indexing strategies

**Web Interface**:
- Modern, responsive design
- Improved mobile experience
- Better accessibility (WCAG 2.1)

**ORM Improvements**:
- Better computed field caching
- Improved SQL query optimization
- Enhanced Many2many field handling

#### Tower-Specific Features (17.0)

**Core Functionality**:
- ✅ SSH command execution
- ✅ Python code commands
- ✅ Flight plans with dependencies
- ✅ Template rendering with Jinja2
- ✅ Variable management
- ✅ Secret/vault integration
- ✅ Multi-server operations

**Modules Available**:
- `cetmix_tower_server` - Core functionality
- `cetmix_tower_yaml` - YAML import/export
- `cetmix_tower_git` - Git integration
- `cetmix_tower_webhook` - Webhook triggers
- `cetmix_tower_aws` - AWS integration
- `cetmix_tower_ovh` - OVH Cloud integration
- `cetmix_tower_server_queue` - Queue jobs
- `cetmix_tower_server_notify_backend` - Notifications

### Known Issues (17.0)

#### Performance Considerations

1. **Large Variable Sets**
   ```python
   # Issue: Slow rendering with many variables
   # Workaround: Limit variable scope
   variables = server.get_variable_values(required_only)
   ```

2. **Multiple Server Operations**
   ```python
   # Issue: Sequential processing
   # Recommendation: Use queue jobs for parallel execution
   servers.with_delay().run_command(command)
   ```

#### Compatibility Notes

- **Paramiko**: Requires paramiko >= 3.0
- **PyYAML**: Safe loading only (for security)
- **SSH Keys**: Ed25519 fully supported

### Best Practices for 17.0

```python
# ✅ Use modern field syntax
custom_field = fields.Char(
    string="Custom Field",
    compute='_compute_custom_field',
    store=True,
)

# ✅ Use @api.depends for computed fields
@api.depends('field1', 'field2')
def _compute_custom_field(self):
    for record in self:
        record.custom_field = f"{record.field1} - {record.field2}"

# ✅ Use proper context managers
with self.env.cr.savepoint():
    # Database operations
    pass
```

### Recommendations for 17.0

1. **Keep Updated**
   ```bash
   # Regular module updates
   git pull origin 17.0
   odoo-bin -c odoo.conf -d db -u cetmix_tower_server --stop-after-init
   ```

2. **Monitor Deprecations**
   ```bash
   # Check logs for deprecation warnings
   grep -i deprecat /var/log/odoo/odoo.log
   ```

3. **Optimize Performance**
   ```ini
   # odoo.conf optimizations for 17.0
   workers = 4
   max_cron_threads = 2
   limit_memory_soft = 2147483648
   limit_memory_hard = 2684354560
   ```

## Odoo 18.0 (Future)

### Expected Release: October 2024

### Anticipated Features

#### Platform Changes (Based on Odoo Roadmap)

**Python Support**:
- Python 3.11, 3.12, 3.13
- Deprecation of Python 3.10
- Better async/await support

**PostgreSQL Support**:
- PostgreSQL 13+ required
- PostgreSQL 12 deprecated
- New JSONB features

**Web Interface**:
- Further modernization
- Improved OWL framework
- Better web components

**ORM Changes**:
- Improved multi-company support
- Better computed field performance
- Enhanced search capabilities

### Breaking Changes to Anticipate

#### 1. Deprecated APIs

```python
# Odoo 17.0 (works)
@api.multi
def old_method(self):
    pass

# Odoo 18.0 (likely removed)
# Remove @api.multi decorator
def old_method(self):
    pass
```

#### 2. View Syntax Changes

```xml
<!-- Odoo 17.0 -->
<field name="arch" type="xml">
    <tree>
        <field name="name"/>
    </tree>
</field>

<!-- Odoo 18.0 (anticipated change) -->
<!-- Possible new syntax for responsive views -->
<field name="arch" type="xml">
    <tree responsive="1">
        <field name="name" required="1"/>
    </tree>
</field>
```

#### 3. Security Changes

```python
# Odoo 17.0
@api.model
def create(self, vals):
    return super().create(vals)

# Odoo 18.0 (enhanced security)
@api.model_create_multi
def create(self, vals_list):
    # Enhanced security checks
    return super().create(vals_list)
```

### Migration Path: 17.0 → 18.0

#### Pre-Migration Steps

1. **Review Code for Deprecations**
   ```bash
   # Search for deprecated patterns
   grep -r "@api.multi" custom_modules/
   grep -r "@api.one" custom_modules/
   ```

2. **Update Python Dependencies**
   ```bash
   # Update requirements.txt
   pip3 install --upgrade -r requirements.txt
   ```

3. **Test on 18.0 Beta**
   ```bash
   # Create test environment with Odoo 18.0
   git clone https://github.com/odoo/odoo.git --branch 18.0 --depth 1
   # Test Tower modules
   ```

#### Migration Procedure

```bash
# 1. Backup
pg_dump production_database | gzip > backup_pre_18.sql.gz
rsync -av /var/lib/odoo/filestore/ /backups/filestore_pre_18/

# 2. Stop services
sudo systemctl stop odoo

# 3. Update Odoo to 18.0
cd /opt/odoo/odoo
git fetch --all
git checkout 18.0
pip3 install -r requirements.txt

# 4. Update Tower modules
cd /opt/odoo/cetmix-tower
git fetch --all
git checkout 18.0  # When available

# 5. Run migration
odoo-bin -c /etc/odoo/odoo.conf -d production_database \
  -u cetmix_tower_server,cetmix_tower_yaml \
  --stop-after-init

# 6. Start services
sudo systemctl start odoo

# 7. Verify
tail -f /var/log/odoo/odoo.log
```

### Tower Compatibility Plan for 18.0

**Expected Tower Updates**:

1. **Core Module** (`cetmix_tower_server`)
   - Updated ORM patterns
   - Enhanced security
   - Performance improvements
   - New Python library support

2. **YAML Module** (`cetmix_tower_yaml`)
   - Updated YAML parser
   - Enhanced validation
   - Better error handling

3. **Integration Modules**
   - AWS SDK updates
   - OVH API updates
   - Webhook enhancements

**Backward Compatibility**:
- Minor version (18.0.x) updates: Full compatibility
- Major breaking changes: Migration scripts provided

### New Features to Leverage

#### 1. Improved Performance

```python
# Odoo 18.0 - Better batch operations
class CxTowerServer(models.Model):
    _inherit = "cx.tower.server"

    def run_command_batch(self, command, options=None):
        """Optimized batch command execution"""
        # Leverage new batch API
        results = self.env['cx.tower.command.log'].create_batch([
            self._prepare_log_vals(srv, command)
            for srv in self
        ])
        return results
```

#### 2. Enhanced Security

```python
# Odoo 18.0 - Enhanced field-level security
class CxTowerServer(models.Model):
    _inherit = "cx.tower.server"

    ssh_password = fields.Char(
        groups="base.group_system",
        readonly=True,
        copy=False,
        # New in 18.0: encrypted field storage
        encrypted=True,
    )
```

## Odoo 19.0+ (Long-term)

### Long-term Planning

#### Strategic Considerations

1. **Technology Stack**
   - Python 3.12+ required
   - PostgreSQL 14+ required
   - Modern JavaScript (ES2023+)

2. **Architecture Evolution**
   - Microservices support
   - Better API gateway
   - Enhanced multi-tenancy

3. **Cloud Native**
   - Kubernetes-ready
   - Better container support
   - Cloud-native storage

#### Tower Evolution

**Anticipated Features**:
- Enhanced automation
- AI/ML integration
- Improved scaling
- Better monitoring
- Advanced security

**Preparation Steps**:

1. **Code Modernization**
   ```python
   # Use modern Python features
   from typing import List, Optional
   from dataclasses import dataclass

   @dataclass
   class ServerConfig:
       host: str
       port: int
       username: str
       timeout: Optional[int] = 30
   ```

2. **Modular Architecture**
   ```python
   # Prepare for microservices
   class TowerAPIService:
       """Standalone API service for Tower operations"""
       def execute_command(self, server_id: int, command_id: int):
           # Can be extracted to microservice
           pass
   ```

3. **Cloud Readiness**
   ```yaml
   # kubernetes/tower-deployment.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: tower-server
   spec:
     replicas: 3
     selector:
       matchLabels:
         app: tower
   ```

## Custom Module Migration

### Inheritance Advantages During Upgrades

**Properly inherited modules survive upgrades**:

```python
# custom_tower_extension/models/cx_tower_server.py
from odoo import fields, models

class CxTowerServer(models.Model):
    _inherit = "cx.tower.server"

    # This survives upgrades!
    custom_field = fields.Char("Custom Field")

    def custom_method(self):
        """This survives upgrades!"""
        result = super().custom_method() if hasattr(super(), 'custom_method') else None
        # Your custom logic
        return result
```

### Migration Checklist for Custom Modules

#### Before Upgrade

- [ ] Review custom module code
- [ ] Check for deprecated APIs
- [ ] Update `__manifest__.py` version
- [ ] Test on target Odoo version
- [ ] Document required changes
- [ ] Create backup of current code

#### Code Updates

```python
# Update version in __manifest__.py
{
    'name': 'Custom Tower Extension',
    'version': '18.0.1.0.0',  # Update for target version
    'depends': ['cetmix_tower_server'],
    # ...
}
```

```python
# Update deprecated code patterns
# OLD (Odoo 17.0)
@api.multi
def my_method(self):
    for rec in self:
        # ...

# NEW (Odoo 18.0+)
def my_method(self):
    for rec in self:
        # ...
```

#### After Upgrade

- [ ] Test all custom functionality
- [ ] Verify data integrity
- [ ] Check logs for warnings
- [ ] Update documentation
- [ ] Train users on changes

### Version Detection in Code

```python
from odoo import release

class CxTowerServer(models.Model):
    _inherit = "cx.tower.server"

    def version_specific_method(self):
        """Method with version-specific logic"""
        version = release.version_info

        if version[0] >= 19:
            # Odoo 19+ specific code
            return self._method_v19()
        elif version[0] == 18:
            # Odoo 18 specific code
            return self._method_v18()
        else:
            # Odoo 17 and earlier
            return self._method_v17()
```

## Testing Strategies

### Test Environment Setup

```bash
# Create test database from production
pg_dump production_database | psql test_database_18

# Copy filestore
rsync -av /var/lib/odoo/filestore/production_database/ \
  /var/lib/odoo/filestore/test_database_18/

# Test upgrade
odoo-bin -c test.conf -d test_database_18 \
  -u cetmix_tower_server,custom_tower_extension \
  --test-enable \
  --stop-after-init
```

### Automated Testing

```python
# tests/test_upgrade.py
from odoo.tests import TransactionCase

class TestTowerUpgrade(TransactionCase):

    def test_server_fields_exist(self):
        """Verify all custom fields exist after upgrade"""
        server = self.env['cx.tower.server'].create({
            'name': 'Test Server',
            'ip_v4_address': '192.168.1.1',
            'ssh_username': 'test',
            'custom_field': 'custom_value',  # Your custom field
        })

        self.assertTrue(hasattr(server, 'custom_field'))
        self.assertEqual(server.custom_field, 'custom_value')

    def test_custom_methods_work(self):
        """Verify custom methods work after upgrade"""
        server = self.env['cx.tower.server'].browse(1)
        result = server.custom_method()
        self.assertIsNotNone(result)
```

```bash
# Run tests
odoo-bin -c test.conf -d test_database_18 \
  --test-enable \
  --test-tags custom_tower_extension \
  --stop-after-init
```

### Manual Testing Checklist

#### Core Functionality

- [ ] SSH connection to servers
- [ ] Command execution
- [ ] Flight plan execution
- [ ] Variable rendering
- [ ] File operations
- [ ] Webhook triggers

#### Custom Functionality

- [ ] Custom fields accessible
- [ ] Custom methods work
- [ ] Custom views display correctly
- [ ] Custom reports generate
- [ ] Custom workflows function

#### Integration Testing

- [ ] AWS integration (if used)
- [ ] OVH integration (if used)
- [ ] Git integration (if used)
- [ ] Webhook integration (if used)
- [ ] Queue jobs (if used)

## Upgrade Checklists

### Pre-Upgrade Checklist

**Planning** (2-4 weeks before):
- [ ] Review target version release notes
- [ ] Identify breaking changes
- [ ] Update custom module code
- [ ] Test on staging environment
- [ ] Plan rollback strategy
- [ ] Schedule maintenance window
- [ ] Notify users

**Preparation** (1 week before):
- [ ] Final backup verification
- [ ] Test restore procedure
- [ ] Update documentation
- [ ] Prepare rollback scripts
- [ ] Review monitoring setup

**Day Before**:
- [ ] Create fresh backup
- [ ] Verify backup integrity
- [ ] Test staging upgrade
- [ ] Brief support team
- [ ] Confirm maintenance window

### Upgrade Day Checklist

**Start of Maintenance Window**:
- [ ] Send notification to users
- [ ] Stop services
- [ ] Create final backup
- [ ] Verify backup completed

**Upgrade Process**:
- [ ] Update Odoo code
- [ ] Update Tower modules
- [ ] Update custom modules
- [ ] Run database migration
- [ ] Check for errors

**Verification**:
- [ ] Start services
- [ ] Check logs
- [ ] Test SSH connectivity
- [ ] Run sample commands
- [ ] Verify custom modules
- [ ] Test critical workflows

**Completion**:
- [ ] Monitor for issues
- [ ] Notify users
- [ ] Update documentation
- [ ] Schedule follow-up check

### Post-Upgrade Checklist

**First Day**:
- [ ] Monitor logs continuously
- [ ] Check system resources
- [ ] Verify all integrations
- [ ] Respond to user issues
- [ ] Document any problems

**First Week**:
- [ ] Review error logs daily
- [ ] Collect user feedback
- [ ] Monitor performance
- [ ] Address any issues
- [ ] Update runbooks

**First Month**:
- [ ] Review upgrade process
- [ ] Update procedures
- [ ] Plan improvements
- [ ] Archive upgrade documentation
- [ ] Celebrate success! 🎉

## Best Practices Summary

### ✅ DO:

1. **Test Thoroughly**
   - Use production data copy
   - Test all workflows
   - Involve users

2. **Use Inheritance**
   - Never modify core files
   - Follow Odoo patterns
   - Document customizations

3. **Plan Carefully**
   - Allow extra time
   - Have rollback plan
   - Communicate clearly

4. **Monitor Closely**
   - Watch logs
   - Track performance
   - Respond quickly

### ❌ DON'T:

1. **Skip Testing**
2. **Ignore Deprecation Warnings**
3. **Modify Core Files**
4. **Rush the Process**
5. **Forget Backups**

## Support Resources

- **Odoo Documentation**: [odoo.com/documentation](https://www.odoo.com/documentation)
- **Tower GitHub**: [github.com/cetmix/cetmix-tower](https://github.com/cetmix/cetmix-tower)
- **Community Forum**: Share experiences and get help
- **Professional Support**: Available for complex migrations

## Next Steps

1. Review [Module Upgrades](02-module-upgrades.md) for Tower-specific procedures
2. Test custom modules on target version
3. Create detailed migration plan
4. Schedule testing window
5. Execute upgrade in staging

---

**Remember**: Proper planning and testing are the keys to successful upgrades. Take your time and follow the checklist!

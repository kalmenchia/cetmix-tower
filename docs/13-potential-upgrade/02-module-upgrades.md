---
title: Tower Module Upgrades
description: Guide to upgrading Cetmix Tower modules and maintaining compatibility
author: Cetmix
date: 2025-11-16
version: 1.0
module: cetmix_tower_server
category: upgrades
tags: [module-upgrade, migration, database, compatibility]
---

# Tower Module Upgrades

This guide covers procedures for upgrading Cetmix Tower modules, including database migrations, testing, rollback procedures, and maintaining custom module compatibility.

## Table of Contents

- [Upgrade Types](#upgrade-types)
- [Upgrading Tower Modules](#upgrading-tower-modules)
- [Database Migration Scripts](#database-migration-scripts)
- [Testing After Upgrade](#testing-after-upgrade)
- [Rollback Procedures](#rollback-procedures)
- [Custom Module Compatibility](#custom-module-compatibility)
- [Version-Specific Considerations](#version-specific-considerations)

## Upgrade Types

### Minor Version Upgrades

**Example**: 17.0.1.0.0 → 17.0.1.1.0

**Characteristics**:
- Bug fixes
- Small improvements
- No breaking changes
- Safe to apply

**Risk Level**: 🟢 Low

```bash
# Simple module update
odoo-bin -c odoo.conf -d database -u cetmix_tower_server --stop-after-init
```

### Major Version Upgrades

**Example**: 17.0.1.0.0 → 18.0.1.0.0

**Characteristics**:
- New features
- Possible breaking changes
- Database migrations
- Requires testing

**Risk Level**: 🟡 Medium to 🔴 High

```bash
# Requires careful planning and testing
# See Odoo Version Upgrades guide
```

### Security Patches

**Example**: 17.0.1.0.0 → 17.0.1.0.1

**Characteristics**:
- Security fixes only
- Minimal changes
- Should apply immediately
- Low risk

**Risk Level**: 🟢 Very Low

```bash
# Apply security patches quickly
git pull origin 17.0
odoo-bin -c odoo.conf -d database -u cetmix_tower_server --stop-after-init
```

## Upgrading Tower Modules

### Pre-Upgrade Preparation

#### 1. Review Release Notes

```bash
# Check GitHub releases
https://github.com/cetmix/cetmix-tower/releases

# Review changelog
cat cetmix-tower/CHANGELOG.md

# Check for breaking changes
grep -i "breaking\|deprecated" cetmix-tower/CHANGELOG.md
```

#### 2. Backup Everything

```bash
#!/bin/bash
# backup_before_upgrade.sh

BACKUP_DIR="/backups/tower_upgrade_$(date +%Y%m%d_%H%M%S)"
DB_NAME="production_database"

# Create backup directory
mkdir -p $BACKUP_DIR

# Database backup
echo "Backing up database..."
pg_dump -U odoo $DB_NAME | gzip > $BACKUP_DIR/database.sql.gz

# Filestore backup
echo "Backing up filestore..."
rsync -av /var/lib/odoo/filestore/$DB_NAME/ $BACKUP_DIR/filestore/

# Configuration backup
echo "Backing up configuration..."
cp /etc/odoo/odoo.conf $BACKUP_DIR/odoo.conf.backup

# Custom modules backup
echo "Backing up custom modules..."
tar -czf $BACKUP_DIR/custom_modules.tar.gz /opt/odoo/custom_modules/

# SSH keys backup (encrypted)
echo "Backing up SSH keys..."
tar -czf - /opt/odoo/.ssh/ | openssl enc -aes-256-cbc -salt -out $BACKUP_DIR/ssh_keys.tar.gz.enc

echo "Backup completed: $BACKUP_DIR"
echo "Verify backup integrity before proceeding!"
```

#### 3. Create Test Environment

```bash
# Create test database from production
createdb -U odoo test_upgrade_database
pg_dump production_database | psql test_upgrade_database

# Copy filestore
rsync -av /var/lib/odoo/filestore/production_database/ \
  /var/lib/odoo/filestore/test_upgrade_database/

# Test upgrade on test database first!
```

### Upgrade Procedure

#### Standard Module Update

```bash
#!/bin/bash
# upgrade_tower_modules.sh

# Configuration
ODOO_CONF="/etc/odoo/odoo.conf"
DATABASE="production_database"
MODULES="cetmix_tower_server,cetmix_tower_yaml,cetmix_tower_git,cetmix_tower_webhook"

echo "=== Tower Module Upgrade ==="
echo "Database: $DATABASE"
echo "Modules: $MODULES"
echo ""

# Confirmation
read -p "Proceed with upgrade? (yes/no): " confirm
if [ "$confirm" != "yes" ]; then
    echo "Upgrade cancelled"
    exit 1
fi

# Stop Odoo service
echo "Stopping Odoo service..."
sudo systemctl stop odoo

# Backup
echo "Creating pre-upgrade backup..."
pg_dump -U odoo $DATABASE | gzip > backup_pre_upgrade_$(date +%Y%m%d_%H%M%S).sql.gz

# Update code
echo "Updating Tower modules..."
cd /opt/odoo/cetmix-tower
git pull origin 17.0

# Run upgrade
echo "Running module upgrade..."
odoo-bin -c $ODOO_CONF -d $DATABASE -u $MODULES --stop-after-init

# Check for errors
if [ $? -ne 0 ]; then
    echo "ERROR: Upgrade failed!"
    echo "Check logs: /var/log/odoo/odoo.log"
    exit 1
fi

# Start Odoo service
echo "Starting Odoo service..."
sudo systemctl start odoo

echo "Upgrade completed successfully!"
echo "Monitor logs: sudo journalctl -u odoo -f"
```

#### Upgrade with Custom Modules

```bash
# Upgrade Tower modules and custom modules together
odoo-bin -c /etc/odoo/odoo.conf -d production_database \
  -u cetmix_tower_server,custom_tower_extension,custom_tower_monitoring \
  --stop-after-init
```

### Post-Upgrade Verification

```bash
#!/bin/bash
# verify_upgrade.sh

DATABASE="production_database"

echo "=== Post-Upgrade Verification ==="

# Check module versions
echo "Checking module versions..."
psql -U odoo $DATABASE -c "
  SELECT name, latest_version, state
  FROM ir_module_module
  WHERE name LIKE 'cetmix_tower%' OR name LIKE 'custom_tower%'
  ORDER BY name;
"

# Check for errors in logs
echo ""
echo "Recent errors in logs:"
sudo journalctl -u odoo --since "5 minutes ago" | grep -i error | tail -20

# Check database health
echo ""
echo "Database health check..."
psql -U odoo $DATABASE -c "
  SELECT
    schemaname,
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size
  FROM pg_tables
  WHERE schemaname NOT IN ('pg_catalog', 'information_schema')
  ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC
  LIMIT 10;
"

echo ""
echo "Verification completed!"
```

## Database Migration Scripts

### Understanding Migration Scripts

Migration scripts run automatically during module upgrades:

```
custom_tower_extension/
├── __manifest__.py
├── models/
└── migrations/
    ├── 17.0.1.1.0/
    │   ├── pre-migrate.py
    │   └── post-migrate.py
    └── 17.0.1.2.0/
        ├── pre-migrate.py
        └── post-migrate.py
```

### Pre-Migration Scripts

Run **before** module upgrade:

```python
# custom_tower_extension/migrations/17.0.1.1.0/pre-migrate.py

def migrate(cr, version):
    """
    Pre-migration script for version 17.0.1.1.0

    Runs before module upgrade.
    Use for:
    - Adding columns with default values
    - Renaming columns
    - Preparing data for new structure

    Args:
        cr: Database cursor
        version: Current installed version (before upgrade)
    """
    if not version:
        # Fresh install, skip migration
        return

    # Example: Add new column with default value
    cr.execute("""
        ALTER TABLE cx_tower_server
        ADD COLUMN IF NOT EXISTS environment VARCHAR(20) DEFAULT 'dev'
    """)

    # Example: Rename column
    cr.execute("""
        DO $$
        BEGIN
            IF EXISTS (
                SELECT 1 FROM information_schema.columns
                WHERE table_name='cx_tower_server'
                AND column_name='old_field_name'
            ) THEN
                ALTER TABLE cx_tower_server
                RENAME COLUMN old_field_name TO new_field_name;
            END IF;
        END $$;
    """)

    # Example: Migrate data
    cr.execute("""
        UPDATE cx_tower_server
        SET environment = 'prod'
        WHERE name LIKE '%production%'
    """)
```

### Post-Migration Scripts

Run **after** module upgrade:

```python
# custom_tower_extension/migrations/17.0.1.1.0/post-migrate.py
from odoo import api, SUPERUSER_ID

def migrate(cr, version):
    """
    Post-migration script for version 17.0.1.1.0

    Runs after module upgrade.
    Use for:
    - Data migrations using ORM
    - Creating records
    - Updating computed fields
    - Setting up configurations

    Args:
        cr: Database cursor
        version: Version before upgrade
    """
    if not version:
        # Fresh install, skip migration
        return

    env = api.Environment(cr, SUPERUSER_ID, {})

    # Example: Update servers based on tags
    servers = env['cx.tower.server'].search([])
    for server in servers:
        # Determine environment from tags
        tag_names = server.tag_ids.mapped('name')
        if 'production' in tag_names:
            server.environment = 'prod'
        elif 'staging' in tag_names:
            server.environment = 'staging'
        elif 'development' in tag_names:
            server.environment = 'dev'

    # Example: Create default records
    if not env['custom.datacenter'].search([('name', '=', 'Default')]):
        env['custom.datacenter'].create({
            'name': 'Default',
            'location': 'Default Location',
        })

    # Example: Update computed fields
    servers = env['cx.tower.server'].search([])
    servers._compute_health_status()

    # Example: Execute initialization commands
    default_command = env.ref('custom_tower_extension.command_initialize')
    if default_command:
        for server in servers:
            if not server.custom_initialized:
                server.run_command(default_command)
```

### Migration Best Practices

```python
# ✅ GOOD: Check if migration is needed
def migrate(cr, version):
    if not version:
        return  # Fresh install

    # Safe column addition
    cr.execute("""
        ALTER TABLE cx_tower_server
        ADD COLUMN IF NOT EXISTS custom_field VARCHAR(100)
    """)

# ❌ BAD: No version check
def migrate(cr, version):
    # This runs on fresh installs too!
    cr.execute("ALTER TABLE cx_tower_server ADD COLUMN custom_field VARCHAR(100)")
```

```python
# ✅ GOOD: Use transactions
def migrate(cr, version):
    if not version:
        return

    # Use savepoint for safe rollback
    with cr.savepoint():
        cr.execute("UPDATE cx_tower_server SET environment = 'prod' WHERE ...")

# ❌ BAD: No transaction management
def migrate(cr, version):
    cr.execute("UPDATE cx_tower_server SET environment = 'prod' WHERE ...")
    # If this fails, database is in inconsistent state
```

### Complex Migration Example

```python
# custom_tower_extension/migrations/17.0.2.0.0/post-migrate.py
"""
Complex migration: Restructure monitoring data
- Move monitoring logs to separate table
- Update server references
- Create monitoring configurations
"""
import logging
from odoo import api, SUPERUSER_ID

_logger = logging.getLogger(__name__)

def migrate(cr, version):
    """Migrate monitoring data to new structure"""
    if not version:
        return

    env = api.Environment(cr, SUPERUSER_ID, {})

    _logger.info("Starting monitoring data migration...")

    # Step 1: Create new monitoring log table (if needed)
    cr.execute("""
        CREATE TABLE IF NOT EXISTS custom_monitoring_log (
            id SERIAL PRIMARY KEY,
            server_id INTEGER REFERENCES cx_tower_server(id) ON DELETE CASCADE,
            check_date TIMESTAMP,
            cpu_usage NUMERIC,
            memory_usage NUMERIC,
            disk_usage NUMERIC,
            status VARCHAR(20),
            create_uid INTEGER,
            create_date TIMESTAMP,
            write_uid INTEGER,
            write_date TIMESTAMP
        )
    """)

    # Step 2: Migrate existing monitoring data
    cr.execute("""
        INSERT INTO custom_monitoring_log
        (server_id, check_date, cpu_usage, memory_usage, status, create_date)
        SELECT
            id,
            last_monitoring_check,
            cpu_usage,
            memory_usage,
            monitoring_status,
            write_date
        FROM cx_tower_server
        WHERE last_monitoring_check IS NOT NULL
    """)

    rows_migrated = cr.rowcount
    _logger.info(f"Migrated {rows_migrated} monitoring records")

    # Step 3: Create default monitoring configurations
    MonitoringConfig = env['custom.monitoring.config']
    servers = env['cx.tower.server'].search([('monitoring_enabled', '=', True)])

    for server in servers:
        if not server.monitoring_config_id:
            config = MonitoringConfig.create({
                'name': f'Config for {server.name}',
                'server_id': server.id,
                'check_interval': 5,
                'cpu_threshold': 80.0,
                'memory_threshold': 80.0,
                'disk_threshold': 90.0,
            })
            server.monitoring_config_id = config.id
            _logger.info(f"Created monitoring config for {server.name}")

    # Step 4: Update server statistics
    cr.execute("""
        UPDATE cx_tower_server s
        SET monitoring_log_count = (
            SELECT COUNT(*)
            FROM custom_monitoring_log
            WHERE server_id = s.id
        )
    """)

    _logger.info("Monitoring data migration completed successfully")
```

## Testing After Upgrade

### Automated Testing

```python
# custom_tower_extension/tests/test_post_upgrade.py
from odoo.tests import TransactionCase, tagged

@tagged('post_install', 'at_install')
class TestPostUpgrade(TransactionCase):
    """Test suite for post-upgrade verification"""

    def setUp(self):
        super().setUp()
        self.Server = self.env['cx.tower.server']
        self.Command = self.env['cx.tower.command']

    def test_custom_fields_exist(self):
        """Verify custom fields exist and are accessible"""
        server = self.Server.create({
            'name': 'Test Server',
            'ip_v4_address': '192.168.1.1',
            'ssh_username': 'test',
            'environment': 'dev',
            'custom_field': 'test_value',
        })

        self.assertTrue(hasattr(server, 'environment'))
        self.assertTrue(hasattr(server, 'custom_field'))
        self.assertEqual(server.environment, 'dev')
        self.assertEqual(server.custom_field, 'test_value')

    def test_computed_fields_work(self):
        """Verify computed fields calculate correctly"""
        server = self.Server.create({
            'name': 'Test Server',
            'ip_v4_address': '192.168.1.1',
            'ssh_username': 'test',
            'cpu_usage': 85.5,
            'memory_usage': 70.2,
        })

        # Computed field should calculate
        self.assertIsNotNone(server.health_status)
        self.assertEqual(server.health_status, 'warning')  # CPU > 80

    def test_custom_methods_work(self):
        """Verify custom methods execute correctly"""
        server = self.Server.search([], limit=1)
        if server:
            # Test custom method
            result = server.custom_method()
            self.assertIsNotNone(result)

    def test_relations_work(self):
        """Verify relationships work after migration"""
        datacenter = self.env['custom.datacenter'].create({
            'name': 'Test Datacenter',
            'location': 'Test Location',
        })

        server = self.Server.create({
            'name': 'Test Server',
            'ip_v4_address': '192.168.1.1',
            'ssh_username': 'test',
            'datacenter_id': datacenter.id,
        })

        self.assertEqual(server.datacenter_id, datacenter)
        self.assertIn(server, datacenter.server_ids)

    def test_migration_data_integrity(self):
        """Verify data was migrated correctly"""
        # Check that environment was set correctly
        prod_servers = self.Server.search([('environment', '=', 'prod')])
        for server in prod_servers:
            # Production servers should have been migrated
            self.assertIn('production', server.name.lower() + ' '.join(server.tag_ids.mapped('name')).lower())
```

```bash
# Run tests
odoo-bin -c test.conf -d test_database \
  --test-enable \
  --test-tags custom_tower_extension \
  --stop-after-init
```

### Manual Testing Checklist

#### Core Functionality

- [ ] **SSH Connectivity**
  ```python
  # Test in Odoo shell
  server = env['cx.tower.server'].search([], limit=1)
  server.test_ssh_connection()
  ```

- [ ] **Command Execution**
  ```python
  command = env['cx.tower.command'].search([('action', '=', 'ssh_command')], limit=1)
  server.run_command(command)
  ```

- [ ] **Flight Plans**
  ```python
  plan = env['cx.tower.plan'].search([], limit=1)
  server.run_flight_plan(plan)
  ```

- [ ] **Variable Rendering**
  ```python
  variables = server.get_variable_values(['odoo_version', 'server_path'])
  print(variables)
  ```

#### Custom Functionality

- [ ] **Custom Fields**
  - Visible in forms
  - Searchable
  - Computed correctly

- [ ] **Custom Methods**
  - Execute without errors
  - Return expected results

- [ ] **Custom Views**
  - Display correctly
  - XPath modifications work
  - No JavaScript errors

#### Data Integrity

- [ ] **Record Counts**
  ```sql
  -- Before and after should match
  SELECT COUNT(*) FROM cx_tower_server;
  SELECT COUNT(*) FROM cx_tower_command;
  ```

- [ ] **Relationships**
  ```sql
  -- Check foreign key integrity
  SELECT COUNT(*) FROM cx_tower_server WHERE datacenter_id IS NOT NULL;
  ```

- [ ] **Computed Fields**
  ```python
  # Trigger recomputation
  servers = env['cx.tower.server'].search([])
  servers._compute_health_status()
  servers._compute_backup_count()
  ```

## Rollback Procedures

### Quick Rollback

```bash
#!/bin/bash
# rollback_upgrade.sh

BACKUP_DIR="/backups/tower_upgrade_20250116_120000"
DB_NAME="production_database"

echo "=== EMERGENCY ROLLBACK ==="
echo "Backup directory: $BACKUP_DIR"
echo "Database: $DB_NAME"
echo ""

read -p "Proceed with rollback? (yes/no): " confirm
if [ "$confirm" != "yes" ]; then
    echo "Rollback cancelled"
    exit 1
fi

# Stop services
echo "Stopping Odoo..."
sudo systemctl stop odoo nginx

# Drop current database
echo "Dropping current database..."
psql -U postgres -c "DROP DATABASE IF EXISTS $DB_NAME;"

# Restore database
echo "Restoring database from backup..."
psql -U postgres -c "CREATE DATABASE $DB_NAME OWNER odoo;"
gunzip < $BACKUP_DIR/database.sql.gz | psql -U odoo $DB_NAME

# Restore filestore
echo "Restoring filestore..."
rm -rf /var/lib/odoo/filestore/$DB_NAME
rsync -av $BACKUP_DIR/filestore/ /var/lib/odoo/filestore/$DB_NAME/

# Restore configuration
echo "Restoring configuration..."
cp $BACKUP_DIR/odoo.conf.backup /etc/odoo/odoo.conf

# Restore custom modules (if needed)
echo "Restoring custom modules..."
tar -xzf $BACKUP_DIR/custom_modules.tar.gz -C /

# Restore code version
echo "Reverting code..."
cd /opt/odoo/cetmix-tower
git checkout $(cat $BACKUP_DIR/git_commit.txt)

# Start services
echo "Starting services..."
sudo systemctl start odoo nginx

echo "Rollback completed!"
echo "Verify system: sudo journalctl -u odoo -f"
```

### Partial Rollback (Module Only)

```bash
# Rollback specific module without database restore
odoo-bin -c /etc/odoo/odoo.conf -d production_database \
  --update=custom_tower_extension \
  --stop-after-init

# Or uninstall and reinstall
# Note: This will lose module data!
```

## Custom Module Compatibility

### Maintaining Compatibility

#### Version Detection

```python
# custom_tower_extension/__manifest__.py
{
    'name': 'Custom Tower Extension',
    'version': '17.0.1.0.0',  # Must match Odoo series
    'depends': [
        'cetmix_tower_server',
    ],
    'external_dependencies': {
        'python': ['requests>=2.28.0'],
    },
}
```

#### Conditional Code

```python
# Handle version differences
from odoo import release

if release.version_info[0] >= 18:
    # Odoo 18+ code
    from odoo.tools import safe_eval
else:
    # Odoo 17 code
    from odoo.tools.safe_eval import safe_eval
```

### Dependency Management

```python
# custom_tower_extension/__manifest__.py
{
    'depends': [
        'cetmix_tower_server',  # Core Tower module
        'cetmix_tower_yaml',     # Optional: for YAML support
    ],
    # Auto-install if dependencies installed
    'auto_install': False,
}
```

### Testing Compatibility

```bash
# Test on multiple Odoo versions
docker run -it odoo:17.0 /bin/bash
docker run -it odoo:18.0 /bin/bash

# Test custom module
odoo-bin -d test_db -i custom_tower_extension --test-enable
```

## Version-Specific Considerations

### Odoo 17.0

**Current considerations**:
- Fully supported
- Regular updates available
- No breaking changes expected

**Update frequency**: Monthly minor updates

### Odoo 18.0 (When Available)

**Migration considerations**:
- Review API changes
- Update deprecated code
- Test thoroughly
- Plan migration window

**Expected challenges**:
- ORM changes
- View syntax updates
- Security enhancements

### Custom Modules

**Best practices**:
- Follow inheritance patterns
- Avoid deprecated APIs
- Test on each new version
- Maintain migration scripts

## Summary Checklist

### Before Upgrade

- [ ] Review release notes
- [ ] Backup database
- [ ] Backup filestore
- [ ] Backup configuration
- [ ] Test on staging
- [ ] Update custom modules
- [ ] Schedule maintenance

### During Upgrade

- [ ] Stop services
- [ ] Update code
- [ ] Run upgrade
- [ ] Check logs
- [ ] Verify modules
- [ ] Start services

### After Upgrade

- [ ] Run tests
- [ ] Verify functionality
- [ ] Monitor logs
- [ ] Check performance
- [ ] Collect feedback
- [ ] Document changes

---

**Remember**: Always test upgrades in staging first, maintain good backups, and use inheritance for custom modules!

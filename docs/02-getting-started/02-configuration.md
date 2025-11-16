---
title: "Configuration Guide"
version: "1.0"
last_updated: "2025-11-16"
maintained_by: "E-Global SCM Development Team"
status: "Active"
audience: "System Administrators, End Users, Developers"
---

# Configuration Guide

This guide covers the configuration of Cetmix Tower for different user roles and scenarios.

---

## Table of Contents

- [For System Administrators](#for-system-administrators)
  - [Initial Configuration](#initial-configuration)
  - [Settings Configuration](#settings-configuration)
  - [User and Access Rights Setup](#user-and-access-rights-setup)
  - [System Parameters](#system-parameters)
  - [Cron Job Configuration](#cron-job-configuration)
  - [Security Settings](#security-settings)
- [For End Users](#for-end-users)
  - [Personal Preferences](#personal-preferences)
  - [Notification Settings](#notification-settings)
- [For Developers](#for-developers)
  - [Configuration File Parameters](#configuration-file-parameters)
  - [Environment Variables](#environment-variables)
  - [Advanced Configuration Options](#advanced-configuration-options)

---

## For System Administrators

### Initial Configuration

After installing Cetmix Tower, follow these steps to configure the system.

#### Step 1: Verify Installation

1. Login to Odoo with administrator credentials
2. Check that the **Tower** menu appears in the top navigation
3. Navigate to **Settings → Apps**
4. Search for "cetmix tower" and verify modules are installed

#### Step 2: Access Cetmix Tower Settings

**Navigation Path**: Settings → General Settings → Scroll to Cetmix Tower section

Or directly:
**Settings → Tower** (if available in your configuration)

#### Step 3: Configure Basic Settings

On first access, you'll see the main Cetmix Tower configuration panel.

### Settings Configuration

#### Command Timeout

**Location**: Settings → General Settings → Cetmix Tower

**Field**: `cetmix_tower_command_timeout`

**Description**: Maximum time (in seconds) a command can run before being terminated

**Default Value**: 300 seconds (5 minutes)

**Configuration**:

1. Navigate to Settings → General Settings
2. Scroll to **Cetmix Tower** section
3. Set **Command Timeout** value
4. Click **Save**

**Recommended Values**:
- **Quick commands** (system checks): 60-120 seconds
- **Standard operations** (deployments): 300-600 seconds
- **Long-running tasks** (backups, compilations): 1800-3600 seconds

**Example Scenarios**:

```
Scenario: Deploying an application
Recommended timeout: 600 seconds (10 minutes)
Reason: Allows time for git pull, dependency installation, restart

Scenario: Database backup
Recommended timeout: 1800 seconds (30 minutes)
Reason: Large databases require significant time

Scenario: Quick system check
Recommended timeout: 60 seconds
Reason: Commands like 'df -h' should complete quickly
```

**Note**: Commands exceeding this timeout will receive `COMMAND_TIMED_OUT` status and be terminated.

#### Webhook Configuration

**Location**: Settings → General Settings → Cetmix Tower Webhook (if module installed)

Configure webhook endpoints for receiving external events.

**Fields**:
- **Base URL**: Your Odoo instance URL
- **Authentication**: Webhook authentication settings
- **Allowed IPs**: IP whitelist for webhook requests

### User and Access Rights Setup

Cetmix Tower uses a three-tier access control system.

#### Access Levels

| Level | Group | Permissions | Use Case |
|-------|-------|-------------|----------|
| **User** | `group_user` | View and run assigned commands on selected servers | End users, operators |
| **Manager** | `group_manager` | Create/modify servers, commands, and plans | Team leads, DevOps engineers |
| **Root** | `group_root` | Full control over all servers and configuration | System administrators |

#### Configuring User Access

1. **Navigate to Users**
   - Settings → Users & Companies → Users

2. **Select or Create User**
   - Click on existing user or **Create** new

3. **Set Access Rights**
   - Scroll to **Access Rights** section
   - Find **Cetmix Tower** → **Access Level**
   - Select appropriate level:
     - User
     - Manager
     - Root

4. **Save Changes**
   - Click **Save**

#### Role-Based Access Control (RBAC)

Cetmix Tower supports granular access control through **Access Roles**.

**Server Access Roles**:

Servers, Commands, and Flight Plans have these access fields:
- `access_owner_id` - Owner (user who created it)
- `access_user_ids` - Authorized users
- `access_group_ids` - Authorized groups
- `access_include_child` - Include child companies

**Configuring Access Roles**:

1. Open a Server, Command, or Flight Plan
2. Go to **Access** tab
3. Configure:
   - **Owner**: User who owns this resource
   - **Users**: Specific users who can access
   - **Groups**: User groups who can access
   - **Public**: Make accessible to all users

**Example Configuration**:

```
Server: Production Web Server
├── Owner: admin@company.com
├── Users: devops@company.com, developer@company.com
├── Groups: DevOps Team
└── Public: No

This configuration allows:
- admin: Full access (owner)
- devops & developer: Access based on their Tower access level
- DevOps Team members: Access based on their Tower access level
- Other users: No access
```

#### Multi-Company Setup

If using Odoo multi-company:

1. Navigate to **Settings → Users & Companies → Companies**
2. Configure company hierarchy
3. In Cetmix Tower resources (servers, commands):
   - Set **Company** field
   - Enable **Include Child Companies** if needed

### System Parameters

System parameters control advanced behavior. Access via **Settings → Technical → Parameters → System Parameters**.

#### Key Parameters

| Parameter | Key | Default | Description |
|-----------|-----|---------|-------------|
| Command Timeout | `cetmix_tower_server.command_timeout` | 300 | Global command timeout in seconds |

**Viewing Parameters**:

1. Enable Developer Mode
   - Settings → Activate Developer Mode
2. Navigate to Settings → Technical → Parameters → System Parameters
3. Search for "cetmix_tower"

**Modifying Parameters**:

1. Click on parameter to edit
2. Change **Value** field
3. Save

**Note**: Most parameters can be configured via Settings UI. Only modify system parameters directly if you understand the implications.

### Cron Job Configuration

Cetmix Tower includes several scheduled tasks (cron jobs) for automated operations.

#### Available Cron Jobs

**1. Auto Pull Files from Server**

**XML ID**: `cetmix_tower_server.ir_cron_auto_pull_files_from_server`

**Purpose**: Automatically synchronize files from servers to Tower

**Default Schedule**: Daily

**Configuring**:
1. Settings → General Settings → Cetmix Tower
2. Click **Configure Auto Pull Files Cron**
3. Adjust:
   - **Interval Number**: Frequency value (e.g., 1)
   - **Interval Unit**: Time unit (Minutes, Hours, Days)
   - **Active**: Enable/disable
4. Save

**2. Check Zombie Commands**

**XML ID**: `cetmix_tower_server.ir_cron_check_zombie_commands`

**Purpose**: Detect and clean up orphaned command processes

**Default Schedule**: Every 15 minutes

**Configuring**:
1. Settings → General Settings → Cetmix Tower
2. Click **Configure Zombie Commands Cron**
3. Adjust frequency and activation
4. Save

**Recommended**: Keep this active to prevent resource leaks

**3. Run Scheduled Tasks**

**XML ID**: `cetmix_tower_server.ir_cron_run_scheduled_tasks`

**Purpose**: Execute scheduled commands and flight plans

**Default Schedule**: Every 5 minutes

**Configuring**:
1. Settings → General Settings → Cetmix Tower
2. Click **Configure Run Scheduled Tasks Cron**
3. Adjust frequency
4. Save

**Note**: This cron checks for scheduled tasks and executes them. More frequent checks = more precise scheduling.

#### Manual Cron Configuration

For advanced users:

1. Enable Developer Mode
2. Navigate to **Settings → Technical → Automation → Scheduled Actions**
3. Search for "Cetmix Tower"
4. Click on cron job to configure
5. Adjust settings:
   - **Name**: Job description
   - **Active**: Enable/disable
   - **Interval Number**: Frequency
   - **Interval Type**: days, hours, minutes
   - **Number of Calls**: -1 (unlimited) or specific count
   - **Next Execution Date**: When to run next
6. Save

#### Queue Job Configuration

If `cetmix_tower_server_queue` module is installed:

1. Navigate to **Settings → General Settings**
2. Scroll to **Queue Job** section
3. Configure:
   - **Number of Workers**: Parallel job execution
   - **Job timeout**: Maximum execution time
4. Configure channel settings for Tower jobs

### Security Settings

#### SSH Key Management

**Location**: Tower → Configuration → Keys

**Security Best Practices**:

1. **Use SSH Keys Instead of Passwords**
   - Create dedicated SSH keys for Tower
   - Avoid using personal SSH keys
   - Rotate keys regularly

2. **Store Keys Securely**
   - All SSH keys and passwords stored in Tower are encrypted
   - Use the Vault feature for sensitive data
   - Limit access to keys using access roles

3. **Configure Host Keys**
   - Always verify server host keys on first connection
   - Store trusted host keys to prevent MITM attacks

**Creating SSH Keys for Tower**:

```bash
# On Tower server
ssh-keygen -t rsa -b 4096 -C "tower@yourcompany.com" -f ~/.ssh/tower_key

# Copy public key to managed servers
ssh-copy-id -i ~/.ssh/tower_key.pub user@managed-server
```

**Adding SSH Key to Tower**:

1. Navigate to Tower → Configuration → Keys
2. Click **Create**
3. Fill in:
   - **Name**: Descriptive name (e.g., "Production SSH Key")
   - **Code**: Unique identifier
   - **SSH Key**: Paste private key content
   - **Passphrase**: If key is encrypted
4. Configure access (users who can use this key)
5. Save

#### Password Storage

**Vault Feature**: Tower uses encrypted vaults for storing sensitive data.

**Model**: `cx.tower.vault.mixin`

**Fields Automatically Protected**:
- `ssh_password` (Server passwords)
- `host_key` (Server host keys)

**Security Measures**:
- Encryption at rest using Odoo's encryption capabilities
- Access control through RBAC
- Audit logging of secret access

#### Firewall and Network Security

**Recommended Configuration**:

1. **Restrict Odoo Access**
   ```bash
   # Allow only specific IPs to access Odoo
   sudo ufw allow from 192.168.1.0/24 to any port 8069
   ```

2. **Use HTTPS**
   - Configure SSL/TLS for Odoo
   - Use reverse proxy (nginx, Apache)
   - Redirect HTTP to HTTPS

3. **Limit SSH Access**
   - Use firewall rules for outbound SSH
   - Consider VPN for server management
   - Use jump servers/bastion hosts

4. **Network Segmentation**
   - Place Tower in management network
   - Use VLANs to separate environments
   - Implement least-privilege network access

#### Database Security

**PostgreSQL Configuration**:

1. **Limit Database Access**
   ```bash
   # Edit pg_hba.conf
   # Allow only local connections
   local   all   odoo   md5
   host    all   odoo   127.0.0.1/32   md5
   ```

2. **Use Strong Passwords**
   ```sql
   ALTER USER odoo WITH PASSWORD 'strong_random_password';
   ```

3. **Regular Backups**
   ```bash
   # Automated backup script
   pg_dump -U odoo your_database | gzip > backup_$(date +%Y%m%d).sql.gz
   ```

4. **Encrypt Database Connections**
   - Enable SSL in PostgreSQL
   - Configure Odoo to use SSL connections

#### Audit and Logging

**Enable Audit Logging**:

Cetmix Tower logs all significant operations:
- Command executions
- File modifications
- User access
- Configuration changes

**View Logs**:
- **Command Logs**: Tower → Logs → Command Logs
- **Plan Logs**: Tower → Logs → Flight Plan Logs
- **Server Logs**: Tower → Servers → [Server] → Logs tab
- **System Logs**: Check Odoo server logs

**Log Retention**:

Configure log retention in Settings:
1. Navigate to Settings → Technical → Database Structure → Automated Actions
2. Create automated action to archive old logs
3. Or use database queries to clean old records

**Example cleanup query**:
```sql
-- Archive command logs older than 90 days
DELETE FROM cx_tower_command_log WHERE create_date < NOW() - INTERVAL '90 days';
```

---

## For End Users

### Personal Preferences

Configure your personal Tower preferences.

#### Notification Preferences

**Location**: Click on your username → Preferences

**Email Notifications**:
- Configure which Tower events trigger email notifications
- Available if `cetmix_tower_server_notify_backend` module is installed

**Backend Notifications**:
- Real-time notifications in Odoo interface
- Command completion alerts
- Error notifications

**Configuring**:

1. Go to Preferences (click your name → Preferences)
2. Check notification settings
3. Save changes

#### Language and Localization

**Setting UI Language**:

1. Click your username → Preferences
2. **Language**: Select preferred language
3. Save

**Note**: Tower interface will display in selected language (if translation available)

#### Default Views

Configure default views for Tower objects:

1. Navigate to Tower section (e.g., Servers)
2. Select preferred view (Kanban, List, Form)
3. Odoo remembers your preference

### Notification Settings

Configure how you receive Tower notifications.

#### Email Notifications

**Requirements**: `cetmix_tower_server_notify_backend` module installed

**Configuration**:

1. Ensure your email is set in user preferences
2. Tower will send notifications for:
   - Command completion
   - Command failures
   - Flight plan completion
   - Scheduled task results

**Customizing**:

Administrators can configure email templates:
1. Settings → Technical → Email → Templates
2. Search for "Tower"
3. Customize templates as needed

#### In-App Notifications

**Backend Notification System**:

Cetmix Tower integrates with Odoo's notification system.

**Viewing Notifications**:
- Click bell icon in top navigation
- View Tower-related notifications
- Mark as read or dismiss

**Notification Types**:
- **Success**: Command completed successfully
- **Warning**: Command completed with warnings
- **Error**: Command failed
- **Info**: General information updates

---

## For Developers

### Configuration File Parameters

Odoo configuration file (`odoo.conf`) parameters for Cetmix Tower.

#### Basic Configuration

**Location**: `/etc/odoo/odoo.conf` (or your custom path)

**Example Configuration**:

```ini
[options]
# Addons path including Cetmix Tower
addons_path = /usr/lib/python3/dist-packages/odoo/addons,/opt/cetmix-tower

# Database configuration
db_host = localhost
db_port = 5432
db_user = odoo
db_password = your_secure_password
db_name = False

# Server configuration
http_port = 8069
workers = 4
max_cron_threads = 2

# Logging
logfile = /var/log/odoo/odoo.log
log_level = info

# Performance
limit_time_cpu = 600
limit_time_real = 1200
limit_memory_soft = 2147483648
limit_memory_hard = 2684354560

# Security
admin_passwd = your_master_password
list_db = False
```

#### Performance Tuning for Tower

**Worker Configuration**:

```ini
# Recommended for Tower installations
workers = 4
max_cron_threads = 2
```

**Explanation**:
- `workers`: Number of worker processes for handling requests
- `max_cron_threads`: Threads for scheduled tasks (Tower uses crons)

**Calculation**:
```
workers = (num_cpu * 2) + 1
max_cron_threads = min(2, num_cpu)
```

**Memory Limits**:

```ini
# Increase for large Tower deployments
limit_memory_soft = 2147483648  # 2GB soft limit
limit_memory_hard = 2684354560  # 2.5GB hard limit
```

**Time Limits**:

```ini
# Important for long-running commands
limit_time_cpu = 600      # 10 minutes CPU time
limit_time_real = 1200    # 20 minutes real time
```

**Note**: These should be higher than `cetmix_tower_command_timeout` to prevent premature termination.

#### Queue Job Configuration

If using `cetmix_tower_server_queue`:

```ini
# Queue job configuration
[queue_job]
channels = root:4,root.cetmix_tower:2

# Job timeout (should be >= command timeout)
job_timeout = 1800
```

### Environment Variables

Configure Cetmix Tower behavior via environment variables.

#### Supported Environment Variables

**Database Configuration**:

```bash
export PGHOST=localhost
export PGPORT=5432
export PGUSER=odoo
export PGPASSWORD=your_password
export PGDATABASE=your_database
```

**Odoo Configuration**:

```bash
# Override config file path
export ODOO_RC=/path/to/custom/odoo.conf

# Set addons path
export ODOO_ADDONS_PATH=/odoo/addons,/cetmix-tower

# Logging
export ODOO_LOG_LEVEL=debug
```

**Development Variables**:

```bash
# Enable development mode
export ODOO_DEV_MODE=all

# Auto-reload on code changes
export ODOO_AUTO_RELOAD=1
```

**Usage in systemd Service**:

Create `/etc/systemd/system/odoo.service`:

```ini
[Unit]
Description=Odoo
After=network.target postgresql.service

[Service]
Type=simple
User=odoo
Group=odoo
Environment="PGHOST=localhost"
Environment="PGPORT=5432"
ExecStart=/usr/bin/odoo-bin -c /etc/odoo/odoo.conf
StandardOutput=journal+console

[Install]
WantedBy=multi-user.target
```

### Advanced Configuration Options

#### SSH Connection Pool

Configure SSH connection behavior in Python code.

**File**: `cetmix_tower_server/ssh/ssh.py`

**Customization** (advanced users only):

```python
# Connection timeout (in SSHConnection class)
TIMEOUT = 30  # seconds

# Connection retry attempts
MAX_RETRIES = 3

# Connection pool size
POOL_SIZE = 10
```

**Note**: These are code-level constants. Modifying requires code changes.

#### Command Execution Environment

**Environment Variables in Commands**:

When commands execute on remote servers, certain environment variables are available:

```bash
# Available in command execution context
$CX_TOWER_SERVER_ID       # Server ID
$CX_TOWER_SERVER_NAME     # Server name
$CX_TOWER_COMMAND_ID      # Command ID
$CX_TOWER_LOG_ID          # Log record ID
```

**Custom Variables**:

Define custom variables in Tower:
1. Navigate to Tower → Configuration → Variables
2. Create variables
3. Assign values at server level
4. Reference in commands: `${variable_name}`

#### Python Code Evaluation

Commands support Python code execution with restricted environment.

**Available Modules**:

From `cx_tower_command.py`:
- `requests` - HTTP requests
- `json` - JSON operations
- `hashlib` - Hashing functions
- `hmac` - HMAC operations
- `tldextract` - Domain extraction
- `dns` - DNS operations
- `re` - Regular expressions (in variables)

**Example Python Command**:

```python
# Available in Command → Code → Python
result = requests.get('https://api.example.com/status')
log(result.json())
```

**Security**: The Python environment is sandboxed using `safe_eval`. Not all Python features available.

#### Custom Exit Codes

Configure custom exit codes for error handling.

**Flight Plans**:

```
Field: on_error_action
Values:
  - 'e': Exit with command exit code
  - 'ec': Exit with custom exit code
  - 'n': Run next command

Field: custom_exit_code
Type: Integer
Usage: When on_error_action = 'ec'
```

**Per-Line Actions**:

Flight plan lines can override error behavior:
1. Open Flight Plan
2. Click on Line
3. Set **On Error Action**
4. Set **Custom Exit Code** if needed

#### Logging Configuration

**Log Levels**:

Configure in `odoo.conf`:

```ini
# Global log level
log_level = info

# Per-module log levels
log_handler = odoo.addons.cetmix_tower_server:DEBUG
log_handler = odoo.addons.cetmix_tower_git:INFO
```

**Log Rotation**:

```bash
# Configure logrotate for Odoo logs
sudo nano /etc/logrotate.d/odoo

# Add configuration
/var/log/odoo/*.log {
    daily
    rotate 30
    missingok
    compress
    delaycompress
    notifempty
    copytruncate
}
```

#### Performance Monitoring

**Enable PostgreSQL Query Logging**:

```ini
# In odoo.conf
log_db = True
log_db_level = warning
```

**Profile Odoo Performance**:

```bash
# Run with profiling
odoo-bin -c /etc/odoo/odoo.conf --dev=all --log-level=debug
```

**Monitor Resource Usage**:

```bash
# CPU and memory usage
htop

# PostgreSQL performance
sudo -u postgres psql -c "SELECT * FROM pg_stat_activity;"

# Odoo process statistics
ps aux | grep odoo
```

---

## Configuration Best Practices

### 1. Security First

- Use strong passwords for all accounts
- Enable SSL/TLS for Odoo
- Restrict database access
- Use SSH keys instead of passwords
- Regularly rotate credentials
- Enable audit logging

### 2. Performance Optimization

- Configure appropriate worker count
- Enable queue jobs for async operations
- Set realistic command timeouts
- Regular database maintenance (VACUUM, ANALYZE)
- Monitor resource usage

### 3. Access Control

- Follow principle of least privilege
- Use role-based access control
- Regular access audits
- Document access policies
- Remove unused accounts

### 4. Backup and Recovery

- Regular database backups
- Test restore procedures
- Document recovery steps
- Store backups securely
- Retention policy (e.g., 30 days)

### 5. Monitoring and Alerting

- Configure notification preferences
- Monitor cron job execution
- Track command success rates
- Set up alerting for failures
- Regular log review

### 6. Documentation

- Document custom configurations
- Maintain configuration change log
- Document access policies
- Keep runbooks updated
- Share knowledge with team

---

## Configuration Checklist

After completing configuration, verify:

- [ ] Command timeout is appropriate for your use cases
- [ ] User access levels are correctly assigned
- [ ] Cron jobs are configured and active
- [ ] SSH keys are set up and tested
- [ ] Notification preferences are configured
- [ ] Security settings are in place
- [ ] Logging is enabled and working
- [ ] Backup procedures are tested
- [ ] Performance is acceptable
- [ ] Documentation is updated

---

## Next Steps

- [Take Your First Steps →](03-first-steps.md)
- [Review Common Tasks](04-common-tasks.md)
- [Explore Feature Guides](../04-feature-guides/README.md)
- [Return to Getting Started Overview](README.md)

---

**Need Help?**
- Review [Installation Guide](01-installation.md)
- Check [Technical Guides](../07-technical-guides/README.md)
- Visit [Security & Compliance](../10-security-compliance/README.md)

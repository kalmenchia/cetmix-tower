---
title: "Troubleshooting Guide"
version: "1.0"
last_updated: "2025-11-16"
maintained_by: "E-Global SCM Development Team"
status: "Active"
audience: "Administrator,Developer"
---

# Troubleshooting Guide

Comprehensive troubleshooting guide for common Cetmix Tower issues.

## Table of Contents

1. [Connection Issues](#connection-issues)
2. [Command Execution Issues](#command-execution-issues)
3. [Flight Plan Issues](#flight-plan-issues)
4. [File Management Issues](#file-management-issues)
5. [Variable & Secret Issues](#variable--secret-issues)
6. [Performance Issues](#performance-issues)
7. [Database Issues](#database-issues)
8. [Integration Issues](#integration-issues)

---

## Connection Issues

### SSH Connection Failed (Exit Code: 503)

**Symptoms**:
- Cannot connect to server
- Error: "SSH Connection Error"
- Exit code: 503

**Possible Causes**:
1. Incorrect IP address or port
2. SSH service not running on server
3. Firewall blocking connection
4. Incorrect credentials
5. Host key verification failure

**Solutions**:

**1. Verify Server Connectivity**
```bash
# Test network connectivity
ping <server-ip>

# Test SSH port
telnet <server-ip> 22
# or
nc -zv <server-ip> 22
```

**2. Test SSH Manually**
```bash
ssh -p <port> <username>@<server-ip>
```

**3. Check Host Key**
- In Tower: Server → Test Connection button
- Enable "Skip Host Key Verification" temporarily to test
- Retrieve and save correct host key

**4. Verify Credentials**
- Check username is correct
- If using password: verify password
- If using SSH key: ensure key is properly formatted

**5. Check Server Logs**
```bash
# On the server
sudo tail -f /var/log/auth.log  # Ubuntu/Debian
sudo tail -f /var/log/secure     # CentOS/RHEL
```

---

### "Another Command is Already Running" (Exit Code: -201)

**Symptoms**:
- Cannot run command
- Error: "Another command is already running"

**Cause**: Parallel execution disabled for this command

**Solutions**:

1. **Wait for Previous Command**: Check command logs for running commands
2. **Enable Parallel Execution**:
   - Edit command
   - Enable "Allow Parallel Run"
3. **Kill Zombie Commands**:
   - Check for hung commands
   - Manual cleanup via Settings → Technical → Scheduled Actions → "Check zombie commands"

---

## Command Execution Issues

### Command Timeout (Exit Code: -206)

**Symptoms**:
- Command takes too long
- Killed before completion
- Exit code: -206

**Solutions**:

**1. Increase Timeout**
```python
# Settings → Tower → Configuration
# Increase "Command Timeout" value (seconds)
```

**2. Optimize Command**
```bash
# Instead of this (slow):
apt-get update && apt-get upgrade -y

# Do this (faster):
apt-get update && apt-get upgrade -y --quiet
```

**3. Run in Background**
```bash
# For long-running processes
nohup your-command > /tmp/output.log 2>&1 &
echo "Started in background"
```

---

### Sudo Password Issues

**Symptoms**:
- Sudo commands fail
- "sudo: a password is required"

**Solutions**:

**1. Configure Server for Sudo**
```python
# In Server record
use_sudo = 'p'  # With password
```

**2. Configure Passwordless Sudo** (Recommended)
```bash
# On server, edit sudoers
sudo visudo

# Add line:
your-username ALL=(ALL) NOPASSWD: ALL
```

**3. Use SSH Key with Agent Forwarding**
```bash
# Local machine
ssh-add ~/.ssh/id_rsa

# In server config
use_sudo = 'n'  # Without password
```

---

### Python Code Execution Errors (Exit Code: -203)

**Symptoms**:
- Python command fails
- Error in response
- Exit code: -203

**Common Errors**:

**1. Module Not Available**
```python
# Error: name 'some_module' is not defined

# Solution: Use only available libraries
# Check: Command → Python Libraries help text
```

**2. Syntax Error**
```python
# Error: invalid syntax

# Solution: Validate Python code
import ast
code = """your code here"""
ast.parse(code)  # Test locally first
```

**3. Return Format Error**
```python
# Must set result variable
result = {
    'exit_code': 0,
    'message': 'Success'
}
```

---

## Flight Plan Issues

### Plan Line Condition Failed (Exit Code: -205)

**Symptoms**:
- Flight plan line skipped
- Exit code: -205

**Cause**: Line condition evaluated to False

**Solutions**:

**1. Check Variable Values**
```python
# In plan line condition field
{{ deploy_env }} == 'production'

# Verify variable 'deploy_env' is set
# Check value matches exactly (case-sensitive)
```

**2. Debug Condition**
```python
# Create debug command to print variable value
echo "deploy_env = {{ deploy_env }}"
```

**3. Use Correct Operators**
```python
# Supported operators:
{{ var }} == 'value'   # Equals
{{ var }} != 'value'   # Not equals
{{ var }} > 5          # Greater than (numbers)
{{ var }} < 5          # Less than
{{ var }} >= 5         # Greater or equal
{{ var }} <= 5         # Less or equal
```

---

### Flight Plan Empty (Exit Code: -302)

**Symptoms**:
- Cannot run flight plan
- Error: "Flight plan is empty"

**Solution**: Add at least one line to the flight plan

---

### Plan Not Compatible with Server (Exit Code: -306)

**Symptoms**:
- Cannot run plan on server
- Error: "Flight plan not compatible"

**Causes**:
1. Plan restricted to specific servers
2. Plan restricted to specific OS

**Solutions**:

**1. Check Server Compatibility**
```python
# Flight Plan → Servers tab
# If empty: available for all servers
# If filled: only for listed servers
```

**2. Check Command Compatibility**
```python
# Each command in plan may have:
# - Server restrictions
# - OS restrictions
```

---

## File Management Issues

### File Upload Failed (Exit Code: -400)

**Symptoms**:
- Cannot push file to server
- File sync fails

**Solutions**:

**1. Check Server Permissions**
```bash
# On server, verify directory exists and is writable
ls -la /path/to/directory
```

**2. Create Directory if Missing**
```bash
mkdir -p /path/to/directory
chmod 755 /path/to/directory
```

**3. Check Disk Space**
```bash
df -h /path/to/directory
```

---

### File Download Failed (Exit Code: -401)

**Symptoms**:
- Cannot pull file from server
- File not found

**Solutions**:

**1. Verify File Exists**
```bash
# SSH to server
ls -la /full/path/to/file
```

**2. Check Read Permissions**
```bash
chmod 644 /path/to/file  # Make readable
```

**3. Use Sudo if Needed**
```python
# In File record
# Enable sudo for file operations if needed
```

---

### File Already Exists (Exit Code: -402)

**Symptoms**:
- File creation fails
- "File already exists"

**Solutions**:

**1. Change "If File Exists" Behavior**
```python
# In Command using file template
if_file_exists = 'overwrite'  # or 'skip'
```

**2. Delete Existing File First**
```bash
# Add command before file creation
rm -f /path/to/file
```

---

## Variable & Secret Issues

### Variable Not Substituted

**Symptoms**:
- Variable appears as `{{ var_name }}` in output
- Not replaced with value

**Solutions**:

**1. Check Variable Name**
```python
# Must match exactly (case-sensitive)
# Variable: api_key
# Usage: {{ api_key }}  ✓
# Usage: {{ API_KEY }}  ✗
```

**2. Check Variable Value is Set**
```python
# Navigate to: Tower → Variables → Values
# Ensure value exists for:
# - Global, or
# - Specific server
```

**3. Check Variable Scope**
```python
# Variable resolution order:
# 1. Custom values passed to command
# 2. Server-specific values
# 3. Global values
```

---

### Secret Not Replaced

**Symptoms**:
- Secret reference appears in output
- Format: `#!cxtower.secret.NAME!#`

**Solutions**:

**1. Check Secret Exists**
```python
# Navigate to: Tower → Secrets → Keys
# Verify secret with reference 'NAME' exists
```

**2. Check Secret Value is Set**
```python
# Secret Values tab
# Ensure global or server-specific value exists
```

**3. Verify Reference Format**
```python
# Correct: #!cxtower.secret.API_KEY!#
# Wrong: #cxtower.secret.API_KEY#
# Wrong: {{ secret.API_KEY }}
```

---

## Performance Issues

### Slow Command Execution

**Symptoms**:
- Commands take longer than expected
- Server response slow

**Solutions**:

**1. Check Server Load**
```bash
# On server
top
htop
vmstat 1
```

**2. Check Network Latency**
```bash
# From Odoo server to target server
ping <server-ip>
mtr <server-ip>
```

**3. Optimize Command**
```bash
# Add --quiet flags
# Reduce output verbosity
# Use efficient commands
```

**4. Use Queue Jobs**
```python
# For long-running commands
# Install: cetmix_tower_server_queue
# Commands run asynchronously
```

---

### Database Performance

**Symptoms**:
- Slow Tower UI
- Queries timeout

**Solutions**:

**1. Vacuum Database**
```sql
-- As postgres user
VACUUM ANALYZE;
```

**2. Reindex Tables**
```sql
REINDEX DATABASE odoo;
```

**3. Check Indexes**
```sql
-- Find missing indexes
SELECT schemaname, tablename, attname, n_distinct, correlation
FROM pg_stats
WHERE tablename IN ('cx_tower_server', 'cx_tower_command_log')
ORDER BY abs(correlation) DESC;
```

**4. Archive Old Logs**
```python
# Archive command logs older than 90 days
# Settings → Technical → Scheduled Actions
# Create custom cleanup cron
```

---

## Database Issues

### Duplicate Key Errors

**Symptoms**:
- Cannot create record
- "duplicate key value violates unique constraint"

**Solutions**:

**1. Check Existing Records**
```sql
SELECT * FROM cx_tower_variable
WHERE name = 'duplicate_name';
```

**2. Update Instead of Create**
```python
# Use write() instead of create()
existing = env['cx.tower.variable'].search([('name', '=', 'api_key')])
if existing:
    existing.write({'validation_pattern': 'new_pattern'})
else:
    env['cx.tower.variable'].create({'name': 'api_key'})
```

---

### Foreign Key Violations

**Symptoms**:
- Cannot delete record
- "foreign key constraint violation"

**Solutions**:

**1. Check Dependencies**
```sql
-- Find dependent records
SELECT * FROM cx_tower_plan_line
WHERE command_id = <command_to_delete>;
```

**2. Delete in Correct Order**
```python
# Delete dependent records first
plan_lines.unlink()
command.unlink()
```

**3. Archive Instead of Delete**
```python
# Set active = False instead
record.active = False
```

---

## Integration Issues

### Git Integration Fails

**Symptoms**:
- Cannot clone repository
- Git commands fail

**Solutions**:

**1. Check Git Installation**
```bash
# On server
git --version
which git
```

**2. Verify Git Credentials**
```bash
# Test clone manually
git clone <repo-url>
```

**3. Check SSH Keys for Git**
```bash
# Add SSH key to ssh-agent
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa

# Test GitHub connection
ssh -T git@github.com
```

---

### Webhook Not Triggered

**Symptoms**:
- Webhook doesn't execute
- No log entry

**Solutions**:

**1. Check Webhook URL**
```python
# Correct format:
https://your-odoo.com/tower/webhook/<webhook_reference>
```

**2. Test Webhook**
```bash
curl -X POST https://your-odoo.com/tower/webhook/deploy \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"server_id": 1, "action": "deploy"}'
```

**3. Check Webhook Logs**
```python
# Navigate to: Tower → Webhooks → Logs
# View recent webhook calls and responses
```

---

### AWS/OVH Integration Issues

**Symptoms**:
- Cannot connect to cloud provider
- API calls fail

**Solutions**:

**1. Verify API Credentials**
```python
# AWS: Check access key and secret
# OVH: Check application key, secret, consumer key
```

**2. Test API Access**
```python
# AWS
import boto3
ec2 = boto3.client('ec2',
    aws_access_key_id='KEY',
    aws_secret_access_key='SECRET',
    region_name='us-east-1'
)
ec2.describe_instances()

# OVH
import ovh
client = ovh.Client(
    endpoint='ovh-eu',
    application_key='KEY',
    application_secret='SECRET',
    consumer_key='CONSUMER'
)
client.get('/me')
```

**3. Check Network Access**
```bash
# Ensure Odoo server can reach cloud APIs
curl -I https://ec2.amazonaws.com
curl -I https://eu.api.ovh.com
```

---

## Diagnostic Tools

### Enable Debug Mode

**Via URL**:
```
https://your-odoo.com/web?debug=1
```

**Via Settings**:
- Activate Developer Mode
- Settings → Technical menu appears

### View Logs

**Odoo Server Log**:
```bash
tail -f /var/log/odoo/odoo-server.log
```

**Tower Command Logs**:
```python
# Navigate to: Tower → Logs → Command Logs
# Filter by date, server, command
# View full output and errors
```

**PostgreSQL Logs**:
```bash
tail -f /var/log/postgresql/postgresql-14-main.log
```

### Database Queries

**List All Servers**:
```sql
SELECT id, name, ip_v4_address, active, status
FROM cx_tower_server
ORDER BY name;
```

**Recent Command Logs**:
```sql
SELECT
    cl.id,
    cl.start_date,
    s.name as server,
    c.name as command,
    cl.command_status,
    cl.duration
FROM cx_tower_command_log cl
JOIN cx_tower_server s ON s.id = cl.server_id
JOIN cx_tower_command c ON c.id = cl.command_id
ORDER BY cl.start_date DESC
LIMIT 20;
```

**Find Failed Commands**:
```sql
SELECT *
FROM cx_tower_command_log
WHERE command_status < 0
ORDER BY start_date DESC;
```

---

## Getting More Help

If you can't resolve the issue:

1. **Check Documentation**: Review relevant feature guides
2. **Search GitHub Issues**: [https://github.com/cetmix/cetmix-tower/issues](https://github.com/cetmix/cetmix-tower/issues)
3. **Report Bug**: See [Bug Reporting Guide](../12-issues-bugs/10-bug-reporting-guide.md)
4. **Community Support**: Odoo forums and discussions

---

## Related Documentation

- [Bug Reporting Guide](../12-issues-bugs/10-bug-reporting-guide.md)
- [Security Guidelines](../10-security-compliance/01-security-guidelines.md)
- [Deployment Guide](01-deployment-guide.md)
- [Common Tasks](../02-getting-started/04-common-tasks.md)

---

**Last Updated**: 2025-11-16
**Next Review**: 2026-02-16
**Maintained By**: E-Global SCM Development Team

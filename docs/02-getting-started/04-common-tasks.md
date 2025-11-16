---
title: "Common Tasks Quick Reference"
version: "1.0"
last_updated: "2025-11-16"
maintained_by: "E-Global SCM Development Team"
status: "Active"
audience: "All Users"
---

# Common Tasks Quick Reference

This guide provides step-by-step instructions for frequently performed tasks in Cetmix Tower.

---

## Table of Contents

1. [Adding a New Server](#1-adding-a-new-server)
2. [Creating and Running Commands](#2-creating-and-running-commands)
3. [Creating a Simple Flight Plan](#3-creating-a-simple-flight-plan)
4. [Managing Files](#4-managing-files)
5. [Setting Up Variables](#5-setting-up-variables)
6. [Storing Secrets](#6-storing-secrets)
7. [Scheduling Tasks](#7-scheduling-tasks)
8. [Viewing Logs](#8-viewing-logs)
9. [Managing Users and Access](#9-managing-users-and-access)
10. [Exporting and Importing Configurations](#10-exporting-and-importing-configurations)

---

## 1. Adding a New Server

**Goal**: Add a server to Tower for management

**Time**: 3-5 minutes

**Prerequisites**: SSH access to target server

### Steps

1. **Navigate to Servers**
   ```
   Tower → Servers → Servers → Create
   ```

2. **Fill Required Fields**
   ```
   Name: [Descriptive name, e.g., "Production Web Server"]
   Code: [Unique identifier, e.g., "prod-web-01"]
   IP Address: [Server IP or hostname, e.g., "192.168.1.100"]
   SSH Port: 22
   SSH Username: [SSH user, e.g., "ubuntu"]
   ```

3. **Configure Authentication**

   **Option A: SSH Key (Recommended)**
   - Key field → Select existing key or create new

   **Option B: Password**
   - SSH Password field → Enter password

4. **Optional: Add Tags**
   ```
   Tags tab → Add → Select tags (e.g., "Production", "Web")
   ```

5. **Optional: Set Operating System**
   ```
   OS field → Select (e.g., "Ubuntu 22.04")
   ```

6. **Save**
   ```
   Click Save button
   ```

7. **Test Connection**
   ```
   Click "Test Connection" button
   Wait for confirmation
   ```

### Expected Result

✓ Server appears in Servers list
✓ Test connection succeeds
✓ Server is ready for command execution

### Troubleshooting

- **Connection fails**: Verify IP, port, username, and authentication credentials
- **Host key error**: Accept host key when prompted or add to known hosts
- **Permission denied**: Check SSH user permissions on target server

### Related Documentation

- [First Steps: Adding Your First Server](03-first-steps.md#adding-your-first-server)
- [Server Management Guide](../04-feature-guides/01-server-management.md)

---

## 2. Creating and Running Commands

**Goal**: Create a new command and execute it on a server

**Time**: 3-5 minutes

**Prerequisites**: At least one server configured in Tower

### Creating a Command

#### Steps

1. **Navigate to Commands**
   ```
   Tower → Commands → Create
   ```

2. **Basic Information**
   ```
   Name: [Descriptive name, e.g., "Check System Load"]
   Code: [Unique identifier, e.g., "check-load"]
   Reference: [Reference code, e.g., "sys_load"]
   ```

3. **Write Command Code**
   ```
   Check "Shell" checkbox
   Code field → Enter command, e.g.:
   ```
   ```bash
   uptime
   free -h
   df -h
   ```

4. **Configure Options**
   ```
   Shell: ✓ Checked
   Use Sudo: ☐ (check if needed)
   Timeout: 60 (seconds, optional)
   Allow Parallel Run: ☐ (optional)
   ```

5. **Optional: Assign to Specific Servers**
   ```
   Servers tab → Add → Select servers
   Or leave empty for all servers
   ```

6. **Save**
   ```
   Click Save
   ```

### Running a Command

#### Method 1: From Command Form

1. **Open Command**
   ```
   Tower → Commands → [Select command]
   ```

2. **Run Command**
   ```
   Click "Run Command" button
   ```

3. **Select Server(s)**
   ```
   Choose one or more servers
   Click Run
   ```

4. **View Result**
   ```
   Navigate to Tower → Logs → Command Logs
   Find your execution
   Review output
   ```

#### Method 2: From Server Form

1. **Open Server**
   ```
   Tower → Servers → [Select server]
   ```

2. **Run Command**
   ```
   Click "Run Command" button (or action menu)
   ```

3. **Select Command**
   ```
   Choose command from list
   Click Run
   ```

### Expected Result

✓ Command executes on selected server(s)
✓ Log entry created in Command Logs
✓ Output visible in log details

### Common Command Examples

**Check Disk Space**:
```bash
df -h
```

**View Running Processes**:
```bash
ps aux | head -20
```

**Check Service Status**:
```bash
sudo systemctl status nginx
```

**Update Package List**:
```bash
sudo apt update
```

**View System Info**:
```bash
uname -a
cat /etc/os-release
```

### Troubleshooting

- **Command not found**: Verify command syntax, check if binary exists on target system
- **Permission denied**: Enable "Use Sudo" option in command settings
- **Timeout**: Increase timeout value in command settings

### Related Documentation

- [Commands Guide](../04-feature-guides/02-commands-execution.md)
- [First Steps: Creating Your First Command](03-first-steps.md#creating-your-first-command)

---

## 3. Creating a Simple Flight Plan

**Goal**: Create a workflow combining multiple commands

**Time**: 5-10 minutes

**Prerequisites**: Commands already created in Tower

### Steps

1. **Navigate to Flight Plans**
   ```
   Tower → Flight Plans → Create
   ```

2. **Basic Information**
   ```
   Name: [Descriptive name, e.g., "Deploy Web Application"]
   Code: [Unique identifier, e.g., "deploy-web-app"]
   Reference: [Reference code, e.g., "deploy_web"]
   ```

3. **Configure Error Handling**
   ```
   On Error: "Exit with command exit code" (or choose alternative)
   ```

4. **Add Flight Plan Lines**

   **Line 1: Update Code**
   ```
   Lines tab → Add a line
   Sequence: 10
   Name: Pull Latest Code
   Command: [Select: "Git Pull" command]
   On Error Action: Exit with exit code
   ```

   **Line 2: Install Dependencies**
   ```
   Lines tab → Add a line
   Sequence: 20
   Name: Install Dependencies
   Command: [Select: "NPM Install" command]
   On Error Action: Exit with exit code
   ```

   **Line 3: Restart Service**
   ```
   Lines tab → Add a line
   Sequence: 30
   Name: Restart Application
   Command: [Select: "Restart Service" command]
   On Error Action: Exit with exit code
   ```

5. **Optional: Assign to Servers**
   ```
   Servers tab → Add → Select servers
   Or leave empty for all servers
   ```

6. **Save**
   ```
   Click Save
   ```

7. **Run Flight Plan**
   ```
   Click "Run Plan" button
   Select server(s)
   Click Run
   ```

### Expected Result

✓ Flight plan executes all lines in sequence
✓ Each command runs on selected server
✓ Flight plan log created showing overall result
✓ Individual command logs created for each line

### Flight Plan Best Practices

1. **Sequence Numbers**: Use increments of 10 (10, 20, 30) to allow inserting lines later
2. **Descriptive Names**: Each line should clearly state what it does
3. **Error Handling**: Consider whether to continue or stop on errors
4. **Conditions**: Use conditions to control when lines execute
5. **Testing**: Test flight plan on development servers first

### Example Flight Plans

**System Update Workflow**:
```
1. Check Current Version
2. Backup Configuration
3. Run apt update
4. Run apt upgrade
5. Restart Services
6. Verify Services Running
```

**Backup Workflow**:
```
1. Stop Application
2. Dump Database
3. Archive Files
4. Copy to Backup Server
5. Start Application
6. Verify Backup Integrity
```

### Troubleshooting

- **Plan stops unexpectedly**: Check error handling settings, review failed command log
- **Line doesn't execute**: Check line conditions, verify sequence order
- **Wrong order**: Adjust sequence numbers (e.g., 10, 20, 30)

### Related Documentation

- [Flight Plans Guide](../04-feature-guides/03-flight-plans-workflows.md)
- [Advanced Workflows](../07-technical-guides/02-flight-plans-advanced.md)

---

## 4. Managing Files

**Goal**: Upload, download, and synchronize files between Tower and servers

**Time**: 3-5 minutes per file

**Prerequisites**: Server configured with working SSH connection

### Uploading a File to Server

#### Steps

1. **Navigate to Files**
   ```
   Tower → Files → Create
   ```

2. **Basic Information**
   ```
   Name: [Descriptive name, e.g., "Nginx Configuration"]
   Code: [Unique identifier, e.g., "nginx-conf"]
   ```

3. **Set File Details**
   ```
   Server: [Select target server]
   Remote Path: [Full path on server, e.g., "/etc/nginx/nginx.conf"]
   ```

4. **Upload File Content**

   **Option A: Upload File**
   ```
   Content tab → Upload File button
   Select file from your computer
   ```

   **Option B: Paste Content**
   ```
   Content tab → Content field
   Paste or type file content
   ```

5. **Configure Options**
   ```
   Auto Pull: ☐ (check to sync from server automatically)
   Pull Interval: [If auto pull enabled, set interval]
   Use Sudo: ☐ (check if needed for file operations)
   ```

6. **Save**
   ```
   Click Save
   ```

7. **Push to Server**
   ```
   Click "Push to Server" button (or action menu)
   Confirm action
   ```

### Expected Result

✓ File uploaded to specified path on server
✓ File record created in Tower
✓ Sync status shows "Synchronized"

### Downloading a File from Server

#### Steps

1. **Create File Record**
   ```
   Tower → Files → Create
   Name: [e.g., "Application Log"]
   Server: [Select server]
   Remote Path: [e.g., "/var/log/app/app.log"]
   ```

2. **Pull from Server**
   ```
   Click "Pull from Server" button
   Wait for completion
   ```

3. **View Content**
   ```
   Content tab → View file content
   ```

4. **Download to Local**
   ```
   Click "Download" button (if available)
   Or copy content from Content field
   ```

### Using File Templates

**Goal**: Reusable file templates with variable substitution

#### Steps

1. **Create File Template**
   ```
   Tower → Files → Templates → Create
   Name: [e.g., "Nginx Site Configuration Template"]
   ```

2. **Add Template Content**
   ```
   Content field → Add content with variables:
   ```
   ```nginx
   server {
       listen 80;
       server_name ${domain_name};
       root /var/www/${app_name};
   }
   ```

3. **Save Template**

4. **Create File from Template**
   ```
   Tower → Files → Create
   Template: [Select your template]
   Server: [Select server]
   Remote Path: [e.g., "/etc/nginx/sites-available/mysite"]
   ```

5. **Set Variable Values**
   ```
   Variables are populated from server's variable values
   Or enter manually if prompted
   ```

6. **Push to Server**

### Expected Result

✓ Template created with variable placeholders
✓ File created with variables replaced
✓ File pushed to server with actual values

### Common File Operations

**Upload configuration file**:
```
Remote Path: /etc/nginx/nginx.conf
Use Sudo: ✓
```

**Download log file**:
```
Remote Path: /var/log/syslog
Auto Pull: ✓ (to monitor)
Pull Interval: 3600 (hourly)
```

**Deploy application code**:
```
Remote Path: /var/www/app/index.html
Content: [Upload file]
```

### Troubleshooting

- **Permission denied**: Enable "Use Sudo" option
- **File not found**: Verify remote path is correct and exists
- **Push fails**: Check SSH connection, verify write permissions

### Related Documentation

- [File Management Guide](../04-feature-guides/04-file-management-sync.md)
- [File Templates](../04-feature-guides/04-file-management-sync.md#file-templates)

---

## 5. Setting Up Variables

**Goal**: Create reusable variables for commands and files

**Time**: 2-3 minutes per variable

**Prerequisites**: Understanding of where variables will be used

### Creating a String Variable

#### Steps

1. **Navigate to Variables**
   ```
   Tower → Configuration → Variables → Create
   ```

2. **Basic Information**
   ```
   Name: [Descriptive name, e.g., "Application Name"]
   Code: [Unique identifier, e.g., "app_name"]
   Reference: [Reference in commands, e.g., "app"]
   Type: String
   ```

3. **Optional: Add Validation**
   ```
   Validation Pattern: [Regex, e.g., "^[a-z0-9_]+$"]
   Validation Message: [e.g., "Only lowercase, numbers, and underscores allowed"]
   ```

4. **Optional: Add Expression**
   ```
   Applied Expression: [Python code to transform value]
   Example: result = value.lower().replace(' ', '_')
   ```

5. **Save**

### Creating an Options Variable

#### Steps

1. **Create Variable**
   ```
   Tower → Configuration → Variables → Create
   Name: Environment
   Code: environment
   Reference: env
   Type: Options
   ```

2. **Add Options**
   ```
   Options tab → Add a line
   Value: development

   Add a line
   Value: staging

   Add a line
   Value: production
   ```

3. **Save**

### Assigning Variable Values to Servers

#### Steps

1. **Open Server**
   ```
   Tower → Servers → [Select server]
   ```

2. **Add Variable Value**
   ```
   Variables tab → Add a line
   Variable: [Select variable, e.g., "Application Name"]
   Value: [Enter value, e.g., "my-app"]
   ```

3. **Repeat for All Variables**

4. **Save Server**

### Using Variables in Commands

**Reference Syntax**: `${variable_code}`

**Example Command**:
```bash
#!/bin/bash
echo "Deploying ${app_name} to ${environment} environment"
cd /var/www/${app_name}
git pull origin ${branch_name}
```

**When Run on Server**:
- `app_name = "my-app"`
- `environment = "production"`
- `branch_name = "main"`

**Actual Execution**:
```bash
#!/bin/bash
echo "Deploying my-app to production environment"
cd /var/www/my-app
git pull origin main
```

### Expected Result

✓ Variable created and available system-wide
✓ Server-specific values assigned
✓ Variables properly substituted in commands
✓ Validation enforced (if configured)

### Common Variable Patterns

**Deployment Variables**:
```
app_name (string)
environment (options: dev, staging, prod)
branch_name (string)
domain_name (string)
```

**Database Variables**:
```
db_host (string)
db_port (string, default: 5432)
db_name (string)
db_user (string)
```

**Path Variables**:
```
app_root (string, e.g., /var/www/app)
log_path (string, e.g., /var/log/app)
config_path (string, e.g., /etc/app)
```

### Troubleshooting

- **Variable not substituted**: Check reference syntax `${variable_code}`, verify variable assigned to server
- **Validation fails**: Check regex pattern, verify value matches pattern
- **Wrong value used**: Verify server-specific value is set, check for typos in variable code

### Related Documentation

- [Variables Guide](../04-feature-guides/05-variables-parameters.md)
- [Variable Validation](../07-technical-guides/03-variable-system.md)

---

## 6. Storing Secrets

**Goal**: Securely store SSH keys, passwords, and sensitive data

**Time**: 2-5 minutes

**Prerequisites**: Admin or Root access level

### Storing SSH Keys

#### Steps

1. **Navigate to Keys**
   ```
   Tower → Configuration → Keys → Create
   ```

2. **Basic Information**
   ```
   Name: [Descriptive name, e.g., "Production SSH Key"]
   Code: [Unique identifier, e.g., "prod-ssh-key"]
   Reference: [Reference code, e.g., "prod_key"]
   ```

3. **Add SSH Key**
   ```
   SSH Key field → Paste private key content
   ```

   **Example** (generating new key):
   ```bash
   ssh-keygen -t rsa -b 4096 -f ~/.ssh/tower_prod_key
   cat ~/.ssh/tower_prod_key
   # Copy output
   ```

4. **Optional: Add Passphrase**
   ```
   Passphrase field → Enter if key is encrypted
   ```

5. **Configure Access**
   ```
   Access tab → Set who can use this key
   Users: [Add authorized users]
   Groups: [Add authorized groups]
   ```

6. **Save**

7. **Deploy Public Key to Servers**
   ```bash
   ssh-copy-id -i ~/.ssh/tower_prod_key.pub user@server
   ```

### Using SSH Keys in Servers

#### Steps

1. **Open Server**
   ```
   Tower → Servers → [Select server]
   ```

2. **Assign Key**
   ```
   Key field → Select the key you created
   ```

3. **Remove Password** (if previously set)
   ```
   SSH Password field → Clear
   ```

4. **Save**

5. **Test Connection**
   ```
   Click "Test Connection"
   Verify success
   ```

### Storing Passwords Securely

**SSH Passwords**: Stored in server record, automatically encrypted

#### Steps

1. **Open Server**
   ```
   Tower → Servers → [Select server]
   ```

2. **Set Password**
   ```
   SSH Password field → Enter password
   ```

3. **Save**
   ```
   Password is encrypted in database
   ```

### Vault Features

Tower automatically encrypts these fields:
- `ssh_password` - Server SSH passwords
- `host_key` - Server host keys
- Any custom fields using `cx.tower.vault.mixin`

**Encryption Details**:
- Uses Odoo's built-in encryption
- Stored encrypted at rest
- Decrypted only during use
- Access controlled by RBAC

### Expected Result

✓ SSH keys stored securely in Tower
✓ Keys accessible only to authorized users
✓ Servers authenticate using keys
✓ Passwords encrypted in database

### Best Practices

1. **Use SSH Keys Over Passwords**
   - More secure
   - No password storage needed
   - Supports key rotation

2. **Limit Key Access**
   - Configure access control on key records
   - Only grant access to necessary users
   - Use different keys for different environments

3. **Rotate Keys Regularly**
   - Create new keys every 90-180 days
   - Update server configurations
   - Delete old keys

4. **Use Dedicated Keys**
   - Create Tower-specific keys
   - Don't use personal SSH keys
   - Label keys clearly

5. **Monitor Key Usage**
   - Review access logs
   - Audit who uses which keys
   - Remove unused keys

### Troubleshooting

- **Key authentication fails**: Verify public key is on server, check key format
- **Permission denied**: Ensure private key permissions are correct (600)
- **Can't see key**: Check access control settings on key record

### Related Documentation

- [Security Guide](../10-security-compliance/01-security-best-practices.md)
- [SSH Configuration](../07-technical-guides/04-ssh-connections.md)

---

## 7. Scheduling Tasks

**Goal**: Automate command execution at specific times or intervals

**Time**: 3-5 minutes

**Prerequisites**: Command or Flight Plan created, server configured

### Creating a Scheduled Task

#### Steps

1. **Navigate to Scheduled Tasks**
   ```
   Tower → Scheduled Tasks → Create
   ```

2. **Basic Information**
   ```
   Name: [Descriptive name, e.g., "Daily Database Backup"]
   Code: [Unique identifier, e.g., "daily-db-backup"]
   ```

3. **Select Action Type**
   ```
   Action field → Select "Command" or "Flight Plan"
   ```

4. **Select Action**
   ```
   Command field → [Select command, e.g., "Backup Database"]
   Or
   Flight Plan field → [Select flight plan]
   ```

5. **Select Server**
   ```
   Server field → [Select target server]
   ```

6. **Configure Schedule**

   **Option A: Interval-Based**
   ```
   Scheduling Type: Interval
   Interval: 1
   Interval Unit: Days
   Next Run: [Set first execution time, e.g., tomorrow 2:00 AM]
   ```

   **Option B: Cron-Based** (Advanced)
   ```
   Scheduling Type: Cron
   Cron Expression: [e.g., "0 2 * * *" for daily at 2 AM]
   ```

7. **Optional: Set Variable Values**
   ```
   Variables tab → Add variable values if command uses variables
   ```

8. **Activate**
   ```
   Active: ✓ Check
   ```

9. **Save**

### Expected Result

✓ Scheduled task created and active
✓ Will execute at configured time
✓ Creates log entries on each execution

### Common Scheduling Patterns

**Daily Backup** (2 AM):
```
Scheduling Type: Interval
Interval: 1
Interval Unit: Days
Next Run: Tomorrow 02:00:00
```

**Every 6 Hours**:
```
Scheduling Type: Interval
Interval: 6
Interval Unit: Hours
Next Run: [6 hours from now]
```

**Weekly on Sunday** (via cron):
```
Scheduling Type: Cron
Cron Expression: 0 0 * * 0
```

**Every 15 Minutes**:
```
Scheduling Type: Interval
Interval: 15
Interval Unit: Minutes
Next Run: [15 minutes from now]
```

### Cron Expression Reference

```
* * * * *
│ │ │ │ │
│ │ │ │ └─── Day of week (0-6, Sunday=0)
│ │ │ └───── Month (1-12)
│ │ └─────── Day of month (1-31)
│ └───────── Hour (0-23)
└─────────── Minute (0-59)
```

**Examples**:
- `0 2 * * *` - Daily at 2 AM
- `0 */6 * * *` - Every 6 hours
- `30 1 * * 0` - Weekly on Sunday at 1:30 AM
- `0 0 1 * *` - Monthly on 1st at midnight
- `*/15 * * * *` - Every 15 minutes

### Viewing Scheduled Task Execution

1. **Check Scheduled Task Record**
   ```
   Tower → Scheduled Tasks → [Select task]
   Next Run field shows next execution time
   Last Run field shows last execution time
   ```

2. **View Execution Logs**
   ```
   Tower → Logs → Command Logs (or Plan Logs)
   Filter by server or command name
   ```

3. **Monitor Cron Job**
   ```
   Settings → Technical → Automation → Scheduled Actions
   Search for "Tower"
   View "ir_cron_run_scheduled_tasks"
   ```

### Pausing a Scheduled Task

1. **Open Scheduled Task**
2. **Uncheck Active**
3. **Save**
4. Task won't execute until re-activated

### Troubleshooting

- **Task doesn't run**: Verify Active is checked, check Next Run time, verify cron job is running
- **Wrong schedule**: Review interval settings or cron expression
- **Execution fails**: Check command logs for errors, verify server connection

### Related Documentation

- [Scheduled Tasks Guide](../04-feature-guides/07-scheduled-tasks-automation.md)
- [Cron Job Configuration](02-configuration.md#cron-job-configuration)

---

## 8. Viewing Logs

**Goal**: Review command execution history and troubleshoot issues

**Time**: 1-2 minutes

**Prerequisites**: Commands have been executed

### Viewing Command Logs

#### Steps

1. **Navigate to Command Logs**
   ```
   Tower → Logs → Command Logs
   ```

2. **Browse Recent Logs**
   ```
   Logs displayed in reverse chronological order (newest first)
   ```

3. **Filter Logs**

   **By Status**:
   ```
   Click Filters → Select:
   - Success
   - Error
   - Timeout
   - Running
   ```

   **By Server**:
   ```
   Search bar → Type server name
   Or use Filters → Group By → Server
   ```

   **By Command**:
   ```
   Search bar → Type command name
   ```

   **By Date**:
   ```
   Filters → Creation Date → Select range
   ```

4. **Open Log Details**
   ```
   Click on log entry to view full details
   ```

5. **Review Results**
   ```
   Result tab:
   - Standard Output: Command output
   - Standard Error: Error messages
   - Exit Code: Numeric result (0 = success)

   Details tab:
   - Execution metadata
   - Timestamps
   - Variables used
   ```

### Viewing Flight Plan Logs

#### Steps

1. **Navigate to Flight Plan Logs**
   ```
   Tower → Logs → Flight Plan Logs
   ```

2. **Open Log**
   ```
   Click on flight plan log entry
   ```

3. **View Overall Status**
   ```
   Status field shows plan result
   Duration shows total execution time
   ```

4. **View Line Executions**
   ```
   Lines tab shows each command execution
   Click on line to see command log details
   ```

5. **Trace Execution Flow**
   ```
   See which lines executed
   Identify where errors occurred
   View conditional execution results
   ```

### Viewing Server Logs

#### Steps

1. **Open Server Record**
   ```
   Tower → Servers → [Select server]
   ```

2. **Click Logs Tab**
   ```
   Shows all activity for this server
   ```

3. **Or Use Server Logs Menu**
   ```
   Tower → Logs → Server Logs
   Filter by server
   ```

### Understanding Log Statuses

| Status | Icon/Color | Meaning | Action |
|--------|-----------|---------|--------|
| **SUCCESS** | ✓ Green | Command completed with exit code 0 | Review output for results |
| **ERROR** | ✗ Red | Command failed (exit code > 0) | Check Standard Error for cause |
| **TIMEOUT** | ⏱ Orange | Command exceeded timeout | Increase timeout or optimize command |
| **RUNNING** | ▶ Blue | Command currently executing | Wait for completion |
| **PENDING** | ⏸ Gray | Queued, not yet started | Normal for queue_job |

### Common Log Patterns

**Successful Command**:
```
Status: SUCCESS
Exit Code: 0
Duration: 2 seconds
Standard Output: [command output]
Standard Error: (empty)
```

**Failed Command**:
```
Status: ERROR
Exit Code: 1
Duration: 1 second
Standard Output: (may be empty)
Standard Error: bash: command-not-found: command not found
```

**Timed Out Command**:
```
Status: TIMEOUT
Exit Code: 124
Duration: 300 seconds
Standard Output: [partial output]
Standard Error: Command execution exceeded timeout
```

### Exporting Logs

1. **Select Logs**
   ```
   In list view, select checkboxes for logs to export
   ```

2. **Export**
   ```
   Action menu → Export
   Choose format (CSV, Excel)
   Download file
   ```

### Searching Logs

**Search by Multiple Criteria**:
```
Click search bar dropdown
Add multiple filters:
- Server = "Production Web Server"
- Command = "Check Disk Space"
- Creation Date > "2025-11-01"
```

**Save Search as Favorite**:
```
Configure your filters
Click Favorites → Save current search
Name: "Production Disk Checks"
```

### Troubleshooting with Logs

**Issue: Command fails intermittently**
- Compare successful vs failed logs
- Check Standard Error for differences
- Look for timeout issues
- Review server load at execution times

**Issue: No output in logs**
- Verify command produces output
- Check if output goes to stderr instead of stdout
- Ensure command completes (check status)

**Issue: Cannot find specific log**
- Use advanced search filters
- Check time range
- Verify command actually executed
- Check different log types (Command vs Plan)

### Related Documentation

- [Logging and Monitoring](../07-technical-guides/05-logging-monitoring.md)
- [Troubleshooting Guide](../12-issues-bugs/02-troubleshooting-guide.md)

---

## 9. Managing Users and Access

**Goal**: Configure user accounts and access permissions

**Time**: 3-5 minutes per user

**Prerequisites**: Admin or Root access level

### Creating a New User

#### Steps

1. **Navigate to Users**
   ```
   Settings → Users & Companies → Users → Create
   ```

2. **Basic Information**
   ```
   Name: [Full name, e.g., "John Doe"]
   Email Address: [e.g., "john.doe@company.com"]
   ```

3. **Set Authentication**
   ```
   Password: [Click "Change Password" or set on first login]
   ```

4. **Configure Access Rights**
   ```
   Access Rights tab
   Scroll to "Cetmix Tower" section
   Access Level → Select:
   - User (read/execute)
   - Manager (manage resources)
   - Root (full control)
   ```

5. **Optional: Add to Groups**
   ```
   Access Rights tab
   Groups → Add other relevant groups
   ```

6. **Save**

### Expected Result

✓ User can login to Odoo
✓ User has appropriate Tower access
✓ User sees Tower menu (if any access granted)

### Configuring Resource-Level Access

#### Sharing a Server with Specific Users

1. **Open Server**
   ```
   Tower → Servers → [Select server]
   ```

2. **Configure Access**
   ```
   Access tab:
   - Owner: [Current owner, usually creator]
   - Users: Add → [Select users who should access]
   - Groups: Add → [Select user groups]
   - Public: ☐ (check to allow all users)
   ```

3. **Save**

#### Sharing a Command

Same process as servers:
```
Tower → Commands → [Select command] → Access tab
Configure Users, Groups, or Public access
```

### Understanding Access Levels

**User**:
- View assigned servers, commands, flight plans
- Run assigned commands
- View logs for their executions
- Cannot create or modify resources

**Manager**:
- All User permissions
- Create servers, commands, flight plans
- Modify owned/assigned resources
- Delete owned resources
- Manage access on owned resources

**Root**:
- All Manager permissions
- View all resources system-wide
- Modify any resource
- Delete any resource
- Configure system settings

### Creating User Groups

#### Steps

1. **Navigate to Groups**
   ```
   Settings → Users & Companies → Groups → Create
   ```

2. **Basic Information**
   ```
   Name: [e.g., "DevOps Team"]
   ```

3. **Add Users**
   ```
   Users tab → Add → Select users
   ```

4. **Save**

5. **Use Group in Tower Resources**
   ```
   Open server/command/plan
   Access tab → Groups → Add "DevOps Team"
   ```

### Common Access Patterns

**Pattern 1: Production Server Access**
```
Server: Production Database Server
├── Owner: dba@company.com (Root)
├── Users: sysadmin@company.com (Manager)
├── Groups: DBA Team (Manager)
└── Public: No

Only DBAs and sysadmin can access
```

**Pattern 2: Public Monitoring Commands**
```
Command: Check System Health
├── Owner: admin@company.com (Root)
├── Users: (empty)
├── Groups: (empty)
└── Public: Yes

All Tower users can run this command
```

**Pattern 3: Team-Based Management**
```
Flight Plan: Deploy Web Application
├── Owner: lead@company.com (Manager)
├── Users: (empty)
├── Groups: Web Development Team (Manager)
└── Public: No

Only Web Development Team can manage and run
```

### Troubleshooting

- **User can't see Tower menu**: Verify Cetmix Tower access level is set (User minimum)
- **User can't see specific server**: Check server's Access tab, add user/group
- **User can run but not edit**: User level = User (read-only), upgrade to Manager
- **User sees "Access Denied"**: Resource access not granted, check Access tab

### Related Documentation

- [User Management](../03-user-guides/01-system-administrator-guide.md#user-management)
- [Access Control](../10-security-compliance/02-access-control-rbac.md)

---

## 10. Exporting and Importing Configurations

**Goal**: Export Tower configurations to YAML and import them to another instance

**Time**: 5-10 minutes

**Prerequisites**: `cetmix_tower_yaml` module installed

### Exporting Configuration

#### Steps

1. **Navigate to Resource**
   ```
   Example: Tower → Commands → [Select command]
   ```

2. **Open Export Wizard**
   ```
   Action menu → Export to YAML
   Or click "Export to YAML" button
   ```

3. **Configure Export Options**
   ```
   Include Dependencies: ✓ (include related records)
   Export Format: YAML
   ```

4. **Generate Export**
   ```
   Click "Export" button
   Wait for processing
   ```

5. **Download YAML File**
   ```
   Click "Download" button
   Save file locally
   ```

### Exporting Multiple Resources

#### Steps

1. **Navigate to List View**
   ```
   Example: Tower → Servers → Servers
   ```

2. **Select Resources**
   ```
   Check boxes for resources to export
   ```

3. **Action Menu**
   ```
   Action menu → Export to YAML
   ```

4. **Configure and Download**
   ```
   Same as single export
   ```

### Importing Configuration

#### Steps

1. **Navigate to Import**
   ```
   Tower → YAML → Import
   Or specific resource → Import from YAML
   ```

2. **Upload YAML File**
   ```
   Click "Upload" or choose file
   Select your YAML file
   ```

3. **Preview Import**
   ```
   Review what will be imported:
   - New records to create
   - Existing records to update
   - Dependencies to resolve
   ```

4. **Configure Import Options**
   ```
   Update Existing: ✓ (update if code matches)
   Create Missing: ✓ (create if doesn't exist)
   Skip Errors: ☐ (stop on first error)
   ```

5. **Execute Import**
   ```
   Click "Import" button
   Wait for processing
   ```

6. **Review Results**
   ```
   Check import log for:
   - Records created
   - Records updated
   - Errors encountered
   ```

### Expected Result

✓ Configuration exported to YAML file
✓ YAML file can be version controlled
✓ Configuration imported to another Tower instance
✓ Resources recreated with same codes

### Common Export/Import Scenarios

**Scenario 1: Migrate Development to Production**
```
1. Export commands from dev instance
2. Review YAML, adjust variables if needed
3. Import to production instance
4. Verify and test
```

**Scenario 2: Backup Configuration**
```
1. Export all servers, commands, plans
2. Store YAML files in git repository
3. Version control your infrastructure
```

**Scenario 3: Share Templates**
```
1. Export reusable commands/plans
2. Share YAML with team or community
3. Others import to their instances
```

### Example YAML Structure

**Command Export**:
```yaml
version: 1.0
author: admin
commands:
  - code: check-disk-space
    name: Check Disk Space
    shell: true
    code_text: |
      df -h
    timeout: 60
```

**Server Export**:
```yaml
version: 1.0
servers:
  - code: prod-web-01
    name: Production Web Server
    ip_address: 192.168.1.100
    ssh_port: 22
    ssh_username: ubuntu
    tags:
      - production
      - web
```

### Best Practices

1. **Use Meaningful Codes**
   - Codes are used to match resources during import
   - Use consistent naming conventions
   - Example: `prod-web-01`, `check-disk-space`

2. **Export with Dependencies**
   - Include related resources (tags, variables, keys)
   - Ensures complete migration

3. **Version Control YAML**
   - Store in Git repository
   - Track changes over time
   - Enable rollback if needed

4. **Review Before Import**
   - Check for sensitive data (passwords, keys)
   - Remove or replace environment-specific values
   - Validate YAML syntax

5. **Test in Staging**
   - Import to staging environment first
   - Verify functionality
   - Then import to production

### Troubleshooting

- **Import fails**: Check YAML syntax, verify dependencies exist, check for duplicate codes
- **Resources not updated**: Verify "Update Existing" is checked, check code matching
- **Sensitive data exposed**: Review YAML before committing, use .gitignore for secrets
- **Dependencies missing**: Export with dependencies, or create dependencies first

### Related Documentation

- [YAML Export/Import Guide](../04-feature-guides/09-yaml-export-import.md)
- [Configuration Management](../09-deployment-operations/02-configuration-management.md)

---

## Quick Reference Cheat Sheet

### Navigation Shortcuts

```
Servers:           Tower → Servers → Servers
Commands:          Tower → Commands
Flight Plans:      Tower → Flight Plans
Files:             Tower → Files
Variables:         Tower → Configuration → Variables
Keys:              Tower → Configuration → Keys
Scheduled Tasks:   Tower → Scheduled Tasks
Command Logs:      Tower → Logs → Command Logs
Settings:          Settings → General Settings → Cetmix Tower
```

### Common Keyboard Shortcuts

```
Create new:        Alt + C (in list view)
Save:              Ctrl + S (in form view)
Discard:           Ctrl + D (in form view)
Search:            Ctrl + K (in list view)
Filter:            Ctrl + F (in list view)
```

### Quick Actions

**Run Command on Server**:
```
Server form → Run Command → Select command → Run
```

**View Latest Logs**:
```
Tower → Logs → Command Logs → (sorted newest first)
```

**Test Server Connection**:
```
Server form → Test Connection button
```

**Export Resource**:
```
Resource form → Action menu → Export to YAML
```

### Common Variable References

```
${app_name}        - Application name
${environment}     - Environment (dev/staging/prod)
${domain_name}     - Domain name
${db_name}         - Database name
${branch_name}     - Git branch
```

### Exit Code Reference

```
0    - Success
1    - General error
2    - Misuse of shell command
124  - Timeout
126  - Command not executable
127  - Command not found
130  - Terminated by Ctrl+C
```

---

## Summary

This guide covered the most common tasks in Cetmix Tower:

✓ Adding and managing servers
✓ Creating and executing commands
✓ Building flight plans for workflows
✓ Managing files and templates
✓ Setting up variables
✓ Storing secrets securely
✓ Scheduling automated tasks
✓ Viewing and analyzing logs
✓ Managing users and access
✓ Exporting and importing configurations

For more detailed information on each topic, refer to the Feature Guides section.

---

## Next Steps

- **Master Advanced Features**: [Feature Guides](../04-feature-guides/README.md)
- **Learn Role-Specific Workflows**: [User Guides](../03-user-guides/README.md)
- **Explore Technical Details**: [Technical Guides](../07-technical-guides/README.md)
- **Return to Getting Started**: [Getting Started Overview](README.md)

---

**Need More Help?**
- [Troubleshooting Guide](../12-issues-bugs/02-troubleshooting-guide.md)
- [FAQ](../12-issues-bugs/03-faq.md)
- [Contact Support](../12-issues-bugs/01-bug-reporting-process.md)

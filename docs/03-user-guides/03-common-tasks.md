---
title: "Common Tasks"
section: "03-user-guides"
document_id: "UG-003"
version: "17.0"
audience: "End Users"
last_updated: "2025-11-16"
tags: ["tasks", "workflows", "commands", "servers", "daily-operations"]
---

# Common Tasks

This guide provides step-by-step instructions for the most common daily operations in Cetmix Tower.

## Table of Contents

1. [Running Commands on Servers](#running-commands-on-servers)
2. [Checking Command Logs](#checking-command-logs)
3. [Managing Files](#managing-files)
4. [Viewing Server Status](#viewing-server-status)
5. [Using Shortcuts](#using-shortcuts)
6. [Understanding Notifications](#understanding-notifications)

---

## Running Commands on Servers

Running commands is one of the primary functions of Cetmix Tower. Commands can be executed individually or as part of flight plans.

### Running a Single Command

**Prerequisites:**
- Command exists in Tower
- You have access to the command (based on access level)
- Target server is accessible

**Step-by-Step Process:**

**Method 1: From Command Form**

1. Navigate to: `Cetmix Tower → Commands → Commands`
2. Open the desired command (click on name)
3. Click the **Run** button (usually top-right or in action menu)
4. The **Run Command Wizard** opens:

   ```
   ┌─────────────────────────────────────────────┐
   │ Run Command: Restart Nginx Service          │
   ├─────────────────────────────────────────────┤
   │ Select Servers:                             │
   │ ☑ Production Web Server 01                  │
   │ ☑ Production Web Server 02                  │
   │ □ Development Server                        │
   │                                             │
   │ Variables:                                  │
   │ (None required for this command)            │
   │                                             │
   │ [Cancel]                  [Run Command]     │
   └─────────────────────────────────────────────┘
   ```

5. Select target servers (checkboxes)
6. Fill in any required variable values
7. Click **Run Command**
8. Command executes on selected servers
9. View execution progress and results

**Method 2: From Server Form**

1. Navigate to: `Cetmix Tower → Servers → Servers`
2. Open the desired server
3. Go to **Commands** tab
4. Find the command in the list
5. Click the **Run** icon (▶️ play button)
6. Follow wizard steps as above

**Method 3: Quick Run from Server Status**

1. Open server in form view
2. Look for **Quick Actions** section
3. Click on frequently-used command
4. Command runs immediately (if no variables required)

---

### Running Commands with Variables

Some commands use variables for flexibility.

**Example: Deploy Application with Version Variable**

```
Command: Deploy Application
Code: git checkout ${app_version} && docker-compose up -d
Variable: app_version (String variable)
```

**Running with Variables:**

1. Start run command wizard (any method above)
2. In **Variables** section:
   ```
   ┌─────────────────────────────────────────────┐
   │ Variables:                                  │
   │ App Version: [v2.5.1        ]              │
   │                                             │
   │ Custom Variables:                           │
   │ + Add variable                              │
   └─────────────────────────────────────────────┘
   ```
3. Enter variable value (e.g., "v2.5.1")
4. Add custom variables if needed (click "+ Add variable")
5. Click **Run Command**

**Variable Types:**

- **String**: Free text input
- **Options**: Dropdown selection from predefined options
- **Custom**: Runtime-defined variables

---

### Monitoring Command Execution

After running a command, monitor its progress:

**Real-time Monitoring:**

1. After clicking **Run Command**, a notification appears
2. Click notification to view **Command Log**
3. Watch execution status:
   - 🔵 **Running**: Command executing
   - 🟢 **Success**: Completed successfully (exit code 0)
   - 🔴 **Failed**: Execution failed (non-zero exit code)
   - 🟡 **Timeout**: Exceeded time limit
   - ⚫ **Unknown**: Status unclear

**Command Log View:**

```
┌─────────────────────────────────────────────────────┐
│ Command: Restart Nginx Service                      │
│ Server: Production Web Server 01                    │
│ Status: ● Running                                   │
│ Started: 2025-11-16 14:30:25                        │
│ Duration: 00:00:03                                  │
├─────────────────────────────────────────────────────┤
│ Output:                                             │
│ Stopping nginx service...                           │
│ Starting nginx service...                           │
│ nginx service started successfully                  │
│                                                     │
│ Exit Code: (pending)                                │
└─────────────────────────────────────────────────────┘
```

**Viewing Output:**
- **Standard Output**: Shows normal command output
- **Error Output**: Shows error messages (if any)
- **Exit Code**: Command completion status (0 = success)

---

### Running Flight Plans

Flight plans execute multiple commands in sequence.

**Step-by-Step Process:**

1. Navigate to: `Cetmix Tower → Flight Plans → Flight Plans`
2. Open desired flight plan
3. Click **Run** button
4. The **Run Flight Plan Wizard** opens:

   ```
   ┌─────────────────────────────────────────────┐
   │ Run Flight Plan: Deploy Web Application     │
   ├─────────────────────────────────────────────┤
   │ Select Servers:                             │
   │ ☑ Production Web Server 01                  │
   │                                             │
   │ Plan Variables:                             │
   │ App Version: [v2.5.1        ]              │
   │ Environment: [Production ▼  ]              │
   │                                             │
   │ Plan Lines (5 steps):                       │
   │ 1. Stop Application Service                 │
   │ 2. Backup Current Version                   │
   │ 3. Pull New Code                            │
   │ 4. Run Database Migrations                  │
   │ 5. Start Application Service                │
   │                                             │
   │ [Cancel]                  [Run Plan]        │
   └─────────────────────────────────────────────┘
   ```

5. Select target servers
6. Fill in plan-level variables
7. Review plan steps
8. Click **Run Plan**

**Monitoring Plan Execution:**

1. Plan execution starts
2. View **Flight Plan Log** for progress
3. Each line shows status:
   ```
   ✓ 1. Stop Application Service - Success (2s)
   ✓ 2. Backup Current Version - Success (15s)
   ⏳ 3. Pull New Code - Running... (8s elapsed)
   ⏸ 4. Run Database Migrations - Pending
   ⏸ 5. Start Application Service - Pending
   ```

**Plan Error Handling:**

If a step fails, the plan follows configured error action:
- **Exit with command exit code**: Plan stops, uses failing command's exit code
- **Exit with custom exit code**: Plan stops, uses predefined exit code
- **Run next command**: Plan continues to next step

---

## Checking Command Logs

Command logs provide historical record of all command executions.

### Accessing Command Logs

**Method 1: From Command Logs Menu**

1. Navigate to: `Cetmix Tower → Commands → Command Logs`
2. View list of all command executions
3. Filter and search as needed
4. Click log entry to view details

**Method 2: From Command Form**

1. Open command in form view
2. Click **Logs** smart button at top
   ```
   [25 Logs]
   ```
3. View all executions of this command
4. Click specific log for details

**Method 3: From Server Form**

1. Open server in form view
2. Go to **Logs** tab or click **Logs** smart button
3. View all commands executed on this server
4. Filter by command, date, status, etc.

---

### Understanding Log Details

Each command log contains detailed execution information:

**Log Information:**

```
┌─────────────────────────────────────────────────────┐
│ Command Log #1234                                   │
├─────────────────────────────────────────────────────┤
│ Command: Restart Nginx Service                      │
│ Server: Production Web Server 01                    │
│ Executed By: Admin User                             │
│ Status: ● Success                                   │
│                                                     │
│ Timing:                                             │
│ Started: 2025-11-16 14:30:25                        │
│ Finished: 2025-11-16 14:30:28                       │
│ Duration: 00:00:03                                  │
│                                                     │
│ Standard Output:                                    │
│ Stopping nginx service...                           │
│ Starting nginx service...                           │
│ nginx service started successfully                  │
│                                                     │
│ Error Output: (none)                                │
│                                                     │
│ Exit Code: 0                                        │
│                                                     │
│ Variables Used:                                     │
│ (None)                                              │
└─────────────────────────────────────────────────────┘
```

**Key Fields:**

- **Status**: Execution result (Success, Failed, Timeout, etc.)
- **Started/Finished**: Timestamp information
- **Duration**: How long command took
- **Standard Output**: Normal command output
- **Error Output**: Error messages (if any)
- **Exit Code**: Return code (0 = success, others = error)
- **Variables Used**: Variable values at execution time

---

### Filtering and Searching Logs

**Common Filters:**

```
Status:
  - Success (exit code 0)
  - Failed (non-zero exit code)
  - Timeout
  - Error (SSH or connection errors)

Date Range:
  - Today
  - Last 7 Days
  - Last 30 Days
  - Custom Date Range

Server:
  - Specific server
  - Server group/tag

Command:
  - Specific command
  - Command category
```

**Example Search:**

```
Find all failed commands on production servers this week:

Filters:
  Status = Failed
  Server Tags contains "Production"
  Created On >= Start of This Week

Results: 3 failed command executions found
```

---

### Troubleshooting from Logs

**Common Issues and Solutions:**

**1. Command Timeout**
```
Status: Timeout
Duration: 300s (limit: 300s)
Solution: Increase command timeout or optimize command
```

**2. Permission Denied**
```
Error Output: Permission denied
Exit Code: 1
Solution: Check user permissions, may need sudo
```

**3. Command Not Found**
```
Error Output: command not found: xyz
Exit Code: 127
Solution: Install missing package or check PATH
```

**4. SSH Connection Failed**
```
Status: Error
Message: SSH connection failed
Solution: Check server connectivity, credentials, firewall
```

---

## Managing Files

Cetmix Tower allows you to manage files on remote servers, either pushing files from Tower or pulling files from servers.

### Pushing Files to Servers (Tower → Server)

**Use Case:** Deploy configuration files, scripts, or application files to servers

**Step-by-Step Process:**

1. Navigate to: `Cetmix Tower → Files → Files`
2. Create or open file to push
3. Verify file settings:
   - **Source**: Tower
   - **Server Directory**: Target path (e.g., `/etc/nginx/conf.d`)
   - **Name**: Filename (e.g., `app.conf`)
   - **Code**: File content

   ```
   ┌─────────────────────────────────────────────┐
   │ File: nginx-app.conf                        │
   ├─────────────────────────────────────────────┤
   │ Source: ● Tower → Server                    │
   │ Server Directory: /etc/nginx/conf.d         │
   │ Servers: Production Web Server 01           │
   │                                             │
   │ File Content:                               │
   │ ┌─────────────────────────────────────────┐ │
   │ │ server {                                │ │
   │ │   listen 80;                            │ │
   │ │   server_name ${domain_name};           │ │
   │ │   location / {                          │ │
   │ │     proxy_pass http://localhost:8000;   │ │
   │ │   }                                     │ │
   │ │ }                                       │ │
   │ └─────────────────────────────────────────┘ │
   └─────────────────────────────────────────────┘
   ```

4. Click **Sync** button (or **Action** → **Sync to Server**)
5. File pushes to server at specified path
6. View sync result in log

**Variables in Files:**

Files support variable substitution:
```
server_name ${domain_name};
root ${app_root_path};
```

Variables resolve at sync time using server-specific values.

---

### Pulling Files from Servers (Server → Tower)

**Use Case:** Retrieve log files, backups, or configuration for review

**Step-by-Step Process:**

1. Navigate to: `Cetmix Tower → Files → Files`
2. Create or open file to pull
3. Set file settings:
   - **Source**: Server
   - **Server Directory**: Source path (e.g., `/var/log/nginx`)
   - **Name**: Filename (e.g., `access.log`)
   - **Servers**: Source server(s)

4. Click **Sync** button (or **Action** → **Pull from Server**)
5. File downloads from server to Tower
6. View file content in Tower
7. Download to local computer if needed

**Example: Pulling Log File**

```
File: application.log
Source: Server → Tower
Server Directory: /var/log/myapp
Name: application.log
Servers: Production Web Server 01

After Sync:
File content appears in Tower, can be:
  - Viewed in browser
  - Downloaded locally
  - Searched/analyzed
```

---

### Auto-Sync Configuration

Enable automatic file synchronization for hands-free updates.

**Configuring Auto-Sync:**

1. Open file in form view
2. Check **Auto Sync** checkbox
3. Select **Auto Sync Interval**:
   - 15 minutes
   - 30 minutes
   - 1 hour
   - 1 day
   - 1 week
   - 1 month

4. Set **Next Sync Date** (usually auto-calculated)
5. Save file

**Auto-Sync Behavior:**

- Tower runs scheduled sync jobs via cron
- Files sync at specified interval
- Sync logs created automatically
- Errors trigger notifications (if configured)

**Example Auto-Sync:**
```
File: Database Backup
Source: Server → Tower
Server Directory: /backup/db
Auto Sync: ✓ Enabled
Auto Sync Interval: 1 day
Next Sync: 2025-11-17 02:00:00

Result: Daily backup downloaded to Tower at 2 AM
```

---

### Using File Templates

File templates allow you to create files from predefined templates with variable substitution.

**Step-by-Step Process:**

1. Navigate to: `Cetmix Tower → Files → File Templates`
2. Create or open template
3. Define template content with variables:
   ```
   server {
     listen 80;
     server_name ${domain_name};
     root ${web_root};
   }
   ```

4. Go to: `Cetmix Tower → Files → Files`
5. Create new file
6. Select **Template** field → Choose your template
7. File auto-populates with template content
8. Set variable values (per server or globally)
9. Save and sync file

**Benefits:**
- Consistency across multiple servers
- Easy updates to multiple files
- Centralized template management
- Variable-driven configuration

---

## Viewing Server Status

Monitor server health and connectivity status.

### Server Status Indicators

**From Servers List View:**

```
┌────────────────────────────────────────────────────┐
│ Name              │ IP Address    │ Status         │
├────────────────────────────────────────────────────┤
│ Web Server 01     │ 192.168.1.10 │ 🟢 Online      │
│ DB Server 01      │ 192.168.1.20 │ 🟢 Online      │
│ Test Server       │ 192.168.1.30 │ 🔴 Offline     │
│ Staging Server    │ 192.168.1.40 │ ⚫ Unknown     │
└────────────────────────────────────────────────────┘
```

**Status Types:**

- **🟢 Online**: Server reachable and responsive
- **🔴 Offline**: Server unreachable or not responding
- **⚫ Unknown**: Status not yet checked or unclear
- **🟡 Warning**: Server reachable but issues detected

---

### Checking Server Details

**From Server Form View:**

```
┌─────────────────────────────────────────────────────┐
│ Web Server 01                    Status: 🟢 Online  │
├─────────────────────────────────────────────────────┤
│ Connection Info:                                    │
│ IP Address: 192.168.1.10                           │
│ Port: 22                                            │
│ Last Connected: 2025-11-16 14:25:30                │
│ Uptime: 45 days 12 hours                            │
│                                                     │
│ System Info:                                        │
│ OS: Ubuntu 22.04 LTS                                │
│ Architecture: x86_64                                │
│ Kernel: 5.15.0-89-generic                           │
│                                                     │
│ Quick Actions:                                      │
│ [Check Status] [Restart Service] [View Logs]       │
└─────────────────────────────────────────────────────┘
```

**Available Information:**
- Connection status and history
- System information (OS, version, architecture)
- Last successful connection
- Server uptime (if available)
- Quick action buttons

---

### Testing Server Connection

**Manual Connection Test:**

1. Open server in form view
2. Click **Action** → **Test Connection**
3. Tower attempts SSH connection
4. Result displayed:
   - ✅ Connection successful
   - ❌ Connection failed (with error message)

**Connection Test Results:**

```
✅ Success:
Server is reachable
SSH connection established
Authentication successful
Status updated to: Online

❌ Failed:
Error: Connection timeout
Unable to connect to 192.168.1.10:22
Please check:
  - Server is powered on
  - Network connectivity
  - Firewall rules
  - SSH service running
```

---

### Server Logs

View detailed server activity:

1. Open server in form view
2. Click **Logs** smart button or go to **Logs** tab
3. View server logs:
   - Connection logs
   - Command executions
   - File syncs
   - Status changes

**Server Log Example:**

```
2025-11-16 14:30:25 | Command | Restart Nginx | Success
2025-11-16 14:15:10 | File Sync | app.conf | Success
2025-11-16 14:00:00 | Connection | Status Check | Online
2025-11-16 13:45:30 | Command | Deploy App | Success
2025-11-16 13:30:15 | Connection | Status Check | Online
```

---

## Using Shortcuts

Shortcuts provide quick access to frequently-used operations.

### What are Shortcuts?

Shortcuts are quick-launch links to:
- Frequently-run commands
- Common flight plans
- Specific server operations
- Custom actions

### Creating Shortcuts

**Step-by-Step Process:**

1. Navigate to: `Cetmix Tower → Tools → Shortcuts`
2. Click **Create**
3. Configure shortcut:
   ```
   ┌─────────────────────────────────────────────┐
   │ Create Shortcut                             │
   ├─────────────────────────────────────────────┤
   │ Name: Restart Production Web Servers        │
   │ Reference: restart_prod_web                 │
   │                                             │
   │ Type: ● Command  ○ Flight Plan             │
   │                                             │
   │ Command: Restart Nginx Service              │
   │ Servers: Production Web Server 01           │
   │          Production Web Server 02           │
   │                                             │
   │ Variable Values:                            │
   │ (Set default values if needed)              │
   │                                             │
   │ [Cancel]                 [Save]             │
   └─────────────────────────────────────────────┘
   ```

4. Click **Save**
5. Shortcut appears in shortcuts list

### Using Shortcuts

**Method 1: From Shortcuts List**

1. Navigate to: `Cetmix Tower → Tools → Shortcuts`
2. Find your shortcut
3. Click **Execute** or **Run** button
4. Command/plan runs with preconfigured settings

**Method 2: From Dashboard (if configured)**

1. Dashboard widgets can display shortcuts
2. Click shortcut widget
3. One-click execution

**Method 3: From Bookmarks**

1. Bookmark shortcut URL in browser
2. Click bookmark to run anytime

---

### Shortcut Benefits

**Time Savings:**
- No need to navigate menus
- Pre-filled variables
- Pre-selected servers
- One-click execution

**Consistency:**
- Same settings every time
- Reduces configuration errors
- Standardizes operations

**Examples of Useful Shortcuts:**

```
1. Daily Backups
   - Command: Run Backup Script
   - Servers: All Production Servers
   - Schedule: Daily at 2 AM

2. Deploy Latest Version
   - Flight Plan: Deploy Application
   - Servers: Production Web Servers
   - Variables: version=latest

3. Restart Services
   - Command: Restart All Services
   - Servers: Specific server group

4. Health Check
   - Command: System Health Check
   - Servers: All Servers
   - Run: Every 15 minutes
```

---

## Understanding Notifications

Cetmix Tower provides notifications to keep you informed of important events.

### Notification Types

**1. In-App Notifications**

Appear in Odoo notification area (bell icon):
- Command completion
- Plan execution results
- Errors and failures
- Scheduled task results

**2. Email Notifications** (if configured)

Sent to your email:
- Critical errors
- Long-running operations complete
- Scheduled reports
- Activity reminders

**3. Activity Notifications**

Odoo activities system:
- Assigned tasks
- Follow-up reminders
- Scheduled activities

---

### Configuring Notifications

**User Preferences:**

1. Click your name → **Preferences**
2. Go to **Notifications** section
3. Configure:
   - Email notification frequency
   - Notification types to receive
   - Quiet hours (if available)

**Record-Level Notifications:**

**Follow Records:**
1. Open record (server, command, plan)
2. Click **Follow** button (star icon)
3. Receive notifications for changes and activities

**Unfollow:**
1. Click **Following** button (filled star)
2. Stop receiving notifications for this record

---

### Managing Notifications

**Viewing Notifications:**

1. Click bell icon (top-right)
2. View recent notifications
3. Click notification to view details
4. Mark as read or clear

**Notification Actions:**

```
┌─────────────────────────────────────────────┐
│ Notifications                         [●3]  │
├─────────────────────────────────────────────┤
│ ● Command "Deploy App" completed            │
│   Success on 2 servers                      │
│   [View Details] [Dismiss]                  │
│                                             │
│ ● Flight Plan "Backup" failed               │
│   Error on Step 3                           │
│   [View Log] [Dismiss]                      │
│                                             │
│ ● File sync completed                       │
│   3 files synced successfully               │
│   [Dismiss]                                 │
└─────────────────────────────────────────────┘
```

**Actions:**
- **View Details**: Open related record
- **View Log**: Jump to execution log
- **Dismiss**: Mark as read and clear
- **Mark All Read**: Clear all notifications

---

### Notification Examples

**Successful Command:**
```
✓ Command Completed
"Restart Nginx Service" completed successfully
Server: Production Web Server 01
Duration: 3 seconds
Exit Code: 0
[View Log]
```

**Failed Command:**
```
✗ Command Failed
"Deploy Application" failed
Server: Production Web Server 02
Error: Permission denied
Exit Code: 1
[View Log] [Retry]
```

**Plan Completion:**
```
✓ Flight Plan Completed
"Deploy Web Application" completed
5/5 steps successful
Total Duration: 2 minutes 15 seconds
Servers: 2
[View Detailed Log]
```

**Scheduled Task:**
```
⏰ Scheduled Task Reminder
"Database Backup" scheduled in 30 minutes
Target: All Production Servers
[Run Now] [View Task]
```

---

## Summary

You now know how to:

✅ Run commands and flight plans on servers
✅ Check command logs and troubleshoot issues
✅ Manage files (push to and pull from servers)
✅ View server status and health
✅ Create and use shortcuts for efficiency
✅ Configure and manage notifications

### Next Steps

- **[Feature Guides](../04-feature-guides/README.md)**: Deep dive into specific Tower features
- **[Module References](../05-module-references/README.md)**: Technical documentation for developers
- **[Deployment & Operations](../09-deployment-operations/README.md)**: Advanced operational topics

---

**Navigation:**
- [← Previous: Basic Operations](./02-basic-operations.md)
- [↑ User Guides Home](./README.md)
- [↑↑ Documentation Home](../README.md)

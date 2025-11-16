---
title: "First Steps Guide"
version: "1.0"
last_updated: "2025-11-16"
maintained_by: "E-Global SCM Development Team"
status: "Active"
audience: "Administrators, End Users"
---

# First Steps with Cetmix Tower

This guide walks you through your first tasks in Cetmix Tower, from adding a server to running your first command.

---

## Table of Contents

- [For Administrators](#for-administrators)
  - [Adding Your First Server](#adding-your-first-server)
  - [Testing SSH Connectivity](#testing-ssh-connectivity)
  - [Setting Up Basic Variables](#setting-up-basic-variables)
  - [Creating Your First Command](#creating-your-first-command)
  - [Understanding Access Control](#understanding-access-control)
- [For End Users](#for-end-users)
  - [Navigating the Tower Interface](#navigating-the-tower-interface)
  - [Viewing Servers](#viewing-servers)
  - [Running Your First Command](#running-your-first-command)
  - [Viewing Command Logs](#viewing-command-logs)
- [Complete Walkthrough](#complete-walkthrough-basic-workflow)
  - [Scenario: System Health Check](#scenario-system-health-check)

---

## For Administrators

### Adding Your First Server

Let's add your first server to Cetmix Tower.

#### Step 1: Navigate to Servers

1. Login to Odoo
2. Click **Tower** in the top navigation menu
3. Select **Servers** → **Servers**
4. You should see the Servers list view

#### Step 2: Create New Server

1. Click **Create** button (top-left)
2. The server form will open

#### Step 3: Fill in Basic Information

**Required Fields**:

| Field | Description | Example |
|-------|-------------|---------|
| **Name** | Descriptive server name | `Production Web Server` |
| **Code** | Unique identifier (auto-generated if empty) | `prod-web-01` |
| **IP Address** | Server IP or hostname | `192.168.1.100` or `web.example.com` |
| **SSH Port** | SSH connection port | `22` (default) |
| **SSH Username** | User for SSH connection | `ubuntu` or `root` |

**Fill in the form**:

```
Name: My First Server
Code: test-server-01
IP Address: 192.168.1.100
SSH Port: 22
SSH Username: ubuntu
```

#### Step 4: Configure Authentication

Choose one of two authentication methods:

**Option A: SSH Password** (less secure, quick testing)

1. Scroll to **SSH Password** field
2. Enter the SSH password
3. Password is encrypted when saved

**Option B: SSH Key** (recommended, more secure)

1. First, create an SSH key in Tower:
   - Navigate to Tower → Configuration → Keys
   - Click Create
   - Name: `My SSH Key`
   - Code: `my-ssh-key`
   - SSH Key: Paste private key content
   - Save

2. Return to server form
3. **Key** field: Select `My SSH Key`

**Example SSH Key Setup**:

```bash
# On your Tower server (or local machine)
ssh-keygen -t rsa -b 4096 -f ~/.ssh/tower_key
# Press Enter for no passphrase (or set one)

# Copy public key to target server
ssh-copy-id -i ~/.ssh/tower_key.pub ubuntu@192.168.1.100

# Copy private key content for Tower
cat ~/.ssh/tower_key
# Copy the output to Tower's Key record
```

#### Step 5: Set Additional Options

**Partner** (Optional):
- Link server to a contact/partner
- Useful for client server management

**Tags** (Optional):
- Categorize servers (e.g., "Production", "Web Server", "Database")
- Navigate to Tower → Configuration → Tags to create tags first

**Operating System** (Optional):
- Select or create OS profile
- Used for OS-specific command compatibility

**Status** (Optional):
- Mark server status (e.g., "Active", "Maintenance")
- Informational only

#### Step 6: Save Server

1. Click **Save** button
2. Server is now created but not yet connected

### Testing SSH Connectivity

After creating a server, test the SSH connection.

#### Method 1: Test Connection Button

1. Open your server record (if not already open)
2. Look for the **Test Connection** button (usually in the top-right or action menu)
3. Click **Test Connection**
4. Wait for result

**Expected Results**:

**Success**:
```
Connection successful!
Server is reachable via SSH.
Host key verified.
```

**Failure - Authentication**:
```
Authentication failed!
Please check username, password, or SSH key.
```

**Failure - Network**:
```
Connection timeout!
Please verify IP address, port, and network connectivity.
```

**Failure - Host Key**:
```
Host key verification failed!
Server host key has changed or is unknown.
```

#### Method 2: Run Test Command

1. Navigate to Tower → Commands
2. Find or create a simple test command:
   - Name: `Test Connection`
   - Code: `echo "Connection successful"`
   - Shell: Checked
3. Assign to your server (or leave blank for all servers)
4. Save command
5. Click **Run Command**
6. Select your server
7. Click **Run**
8. Check logs for result

#### Troubleshooting Connection Issues

**Issue: Authentication Failed**

- Verify SSH username is correct
- Check password (if using password auth)
- Verify SSH key is correct (if using key auth)
- Test manual SSH: `ssh -i /path/to/key username@server_ip`

**Issue: Connection Timeout**

- Verify IP address is correct
- Check port (default: 22)
- Verify network connectivity: `ping server_ip`
- Check firewall rules on both Tower and target server
- Verify SSH service is running: `sudo systemctl status sshd`

**Issue: Host Key Verification Failed**

1. When first connecting, Tower may prompt to accept host key
2. Click **Accept** to trust the server
3. Alternatively, manually add to known hosts:
   ```bash
   ssh-keyscan -H 192.168.1.100 >> ~/.ssh/known_hosts
   ```

### Setting Up Basic Variables

Variables allow you to create reusable, parameterized commands.

#### What Are Variables?

Variables are placeholders that get replaced with actual values when commands execute.

**Use Cases**:
- Environment-specific values (domain names, ports)
- User-specific values (usernames, project names)
- Dynamic values (dates, timestamps)

#### Creating Your First Variable

1. Navigate to **Tower → Configuration → Variables**
2. Click **Create**

**Example 1: Simple String Variable**

```
Name: Application Name
Code: app_name
Type: String
Reference: app_name
Description: Name of the application to deploy
```

**Example 2: Options Variable**

```
Name: Environment
Code: environment
Type: Options
Reference: env
Options:
  - development
  - staging
  - production
Description: Target environment for deployment
```

#### Step-by-Step: Creating Environment Variable

1. **Basic Information**:
   - Name: `Environment`
   - Code: `environment`
   - Reference: `env`
   - Type: `Options`

2. **Add Options**:
   - Click **Add a line** in Options section
   - Value: `development`
   - Click **Add a line** again
   - Value: `staging`
   - Click **Add a line** again
   - Value: `production`

3. **Save Variable**

#### Assigning Variable Values to Servers

Variables can have different values per server.

1. Open your server record
2. Go to **Variables** tab
3. Click **Add a line**
4. Select **Variable**: `Environment`
5. Enter **Value**: `production`
6. Save

#### Using Variables in Commands

Reference variables in commands using syntax: `${variable_code}`

**Example Command**:
```bash
echo "Deploying to ${environment} environment"
echo "Application: ${app_name}"
```

**When run on server with values**:
- `environment = production`
- `app_name = MyApp`

**Actual command executed**:
```bash
echo "Deploying to production environment"
echo "Application: MyApp"
```

### Creating Your First Command

Commands are executable scripts or code that run on servers.

#### Step 1: Navigate to Commands

1. Click **Tower** → **Commands**
2. Click **Create**

#### Step 2: Fill in Basic Information

```
Name: Check Disk Space
Code: check-disk-space
Reference: disk_check
```

**Description** (optional but recommended):
```
Checks available disk space on the server using df command
```

#### Step 3: Write Command Code

Choose the code type:

**Option A: Shell Command** (most common)

1. Check **Shell** checkbox
2. In **Code** field, enter:
   ```bash
   df -h
   ```

**Option B: Python Code**

1. Uncheck **Shell** checkbox
2. In **Code** field, enter Python code:
   ```python
   import subprocess
   result = subprocess.run(['df', '-h'], capture_output=True, text=True)
   log(result.stdout)
   ```

**For this example, use Option A (Shell Command)**

#### Step 4: Configure Command Options

**Important Fields**:

| Field | Value | Description |
|-------|-------|-------------|
| **Shell** | ✓ Checked | Execute as shell command |
| **Use Sudo** | ☐ Optional | Run with sudo privileges |
| **Timeout** | 60 | Command timeout in seconds (optional, uses global default if empty) |
| **Allow Parallel Run** | ☐ Unchecked | Allow multiple instances on same server |

**For this example**:
- Shell: Checked
- Use Sudo: Unchecked
- Timeout: 60

#### Step 5: Assign to Servers (Optional)

**Option A: Assign to Specific Servers**

1. Go to **Servers** tab
2. Click **Add a line**
3. Select your server(s)
4. Commands will only run on these servers

**Option B: Allow All Servers**

1. Leave **Servers** tab empty
2. Command can run on any server

**For this example**: Leave empty (allow all servers)

#### Step 6: Configure OS Compatibility (Optional)

If command is OS-specific:

1. Go to **Operating Systems** tab
2. Select compatible OS(es)
3. Command will only be available for servers with matching OS

**For this example**: Leave empty (all OS)

#### Step 7: Set Access Control (Optional)

Control who can run this command:

1. Go to **Access** tab
2. Configure:
   - **Owner**: You (auto-set)
   - **Users**: Specific users who can run this command
   - **Groups**: User groups who can access
3. Leave empty for public access

#### Step 8: Save Command

1. Click **Save**
2. Command is now ready to use

### Understanding Access Control

Cetmix Tower uses multi-level access control.

#### Level 1: User Access Levels

Set in **Settings → Users → Access Rights → Cetmix Tower**

| Level | Can View | Can Create/Edit | Can Delete | Can Run |
|-------|----------|-----------------|------------|---------|
| **User** | Assigned resources | No | No | Assigned commands |
| **Manager** | Owned/assigned resources | Yes (owned/assigned) | Yes (owned) | All assigned |
| **Root** | All resources | Yes (all) | Yes (all) | All |

#### Level 2: Resource-Level Access

Each resource (Server, Command, Flight Plan) has access settings:

**Access Fields**:
- **Owner**: User who created it (full control)
- **Users**: Specific users with access
- **Groups**: User groups with access
- **Public**: Available to all users (respecting access level)

**Access Logic**:

```
User can access if:
  1. User is Owner, OR
  2. User is in Users list, OR
  3. User belongs to a Group in Groups list, OR
  4. Resource is marked as Public

AND

  User's Tower access level permits the action
```

#### Example Access Scenarios

**Scenario 1: Restricted Production Server**

```
Server: Production Database
├── Owner: dba@company.com
├── Users: sysadmin@company.com
├── Groups: DBA Team
├── Public: No

Access:
- dba@company.com: Full access (owner)
- sysadmin@company.com: Access based on their Tower level
- DBA Team members: Access based on their Tower level
- Other users: No access
```

**Scenario 2: Public Command**

```
Command: Check Disk Space
├── Owner: admin@company.com
├── Users: (empty)
├── Groups: (empty)
├── Public: Yes

Access:
- All Tower Users: Can run on assigned servers
- All Tower Managers: Can run and edit
- All Tower Roots: Full access
```

#### Setting Up Role-Based Access

**Example: DevOps Team Setup**

1. **Create User Group**:
   - Settings → Users & Companies → Groups
   - Create: "DevOps Team"
   - Add members

2. **Configure Server Access**:
   - Open production servers
   - Access tab → Groups → Add "DevOps Team"
   - Save

3. **Configure Command Access**:
   - Open sensitive commands
   - Access tab → Groups → Add "DevOps Team"
   - Save

4. **Assign User Access Levels**:
   - DevOps lead: Manager or Root
   - DevOps members: Manager
   - Other users: User (read-only)

---

## For End Users

### Navigating the Tower Interface

Get familiar with the Cetmix Tower interface.

#### Main Menu Structure

Click **Tower** in the top navigation to see:

```
Tower
├── Servers
│   ├── Servers (cx.tower.server)
│   └── Templates (cx.tower.server.template)
├── Commands
│   └── Commands (cx.tower.command)
├── Flight Plans
│   └── Flight Plans (cx.tower.plan)
├── Files
│   ├── Files (cx.tower.file)
│   └── Templates (cx.tower.file.template)
├── Logs
│   ├── Command Logs (cx.tower.command.log)
│   ├── Flight Plan Logs (cx.tower.plan.log)
│   └── Server Logs (cx.tower.server.log)
├── Configuration
│   ├── Variables (cx.tower.variable)
│   ├── Keys (cx.tower.key)
│   ├── Tags (cx.tower.tag)
│   ├── Operating Systems (cx.tower.os)
│   └── Shortcuts (cx.tower.shortcut)
└── Scheduled Tasks
    └── Scheduled Tasks (cx.tower.scheduled.task)
```

#### View Types

Odoo provides different view types for Tower objects:

**List View** (default for most objects):
- Shows multiple records in a table
- Quick search and filters
- Bulk actions

**Kanban View** (for servers, commands):
- Card-based layout
- Drag-and-drop organization
- Visual status indicators

**Form View** (detail view):
- Full record details
- Edit fields
- Related data tabs

**Switch Views**:
- Use view switcher icons in top-right
- Options: List, Kanban, Form, Calendar (if available)

#### Search and Filters

**Search Bar**:
- Top of list/kanban views
- Type to search by name, code, IP address, etc.

**Filters** (click filter icon):
- Pre-defined filters (e.g., "Active Servers")
- Group by options (e.g., "Group by Partner")
- Custom filters

**Favorites**:
- Save frequently used search/filter combinations
- Click Favorites → Save Current Search

### Viewing Servers

Explore the servers in your Tower instance.

#### Step 1: Access Servers

1. Navigate to **Tower → Servers → Servers**
2. You'll see the list of all servers you have access to

#### Step 2: Understanding Server Views

**List View Columns**:
- **Name**: Server name
- **Code**: Unique identifier
- **IP Address**: Server IP or hostname
- **Partner**: Associated partner/client
- **Status**: Current status
- **Tags**: Server tags

**Kanban View Cards**:
- Server name and code
- IP address
- Quick action buttons
- Color-coded by status or tags

#### Step 3: Server Details

1. Click on a server to open form view
2. **Main Tab**: Connection details, authentication
3. **Variables Tab**: Server-specific variable values
4. **Files Tab**: Files managed on this server
5. **Commands Tab**: Available commands
6. **Logs Tab**: Server activity logs

#### Step 4: Server Actions

From server form, you can (depending on access level):
- **Test Connection**: Verify SSH connectivity
- **Run Command**: Execute command on this server
- **View Logs**: See command execution history
- **Edit**: Modify server details (Manager+)
- **Duplicate**: Create similar server (Manager+)

### Running Your First Command

Execute a command on a server.

#### Method 1: From Command Form

1. **Navigate to Command**:
   - Tower → Commands
   - Find or search for a command (e.g., "Check Disk Space")
   - Click to open

2. **Click Run Command**:
   - Button in top-right (or action menu)
   - Run Command wizard opens

3. **Select Server(s)**:
   - Choose one or more servers
   - Only servers you have access to appear
   - Click server(s) to select

4. **Configure Variables** (if command uses variables):
   - Enter or select values for each variable
   - Example: Environment = "production"

5. **Click Run**:
   - Command is submitted
   - Notification appears

6. **View Progress**:
   - Command Log record is created
   - Navigate to Tower → Logs → Command Logs
   - Find your log entry
   - Status shows: Pending → Running → Success/Error

#### Method 2: From Server Form

1. **Navigate to Server**:
   - Tower → Servers
   - Open your server

2. **Click Run Command** (button or action menu)

3. **Select Command**:
   - Choose from available commands
   - Only commands you can run appear

4. **Configure and Run**:
   - Same as Method 1 from step 4

#### Method 3: Quick Run from List

Some views offer quick run options:

1. In server list view
2. Click action menu (⋮) on server row
3. Select **Run Command**
4. Follow wizard

#### Understanding Command Execution

**Execution Flow**:

```
1. User clicks "Run Command"
2. Tower validates access and parameters
3. Command is queued (if using queue_job)
4. SSH connection to server is established
5. Command code is executed on server
6. Output is captured
7. Result is logged
8. User receives notification
9. SSH connection is closed
```

**Execution Time**:
- Simple commands: 1-5 seconds
- Complex commands: Varies
- Check command logs for status

### Viewing Command Logs

Track and review command execution results.

#### Accessing Command Logs

**Method 1: Direct Navigation**

1. Tower → Logs → Command Logs
2. You'll see list of all command executions

**Method 2: From Server**

1. Open server record
2. Logs tab
3. Shows logs for this server only

**Method 3: From Command**

1. Open command record
2. Click **Logs** smart button (shows count)
3. View all executions of this command

#### Understanding Log Fields

| Field | Description | Example |
|-------|-------------|---------|
| **Name** | Log identifier | `Command: Check Disk Space on My First Server` |
| **Server** | Target server | `My First Server` |
| **Command** | Executed command | `Check Disk Space` |
| **Status** | Execution result | `SUCCESS`, `ERROR`, `TIMEOUT` |
| **Started** | Execution start time | `2025-11-16 10:30:00` |
| **Ended** | Execution end time | `2025-11-16 10:30:05` |
| **Duration** | Total execution time | `5 seconds` |
| **Exit Code** | Command exit code | `0` (success), `1` (error) |

#### Viewing Log Details

1. Click on a log record
2. **Result** tab:
   - **Standard Output** (stdout): Command output
   - **Standard Error** (stderr): Error messages
   - **Exit Code**: Numeric result code
3. **Details** tab:
   - Execution metadata
   - Variables used
   - Timestamps

#### Interpreting Results

**Success** (Exit Code 0):
```
Status: SUCCESS
Exit Code: 0
Standard Output:
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1       100G   45G   55G  45% /
```

**Error** (Exit Code > 0):
```
Status: ERROR
Exit Code: 1
Standard Error:
bash: command-not-found: command not found
```

**Timeout**:
```
Status: TIMEOUT
Duration: 300 seconds
Message: Command execution exceeded timeout limit
```

#### Filtering and Searching Logs

**Common Filters**:
- **My Logs**: Logs for commands you ran
- **Failed Logs**: Only errors
- **Recent Logs**: Last 7 days
- **Server Logs**: Filter by specific server

**Search Examples**:
- Search by server name: `My First Server`
- Search by command: `Check Disk Space`
- Search by status: Use filter dropdown

---

## Complete Walkthrough: Basic Workflow

Let's put it all together with a complete, real-world scenario.

### Scenario: System Health Check

**Goal**: Set up automated disk space monitoring on a production server

**What We'll Do**:
1. Add a production server
2. Create a disk space check command
3. Run the command
4. View and interpret results
5. (Optional) Schedule for automated execution

---

#### Part 1: Add the Server

1. **Navigate**: Tower → Servers → Create

2. **Fill in details**:
   ```
   Name: Production App Server
   Code: prod-app-01
   IP Address: 192.168.1.50
   SSH Port: 22
   SSH Username: ubuntu
   ```

3. **Set up SSH Key**:
   - On your terminal:
     ```bash
     ssh-keygen -t rsa -b 4096 -f ~/.ssh/tower_prod
     ssh-copy-id -i ~/.ssh/tower_prod.pub ubuntu@192.168.1.50
     ```

   - Create key in Tower:
     - Tower → Configuration → Keys → Create
     - Name: `Production SSH Key`
     - Code: `prod-ssh-key`
     - SSH Key: (paste content of `~/.ssh/tower_prod`)
     - Save

   - Return to server form
   - Key field: Select `Production SSH Key`

4. **Add Tags** (optional):
   - Tags tab → Add: `Production`, `Web Server`

5. **Save Server**

6. **Test Connection**:
   - Click **Test Connection** button
   - Wait for success message
   - If failed, troubleshoot using previous sections

---

#### Part 2: Create Disk Space Command

1. **Navigate**: Tower → Commands → Create

2. **Basic Info**:
   ```
   Name: Check Disk Space and Alert if Low
   Code: disk-check-alert
   Reference: disk_alert
   ```

3. **Command Code**:
   - Check **Shell** checkbox
   - Code field:
     ```bash
     #!/bin/bash
     # Check disk space and alert if usage > 80%

     df -h / | tail -n 1 | awk '{
       usage = int($5)
       if (usage > 80) {
         print "WARNING: Disk usage is " usage "%"
         exit 1
       } else {
         print "OK: Disk usage is " usage "%"
         exit 0
       }
     }'
     ```

4. **Options**:
   - Shell: ✓
   - Use Sudo: ☐
   - Timeout: 30
   - Allow Parallel Run: ☐

5. **Assign to Server**:
   - Servers tab → Add: `Production App Server`

6. **Save Command**

---

#### Part 3: Run the Command

1. **From Command Form**:
   - Click **Run Command** button

2. **Select Server**:
   - Choose `Production App Server`
   - Click **Run**

3. **Wait for Execution**:
   - Notification: "Command is running..."
   - Takes 1-5 seconds typically

4. **Check Notification**:
   - Bell icon (top-right)
   - Should see: "Command completed successfully" (or error)

---

#### Part 4: View Results

1. **Navigate to Logs**:
   - Tower → Logs → Command Logs
   - Or click **Logs** smart button from command

2. **Find Your Log**:
   - Should be at the top (most recent)
   - Name: `Command: Check Disk Space and Alert if Low on Production App Server`

3. **Open Log Record**:
   - Click to open

4. **Review Results**:

   **If Disk Usage < 80%**:
   ```
   Status: SUCCESS
   Exit Code: 0
   Standard Output:
   OK: Disk usage is 45%

   Duration: 2 seconds
   ```

   **If Disk Usage > 80%**:
   ```
   Status: ERROR
   Exit Code: 1
   Standard Output:
   WARNING: Disk usage is 85%

   Duration: 2 seconds
   ```

5. **Interpret**:
   - Exit Code 0 = Success (disk OK)
   - Exit Code 1 = Warning (disk low)
   - Standard Output = Message

---

#### Part 5: Schedule Automated Execution (Optional)

1. **Navigate**: Tower → Scheduled Tasks → Create

2. **Basic Info**:
   ```
   Name: Daily Disk Check - Production App Server
   Code: daily-disk-check-prod
   ```

3. **Configuration**:
   - **Action**: Command
   - **Command**: `Check Disk Space and Alert if Low`
   - **Server**: `Production App Server`

4. **Schedule**:
   - **Scheduling Type**: Interval
   - **Interval**: 1
   - **Interval Unit**: Days
   - **Next Run**: (set to tomorrow morning, e.g., 6:00 AM)

5. **Save**

6. **Activate**:
   - Check **Active** checkbox
   - Save

**Result**: Command will run daily at 6 AM. Check logs daily to monitor disk usage.

---

## What You've Learned

By completing this guide, you now know how to:

✓ Add servers to Cetmix Tower
✓ Configure SSH authentication (password and key)
✓ Test SSH connectivity
✓ Create and configure variables
✓ Create shell commands
✓ Run commands on servers
✓ View and interpret command logs
✓ Set up access control
✓ Navigate the Tower interface
✓ (Optional) Schedule automated tasks

---

## Next Steps

Now that you're comfortable with the basics:

1. **Explore More Commands**:
   - System updates: `sudo apt update && sudo apt upgrade -y`
   - Service status: `sudo systemctl status nginx`
   - Log viewing: `tail -n 100 /var/log/syslog`

2. **Create Flight Plans**:
   - Combine multiple commands into workflows
   - See [Flight Plans Guide](../04-feature-guides/03-flight-plans-workflows.md)

3. **Manage Files**:
   - Upload configuration files
   - Sync files between servers
   - See [File Management Guide](../04-feature-guides/04-file-management-sync.md)

4. **Set Up More Servers**:
   - Add all your infrastructure
   - Organize with tags
   - Configure access for team members

5. **Review Common Tasks**:
   - [Common Tasks Reference](04-common-tasks.md)
   - Quick solutions for frequent operations

---

## Troubleshooting Common First-Time Issues

**Issue: Can't see Tower menu**
- **Solution**: Check user access rights (Settings → Users → Access Rights → Cetmix Tower)

**Issue: Test connection fails**
- **Solution**: Verify IP, port, username, password/key. Test manual SSH from server where Odoo runs.

**Issue: Command runs but no output**
- **Solution**: Check command logs. Command might have succeeded with no output. Verify command syntax.

**Issue: Can't find server in run command wizard**
- **Solution**: Check server assignment in command, check your access rights to the server.

**Issue: Permission denied errors**
- **Solution**: Check SSH user permissions. May need to use sudo. Enable "Use Sudo" in command options.

**Issue: Command times out**
- **Solution**: Increase timeout in command settings or global timeout in Settings → General Settings → Cetmix Tower.

---

## Getting Help

- **Common Tasks**: [Common Tasks Guide](04-common-tasks.md)
- **Feature Documentation**: [Feature Guides](../04-feature-guides/README.md)
- **Technical Details**: [Technical Guides](../07-technical-guides/README.md)
- **Issues**: [Issues & Bugs](../12-issues-bugs/README.md)

---

**Congratulations!** You've completed your first steps with Cetmix Tower.

**Continue Learning**:
- [Common Tasks →](04-common-tasks.md)
- [User Guides →](../03-user-guides/README.md)
- [Return to Getting Started](README.md)

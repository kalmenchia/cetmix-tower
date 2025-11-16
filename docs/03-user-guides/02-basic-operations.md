---
title: "Basic Operations"
section: "03-user-guides"
document_id: "UG-002"
version: "17.0"
audience: "End Users"
last_updated: "2025-11-16"
tags: ["operations", "crud", "create", "edit", "delete", "filters", "export"]
---

# Basic Operations

This guide covers the fundamental operations you'll perform regularly in Cetmix Tower, including creating, editing, and managing records.

## Table of Contents

1. [Creating Records](#creating-records)
2. [Editing and Updating Records](#editing-and-updating-records)
3. [Deleting and Archiving](#deleting-and-archiving)
4. [Using Filters and Groups](#using-filters-and-groups)
5. [Exporting Data](#exporting-data)
6. [Common UI Elements](#common-ui-elements)

---

## Creating Records

Creating new records is one of the most common operations in Cetmix Tower. The process is similar across all resource types (servers, commands, plans, files, variables).

### Basic Creation Steps

**Method 1: From List View**

1. Navigate to the desired resource (e.g., Servers, Commands, Flight Plans)
2. Click the **Create** button (usually top-left)
3. Fill in the required fields (marked with bold labels or asterisks)
4. Fill in optional fields as needed
5. Click **Save** to create the record

**Method 2: Quick Create (where available)**

1. Some views offer "Quick Create" for faster entry
2. Click **+** icon or **Create** button
3. Fill minimal required fields in popup
4. Click **Add** to create immediately
5. Or click **Edit** to open full form

### Creating Servers

**Required Fields:**
- **Name**: Descriptive server name (e.g., "Production Web Server 01")
- **IP Address** or **Domain**: Server connection address
- **SSH Port**: Usually 22 (default)
- **SSH Username**: Login username

**Optional but Recommended:**
- **Tags**: For organization (e.g., "Production", "Web Server")
- **Partner**: Related contact or company
- **Operating System**: For command compatibility
- **Notes**: Additional documentation

**Example: Creating a New Server**

```
1. Go to: Cetmix Tower → Servers → Servers
2. Click [Create]
3. Fill in fields:
   Name: Production Web Server 01
   Reference: prod_web_01 (auto-generated from name)
   IP Address: 192.168.1.100
   SSH Port: 22
   SSH Username: admin
   SSH Authentication: Password
   SSH Password: ••••••••
   Tags: Production, Web Server
4. Click [Save]
```

💡 **Tip:** The `reference` field is auto-generated from the name but can be customized. References must be unique and use only lowercase letters, numbers, and underscores.

---

### Creating Commands

**Required Fields:**
- **Name**: Command description (e.g., "Restart Nginx Service")
- **Code**: Actual command or script to execute

**Optional Fields:**
- **Servers**: Limit to specific servers
- **Operating Systems**: Restrict to compatible OS
- **Tags**: For organization
- **Variables**: Use variables for flexibility
- **Timeout**: Execution timeout in seconds

**Example: Creating a Command**

```
1. Go to: Cetmix Tower → Commands → Commands
2. Click [Create]
3. Fill in fields:
   Name: Restart Nginx Service
   Reference: restart_nginx (auto-generated)
   Code: sudo systemctl restart nginx
   Command Type: Shell
   Timeout: 300
   Tags: Services, Nginx
   OS: Ubuntu, Debian
4. Click [Save]
```

**Command Types:**
- **Shell**: Regular shell commands (bash, sh)
- **Python**: Python code execution
- **Custom**: Module-specific implementations

---

### Creating Flight Plans

Flight plans orchestrate multiple commands in sequence with error handling and conditional logic.

**Required Fields:**
- **Name**: Plan description (e.g., "Deploy Application v2.0")
- **Lines**: At least one plan line with a command

**Plan Configuration:**
- **Servers**: Target servers (or select at runtime)
- **On Error**: How to handle failures
  - `Exit with command exit code`: Stop on error
  - `Exit with custom exit code`: Stop with custom code
  - `Run next command`: Continue despite errors
- **Allow Parallel Run**: Enable concurrent executions

**Example: Creating a Flight Plan**

```
1. Go to: Cetmix Tower → Flight Plans → Flight Plans
2. Click [Create]
3. Fill in basic info:
   Name: Deploy Web Application
   Reference: deploy_web_app (auto-generated)
   Servers: Production Web Server 01
   On Error: Exit with command exit code
4. Add plan lines (in [Lines] tab):
   Line 1:
     - Sequence: 10
     - Command: Stop Application
     - On Success: Run next command
   Line 2:
     - Sequence: 20
     - Command: Pull Latest Code
     - On Success: Run next command
   Line 3:
     - Sequence: 30
     - Command: Start Application
5. Click [Save]
```

📝 **Note:** Plan lines execute in sequence order. Use gaps (10, 20, 30) to allow easy insertion of new steps later.

---

### Creating Files

Files can be pushed from Tower to servers or pulled from servers to Tower.

**Required Fields:**
- **Name**: Filename (without path, e.g., "nginx.conf")
- **Server Directory**: Target path on server
- **Source**: Tower or Server
- **File Type**: Code or Binary

**Optional Fields:**
- **Template**: Use file template for generation
- **Auto Sync**: Enable automatic synchronization
- **Code**: File content (for Tower source)

**Example: Creating a Configuration File**

```
1. Go to: Cetmix Tower → Files → Files
2. Click [Create]
3. Fill in fields:
   Name: app-config.ini
   Reference: app_config_ini (auto-generated)
   Source: Tower
   File Type: Code
   Server Directory: /opt/myapp/config
   Auto Sync: ✓ Enabled
   Auto Sync Interval: 1 day
   Code: [paste configuration content]
4. Click [Save]
```

**File Sources:**
- **Tower → Server**: Push files from Tower to servers
- **Server → Tower**: Pull files from servers to Tower

---

### Creating Variables

Variables allow dynamic values in commands and files.

**Required Fields:**
- **Name**: Variable name (e.g., "Database Password")
- **Reference**: Variable identifier (e.g., "db_password")
- **Type**: String or Options

**Variable Types:**

**String Variables:**
- Free-text entry
- Can include validation patterns
- Support applied expressions (transformations)

**Option Variables:**
- Dropdown selection
- Predefined options
- Controlled values

**Example: Creating a Variable**

```
1. Go to: Cetmix Tower → Variables → Variables
2. Click [Create]
3. Fill in fields:
   Name: Application Environment
   Reference: app_env (auto-generated)
   Type: Options
   Access Level: Manager
4. In [Options] tab, add options:
   - Reference: dev, Name: Development
   - Reference: staging, Name: Staging
   - Reference: prod, Name: Production
5. Click [Save]
```

**Using Variables:**
- In commands: `${app_env}`
- In files: `${app_env}`
- Values set per server/template/plan

---

## Editing and Updating Records

Modify existing records to update information or correct errors.

### Edit Methods

**Method 1: Edit Button (Recommended)**

1. Open record in form view
2. Click **Edit** button (top-left)
3. Modify fields as needed
4. Click **Save** to commit changes
5. Or click **Discard** to cancel

**Method 2: Inline Edit (List View)**

1. From list view, click pencil icon on row
2. Edit fields directly in list
3. Changes save automatically on field exit
4. Or press Enter to confirm

**Method 3: Bulk Edit**

1. Select multiple records (checkboxes in list view)
2. Click **Action** dropdown → **Export** or custom actions
3. Some fields support bulk updates

### Edit Guidelines

**Best Practices:**

✅ **Do:**
- Save frequently to avoid losing changes
- Review changes before saving
- Use descriptive names and references
- Document changes in notes/chatter

❌ **Don't:**
- Edit multiple records simultaneously
- Change critical fields without backup
- Modify references unless necessary
- Skip required fields

### Tracking Changes

Many records support change tracking through the **Chatter**:

- View modification history
- See who changed what and when
- Add notes about changes
- Track related activities

**Accessing Change History:**
1. Open record in form view
2. Scroll to bottom to see Chatter
3. Review logged messages
4. Add notes as needed

---

## Deleting and Archiving

Cetmix Tower supports both deleting and archiving records. Understanding the difference is important.

### Archiving Records (Recommended)

**Purpose:** Hide records without deleting them permanently

**Benefits:**
- Records can be restored later
- Maintains data integrity
- Preserves relationships
- Keeps audit trail

**How to Archive:**

**Single Record:**
1. Open record in form view
2. Click **Action** dropdown
3. Select **Archive**
4. Confirm when prompted

**Multiple Records:**
1. Select records in list view (checkboxes)
2. Click **Action** dropdown
3. Select **Archive**
4. Confirm bulk archive

**Viewing Archived Records:**
1. In list view, click **Filters**
2. Select **Archived**
3. Archived records appear grayed out
4. Can unarchive by selecting **Action** → **Unarchive**

---

### Deleting Records (Permanent)

**Purpose:** Permanently remove records from database

⚠️ **Warning:** Deletion is usually permanent and cannot be undone. Archive instead when possible.

**When to Delete:**
- Test or duplicate records
- Invalid data entries
- Records with no dependencies

**How to Delete:**

**Single Record:**
1. Open record in form view
2. Click **Action** dropdown
3. Select **Delete**
4. Confirm deletion (may require password)

**Multiple Records:**
1. Select records in list view
2. Click **Action** dropdown
3. Select **Delete**
4. Confirm bulk deletion

**Deletion Restrictions:**

Some records cannot be deleted if:
- They have related records (e.g., command used in flight plans)
- They're referenced by logs
- System constraints prevent deletion
- User lacks sufficient permissions

**Error Example:**
```
Cannot delete command "Backup Database" because:
- Used in 3 flight plans
- Has 15 execution logs

Suggestion: Archive instead of delete
```

---

## Using Filters and Groups

Filters and groups help you organize and find records efficiently.

### Standard Filters

Available in most list views:

**Common Filters:**
- **Active**: Show only active records (default)
- **Archived**: Show archived records
- **My Records**: Records created by you
- **Shared with Me**: Records you have access to

**Accessing Filters:**
1. Click **Filters** dropdown (top of list)
2. Select desired filter
3. Multiple filters can combine

### Custom Filters

Create your own filters for specific needs:

**Creating Custom Filter:**
1. Click **Filters** → **Add Custom Filter**
2. Configure filter conditions:
   - Select field (e.g., "Tags")
   - Choose operator (e.g., "contains")
   - Enter value (e.g., "Production")
3. Click **Apply**
4. Optionally save for reuse

**Example Custom Filters:**

```
Servers Online:
  Status = Online

Production Web Servers:
  Tags contains "Production"
  AND Tags contains "Web Server"

Commands Created This Month:
  Created On >= Start of Month
  AND Created On <= Today

High Priority Plans:
  Priority = High
  AND Status != Complete
```

**Saving Custom Filters:**
1. Configure filter
2. Click **Favorites** → **Save current search**
3. Enter filter name
4. Choose: Personal or Shared with team
5. Click **Save**

---

### Group By

Organize records into collapsible groups:

**Using Group By:**
1. Click **Group By** dropdown
2. Select grouping field:
   - Status
   - Tags
   - Partner
   - Created Date
   - Custom fields

**Example Groups:**

```
Group by Status:
  ▼ Online (25 servers)
  ▼ Offline (3 servers)
  ▼ Unknown (1 server)

Group by Tags:
  ▼ Production (15 servers)
  ▼ Development (10 servers)
  ▼ Testing (4 servers)
```

**Multi-Level Grouping:**
1. Apply first group (e.g., Tags)
2. Apply second group (e.g., Status)
3. Records organize hierarchically

```
▼ Production
  ▼ Online (12)
  ▼ Offline (3)
▼ Development
  ▼ Online (8)
  ▼ Offline (2)
```

---

## Exporting Data

Export records to Excel, CSV, or other formats for external use.

### Export Methods

**Method 1: Export All Fields**

1. Select records in list view (or leave unselected for all)
2. Click **Action** → **Export**
3. Choose export format:
   - Excel (XLSX)
   - CSV
   - PDF (some views)
4. Select fields to export
5. Click **Export**
6. Download file

**Method 2: Export Visible Fields**

1. Configure list view with desired columns
2. Select records
3. Click **Action** → **Export**
4. Choose "Visible Fields Only"
5. Export and download

### Export Options

**Field Selection:**
- **All fields**: Export every field in the model
- **Visible fields**: Only columns shown in list view
- **Custom selection**: Choose specific fields

**Export Formats:**

**Excel (XLSX):**
- Preserves formatting
- Multiple sheets for related data
- Best for data analysis

**CSV:**
- Plain text format
- Universal compatibility
- Best for imports to other systems

**PDF:**
- Human-readable format
- Printable reports
- Not suitable for re-import

**Example Export:**

```
Exporting Servers:
1. Go to Servers list
2. Apply filter: Status = Online
3. Select 10 servers (or select all)
4. Click [Action] → [Export]
5. Select format: Excel
6. Choose fields:
   - Name
   - IP Address
   - Status
   - Tags
   - Partner
7. Click [Export]
8. File downloads: servers_export_2025-11-16.xlsx
```

### Import Data (Reverse Operation)

**Importing Records:**
1. Prepare CSV/Excel file with correct format
2. Go to list view
3. Click **Favorites** → **Import records**
4. Upload file
5. Map columns to fields
6. Validate and import

📝 **Note:** Import templates can be downloaded from the import wizard to ensure correct format.

---

## Common UI Elements

Understanding common UI elements helps you work more efficiently.

### Buttons

**Standard Action Buttons:**

| Button | Purpose | Location |
|--------|---------|----------|
| **Create** | Add new record | List view top-left |
| **Save** | Commit changes | Form view top-left |
| **Discard** | Cancel changes | Form view top-left |
| **Edit** | Enable editing | Form view (when viewing) |
| **Action** | Additional operations | Form/List view top-right |

**Smart Buttons:**

Located at top of form views, show related record counts:

```
┌─────────────────────────────────────────────┐
│ [3 Commands] [12 Files] [2 Plans] [5 Logs] │
└─────────────────────────────────────────────┘
```

Click smart buttons to view related records.

---

### Status Indicators

**Visual Status Indicators:**

- **🟢 Green Dot**: Online/Active/Success
- **🔴 Red Dot**: Offline/Error/Failed
- **🟡 Yellow Dot**: Warning/Pending
- **⚫ Gray Dot**: Unknown/Disabled

**Status Badges:**

```
[Active]    - Green badge
[Archived]  - Gray badge
[Running]   - Blue badge, animated
[Failed]    - Red badge
[Success]   - Green badge with checkmark
```

---

### Wizards

Wizards guide you through multi-step processes.

**Common Wizards:**

**1. Run Command Wizard**
- Select servers
- Set variable values
- Configure options
- Execute command

**2. Run Flight Plan Wizard**
- Choose target servers
- Set plan variables
- Review plan steps
- Start execution

**3. File Sync Wizard**
- Select files
- Choose direction
- Confirm servers
- Sync files

**Using Wizards:**
1. Click action that opens wizard
2. Follow step-by-step prompts
3. Fill required information
4. Review summary
5. Click final action button
6. View results

---

### Tabs and Notebooks

Forms often use tabs to organize information:

**Common Tab Patterns:**

**Server Form Tabs:**
- General: Basic information
- Connection: SSH settings
- Variables: Server-specific variables
- Commands: Available commands
- Files: Associated files
- Logs: Activity history

**Navigation:**
- Click tab name to switch
- Unsaved changes persist across tabs
- Red indicators show required fields in other tabs

---

### Chatter and Activities

Located at bottom of form views:

**Chatter Features:**
- **Messages**: Communication and notes
- **Activities**: Scheduled tasks and reminders
- **Followers**: Users tracking this record

**Using Chatter:**

**Send Message:**
1. Click in message box
2. Type message
3. @mention users to notify
4. Click **Send**

**Schedule Activity:**
1. Click **Activities** → **Schedule Activity**
2. Choose activity type
3. Set due date
4. Assign to user
5. Click **Schedule**

**Add Followers:**
1. Click **Followers** icon
2. Select users/channels
3. They receive updates

---

### Many2one and Many2many Fields

**Many2one Fields** (single selection):
- Click dropdown to select
- Type to search
- Click **Create** to add new record inline

**Many2many Fields** (multiple selection):
- Click to select multiple items
- Remove by clicking X on tag
- Add new with **Create** option

**Example:**
```
Tags: [Production] [Web Server] [Ubuntu] [+]
```

---

## Next Steps

Now that you understand basic operations, proceed to:

- **[Common Tasks](./03-common-tasks.md)**: Learn specific daily operations
- **[Feature Guides](../04-feature-guides/README.md)**: Deep dive into specific features

---

**Navigation:**
- [← Previous: Navigation](./01-navigation.md)
- [Next: Common Tasks →](./03-common-tasks.md)
- [↑ User Guides Home](./README.md)
- [↑↑ Documentation Home](../README.md)

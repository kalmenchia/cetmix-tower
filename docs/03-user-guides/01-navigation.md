---
title: "Navigation Guide"
section: "03-user-guides"
document_id: "UG-001"
version: "17.0"
audience: "End Users"
last_updated: "2025-11-16"
tags: ["navigation", "interface", "ui", "menu", "views"]
---

# Navigation Guide

This guide will help you navigate the Cetmix Tower interface efficiently and understand how to access different features and resources.

## Table of Contents

1. [Tower Interface Overview](#tower-interface-overview)
2. [Main Menu Structure](#main-menu-structure)
3. [View Types](#view-types)
4. [Navigation Tips and Shortcuts](#navigation-tips-and-shortcuts)
5. [Search and Filtering](#search-and-filtering)
6. [User Preferences](#user-preferences)

---

## Tower Interface Overview

Cetmix Tower follows the standard Odoo interface layout with specialized features for server management. The interface consists of several key areas:

### Interface Layout

```
┌─────────────────────────────────────────────────────────┐
│  Top Navigation Bar (Apps, Search, User Menu)          │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌──────────────┬──────────────────────────────────┐   │
│  │              │                                  │   │
│  │   Sidebar    │      Main Content Area          │   │
│  │   Menu       │                                  │   │
│  │              │  • List Views                    │   │
│  │  • Servers   │  • Form Views                    │   │
│  │  • Commands  │  • Kanban Views                  │   │
│  │  • Plans     │  • Calendar Views                │   │
│  │  • Files     │                                  │   │
│  │  • Variables │                                  │   │
│  │  • Config    │                                  │   │
│  │              │                                  │   │
│  └──────────────┴──────────────────────────────────┘   │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### Key Interface Elements

**1. Top Navigation Bar**
- **Apps Menu**: Access different Odoo applications
- **Search**: Global search across all records
- **User Menu**: Access preferences, logout, and user settings

**2. Sidebar Menu**
- Quick access to main Tower features
- Collapsible for more screen space
- Customizable based on user permissions

**3. Main Content Area**
- Displays lists, forms, and other views
- Context-sensitive based on selected menu item
- Supports multiple tabs for parallel work

**4. Action Buttons**
- **Create**: Add new records
- **Edit**: Modify existing records
- **Save**: Commit changes
- **Discard**: Cancel changes
- **Action**: Access additional operations (delete, archive, export, etc.)

---

## Main Menu Structure

The Cetmix Tower main menu is organized into logical sections based on resource types and operations.

### Primary Menu Items

#### 1. Servers

**Path:** `Cetmix Tower → Servers`

Access and manage your server infrastructure:

- **Servers**: List of all configured servers
  - View server status and connection details
  - Monitor server health
  - Execute commands on individual servers

- **Server Templates**: Reusable server configurations
  - Create templates for similar servers
  - Quick deployment from templates

- **Server Logs**: Historical server activity
  - Connection logs
  - Operation logs
  - Error tracking

- **Operating Systems**: Supported OS configurations
  - OS-specific command compatibility
  - Version management

**Typical Use Cases:**
- Check server status
- Connect to servers
- Review server logs
- Create new servers from templates

---

#### 2. Commands

**Path:** `Cetmix Tower → Commands`

Manage commands that can be executed on servers:

- **Commands**: Library of available commands
  - Shell commands
  - Python scripts
  - Custom commands with variables

- **Command Logs**: Execution history
  - Success/failure status
  - Output and error messages
  - Execution time and duration

**Typical Use Cases:**
- Run system commands
- Check command execution status
- Review command output
- Debug failed commands

---

#### 3. Flight Plans

**Path:** `Cetmix Tower → Flight Plans`

Create and manage multi-step automation workflows:

- **Flight Plans**: Orchestrated command sequences
  - Multiple commands in order
  - Conditional execution
  - Error handling strategies

- **Flight Plan Lines**: Individual steps in plans
  - Command assignment
  - Execution order
  - Conditions and actions

- **Flight Plan Logs**: Execution history
  - Step-by-step results
  - Overall plan status
  - Performance metrics

**Typical Use Cases:**
- Deploy applications
- Run maintenance routines
- Execute complex workflows
- Automate repetitive tasks

---

#### 4. Files

**Path:** `Cetmix Tower → Files`

Manage files synchronized between Tower and servers:

- **Files**: Individual file records
  - Push files to servers
  - Pull files from servers
  - Auto-sync configuration

- **File Templates**: Reusable file configurations
  - Template-based file generation
  - Variable substitution
  - Multiple server deployment

**Typical Use Cases:**
- Deploy configuration files
- Retrieve log files
- Sync application files
- Manage file templates

---

#### 5. Variables

**Path:** `Cetmix Tower → Variables`

Define and manage variables used in commands and files:

- **Variables**: Variable definitions
  - String variables
  - Option-based variables
  - Validation rules

- **Variable Values**: Assigned values
  - Server-specific values
  - Template-specific values
  - Plan-specific values

**Typical Use Cases:**
- Configure environment variables
- Set deployment parameters
- Manage configuration options
- Define reusable values

---

#### 6. Tools & Utilities

**Path:** `Cetmix Tower → Tools`

Access utility features:

- **Shortcuts**: Quick access to frequent operations
  - Custom shortcuts
  - Command shortcuts
  - Plan shortcuts

- **Scheduled Tasks**: Automated operations
  - Recurring command execution
  - Scheduled file syncs
  - Maintenance windows

- **Tags**: Organize resources
  - Server grouping
  - Command categorization
  - Plan organization

---

#### 7. Configuration

**Path:** `Cetmix Tower → Configuration`

System-wide settings and configuration:

- **Settings**: Global Tower settings
  - Connection timeouts
  - Security settings
  - Integration configuration

- **SSH Keys**: Manage SSH authentication
  - Public/private key pairs
  - Key distribution
  - Security management

- **Access Control**: User permissions
  - User roles (User, Manager, Root)
  - Record-level permissions
  - Team assignments

---

## View Types

Cetmix Tower uses different view types to display and interact with data. Understanding these views will help you work more efficiently.

### List View (Tree View)

**When to use:** Viewing multiple records, comparing data, bulk operations

**Features:**
- Tabular display of records
- Sortable columns (click column header)
- Multi-select for bulk actions
- Quick filters
- Group by functionality
- Export to Excel/CSV

**Example:** Viewing all servers with their status

```
┌────────────────────────────────────────────────────────┐
│ Name          │ IP Address    │ Status  │ Tags        │
├────────────────────────────────────────────────────────┤
│ Web Server 01 │ 192.168.1.10 │ ● Online│ Production  │
│ DB Server 01  │ 192.168.1.20 │ ● Online│ Production  │
│ Test Server   │ 192.168.1.30 │ ○ Offline│ Development│
└────────────────────────────────────────────────────────┘
```

**Tips:**
- Click column headers to sort
- Use checkboxes for bulk operations
- Right-click for context menu
- Adjust column width by dragging

---

### Form View

**When to use:** Viewing/editing detailed information for a single record

**Features:**
- Detailed field display
- Tabbed organization
- Related records (One2many, Many2many)
- Action buttons
- Chatter for messages and activities

**Example:** Server detail form

```
┌─────────────────────────────────────────────────────┐
│ [Save] [Discard] [Action ▼]                        │
├─────────────────────────────────────────────────────┤
│ Name: Web Server 01        Status: ● Online        │
│ IP Address: 192.168.1.10   Port: 22                │
│                                                     │
│ ┌─[Connection]─[Variables]─[Commands]─[Logs]────┐ │
│ │ SSH Username: admin                            │ │
│ │ SSH Password: ••••••                           │ │
│ │ Authentication: Password                       │ │
│ └────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────┘
```

---

### Kanban View

**When to use:** Visual organization, drag-and-drop workflow, status tracking

**Features:**
- Card-based display
- Drag-and-drop between columns
- Color coding
- Quick create
- Grouping by fields

**Example:** Commands organized by status

```
┌─────────────┬─────────────┬─────────────┬─────────────┐
│   Draft     │   Ready     │  Running    │  Complete   │
├─────────────┼─────────────┼─────────────┼─────────────┤
│ ┌─────────┐ │ ┌─────────┐ │ ┌─────────┐ │ ┌─────────┐ │
│ │ Deploy  │ │ │ Backup  │ │ │ Update  │ │ │ Restart │ │
│ │ App v2  │ │ │ DB      │ │ │ Packages│ │ │ Service │ │
│ └─────────┘ │ └─────────┘ │ └─────────┘ │ └─────────┘ │
│             │             │             │             │
│ ┌─────────┐ │             │             │ ┌─────────┐ │
│ │ Config  │ │             │             │ │ Health  │ │
│ │ Nginx   │ │             │             │ │ Check   │ │
│ └─────────┘ │             │             │ └─────────┘ │
└─────────────┴─────────────┴─────────────┴─────────────┘
```

---

### Calendar View

**When to use:** Scheduled tasks, planning operations, time-based viewing

**Features:**
- Day/Week/Month views
- Drag-and-drop scheduling
- Color coding
- Quick create events
- Filter by criteria

**Example:** Scheduled tasks calendar

```
┌───────────────────────────────────────────────────────┐
│ ◄ November 2025 ►        [Day][Week][Month]          │
├───────┬───────┬───────┬───────┬───────┬───────┬───────┤
│  Mon  │  Tue  │  Wed  │  Thu  │  Fri  │  Sat  │  Sun  │
├───────┼───────┼───────┼───────┼───────┼───────┼───────┤
│   14  │   15  │   16  │   17  │   18  │   19  │   20  │
│       │       │ 02:00 │       │       │       │       │
│       │       │ Backup│       │       │       │       │
│       │       │ ────  │       │       │       │       │
│       │       │ 18:00 │       │       │       │       │
│       │       │ Deploy│       │       │       │       │
└───────┴───────┴───────┴───────┴───────┴───────┴───────┘
```

---

## Navigation Tips and Shortcuts

### Keyboard Shortcuts

Master these shortcuts to navigate faster:

| Shortcut | Action |
|----------|--------|
| `Alt + C` | Create new record |
| `Alt + E` | Edit current record |
| `Alt + S` | Save current record |
| `Alt + D` | Discard changes |
| `Alt + P` | Print/PDF export |
| `Ctrl + K` | Open quick search |
| `Escape` | Close dialog/cancel |

### Navigation Patterns

**Breadcrumbs**
- Use breadcrumbs at the top to navigate back
- Click any breadcrumb level to jump to that view

**Browser Back/Forward**
- Browser navigation buttons work within Tower
- Safe to use back button after saving

**Multiple Tabs**
- Open records in new tabs (Ctrl+Click or Middle-Click)
- Keep multiple contexts open simultaneously

### Quick Actions

**From List View:**
1. **Quick Create**: Click "Create" for immediate form
2. **Quick Edit**: Click pencil icon for inline editing
3. **Quick View**: Click eye icon for preview

**From Form View:**
1. **Save & New**: Save and create another record
2. **Save & Close**: Save and return to list
3. **Duplicate**: Create copy of current record

---

## Search and Filtering

Cetmix Tower provides powerful search and filtering capabilities to help you find records quickly.

### Search Bar

Located at the top of list views:

**Basic Search:**
- Type keywords to search across multiple fields
- Searches name, reference, and description fields
- Real-time results as you type

**Example:**
```
Search: "production web"
Results: All servers/commands/plans with "production" or "web"
```

### Filters

**Pre-defined Filters:**
- Click filter dropdown to see available filters
- Common filters: Active, Archived, My Records, Recent

**Custom Filters:**
1. Click "Add Custom Filter"
2. Select field, operator, and value
3. Click "Apply"
4. Save filter for future use

**Example Filters:**
- `Status` `is` `Online` - Show only online servers
- `Tags` `contains` `Production` - Production resources only
- `Created On` `is` `Today` - Recently created records

### Group By

**Purpose:** Organize records into collapsible groups

**How to use:**
1. Click "Group By" dropdown
2. Select grouping field (e.g., Status, Tags, Partner)
3. Records organize into expandable groups

**Multi-level Grouping:**
- Group by multiple fields simultaneously
- Example: Group by Tags → then by Status

**Example:**
```
▼ Production (12 servers)
  ├─ Online (10)
  └─ Offline (2)
▼ Development (5 servers)
  ├─ Online (4)
  └─ Offline (1)
```

### Saved Filters

**Create Personal Filters:**
1. Configure filters and groups
2. Click "Favorites" → "Save current search"
3. Name your filter
4. Choose visibility (Personal/Shared)

**Access Saved Filters:**
- Click "Favorites" dropdown
- Select saved filter
- Filter applies instantly

---

## User Preferences

Customize your Tower experience through user preferences.

### Accessing Preferences

1. Click your name in the top-right corner
2. Select "Preferences"
3. Configure available options

### Key Preference Options

**Language & Localization:**
- Interface language
- Date format
- Time zone
- Number format

**Display Options:**
- Default list size (items per page)
- Default view type (List/Kanban)
- Sidebar auto-hide

**Notifications:**
- Email notification settings
- In-app notifications
- Activity reminders

**Security:**
- Change password
- Two-factor authentication
- API keys management

### Company-Specific Settings

Some preferences may be controlled by company settings:
- Contact your Tower administrator for company-wide settings
- User preferences override company defaults where allowed

---

## Next Steps

Now that you understand how to navigate Cetmix Tower, continue with:

- **[Basic Operations](./02-basic-operations.md)**: Learn how to create, edit, and manage records
- **[Common Tasks](./03-common-tasks.md)**: Step-by-step guides for daily operations

---

**Navigation:**
- [← Back to User Guides](./README.md)
- [Next: Basic Operations →](./02-basic-operations.md)
- [↑ Documentation Home](../README.md)

---
title: "Odoo Documentation Template & Structure Guide"
version: "1.0"
created: "2025-11-13"
maintained_by: "E-Global SCM Development Team"
purpose: "Reference template for creating comprehensive Odoo project documentation"
---

# Odoo Documentation Template & Structure Guide

## Purpose

This document serves as a **blueprint** for creating comprehensive documentation for Odoo custom modules and projects. It captures all the ideas, formats, structures, and layouts used in the Dr Ko Clinics Odoo v14.0 documentation project.

Use this template when:
- Starting documentation for a new Odoo project
- Standardizing documentation across multiple projects
- Creating consistent structure for custom modules
- Ensuring comprehensive coverage of all aspects

---

## Table of Contents

1. [Documentation Philosophy](#documentation-philosophy)
2. [Folder Structure](#folder-structure)
3. [Three-Audience Approach](#three-audience-approach)
4. [Section Breakdown](#section-breakdown)
5. [File Naming Conventions](#file-naming-conventions)
6. [Markdown Standards](#markdown-standards)
7. [YAML Frontmatter](#yaml-frontmatter)
8. [Content Templates](#content-templates)
9. [Integration Options](#integration-options)
10. [Best Practices](#best-practices)

---

## Documentation Philosophy

### Core Principles

1. **Audience-Driven Content**
   - Separate content for Administrators, End Users, and Developers
   - Each audience gets appropriate depth and technical detail
   - Cross-reference between audience levels

2. **Feature-Centric Organization**
   - Organize by business workflows, not just technical modules
   - Group related functionality together
   - Mirror real-world usage patterns

3. **Comprehensive Coverage**
   - Cover setup, configuration, usage, troubleshooting, and development
   - Include real code examples with file paths and line numbers
   - Provide SQL queries, screenshots, and diagrams

4. **Living Documentation**
   - Integrate with issue tracking (GitHub, Jira)
   - Automate updates where possible
   - Regular review cycles

5. **Searchability & Navigation**
   - Clear hierarchy and cross-references
   - Consistent naming patterns
   - Comprehensive README files at each level

---

## Folder Structure

### Root Documentation Directory

```
docs/
├── README.md                          # Main navigation hub
│
├── 01-overview/                       # Project overview
│   ├── README.md
│   ├── 01-project-overview.md
│   ├── 02-system-architecture.md
│   ├── 03-technology-stack.md
│   └── 04-module-list.md
│
├── 02-getting-started/                # Quick start guides
│   ├── README.md
│   ├── 01-installation.md
│   ├── 02-configuration.md
│   ├── 03-first-steps.md
│   └── 04-common-tasks.md
│
├── 03-user-guides/                    # End user documentation
│   ├── README.md
│   ├── 01-navigation.md
│   ├── 02-basic-operations.md
│   └── [feature-specific-guides].md
│
├── 04-feature-guides/                 # Feature-centric documentation
│   ├── README.md
│   ├── setup-configuration/           # Foundation features
│   ├── [business-domain-1]/           # Domain-specific features
│   ├── [business-domain-2]/
│   └── maintenance-operations/        # Admin/maintenance
│
├── 05-module-references/              # Technical module docs
│   ├── README.md
│   ├── custom-modules/
│   └── third-party-modules/
│
├── 06-api-references/                 # API documentation
│   ├── README.md
│   ├── 01-rest-api.md
│   ├── 02-odoo-orm-api.md
│   └── 03-external-integrations.md
│
├── 07-technical-guides/               # Developer documentation
│   ├── README.md
│   ├── models-and-orm.md
│   ├── views-and-qweb.md
│   ├── controllers-and-routing.md
│   └── javascript-and-owl.md
│
├── 08-development-workflows/          # Development processes
│   ├── README.md
│   ├── coding-standards.md
│   ├── testing-guidelines.md
│   ├── git-workflow.md
│   └── code-review-process.md
│
├── 09-deployment-operations/          # DevOps documentation
│   ├── README.md
│   ├── deployment-guide.md
│   ├── backup-restore.md
│   ├── monitoring.md
│   └── troubleshooting.md
│
├── 10-security-compliance/            # Security documentation
│   ├── README.md
│   ├── security-guidelines.md
│   ├── user-permissions.md
│   └── data-privacy.md
│
├── 11-customization-extensions/       # Customization guides
│   ├── README.md
│   ├── custom-fields.md
│   ├── custom-reports.md
│   └── custom-workflows.md
│
├── 12-issues-bugs/                    # Issue tracking
│   ├── README.md
│   ├── 01-known-issues-by-severity.md
│   ├── 02-known-issues-by-module.md
│   ├── 03-known-issues-by-category.md
│   ├── [ISSUE-XXX].md                 # Individual issue docs
│   └── 10-bug-reporting-guide.md
│
└── 13-potential-upgrade/              # Future planning
    ├── README.md
    ├── 01-odoo-version-upgrades.md
    ├── 02-module-upgrades.md
    └── 12-upgrade-roadmap.md
```

### Rationale for Structure

- **Section 01-03**: Foundation and quick start (least technical)
- **Section 04**: Core content - feature-based organization
- **Section 05-08**: Technical documentation (developers)
- **Section 09-11**: Operations and customization
- **Section 12-13**: Issues and future planning

---

## Three-Audience Approach

### Audience Definitions

#### 1. System Administrator
**Who**: IT staff, system admins, configuration managers
**Needs**:
- Installation and configuration
- System maintenance
- User management
- Performance tuning
- Backup and recovery

**Content Depth**: Medium-High
**Technical Level**: Moderate to High

#### 2. End User
**Who**: Daily users (staff, operators, data entry)
**Needs**:
- How to perform daily tasks
- Step-by-step procedures
- Screenshots and examples
- Common scenarios
- Error resolution

**Content Depth**: Medium
**Technical Level**: Low to Moderate

#### 3. Developer
**Who**: Programmers, module developers, technical consultants
**Needs**:
- Code examples with file paths
- API documentation
- Database schema
- Architecture details
- Development workflows

**Content Depth**: High
**Technical Level**: High

### Content Template for Three Audiences

```markdown
# Feature Name

## Overview
[Brief description suitable for all audiences]

---

## For System Administrators

### Configuration
- Configuration steps
- System requirements
- Security considerations
- Performance settings

### Maintenance
- Monitoring requirements
- Troubleshooting steps
- Backup considerations

---

## For End Users

### Getting Started
- Accessing the feature
- Basic navigation
- Common tasks

### Step-by-Step Procedures

#### Procedure 1: [Task Name]
1. Navigate to...
2. Click on...
3. Enter...
4. Save...

**Screenshot Placeholder**: [Image: procedure-1-step-3.png]

### Common Scenarios
[Real-world examples]

---

## For Developers

### Technical Architecture
- Module dependencies
- Database tables
- Key models and methods

### Code Examples

**File**: `module_name/models/model_name.py`
```python
# Example code with line references
def method_name(self):
    # Implementation
    pass
```

### Database Schema

```sql
-- Table structure
CREATE TABLE table_name (
    id SERIAL PRIMARY KEY,
    -- columns
);
```

### API Usage
[API examples]

---

## Related Documentation
- [Link to related feature guide]
- [Link to technical reference]
- [Link to troubleshooting]
```

---

## Section Breakdown

### Section 04: Feature Guides (Main Content Section)

This is the **heart** of the documentation. Organize by business domains:

#### Structure Example

```
04-feature-guides/
├── README.md
│
├── setup-configuration/               # Foundation
│   ├── 01-system-setup.md
│   ├── 02-master-tables.md
│   ├── 03-packages.md
│   ├── 04-discounts-promotions.md
│   └── 05-initial-configuration.md
│
├── [domain-1]/                        # Business Domain 1
│   ├── 06-feature-a.md
│   ├── 07-feature-b.md
│   └── 08-feature-c.md
│
├── [domain-2]/                        # Business Domain 2
│   ├── 09-feature-x.md
│   ├── 10-feature-y.md
│   └── 11-feature-z.md
│
├── reporting-analytics/               # Reporting
│   ├── 22-dashboard.md
│   ├── 23-daily-reports.md
│   └── 24-monthly-reports.md
│
├── performance-optimization/          # Performance
│   ├── 26-queue-jobs.md
│   └── 27-cron-jobs.md
│
└── maintenance-operations/            # Maintenance
    ├── 28-sql-patches.md
    └── 29-temp-fixes.md
```

#### Domain Examples (Customize for Your Project)

**Healthcare/Clinic Example**:
- `patient-management/`
- `appointment-scheduling/`
- `sales-operations/`
- `inventory-management/`
- `reporting-analytics/`

**E-commerce Example**:
- `product-catalog/`
- `order-management/`
- `customer-management/`
- `payment-processing/`
- `shipping-fulfillment/`

**Manufacturing Example**:
- `production-planning/`
- `inventory-control/`
- `quality-management/`
- `maintenance/`
- `procurement/`

**Retail Example**:
- `pos-operations/`
- `inventory-management/`
- `customer-loyalty/`
- `promotions-discounts/`
- `multi-store-management/`

---

## File Naming Conventions

### General Rules

1. **Use lowercase with hyphens**: `feature-name.md` not `FeatureName.md`
2. **Prefix with numbers for ordering**: `01-first.md`, `02-second.md`
3. **Descriptive names**: `patient-search.md` not `search.md`
4. **Consistent patterns**: All guides follow same naming

### Naming Patterns

#### README Files
- `README.md` - Section overview and navigation

#### Guide Files
- `[number]-[feature-name].md` - Feature guides
- Example: `15-take-now-fulfillment.md`

#### Issue Files
- `[ISSUE-ID]-[short-description].md` - Issue documentation
- `PERF-001-take-now-timeout.md`
- `BUG-042-payment-calculation.md`

#### Reference Files
- `[module-name]-reference.md` - Module documentation
- `egl_clinic_sales-reference.md`

---

## Markdown Standards

### Document Structure

Every markdown file should follow this structure:

```markdown
---
title: "Document Title"
version: "1.0"
last_updated: "YYYY-MM-DD"
maintained_by: "Team Name"
status: "Active|Draft|Deprecated"
---

# Document Title

## Overview
[Brief description - 2-3 sentences]

**Audience**: [Administrator|End User|Developer|All]

---

## Table of Contents
1. [Section 1](#section-1)
2. [Section 2](#section-2)
3. [Section 3](#section-3)

---

## Section 1

Content here...

### Subsection 1.1

More content...

---

## Related Documentation

- [Link to related doc 1](../path/to/doc1.md)
- [Link to related doc 2](../path/to/doc2.md)

---

**Last Updated**: YYYY-MM-DD
**Next Review**: YYYY-MM-DD
**Maintained By**: Team Name
```

### Formatting Standards

#### Code Blocks

**Python Example**:
```markdown
**File**: `/path/to/module/models/model_name.py:123-145`

```python
class ModelName(models.Model):
    _name = 'model.name'

    field_name = fields.Char(string="Field Label")

    def method_name(self):
        """Method description."""
        # Implementation at line 130
        result = self.env['other.model'].search([])
        return result
\```
```

**SQL Example**:
```markdown
**Query**: Find duplicate patient records

```sql
SELECT
    identification_id,
    COUNT(*) as duplicate_count,
    STRING_AGG(id::text, ', ') as record_ids
FROM res_partner
WHERE identification_id IS NOT NULL
  AND is_patient = TRUE
GROUP BY identification_id
HAVING COUNT(*) > 1;
\```
```

**XML Example**:
```markdown
**File**: `/path/to/module/views/view_name.xml:45-67`

```xml
<record id="view_form_model" model="ir.ui.view">
    <field name="name">model.form</field>
    <field name="model">model.name</field>
    <field name="arch" type="xml">
        <form>
            <sheet>
                <field name="field_name"/>
            </sheet>
        </form>
    </field>
</record>
\```
```

#### Tables

```markdown
| Column 1 | Column 2 | Column 3 |
|----------|----------|----------|
| Data 1   | Data 2   | Data 3   |
| Data 4   | Data 5   | Data 6   |
```

#### Callouts and Alerts

```markdown
**⚠️ WARNING**: Critical information that could cause data loss

**ℹ️ INFO**: Helpful information or tips

**✓ SUCCESS**: Confirmation or positive outcome

**🐛 BUG**: Known issue or limitation

**📝 NOTE**: Additional context or explanation
```

#### Cross-References

Always provide context with links:

```markdown
**Related Documentation**:
- [Feature Guide 15: Take-Now Process](../04-feature-guides/sales-operations/15-take-now.md)
- [Technical Reference: Models](../07-technical-guides/models-and-orm.md#performance-optimization)
- [Issue PERF-001](../12-issues-bugs/PERF-001-take-now-timeout.md)

**Code References**:
- Package selection logic: `wizard/consume_package_wiz.py:78-137`
- Fulfillment processing: `models/sale.py:1909-2020`
```

---

## YAML Frontmatter

### Standard Frontmatter

Every markdown file should include YAML frontmatter:

```yaml
---
title: "Document Title"
version: "1.0"
last_updated: "YYYY-MM-DD"
maintained_by: "Team Name"
status: "Active"
audience: "Administrator|End User|Developer|All"
module: "module_name"
priority: "High|Medium|Low"
---
```

### Extended Frontmatter (for Issue Tracking)

```yaml
---
title: "Issue Title"
issue_id: "PERF-001"
severity: "Critical|High|Medium|Low"
status: "Open|In Progress|Resolved|Closed"
reported_date: "YYYY-MM-DD"
assigned_to: "Developer Name"
module: "module_name"
category: "Performance|Data|UI|Integration|Configuration"
github_issue: "#123"
jira_ticket: "PROJ-456"
---
```

---

## Content Templates

### Template 1: Feature Guide

```markdown
---
title: "Feature Name"
version: "1.0"
last_updated: "YYYY-MM-DD"
maintained_by: "Team Name"
status: "Active"
audience: "All"
---

# Feature Name

## Overview

[2-3 sentence description of the feature and its purpose]

**Key Benefits**:
- Benefit 1
- Benefit 2
- Benefit 3

**Related Modules**: `module_1`, `module_2`

---

## For System Administrators

### Prerequisites
- Prerequisite 1
- Prerequisite 2

### Installation
1. Install module via Apps menu
2. Configure settings
3. Set up master data

### Configuration

#### Setting 1
**Location**: Settings → Configuration → Feature
**Purpose**: [Explanation]

```python
# Configuration code example
```

### Security & Permissions

| Group | Access Rights |
|-------|---------------|
| Manager | Full access |
| User | Read/Write |
| Portal | Read only |

---

## For End Users

### Accessing the Feature

**Navigation**: Main Menu → Section → Feature

### Common Tasks

#### Task 1: [Task Name]

**Purpose**: [What this accomplishes]

**Steps**:
1. Navigate to [location]
2. Click [button/link]
3. Fill in:
   - Field 1: [description]
   - Field 2: [description]
4. Click "Save"

**Screenshot Placeholder**: [Image: task-1-overview.png]

**Result**: [What happens after completion]

#### Task 2: [Task Name]
[Similar structure]

### Tips & Best Practices
- Tip 1
- Tip 2
- Tip 3

---

## For Developers

### Module Information
- **Module Name**: `module_name`
- **Dependencies**: `module_1`, `module_2`
- **Version**: 14.0.1.0.0

### Technical Architecture

```
Module Structure:
module_name/
├── models/
│   ├── model_a.py
│   └── model_b.py
├── views/
│   └── views.xml
├── wizard/
│   └── wizard.py
└── __manifest__.py
```

### Key Models

#### Model 1: `model.name`

**File**: `module_name/models/model_name.py`

```python
class ModelName(models.Model):
    _name = 'model.name'
    _description = 'Model Description'

    # Fields
    name = fields.Char(string="Name", required=True)
    active = fields.Boolean(default=True)

    # Methods
    def action_confirm(self):
        """Confirm the record."""
        self.state = 'confirmed'
```

### Database Schema

```sql
-- Table: model_name
CREATE TABLE model_name (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    active BOOLEAN DEFAULT TRUE,
    create_date TIMESTAMP,
    write_date TIMESTAMP
);

-- Indexes
CREATE INDEX idx_model_name_active ON model_name(active);
```

### API Usage

```python
# Create record
record = self.env['model.name'].create({
    'name': 'Example',
})

# Search records
records = self.env['model.name'].search([
    ('active', '=', True),
])

# Update record
record.write({'name': 'Updated'})
```

### Common Customizations

#### Customization 1: [Name]
[Description and code example]

---

## Troubleshooting

### Issue 1: [Problem Description]

**Symptoms**:
- Symptom 1
- Symptom 2

**Cause**: [Root cause]

**Solution**:
1. Step 1
2. Step 2

**SQL Diagnostic**:
```sql
-- Query to diagnose issue
```

---

## Related Documentation

- [Related Feature Guide](../path/to/guide.md)
- [Technical Reference](../path/to/reference.md)
- [Known Issues](../12-issues-bugs/01-known-issues-by-severity.md)

---

**Last Updated**: YYYY-MM-DD
**Next Review**: [Date]
**Maintained By**: Team Name
```

### Template 2: Issue Documentation

```markdown
---
title: "Issue Title"
issue_id: "ISSUE-XXX"
severity: "High"
status: "Open"
reported_date: "YYYY-MM-DD"
module: "module_name"
category: "Performance"
---

# ISSUE-XXX: Issue Title

## Overview

**Status**: Open | In Progress | Resolved | Closed
**Severity**: Critical | High | Medium | Low
**Module**: `module_name`
**Reported**: YYYY-MM-DD
**Assigned To**: Developer Name

---

## Problem Description

[Clear description of the issue and its impact]

### Affected Users
- User group 1: [description]
- User group 2: [description]
- Frequency: [how often it occurs]

### User Impact
- **Financial**: [Low|Medium|High|Critical] - [description]
- **Operational**: [Low|Medium|High|Critical] - [description]
- **User Experience**: [Low|Medium|High|Critical] - [description]
- **Data Integrity**: [Low|Medium|High|Critical] - [description]

---

## Steps to Reproduce

1. Step 1: [detailed step]
2. Step 2: [detailed step]
3. Step 3: [detailed step]

**Expected Result**: [what should happen]
**Actual Result**: [what actually happens]

---

## Technical Analysis

### Root Cause

[Detailed explanation of the root cause]

### Affected Components

**File**: `module/models/file.py:123-145`
```python
# Problematic code
def method_name(self):
    # Issue at line 130
    pass
```

### Performance Metrics (if applicable)
- Current performance: [metrics]
- Expected performance: [metrics]
- Performance gap: [percentage or time]

---

## Error Messages

```
[Full error message or stack trace]
```

---

## Diagnostic Queries

### Query 1: Identify affected records

```sql
SELECT
    column1,
    column2,
    COUNT(*) as count
FROM table_name
WHERE condition
GROUP BY column1, column2;
```

---

## Proposed Solutions

### Solution 1: [Solution Name]

**Approach**: [Description]

**Implementation**:
```python
# Solution code
def fixed_method(self):
    # Implementation
    pass
```

**Pros**:
- Pro 1
- Pro 2

**Cons**:
- Con 1
- Con 2

**Effort**: [X days/hours]
**Risk**: [Low|Medium|High]

### Solution 2: [Alternative Solution]
[Similar structure]

---

## Temporary Workaround

**Status**: Active | Not Needed

**Steps**:
1. Workaround step 1
2. Workaround step 2

**Limitations**:
- Limitation 1
- Limitation 2

---

## Testing Plan

### Test Case 1: [Test Name]
**Objective**: [What to verify]

**Steps**:
1. Test step 1
2. Test step 2

**Expected Result**: [What should happen]

### Performance Benchmarks (if applicable)

| Metric | Before | Target | Actual |
|--------|--------|--------|--------|
| Response time | 30s | <5s | TBD |
| Database queries | 150 | <20 | TBD |

---

## Implementation Plan

### Phase 1: [Phase Name]
**Timeline**: Week 1
**Tasks**:
- [ ] Task 1
- [ ] Task 2
- [ ] Task 3

### Phase 2: [Phase Name]
**Timeline**: Week 2
**Tasks**:
- [ ] Task 1
- [ ] Task 2

---

## Related Issues

- [ISSUE-ABC](ISSUE-ABC.md) - Related issue
- [ISSUE-XYZ](ISSUE-XYZ.md) - Similar issue

---

## Related Documentation

- [Feature Guide](../04-feature-guides/domain/feature.md)
- [Technical Reference](../07-technical-guides/reference.md)
- [Known Issues by Severity](01-known-issues-by-severity.md)

---

## Update History

| Date | Status | Notes |
|------|--------|-------|
| YYYY-MM-DD | Open | Issue reported |
| YYYY-MM-DD | In Progress | Analysis complete |
| YYYY-MM-DD | Resolved | Fix deployed |

---

**Last Updated**: YYYY-MM-DD
**Next Review**: YYYY-MM-DD
```

### Template 3: Module Reference

```markdown
---
title: "Module Name Reference"
version: "1.0"
last_updated: "YYYY-MM-DD"
maintained_by: "Team Name"
module: "module_name"
odoo_version: "14.0"
---

# Module Name Reference

## Module Information

| Attribute | Value |
|-----------|-------|
| **Technical Name** | `module_name` |
| **Version** | 14.0.1.0.0 |
| **Category** | [Category] |
| **Author** | [Author Name] |
| **License** | LGPL-3 |
| **Depends** | `base`, `sale`, `stock` |

---

## Overview

[Description of module purpose and functionality]

### Key Features
- Feature 1
- Feature 2
- Feature 3

---

## Installation

### Requirements
- Odoo 14.0+
- Python 3.6+
- PostgreSQL 10+
- Additional Python libraries: [if any]

### Installation Steps
1. Copy module to addons directory
2. Update apps list
3. Install module
4. Configure settings

---

## Models

### Model 1: `model.name`

**Description**: [Model purpose]

**Inherits**: `models.Model`
**Table**: `model_name`

#### Fields

| Field Name | Type | Required | Description |
|------------|------|----------|-------------|
| `name` | Char | Yes | Record name |
| `active` | Boolean | No | Active flag |
| `state` | Selection | Yes | Record state |

#### Methods

##### `action_confirm()`
**Purpose**: Confirm the record
**Parameters**: None
**Returns**: `bool`

```python
def action_confirm(self):
    """Confirm the record and update state."""
    self.ensure_one()
    self.state = 'confirmed'
    return True
```

---

## Views

### Form View: `view_form_model_name`

**XML ID**: `module_name.view_form_model_name`
**Model**: `model.name`

```xml
<record id="view_form_model_name" model="ir.ui.view">
    <field name="name">model.name.form</field>
    <field name="model">model.name</field>
    <field name="arch" type="xml">
        <form>
            <sheet>
                <field name="name"/>
            </sheet>
        </form>
    </field>
</record>
```

---

## Security

### Access Rights

**File**: `security/ir.model.access.csv`

| ID | Name | Model | Group | Read | Write | Create | Delete |
|----|------|-------|-------|------|-------|--------|--------|
| `access_model_manager` | Model Manager | `model.name` | Manager | ✓ | ✓ | ✓ | ✓ |
| `access_model_user` | Model User | `model.name` | User | ✓ | ✓ | ✓ | ✗ |

### Record Rules

```xml
<record id="rule_model_name_personal" model="ir.rule">
    <field name="name">Personal Records</field>
    <field name="model_id" ref="model_model_name"/>
    <field name="groups" eval="[(4, ref('base.group_user'))]"/>
    <field name="domain_force">[('user_id', '=', user.id)]</field>
</record>
```

---

## Data Files

### Master Data

**File**: `data/master_data.xml`

[Description of included master data]

### Demo Data

**File**: `demo/demo_data.xml`

[Description of demo data]

---

## Reports

### Report 1: [Report Name]

**XML ID**: `module_name.report_name`
**Type**: PDF | Excel | HTML
**Model**: `model.name`

```xml
<report
    id="report_name"
    string="Report Title"
    model="model.name"
    report_type="qweb-pdf"
    file="module_name.report_name"
    name="module_name.report_name"
/>
```

---

## Wizards

### Wizard 1: [Wizard Name]

**Model**: `wizard.model.name`
**Type**: `TransientModel`
**Purpose**: [Description]

```python
class WizardName(models.TransientModel):
    _name = 'wizard.model.name'
    _description = 'Wizard Description'

    def action_process(self):
        """Process wizard action."""
        pass
```

---

## Configuration

### Settings

**Location**: Settings → [Section] → [Feature]

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| Setting 1 | Boolean | True | [Description] |
| Setting 2 | Integer | 30 | [Description] |

---

## Dependencies

### Required Modules
- `base` - Odoo base module
- `sale` - Sales management
- `stock` - Inventory management

### Optional Modules
- `account` - Accounting integration
- `purchase` - Purchase integration

---

## Upgrade Notes

### From v13.0 to v14.0
- Migration note 1
- Migration note 2
- Breaking changes

---

## Known Issues

- [ISSUE-001](../12-issues-bugs/ISSUE-001.md) - Issue description
- [PERF-001](../12-issues-bugs/PERF-001.md) - Performance issue

---

## API Examples

### Create Record
```python
record = env['model.name'].create({
    'name': 'Example',
})
```

### Search Records
```python
records = env['model.name'].search([
    ('active', '=', True),
])
```

---

## Related Documentation

- [Feature Guide](../04-feature-guides/domain/feature.md)
- [User Guide](../03-user-guides/feature.md)
- [Technical Guide](../07-technical-guides/models-and-orm.md)

---

**Last Updated**: YYYY-MM-DD
**Maintained By**: Team Name
```

---

## Integration Options

### Option A: Manual Documentation Only

**Pros**:
- Full control over content
- No external dependencies
- Works offline

**Cons**:
- Manual updates required
- Can become stale
- No automation

**When to Use**: Small projects, minimal issue tracking

---

### Option B: Basic Issue Tracker Integration

**Approach**: Reference external issues via URLs

```markdown
### ISSUE-001: Payment Processing Error

**GitHub**: [#451](https://github.com/org/repo/issues/451)
**Status**: See GitHub for current status
```

**Pros**:
- Simple to implement
- No automation needed
- Single source of truth (GitHub/Jira)

**Cons**:
- No local context
- Requires internet access
- Scattered information

**When to Use**: Medium projects, existing issue tracker

---

### Option C: API Integration (Recommended)

**Approach**: Automated sync with GitHub/Jira APIs

#### GitHub Issues Integration

**Setup**:
1. Create GitHub personal access token
2. Configure environment variable: `GITHUB_API_TOKEN`
3. Set up sync script

**Sync Script**: `/scripts/github-integration/sync_github_issues.py`

```python
#!/usr/bin/env python3
"""
GitHub Issues to Markdown Documentation Sync
"""

import os
import requests
import json
from datetime import datetime

GITHUB_CONFIG = {
    'api_base': 'https://api.github.com',
    'repo_owner': 'your-org',
    'repo_name': 'your-repo',
    'api_token': os.getenv('GITHUB_API_TOKEN'),
}

def fetch_issues(state='open', labels=None):
    """Fetch issues from GitHub API."""
    url = f"{GITHUB_CONFIG['api_base']}/repos/{GITHUB_CONFIG['repo_owner']}/{GITHUB_CONFIG['repo_name']}/issues"

    headers = {
        'Accept': 'application/vnd.github.v3+json',
        'Authorization': f"token {GITHUB_CONFIG['api_token']}"
    }

    params = {
        'state': state,
        'labels': ','.join(labels) if labels else None,
        'per_page': 100
    }

    response = requests.get(url, headers=headers, params=params)
    response.raise_for_status()
    return response.json()

def generate_markdown(issues):
    """Generate markdown from issues."""
    markdown = f"# Known Issues\n\n"
    markdown += f"**Last Sync**: {datetime.utcnow().strftime('%Y-%m-%d %H:%M UTC')}\n\n"

    for issue in issues:
        markdown += f"## [{issue['title']}]({issue['html_url']})\n\n"
        markdown += f"**Status**: {issue['state']}\n"
        markdown += f"**Created**: {issue['created_at'][:10]}\n\n"
        markdown += f"{issue['body']}\n\n"
        markdown += "---\n\n"

    return markdown

if __name__ == '__main__':
    issues = fetch_issues(labels=['bug', 'severity:high'])
    markdown = generate_markdown(issues)

    with open('docs/12-issues-bugs/01-known-issues-by-severity.md', 'w') as f:
        f.write(markdown)
```

#### Automation

**Cron Job** (Linux/Mac):
```bash
# Add to crontab: crontab -e
0 */2 * * * cd /path/to/project && python3 scripts/sync_github_issues.py
```

**GitHub Actions** (Automated):
```yaml
# .github/workflows/sync-docs.yml
name: Sync Documentation

on:
  schedule:
    - cron: '0 */2 * * *'  # Every 2 hours
  workflow_dispatch:  # Manual trigger

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2

      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: '3.9'

      - name: Install dependencies
        run: pip install requests

      - name: Sync issues
        env:
          GITHUB_API_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: python3 scripts/sync_github_issues.py

      - name: Commit changes
        run: |
          git config user.name "GitHub Actions"
          git config user.email "actions@github.com"
          git add docs/
          git commit -m "Auto-sync documentation from issues" || exit 0
          git push
```

#### Label Strategy

Use consistent labels for automation:

**Severity Labels**:
- `severity:critical`
- `severity:high`
- `severity:medium`
- `severity:low`

**Module Labels**:
- `module:egl_clinic_sales`
- `module:egl_clinic_package`
- `module:[module_name]`

**Category Labels**:
- `category:performance`
- `category:data`
- `category:ui`
- `category:integration`
- `category:configuration`

**Status Labels**:
- `status:confirmed`
- `status:in-progress`
- `status:pending-review`
- `status:workaround-available`

---

## Best Practices

### 1. Documentation Maintenance

#### Regular Reviews
- **Weekly**: Update issue statuses
- **Monthly**: Review and update feature guides
- **Quarterly**: Full documentation audit
- **Annually**: Major restructuring if needed

#### Version Control
- All documentation in Git
- Clear commit messages
- Branch for major updates
- Review process for changes

#### Change Log
Maintain a changelog at root level:

```markdown
# Documentation Changelog

## [1.2.0] - 2025-11-15
### Added
- New section on API integration
- Performance optimization guides

### Changed
- Updated installation instructions
- Revised security guidelines

### Fixed
- Broken links in Module References
- Code examples in Technical Guides
```

---

### 2. Writing Guidelines

#### Clarity
- Use active voice
- Short sentences
- Clear headings
- Consistent terminology

#### Completeness
- Always include examples
- Provide context
- Link related documentation
- Include troubleshooting

#### Accuracy
- Test all code examples
- Verify file paths
- Check line numbers
- Validate SQL queries

---

### 3. Screenshot Management

#### Organization
```
docs/
└── _assets/
    └── images/
        ├── 01-overview/
        ├── 02-getting-started/
        ├── 04-feature-guides/
        │   ├── setup-configuration/
        │   │   ├── 01-system-setup-overview.png
        │   │   └── 01-system-setup-config.png
        │   └── domain-name/
        └── 12-issues-bugs/
```

#### Naming Convention
```
[section-number]-[feature-name]-[description].png
```

Examples:
- `01-system-setup-main-screen.png`
- `15-take-now-select-package.png`
- `PERF-001-performance-chart.png`

#### Placeholders
During initial documentation, use placeholders:

```markdown
**Screenshot**: [Image: 01-system-setup-main-screen.png]
**Location**: Main Menu → Configuration → System Setup
**Shows**: System setup main configuration screen
```

---

### 4. Code Example Guidelines

#### Always Include File Paths
```markdown
**File**: `egl_clinic_sales/models/sale_order.py:456-478`
```

#### Provide Context
```python
# Context: This method is called when user clicks "Confirm" button
# It processes the fulfillment and creates invoice

def action_confirm_fulfillment(self):
    """Process fulfillment and create invoice."""
    self.ensure_one()

    # Step 1: Validate stock availability
    if not self._check_stock_availability():
        raise UserError(_("Insufficient stock available"))

    # Step 2: Create invoice (line 465)
    invoice = self._create_fulfillment_invoice()

    # Step 3: Process payment
    self._process_fulfillment_payment(invoice)

    return True
```

#### Include Usage Examples
```python
# Usage example:
order = self.env['sale.order'].browse(123)
order.action_confirm_fulfillment()
```

---

### 5. SQL Query Standards

#### Always Include Comments
```sql
-- Purpose: Find orders with pending fulfillment
-- Context: Used in daily operations report
-- Performance: Uses index on state and date_order

SELECT
    so.name AS order_reference,
    so.date_order,
    rp.name AS customer_name,
    so.amount_total,
    COUNT(sol.id) AS line_count
FROM sale_order so
JOIN res_partner rp ON rp.id = so.partner_id
JOIN sale_order_line sol ON sol.order_id = so.id
WHERE so.state = 'sale'
  AND so.fulfillment_state = 'pending'
  AND so.date_order >= CURRENT_DATE - INTERVAL '7 days'
GROUP BY so.id, so.name, so.date_order, rp.name, so.amount_total
ORDER BY so.date_order DESC;
```

#### Explain Results
```sql
-- Expected output:
-- order_reference | date_order | customer_name | amount_total | line_count
-- SO00123        | 2025-11-10 | John Doe      | 1500.00      | 5
-- SO00124        | 2025-11-11 | Jane Smith    | 2300.00      | 8
```

---

### 6. Cross-Referencing

#### Internal Links
Always use relative paths:

```markdown
See [Feature Guide 15: Take-Now Process](../04-feature-guides/sales-operations/15-take-now.md)

For technical details, refer to [Models and ORM](../07-technical-guides/models-and-orm.md#performance-optimization)
```

#### External Links
Provide context:

```markdown
**External Resources**:
- [Odoo 14 Documentation](https://www.odoo.com/documentation/14.0/)
- [PostgreSQL Performance](https://www.postgresql.org/docs/12/performance-tips.html)
- [Python Best Practices](https://peps.python.org/pep-0008/)
```

---

### 7. Issue Documentation Standards

#### Consistent IDs
- Use prefixes: `PERF-`, `BUG-`, `SEC-`, `UI-`
- Sequential numbers: `PERF-001`, `PERF-002`
- Never reuse IDs

#### Performance Issues
Always include:
- Current performance metrics
- Target performance metrics
- Benchmarks before/after
- Testing methodology

#### Root Cause Analysis
- Detailed technical explanation
- Code references with line numbers
- Database query analysis
- Performance profiling data

---

## Customization Guide

### Adapting for Your Project

#### 1. Identify Business Domains
List your project's main functional areas:

```
Examples:
- Healthcare: Patient Management, Appointments, Billing
- E-commerce: Products, Orders, Shipping
- Manufacturing: Production, Quality, Maintenance
- Retail: POS, Inventory, Loyalty
```

#### 2. Map Domains to Folders
```
04-feature-guides/
├── setup-configuration/       # Always include
├── [domain-1]/               # Your domain 1
├── [domain-2]/               # Your domain 2
├── [domain-3]/               # Your domain 3
├── reporting-analytics/      # Always include
├── performance-optimization/ # If applicable
└── maintenance-operations/   # Always include
```

#### 3. Customize Section Numbers
You can add/remove sections based on needs:

**Minimal Structure** (Small projects):
- 01-overview
- 02-getting-started
- 04-feature-guides
- 12-issues-bugs

**Standard Structure** (Medium projects):
- All 13 sections

**Extended Structure** (Large projects):
- All 13 sections
- 14-integrations
- 15-training-materials
- 16-faq

---

## Tools & Resources

### Markdown Editors
- **VS Code** with Markdown extensions
- **Obsidian** for wiki-style linking
- **Typora** for WYSIWYG editing

### Diagramming
- **Draw.io** for architecture diagrams
- **PlantUML** for UML diagrams
- **Mermaid** for embedded diagrams

### Screenshot Tools
- **Flameshot** (Linux)
- **Snagit** (Cross-platform)
- **macOS Screenshot** (Mac)

### Documentation Generators
- **MkDocs** - Static site from markdown
- **Sphinx** - Python documentation
- **GitBook** - Online documentation

---

## Migration Checklist

When creating documentation for a new project:

### Phase 1: Planning
- [ ] Identify all custom modules
- [ ] List business domains/workflows
- [ ] Define audience (Admin, User, Developer)
- [ ] Choose integration approach (A, B, or C)

### Phase 2: Structure Setup
- [ ] Create root docs/ folder
- [ ] Create all 13 section folders
- [ ] Create README.md files
- [ ] Set up scripts/ folder for automation

### Phase 3: Content Creation
- [ ] Write overview section
- [ ] Document getting started
- [ ] Create feature guides (domain by domain)
- [ ] Document technical references
- [ ] Set up issue tracking

### Phase 4: Integration
- [ ] Configure GitHub/Jira integration
- [ ] Set up sync scripts
- [ ] Test automation
- [ ] Establish review process

### Phase 5: Maintenance
- [ ] Define update schedule
- [ ] Assign maintenance responsibilities
- [ ] Set up change tracking
- [ ] Plan regular audits

---

## Example Projects

### Healthcare Clinic System

```
04-feature-guides/
├── setup-configuration/
├── patient-management/
├── appointment-scheduling/
├── medical-records/
├── billing-insurance/
├── pharmacy-inventory/
├── reporting-analytics/
├── performance-optimization/
└── maintenance-operations/
```

### E-commerce Platform

```
04-feature-guides/
├── setup-configuration/
├── product-catalog/
├── order-management/
├── customer-management/
├── payment-processing/
├── shipping-fulfillment/
├── marketing-promotions/
├── reporting-analytics/
└── maintenance-operations/
```

### Manufacturing ERP

```
04-feature-guides/
├── setup-configuration/
├── production-planning/
├── inventory-control/
├── quality-management/
├── maintenance-management/
├── procurement/
├── shop-floor-control/
├── reporting-analytics/
└── maintenance-operations/
```

---

## Conclusion

This template provides a comprehensive framework for documenting Odoo projects. Key takeaways:

1. **Structure Matters**: Consistent organization aids navigation
2. **Three Audiences**: Serve Admin, User, and Developer needs
3. **Feature-Centric**: Organize by business workflows
4. **Living Documentation**: Integrate with issue tracking
5. **Code Examples**: Always include file paths and line numbers
6. **Best Practices**: Regular reviews and updates

### Next Steps

1. Copy this template to your new project
2. Customize folder structure for your domains
3. Set up integration if needed
4. Start with Section 01 (Overview)
5. Build out Section 04 (Feature Guides)
6. Maintain and iterate

---

## Support & Resources

**Template Repository**: [Link to template repo if available]
**Questions**: [Team contact]
**Updates**: Check this document quarterly for updates

---

**Document Version**: 1.0
**Created**: 2025-11-13
**Last Updated**: 2025-11-13
**Maintained By**: E-Global SCM Development Team
**Based On**: Dr Ko Clinics Odoo v14.0 Documentation Project

---

**License**: This template is provided as-is for use in Odoo documentation projects.

---
title: "Views and UI"
section: "07-technical-guides"
document_id: "TG-002"
version: "17.0"
audience: "Frontend Developers"
last_updated: "2025-11-16"
tags: ["views", "ui", "owl", "javascript", "widgets", "scss"]
---

# Views and UI

Comprehensive guide to Cetmix Tower's user interface architecture, including views, OWL components, custom widgets, and styling.

## Table of Contents

1. [View Architecture](#view-architecture)
2. [OWL Components](#owl-components)
3. [Custom Widgets and Fields](#custom-widgets-and-fields)
4. [JavaScript Customizations](#javascript-customizations)
5. [SCSS Styling](#scss-styling)
6. [Asset Bundles](#asset-bundles)

---

## View Architecture

### View Types in Tower

Cetmix Tower uses standard Odoo view types with custom enhancements:

**View Types:**
- **Form Views**: Detailed record editing
- **Tree (List) Views**: Tabular data display
- **Kanban Views**: Card-based organization
- **Calendar Views**: Time-based scheduling
- **Search Views**: Filtering and grouping

### View File Locations

```
/home/user/cetmix-tower/cetmix_tower_server/views/
├── cx_tower_server_view.xml
├── cx_tower_command_view.xml
├── cx_tower_plan_view.xml
├── cx_tower_plan_line_view.xml
├── cx_tower_file_view.xml
├── cx_tower_variable_view.xml
├── cx_tower_command_log_view.xml
├── cx_tower_plan_log_view.xml
└── ...
```

---

### Form View Structure

**Example: Server Form View**

```xml
<!-- cx_tower_server_view.xml -->
<record id="cx_tower_server_view_form" model="ir.ui.view">
    <field name="name">cx.tower.server.view.form</field>
    <field name="model">cx.tower.server</field>
    <field name="arch" type="xml">
        <form string="Server">
            <!-- Header with status widget -->
            <header>
                <field name="status" widget="server_status"/>
                <button name="test_connection" type="object"
                        string="Test Connection"/>
            </header>

            <!-- Smart buttons -->
            <div class="oe_button_box" name="button_box">
                <button name="action_view_commands" type="object"
                        class="oe_stat_button" icon="fa-terminal">
                    <field name="command_count" widget="statinfo"/>
                </button>
                <button name="action_view_files" type="object"
                        class="oe_stat_button" icon="fa-file">
                    <field name="file_count" widget="statinfo"/>
                </button>
            </div>

            <!-- Body with notebook -->
            <sheet>
                <group>
                    <group>
                        <field name="name"/>
                        <field name="reference"/>
                        <field name="ip_address"/>
                        <field name="ssh_port"/>
                    </group>
                    <group>
                        <field name="partner_id"/>
                        <field name="os_id"/>
                        <field name="tag_ids" widget="many2many_tags"/>
                    </group>
                </group>

                <notebook>
                    <page string="Connection">
                        <!-- SSH connection fields -->
                    </page>
                    <page string="Variables">
                        <field name="variable_value_ids"/>
                    </page>
                    <page string="Commands">
                        <field name="command_ids"/>
                    </page>
                </notebook>
            </sheet>

            <!-- Chatter -->
            <div class="oe_chatter">
                <field name="message_follower_ids"/>
                <field name="message_ids"/>
            </div>
        </form>
    </field>
</record>
```

**Key Elements:**
- **Header**: Action buttons and status fields
- **Button Box**: Smart buttons for related records
- **Sheet**: Main content area
- **Notebook**: Tabbed organization
- **Chatter**: Messages and activities

---

### Tree View with Custom Attributes

```xml
<record id="cx_tower_command_view_tree" model="ir.ui.view">
    <field name="name">cx.tower.command.view.tree</field>
    <field name="model">cx.tower.command</field>
    <field name="arch" type="xml">
        <tree string="Commands"
              decoration-muted="not active"
              decoration-success="access_level=='1'"
              decoration-warning="access_level=='2'"
              decoration-danger="access_level=='3'">
            <field name="active" invisible="1"/>
            <field name="name"/>
            <field name="reference"/>
            <field name="action"/>
            <field name="access_level" invisible="1"/>
            <field name="tag_ids" widget="many2many_tags"/>
            <button name="run_command" type="object"
                    icon="fa-play" title="Run Command"/>
        </tree>
    </field>
</record>
```

**Decorations:**
- `decoration-muted`: Gray for inactive
- `decoration-success`: Green for User level
- `decoration-warning`: Yellow for Manager level
- `decoration-danger`: Red for Root level

---

### Kanban View

```xml
<record id="cx_tower_server_view_kanban" model="ir.ui.view">
    <field name="name">cx.tower.server.view.kanban</field>
    <field name="model">cx.tower.server</field>
    <field name="arch" type="xml">
        <kanban class="o_kanban_mobile">
            <field name="id"/>
            <field name="name"/>
            <field name="status"/>
            <field name="ip_address"/>
            <field name="color"/>

            <templates>
                <t t-name="kanban-box">
                    <div t-attf-class="oe_kanban_color_#{kanban_getcolor(record.color.raw_value)} oe_kanban_global_click">
                        <div class="o_kanban_record_top">
                            <div class="o_kanban_record_headings">
                                <strong class="o_kanban_record_title">
                                    <field name="name"/>
                                </strong>
                            </div>
                            <field name="status" widget="server_status"/>
                        </div>
                        <div class="o_kanban_record_body">
                            <field name="ip_address"/>
                        </div>
                        <div class="o_kanban_record_bottom">
                            <field name="tag_ids" widget="many2many_tags"/>
                        </div>
                    </div>
                </t>
            </templates>
        </kanban>
    </field>
</record>
```

---

## OWL Components

Tower uses Odoo Web Library (OWL) for modern reactive components.

### Component File Structure

```
/home/user/cetmix-tower/cetmix_tower_server/static/src/components/
├── ace_variables/
│   ├── ace_variables.esm.js        # Main component
│   ├── ace_variables.xml           # Template
│   ├── ace_variables.scss          # Styles
│   ├── code_editor_tower.esm.js    # Code editor
│   ├── autocomplete_popup.esm.js   # Autocomplete
│   └── ...
├── server_status/
│   ├── server_status_field.esm.js  # Status widget
│   └── server_status_field.scss    # Styles
└── ...
```

---

### Custom Component: Server Status Field

**JavaScript Component:**

```javascript
/** @odoo-module */
// File: static/src/components/server_status/server_status_field.esm.js

import {registry} from "@web/core/registry";
import {
    StateSelectionField,
    stateSelectionField,
} from "@web/views/fields/state_selection/state_selection_field";

import {STATUS_COLORS, STATUS_COLOR_PREFIX} from "../../utils/server_utils.esm";

export class ServerStatusField extends StateSelectionField {
    setup() {
        super.setup();
        this.colorPrefix = STATUS_COLOR_PREFIX;
        this.colors = STATUS_COLORS;
    }

    get options() {
        return [[false, "Undefined"], ...super.options];
    }
}

export const serverStatusField = {
    ...stateSelectionField,
    component: ServerStatusField,
};

// Register custom widget
registry.category("fields").add("server_status", serverStatusField);
```

**Utility Constants:**

```javascript
// File: static/src/utils/server_utils.esm.js
export const STATUS_COLOR_PREFIX = "o_status";

export const STATUS_COLORS = {
    online: "success",
    offline: "danger",
    unknown: "muted",
    warning: "warning",
};
```

**SCSS Styling:**

```scss
// File: static/src/components/server_status/server_status_field.scss
.o_status {
    &.o_status_success {
        color: $success;
        &::before {
            content: "●";
            margin-right: 4px;
        }
    }

    &.o_status_danger {
        color: $danger;
        &::before {
            content: "●";
            margin-right: 4px;
        }
    }

    &.o_status_muted {
        color: $text-muted;
        &::before {
            content: "●";
            margin-right: 4px;
        }
    }
}
```

**Usage in View:**

```xml
<field name="status" widget="server_status"/>
```

---

### ACE Variables Component

Advanced code editor with variable autocomplete.

**Component Structure:**

```javascript
/** @odoo-module */
// File: static/src/components/ace_variables/ace_variables.esm.js

import {Component} from "@odoo/owl";
import {registry} from "@web/core/registry";
import {standardFieldProps} from "@web/views/fields/standard_field_props";

export class AceVariablesField extends Component {
    setup() {
        this.aceEditor = null;
        this.variables = [];

        // Initialize ACE editor
        onMounted(() => {
            this.initializeEditor();
            this.setupAutocomplete();
        });
    }

    initializeEditor() {
        const editor = ace.edit(this.editorRef.el);
        editor.setTheme("ace/theme/monokai");
        editor.session.setMode("ace/mode/sh");
        editor.setOptions({
            enableBasicAutocompletion: true,
            enableLiveAutocompletion: true,
        });

        this.aceEditor = editor;
    }

    setupAutocomplete() {
        // Custom autocomplete for variables
        const variableCompleter = {
            getCompletions: (editor, session, pos, prefix, callback) => {
                const completions = this.variables.map(v => ({
                    caption: `{{ ${v.reference} }}`,
                    value: `{{ ${v.reference} }}`,
                    meta: v.name,
                    score: 1000,
                }));
                callback(null, completions);
            }
        };

        this.aceEditor.completers.push(variableCompleter);
    }

    updateValue(newValue) {
        this.props.update(newValue);
    }
}

AceVariablesField.template = "cetmix_tower_server.AceVariablesField";
AceVariablesField.props = {...standardFieldProps};

registry.category("fields").add("ace_variables", {
    component: AceVariablesField,
});
```

**XML Template:**

```xml
<!-- File: static/src/components/ace_variables/ace_variables.xml -->
<templates>
    <t t-name="cetmix_tower_server.AceVariablesField">
        <div class="ace_variables_field">
            <div class="ace_editor_container">
                <div t-ref="editorRef" class="ace_editor"/>
            </div>
            <div class="variable_hints">
                <span class="hint_label">Available Variables:</span>
                <t t-foreach="variables" t-as="variable" t-key="variable.id">
                    <span class="variable_hint"
                          t-esc="`{{ ${variable.reference} }}`"
                          t-on-click="() => this.insertVariable(variable)"/>
                </t>
            </div>
        </div>
    </t>
</templates>
```

**Usage:**

```xml
<field name="code" widget="ace_variables"/>
```

---

## Custom Widgets and Fields

### Many2many Tags with Colors

```xml
<field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}"/>
```

### Status Badge

```xml
<field name="status" widget="badge"
       decoration-success="status == 'success'"
       decoration-danger="status == 'failed'"
       decoration-info="status == 'running'"/>
```

### Progress Bar

```xml
<field name="completion_percentage" widget="progressbar"/>
```

### Handle (Drag and Drop)

```xml
<field name="sequence" widget="handle"/>
```

---

## JavaScript Customizations

### Form Controller Extension

```javascript
/** @odoo-module */

import {FormController} from "@web/views/form/form_controller";
import {patch} from "@web/core/utils/patch";

patch(FormController.prototype, "tower.FormController", {
    async onSave(record) {
        // Custom save logic
        const result = await super.onSave(record);

        // Additional actions after save
        if (result && this.props.resModel === "cx.tower.server") {
            this.notification.add("Server saved successfully!", {
                type: "success"
            });
        }

        return result;
    },
});
```

---

### Custom Actions

```javascript
/** @odoo-module */

import {registry} from "@web/core/registry";

async function runCommandAction(env, action) {
    const {context, res_id} = action;

    // Open wizard
    return env.services.action.doAction({
        type: "ir.actions.act_window",
        res_model: "cx.tower.command.run.wizard",
        views: [[false, "form"]],
        target: "new",
        context: {
            ...context,
            default_command_id: res_id,
        },
    });
}

registry.category("actions").add("run_command", runCommandAction);
```

**Usage in View:**

```xml
<button name="run_command" type="action" string="Run Command"/>
```

---

## SCSS Styling

### File Organization

```
/home/user/cetmix-tower/cetmix_tower_server/static/src/
├── components/
│   ├── ace_variables/
│   │   └── ace_variables.scss
│   └── server_status/
│       └── server_status_field.scss
```

### Component Styling Example

```scss
// File: static/src/components/ace_variables/ace_variables.scss

.ace_variables_field {
    display: flex;
    flex-direction: column;
    height: 400px;

    .ace_editor_container {
        flex: 1;
        border: 1px solid $border-color;
        border-radius: $border-radius;

        .ace_editor {
            height: 100%;
            font-family: $font-family-monospace;
            font-size: 14px;
        }
    }

    .variable_hints {
        margin-top: 8px;
        padding: 8px;
        background-color: $gray-100;
        border-radius: $border-radius;

        .hint_label {
            font-weight: bold;
            margin-right: 8px;
        }

        .variable_hint {
            display: inline-block;
            margin: 2px;
            padding: 2px 8px;
            background-color: $primary;
            color: white;
            border-radius: 3px;
            cursor: pointer;
            font-family: $font-family-monospace;

            &:hover {
                background-color: darken($primary, 10%);
            }
        }
    }
}

// Dark theme support
body[data-theme="dark"] {
    .ace_variables_field {
        .ace_editor_container {
            border-color: $border-color-dark;
        }

        .variable_hints {
            background-color: $gray-800;
        }
    }
}
```

### Theme Variables

```scss
// Using Odoo's Bootstrap variables
$primary: #017e84;
$success: #28a745;
$danger: #dc3545;
$warning: #ffc107;
$info: #17a2b8;

$gray-100: #f8f9fa;
$gray-800: #343a40;

$border-color: #dee2e6;
$border-radius: 0.25rem;

$font-family-monospace: 'Monaco', 'Menlo', 'Ubuntu Mono', monospace;
```

---

## Asset Bundles

### Asset Declaration in Manifest

```python
# File: /home/user/cetmix-tower/cetmix_tower_server/__manifest__.py

"assets": {
    "web.assets_backend": [
        # Component templates (XML)
        "cetmix_tower_server/static/src/components/**/*.xml",

        # JavaScript files
        "cetmix_tower_server/static/src/**/*.js",

        # Stylesheets
        "cetmix_tower_server/static/src/**/*.scss",
    ],
},
```

### Asset Loading Order

1. **Utility files** (loaded first)
2. **Component JavaScript**
3. **Component templates**
4. **Stylesheets** (loaded last)

### Lazy Loading

For heavy components:

```python
"assets": {
    "web.assets_backend": [...],
    "cetmix_tower_server.ace_editor": [
        "cetmix_tower_server/static/lib/ace/ace.js",
        "cetmix_tower_server/static/lib/ace/mode-sh.js",
        "cetmix_tower_server/static/lib/ace/theme-monokai.js",
    ],
},
```

---

## Best Practices

### Component Design

✅ **Do:**
- Keep components focused and reusable
- Use props for configuration
- Emit events for parent communication
- Leverage OWL lifecycle hooks

❌ **Don't:**
- Mix business logic with UI
- Directly manipulate DOM
- Create circular dependencies

### Performance

✅ **Optimize:**
- Use `t-memo` for expensive computations
- Lazy load heavy libraries
- Minimize DOM updates
- Use virtual scrolling for long lists

### Accessibility

✅ **Ensure:**
- Proper ARIA labels
- Keyboard navigation
- Screen reader support
- Color contrast ratios

---

## Summary

Key Takeaways:

✅ **View Architecture**: Standard Odoo views with custom enhancements
✅ **OWL Components**: Modern reactive UI components
✅ **Custom Widgets**: Server status, ACE editor with variables
✅ **Asset Management**: Organized bundles for performance
✅ **Styling**: Component-based SCSS with theme support

---

**Next Steps:**
- [Security and Access →](./03-security-and-access.md)
- [Models and ORM ←](./01-models-and-orm.md)

---

**Navigation:**
- [← Technical Guides Home](./README.md)
- [↑ Documentation Home](../README.md)

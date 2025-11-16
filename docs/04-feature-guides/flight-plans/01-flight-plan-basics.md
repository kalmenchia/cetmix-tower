---
title: Flight Plan Basics
audience: [administrators, end-users, developers]
topics: [plan-creation, plan-lines, execution-order, error-handling, plan-logs]
---

# Flight Plan Basics

This guide covers the fundamental concepts and operations for creating and managing flight plans in Cetmix Tower.

## Table of Contents

- [For Administrators](#for-administrators)
  - [What are Flight Plans](#what-are-flight-plans)
  - [Creating Flight Plans](#creating-flight-plans)
  - [Adding Plan Lines](#adding-plan-lines)
  - [Execution Order and Sequence](#execution-order-and-sequence)
  - [Error Handling Strategies](#error-handling-strategies)
  - [Plan Compatibility with Servers](#plan-compatibility-with-servers)
- [For End Users](#for-end-users)
  - [Running Flight Plans](#running-flight-plans)
  - [Monitoring Plan Execution](#monitoring-plan-execution)
  - [Understanding Plan Logs](#understanding-plan-logs)
- [For Developers](#for-developers)
  - [Flight Plan Model](#flight-plan-model)
  - [Execution Orchestration](#execution-orchestration)
  - [Plan Log Model](#plan-log-model)
  - [Code Examples](#code-examples)

---

## For Administrators

### What are Flight Plans

Flight Plans are ordered sequences of commands that execute on servers. They provide:

**Orchestration**: Run multiple commands in a defined order
**Conditional Logic**: Execute lines based on conditions
**Error Handling**: Control what happens when commands fail
**Variable Sharing**: Pass data between commands
**Reusability**: Execute the same workflow on multiple servers

**Use Cases**:
- Application deployment
- Server provisioning and setup
- Database backup and restore
- System updates and upgrades
- Disaster recovery procedures
- Complex maintenance tasks

### Creating Flight Plans

#### Method 1: Via UI

1. Navigate to **Cetmix Tower > Flight Plans**
2. Click **Create**
3. Fill in basic information:
   - **Name**: Descriptive plan name
   - **On Error**: Default error handling strategy
   - **Custom Exit Code**: (optional) Exit code for custom error handling
4. Configure optional settings:
   - **Servers**: Limit to specific servers (empty = all compatible servers)
   - **Tags**: Organize plans
   - **Note**: Documentation
5. Click **Save**
6. Add plan lines (see next section)

#### Method 2: Programmatic Creation

See [For Developers](#for-developers) section for code examples.

### Adding Plan Lines

Plan lines define the commands to execute and their configuration.

#### Adding a Line

1. Open the flight plan
2. Go to **Lines** tab
3. Click **Add a line**
4. Configure the line:
   - **Sequence**: Execution order (lower numbers run first)
   - **Action**: Command type (auto-filled based on command)
   - **Command**: Select command to execute
   - **Path**: (optional) Override command's default path
   - **Use Sudo**: Enable sudo for this line
   - **Condition**: (optional) Execution condition
5. Click **Save & Close** or **Save & New**

#### Line Fields

**Required**:
- `command_id`: Command to execute

**Optional**:
- `sequence`: Execution order (default: 10)
- `path`: Override default command path
- `use_sudo`: Run with sudo (uses server's sudo configuration)
- `condition`: Conditional execution (Jinja expression)
- `action_ids`: Post-execution actions

**Auto-Computed**:
- `name`: Taken from command name
- `action`: Taken from command action type
- `note`: Taken from command note

**Defined**: `cetmix_tower_server/models/cx_tower_plan_line.py:11-106`

### Execution Order and Sequence

Lines execute in order determined by the `sequence` field.

**Sequencing Rules**:
- Lines ordered by sequence (ascending)
- Default sequence is 10
- Common pattern: increment by 10 (10, 20, 30...) for easy insertion
- Equal sequences: undefined order (avoid this)

**Example**:
```
Sequence 10: Stop service
Sequence 20: Update code
Sequence 30: Run migrations
Sequence 40: Start service
```

**Adding Between Lines**:
To insert a line between 20 and 30, use sequence 25:
```
Sequence 10: Stop service
Sequence 20: Update code
Sequence 25: Clear cache  ← NEW
Sequence 30: Run migrations
Sequence 40: Start service
```

### Error Handling Strategies

#### Plan-Level Error Handling

Set the default strategy in the flight plan's **On Error** field:

| Strategy | Value | Behavior |
|----------|-------|----------|
| Exit with command exit code | `'e'` | Stop execution, return command's exit code (default) |
| Exit with custom exit code | `'ec'` | Stop execution, return plan's custom_exit_code |
| Run next command | `'n'` | Continue to next line despite error |

**Defined**: `cetmix_tower_server/models/cx_tower_plan.py:64-75`

```python
on_error_action = fields.Selection([
    ('e', 'Exit with command exit code'),
    ('ec', 'Exit with custom exit code'),
    ('n', 'Run next command'),
], required=True, default='e')
```

#### Line-Level Error Handling

Override plan-level strategy with **Plan Line Actions**.

**Creating Actions**:
1. Open plan line
2. Go to **Actions** tab
3. Click **Add a line**
4. Configure:
   - **Condition**: When to trigger (e.g., `==`, `!=`, `>`, `<`)
   - **Value**: Exit code to match
   - **Action**: What to do
   - **Custom Exit Code**: (optional) Exit code to use
   - **Variable Values**: (optional) Variables to set
5. Save

**Action Configuration**:
- `condition`: Comparison operator (==, !=, >, <, >=, <=)
- `value_char`: Exit code value to compare
- `action`: Action to take ('e', 'ec', 'n')
- `custom_exit_code`: Custom exit code (if action='ec')
- `variable_value_ids`: Variables to update on this action

**Defined**: `cetmix_tower_server/models/cx_tower_plan_line_action.py`

**Example Actions**:
```
Action 1: If exit code == 0, action = 'n' (continue)
Action 2: If exit code == 1, action = 'ec', custom_exit_code = 100
Action 3: If exit code > 1, action = 'e' (exit with command code)
```

#### Execution Flow

**Action Evaluation** (`_get_next_action_values`):
1. Check line condition (if set)
2. Iterate through line actions
3. Evaluate each action's condition against exit code
4. Execute first matching action
5. If no match, use plan's default error handling

**Location**: `cetmix_tower_server/models/cx_tower_plan.py:246-322`

**Next Action Execution** (`_run_next_action`):
1. Get action, exit code, and next line
2. Update variable values if specified
3. If action='n' and next line exists, execute next line
4. If action='e' or 'ec', finish plan with exit code

**Location**: `cetmix_tower_server/models/cx_tower_plan.py:349-378`

### Conditional Execution

Lines can have conditions that determine if they execute.

#### Setting Conditions

**Field**: `condition` (Char)

**Syntax**: Jinja-style Python expression
```python
{{ variable_name }} == 'value'
{{ odoo_version }} > '15.0'
{{ needs_restart }} == True
```

**Evaluation**:
1. Variables extracted from condition
2. Variable values retrieved for server
3. Condition rendered with values (pythonic mode)
4. Evaluated using `safe_eval()`

**Method**: `line._is_executable_line(server, variable_values)`
**Location**: `cetmix_tower_server/models/cx_tower_plan_line.py:207-246`

#### Condition Examples

**Check Odoo Version**:
```python
{{ odoo_version }} == '16.0'
```

**Check Environment**:
```python
{{ env_type }} == 'production'
```

**Check Boolean Flag**:
```python
{{ needs_migration }} == True
```

**Complex Condition**:
```python
{{ odoo_version }} >= '16.0' and {{ env_type }} == 'production'
```

**Using Custom Values** (from previous Python commands):
```python
{{ deployment_validated }} == True
```

#### Skipped Lines

When condition evaluates to False:
- Line is marked as skipped
- Command log created with status PLAN_LINE_CONDITION_CHECK_FAILED (-205)
- Log field `is_skipped=True`
- Execution continues to next line

**Method**: `line._skip()`
**Location**: `cetmix_tower_server/models/cx_tower_plan_line.py:248-273`

### Plan Compatibility with Servers

Restrict which servers can run a flight plan.

#### Server Compatibility

**Field**: `server_ids` (Many2many)

- **Empty**: Plan can run on any server
- **With servers**: Plan limited to those servers only

#### Nested Plan Compatibility

Flight plans check nested plan compatibility:
- If plan includes a "Run flight plan" command
- System verifies nested plan is compatible with target server
- Recursive check through all nested plans

**Check Method**: `_is_plan_incompatible_with_server()`
**Location**: `cetmix_tower_server/models/cx_tower_plan.py:137-172`

```python
def _is_plan_incompatible_with_server(self, server):
    """Check if plan is compatible with server"""
    # Check plan's server_ids
    if self.server_ids and server.id not in self.server_ids.ids:
        return "Flight plan is not compatible with the server"

    # Check each command compatibility
    for command in self.command_ids:
        if not command._check_server_compatibility(server):
            return f"Command {command.name} is not compatible"

        # Check nested plans
        if command.action == 'plan':
            nested_check = command.flight_plan_id._is_plan_incompatible_with_server(server)
            if nested_check:
                return nested_check

    return False
```

### Access Control

#### Access Levels

Flight plans inherit access control:
- Public: Anyone can execute
- User: Specific users
- Manager: Only managers

#### Command Access Validation

**Warning Field**: `access_level_warn_msg`

Shows warning if commands in the plan have higher access level than the plan itself.

**Example**:
- Plan access level: User (20)
- Command access level: Manager (30)
- Warning: "Command 'Deploy' has higher access level than the plan"

**Compute**: `_compute_command_access_level()`
**Location**: `cetmix_tower_server/models/cx_tower_plan.py:93-114`

### Parallel Execution Control

**Field**: `allow_parallel_run` (Boolean)

- `False`: Only one instance of this plan can run per server
- `True`: Multiple instances allowed

If blocked, status code -301 (ANOTHER_PLAN_RUNNING) is returned.

**Check**: During `_run_single()` execution
**Location**: `cetmix_tower_server/models/cx_tower_plan.py:227-240`

### Tags and Organization

**Field**: `tag_ids` (Many2many to cx.tower.tag)

Use tags to:
- Group related plans
- Categorize by purpose (deployment, backup, maintenance)
- Filter in UI
- Organize by project or customer

---

## For End Users

### Running Flight Plans

#### From Server Record

1. Open the server record
2. Click **Run Flight Plan** button
3. Select flight plan from list
4. (Optional) Provide custom variable values
5. Click **Run**
6. Plan begins executing immediately

#### From Flight Plan Record

1. Open the flight plan record
2. Click **Run on Servers** button (if available)
3. Select target servers
4. (Optional) Provide custom variable values
5. Click **Run**

### Monitoring Plan Execution

#### Real-Time Monitoring

**Via Server**:
1. Open server record
2. Click **Flight Plan Logs** button or tab
3. Find the running plan (filter by "Is Running")
4. Open the plan log record
5. View current execution status

**Fields to Watch**:
- **Is Running**: True while executing
- **Plan Line Executed**: Currently running line
- **Start Date**: When execution began
- **Duration (sec)**: Current runtime
- **Command Logs**: Completed command logs

#### Stopping a Running Plan

1. Open the plan log record
2. Click **Stop** button
3. Confirm action
4. Plan status becomes "Stopped" (-308)
5. Currently running command is also stopped

**Method**: `action_stop()`
**Location**: `cetmix_tower_server/models/cx_tower_plan_log.py:223-247`

### Understanding Plan Logs

#### Plan Log Fields

**Identification**:
- **Name**: Server + Plan name
- **Label**: Custom tracking label
- **Server**: Where it ran
- **Plan**: Which plan executed

**Timing**:
- **Started**: Start timestamp
- **Finished**: End timestamp
- **Duration**: Total execution time (seconds)
- **Duration (sec)**: Current duration (if running)

**Status**:
- **Status**: Plan exit code
  - `0`: Success
  - `-301`: Another plan running
  - `-302`: Plan is empty
  - `-306`: Plan not compatible
  - `-308`: Stopped by user
- **Is Running**: True during execution
- **Is Stopped**: True if manually stopped

**Execution**:
- **Plan Line Executed**: Currently running line
- **Command Logs**: All command logs from this plan
- **Custom Message**: Additional information
- **Variable Values**: Custom values passed to plan

**Defined**: `cetmix_tower_server/models/cx_tower_plan_log.py:8-91`

#### Reading Command Logs

Each line execution creates a command log:

1. Open plan log
2. Go to **Command Logs** tab
3. View logs in sequence order
4. Check each command's:
   - Status (exit code)
   - Response (output)
   - Error (if any)
   - Duration
   - Variables used

#### Success Criteria

A flight plan is successful when:
- All lines execute without critical errors (or errors are handled)
- Final status code is 0
- **Finished** date is set
- **Is Running** is False

#### Failure Analysis

When a plan fails:
1. Check **Status** code
2. Find the last executed line (**Plan Line Executed**)
3. Review that line's command log
4. Check error message and response
5. Verify variable values were correct
6. Review error handling configuration

---

## For Developers

### Flight Plan Model

**Model Name**: `cx.tower.plan`

**Location**: `cetmix_tower_server/models/cx_tower_plan.py`

**Inheritance**:
```python
_inherit = [
    "cx.tower.reference.mixin",     # Reference tracking
    "cx.tower.access.mixin",        # Access control (users)
    "cx.tower.access.role.mixin",   # Access control (roles)
]
```

**Description**: "Cetmix Tower Flight Plan"

**Order**: `name asc`

#### Key Fields

```python
# Basic
active = fields.Boolean(default=True)
color = fields.Integer()
server_ids = fields.Many2many('cx.tower.server')
tag_ids = fields.Many2many('cx.tower.tag')
note = fields.Text()

# Lines
line_ids = fields.One2many('cx.tower.plan.line', 'plan_id', auto_join=True, copy=True)
command_ids = fields.Many2many('cx.tower.command', compute='_compute_command_ids', store=True)

# Error Handling
on_error_action = fields.Selection([...], required=True, default='e')
custom_exit_code = fields.Integer()

# Execution Control
allow_parallel_run = fields.Boolean()

# Access
user_ids = fields.Many2many(relation='cx_tower_plan_user_rel')
manager_ids = fields.Many2many(relation='cx_tower_plan_manager_rel')

# Validation
access_level_warn_msg = fields.Text(compute='_compute_command_access_level')
```

**Defined**: Lines 17-91

### Execution Orchestration

#### Main Entry Point

**Method**: `server.run_flight_plan(flight_plan, **kwargs)`

**Location**: `cetmix_tower_server/models/cx_tower_server.py:1033-1053`

**Calls**: `flight_plan._run_single(server, **kwargs)`

#### Single Execution

**Method**: `_run_single(server, **kwargs)`

**Location**: `cetmix_tower_server/models/cx_tower_plan.py:178-244`

**Parameters**:
- `server`: cx.tower.server record
- `kwargs`: Optional arguments
  - `plan_log`: Values for plan log
  - `log`: Values for command logs
  - `key`: Values for key parser
  - `variable_values`: Custom variable values

**Returns**: cx.tower.plan.log record

**Process**:
```python
def _run_single(self, server, **kwargs):
    """Run single Flight Plan on a single server"""
    self.ensure_one()
    server.ensure_one()

    # Check access
    self.check_access_rights("read")
    self.check_access_rule("read")

    plan_log_obj = self.env["cx.tower.plan.log"].sudo()

    # Check compatibility (unless from_command context)
    if not self.env.context.get("from_command"):
        incompatibility = self._is_plan_incompatible_with_server(server)
        if incompatibility:
            # Create error log and return
            return plan_log_obj.record(
                server, self, PLAN_NOT_COMPATIBLE_WITH_SERVER,
                plan_log={'custom_message': incompatibility}, **kwargs
            )

    # Check parallel run
    if not self.allow_parallel_run or self.env.context.get('prevent_plan_recursion'):
        if plan_log_obj.search_count([...running plans...]) > 0:
            return plan_log_obj.record(
                server, self, ANOTHER_PLAN_RUNNING, **kwargs
            )

    # Start execution
    return plan_log_obj.start(server, self, fields.Datetime.now(), **kwargs)
```

#### Plan Log Start

**Method**: `plan_log.start(server, plan, start_date, **kwargs)`

**Location**: `cetmix_tower_server/models/cx_tower_plan_log.py:132-201`

**Process**:
```python
def start(self, server, plan, start_date=None, **kwargs):
    """Start plan execution"""
    # Create plan log record
    vals = {
        'server_id': server.id,
        'plan_id': plan.id,
        'is_running': True,
        'start_date': start_date or fields.Datetime.now(),
    }

    # Apply kwargs
    plan_log_kwargs = kwargs.get('plan_log')
    if plan_log_kwargs:
        vals.update(plan_log_kwargs)

    # Apply variable values
    variable_values = kwargs.get('variable_values')
    if variable_values:
        vals['variable_values'] = variable_values

    plan_log = self.sudo().create(vals)

    # Find and run first executable line
    for line, is_executable in get_executable_line(plan, server, variable_values):
        if is_executable:
            line._run(server, plan_log, **kwargs)
            break
        else:
            if not self._context.get('no_command_log'):
                line._skip(server, plan_log)
            break
    else:
        # No executable lines found
        plan_log.finish(plan_status=PLAN_IS_EMPTY)

    return plan_log
```

#### Line Execution

**Method**: `line._run(server, plan_log_record, **kwargs)`

**Location**: `cetmix_tower_server/models/cx_tower_plan_line.py:168-205`

**Process**:
```python
def _run(self, server, plan_log_record, **kwargs):
    """Run command from Flight Plan line"""
    self.ensure_one()

    # Set current line in plan log
    plan_log_record.plan_line_executed_id = self

    # Handle nested plan tracking
    flight_plan_command_log = kwargs.get('flight_plan_command_log')
    if flight_plan_command_log:
        flight_plan_command_log.triggered_plan_log_id = plan_log_record.id

    # Pass plan_log to command
    log_vals = kwargs.get('log', {})
    log_vals.update({'plan_log_id': plan_log_record.id})
    kwargs.update({'log': log_vals})

    # Set sudo value
    use_sudo = self.use_sudo and server.use_sudo

    # Execute command
    command_as_root = self.sudo().command_id
    path = self.path or command_as_root.path
    server.run_command(command_as_root, path, sudo=use_sudo, **kwargs)
```

#### Next Action Determination

**Method**: `_get_next_action_values(command_log)`

**Location**: `cetmix_tower_server/models/cx_tower_plan.py:246-322`

**Returns**: `(action, exit_code, next_line)`

**Process**:
```python
def _get_next_action_values(self, command_log):
    """Get next action based on command result"""
    # Get current line and exit code
    current_line = command_log.plan_log_id.plan_line_executed_id
    exit_code = command_log.command_status

    # Check line condition
    variable_values = command_log.variable_values or command_log.plan_log_id.variable_values
    if not current_line._is_executable_line(server, variable_values):
        return self._get_next_action_state('n', PLAN_LINE_CONDITION_CHECK_FAILED, current_line)

    # Check plan action lines
    for action_line in current_line.action_ids:
        conditional_expression = f"{exit_code} {action_line.condition} {action_line.value_char}"
        if expr_eval(conditional_expression):
            action = action_line.action

            # Use custom exit code if needed
            if action == 'ec' and action_line.custom_exit_code:
                exit_code = action_line.custom_exit_code

            # Update variables if specified
            for variable_value in action_line.variable_value_ids:
                # Update or create server variable value
                ...

            return self._get_next_action_state(action, exit_code, current_line)

    # No action matched, use defaults
    return self._get_next_action_state(None, exit_code, current_line)
```

#### Plan Completion

**Method**: `plan_log.finish(plan_status, **kwargs)`

**Location**: `cetmix_tower_server/models/cx_tower_plan_log.py:249-280`

**Process**:
```python
def finish(self, plan_status, **kwargs):
    """Finish plan execution"""
    values = {
        'is_running': False,
        'plan_status': plan_status,
        'finish_date': fields.Datetime.now(),
    }

    if kwargs:
        values.update(kwargs)

    self.sudo().write(values)

    # Call hook
    self._plan_finished()

    # Handle server deletion if needed
    if self.server_id._is_being_deleted() and self.server_id.plan_delete_id == self.plan_id:
        if plan_status == 0:
            self.with_context(server_force_delete=True).server_id.unlink()
        else:
            self.server_id.status = 'delete_error'
```

### Plan Log Model

**Model Name**: `cx.tower.plan.log`

**Location**: `cetmix_tower_server/models/cx_tower_plan_log.py`

**Description**: "Cetmix Tower Flight Plan Log"

**Order**: `start_date desc, id desc`

#### Key Methods

**`start(server, plan, start_date, **kwargs)`**: Begin execution
**`stop()`**: Force stop execution
**`finish(plan_status, **kwargs)`**: Complete execution
**`record(server, plan, status, ...)`**: Record without running
**`_plan_finished()`**: Hook for post-completion actions
**`_plan_command_finished(command_log)`**: Triggered when command finishes

### Code Examples

#### Example 1: Create Flight Plan Programmatically

```python
# File: custom_module/models/plan_creator.py

def create_deployment_plan(self, name):
    """Create standard deployment flight plan"""
    plan_obj = self.env['cx.tower.plan']
    line_obj = self.env['cx.tower.plan.line']

    # Create plan
    plan = plan_obj.create({
        'name': name,
        'on_error_action': 'e',  # Exit on error
        'tag_ids': [(6, 0, [self.env.ref('my_module.tag_deployment').id])],
    })

    # Add lines
    line_obj.create({
        'plan_id': plan.id,
        'sequence': 10,
        'command_id': self.env.ref('my_module.cmd_stop_service').id,
    })

    line_obj.create({
        'plan_id': plan.id,
        'sequence': 20,
        'command_id': self.env.ref('my_module.cmd_git_pull').id,
        'path': '/opt/myapp',
    })

    line_obj.create({
        'plan_id': plan.id,
        'sequence': 30,
        'command_id': self.env.ref('my_module.cmd_migrate').id,
        'condition': "{{ needs_migration }} == True",
    })

    line_obj.create({
        'plan_id': plan.id,
        'sequence': 40,
        'command_id': self.env.ref('my_module.cmd_start_service').id,
    })

    return plan
```

#### Example 2: Execute Plan with Custom Variables

```python
# File: custom_module/models/plan_executor.py

def deploy_with_version(self, server, version):
    """Deploy specific version to server"""
    plan = self.env.ref('my_module.plan_deployment')

    # Execute with custom variable values
    plan_log = server.run_flight_plan(
        plan,
        variable_values={
            'app_version': version,
            'needs_migration': True,
            'env_type': 'production'
        },
        plan_log={
            'label': f'deploy_v{version}'
        }
    )

    return plan_log
```

#### Example 3: Monitor Plan Execution

```python
# File: custom_module/models/plan_monitor.py

def check_plan_status(self, plan_log):
    """Check if plan is still running or completed"""
    if plan_log.is_running:
        current_line = plan_log.plan_line_executed_id
        duration = plan_log.duration_current

        return {
            'status': 'running',
            'current_line': current_line.name if current_line else 'Starting',
            'duration': duration,
        }
    else:
        return {
            'status': 'completed',
            'exit_code': plan_log.plan_status,
            'duration': plan_log.duration,
            'success': plan_log.plan_status == 0,
        }
```

#### Example 4: Plan with Actions

```python
# File: custom_module/models/plan_with_actions.py

def create_smart_deployment(self):
    """Create deployment plan with conditional actions"""
    plan = self.env['cx.tower.plan'].create({
        'name': 'Smart Deployment',
        'on_error_action': 'e',
    })

    # Create update line
    update_line = self.env['cx.tower.plan.line'].create({
        'plan_id': plan.id,
        'sequence': 10,
        'command_id': self.env.ref('my_module.cmd_update_code').id,
    })

    # Add action: on success (exit code 0), continue
    self.env['cx.tower.plan.line.action'].create({
        'line_id': update_line.id,
        'condition': '==',
        'value_char': '0',
        'action': 'n',  # Continue
    })

    # Add action: on 'no changes' (exit code 2), skip migrations
    migration_skip_action = self.env['cx.tower.plan.line.action'].create({
        'line_id': update_line.id,
        'condition': '==',
        'value_char': '2',
        'action': 'n',  # Continue
    })

    # Set variable to skip migrations
    migration_var = self.env.ref('my_module.var_needs_migration')
    self.env['cx.tower.variable.value'].create({
        'plan_line_action_id': migration_skip_action.id,
        'variable_id': migration_var.id,
        'value_char': 'False',
    })

    # Add migration line with condition
    self.env['cx.tower.plan.line'].create({
        'plan_id': plan.id,
        'sequence': 20,
        'command_id': self.env.ref('my_module.cmd_migrate').id,
        'condition': "{{ needs_migration }} == True",
    })

    return plan
```

#### Example 5: Nested Flight Plans

```python
# File: custom_module/models/nested_plans.py

def create_master_plan(self):
    """Create master plan that calls other plans"""

    # Create command that runs a plan
    backup_plan_cmd = self.env['cx.tower.command'].create({
        'name': 'Run Backup Plan',
        'action': 'plan',
        'flight_plan_id': self.env.ref('my_module.plan_backup').id,
    })

    deploy_plan_cmd = self.env['cx.tower.command'].create({
        'name': 'Run Deployment Plan',
        'action': 'plan',
        'flight_plan_id': self.env.ref('my_module.plan_deployment').id,
    })

    verify_plan_cmd = self.env['cx.tower.command'].create({
        'name': 'Run Verification Plan',
        'action': 'plan',
        'flight_plan_id': self.env.ref('my_module.plan_verification').id,
    })

    # Create master plan
    master_plan = self.env['cx.tower.plan'].create({
        'name': 'Master Deployment Workflow',
        'on_error_action': 'e',
    })

    # Add nested plans as lines
    for seq, cmd in enumerate([backup_plan_cmd, deploy_plan_cmd, verify_plan_cmd], start=1):
        self.env['cx.tower.plan.line'].create({
            'plan_id': master_plan.id,
            'sequence': seq * 10,
            'command_id': cmd.id,
        })

    return master_plan
```

## Related Documentation

- [Command Execution](../command-execution/README.md)
- [Server Management](../server-management/README.md)
- [Variable Management](../variable-management/README.md)
- [Logging & Monitoring](../logging-monitoring/README.md)

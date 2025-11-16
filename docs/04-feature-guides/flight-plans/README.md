# Flight Plans

Flight Plans are Cetmix Tower's orchestration feature that enables you to execute multiple commands in sequence with conditional logic, error handling, and variable management.

## Overview

Flight Plans provide:

- **Multi-Step Orchestration**: Execute multiple commands in a defined sequence
- **Conditional Execution**: Run lines based on conditions
- **Error Handling**: Configure actions on success or failure
- **Variable Sharing**: Pass data between commands via custom values
- **Nested Plans**: Execute flight plans within flight plans
- **Comprehensive Logging**: Track execution of every step

## Core Concepts

### Flight Plan (cx.tower.plan)

A flight plan is an ordered sequence of commands that execute on a server. Each plan contains:
- Multiple plan lines (commands)
- Error handling strategy
- Server compatibility rules
- Access control settings

**Location**: `cetmix_tower_server/models/cx_tower_plan.py`

### Flight Plan Line (cx.tower.plan.line)

A line represents one command in the flight plan. Lines have:
- Reference to a command
- Execution sequence (order)
- Optional execution path override
- Optional sudo override
- Execution condition
- Post-execution actions

**Location**: `cetmix_tower_server/models/cx_tower_plan_line.py`

### Plan Line Action (cx.tower.plan.line.action)

Actions define what happens based on command result:
- Condition to match (e.g., exit code == 0)
- Action to take (run next, exit, custom exit code)
- Optional custom exit code
- Variable values to set

**Location**: `cetmix_tower_server/models/cx_tower_plan_line_action.py`

### Flight Plan Log (cx.tower.plan.log)

Log record for each flight plan execution:
- Start and finish timestamps
- Execution status
- Link to all command logs
- Currently executing line
- Custom variable values
- Duration tracking

**Location**: `cetmix_tower_server/models/cx_tower_plan_log.py`

## Feature Guides

### [01. Flight Plan Basics](./01-flight-plan-basics.md)

Comprehensive guide covering:
- Creating flight plans
- Adding and configuring plan lines
- Execution order and sequence
- Error handling strategies
- Plan compatibility with servers
- Running and monitoring plans
- Plan log analysis

**Audiences**: Administrators, End Users, Developers

## Key Models and Files

### Models

- **cx.tower.plan**: Flight plan model
  File: `cetmix_tower_server/models/cx_tower_plan.py`

- **cx.tower.plan.line**: Flight plan line model
  File: `cetmix_tower_server/models/cx_tower_plan_line.py`

- **cx.tower.plan.line.action**: Plan line action model
  File: `cetmix_tower_server/models/cx_tower_plan_line_action.py`

- **cx.tower.plan.log**: Flight plan log model
  File: `cetmix_tower_server/models/cx_tower_plan_log.py`

### Execution Methods

- `_run_single()`: Execute plan on one server
  Location: `cetmix_tower_server/models/cx_tower_plan.py:178-244`

- `_get_next_action_values()`: Determine next action based on result
  Location: `cetmix_tower_server/models/cx_tower_plan.py:246-322`

- `_run_next_action()`: Execute next action
  Location: `cetmix_tower_server/models/cx_tower_plan.py:349-378`

## Common Use Cases

### For Administrators

1. **Deployment Automation**: Create multi-step deployment processes
2. **Server Provisioning**: Orchestrate server setup and configuration
3. **Backup Procedures**: Define backup and verification workflows
4. **Maintenance Tasks**: Schedule routine maintenance operations
5. **Disaster Recovery**: Create recovery procedures

### For End Users

1. **Execute Workflows**: Run pre-configured multi-step processes
2. **Monitor Progress**: Track plan execution in real-time
3. **Review Results**: Check outcomes of each step
4. **Understand Failures**: Identify which step failed and why

### For Developers

1. **Programmatic Orchestration**: Trigger flight plans from code
2. **Custom Actions**: Extend action logic
3. **Variable Workflows**: Pass data between commands
4. **Integration**: Connect with external systems

## Technical Overview

### Execution Flow

```
1. User/System triggers plan execution
   ↓
2. server.run_flight_plan(flight_plan, **kwargs)
   ↓
3. plan._run_single(server, **kwargs)
   ↓
4. Check plan compatibility with server
   ↓
5. Check parallel run restriction
   ↓
6. Create plan log record
   ↓
7. plan_log.start() - Begin execution
   ↓
8. For each line:
   ├─ Check line condition
   ├─ If executable: line._run()
   │  └─ Execute command
   │     └─ On finish: plan._run_next_action()
   │        ├─ Evaluate line actions
   │        ├─ Determine next step
   │        └─ Run next line or exit
   └─ If not executable: line._skip()
      └─ Log skip and continue
   ↓
9. plan_log.finish() - Complete execution
```

### Error Handling Strategy

Plans have a default error handling strategy (`on_error_action`):

| Strategy | Code | Behavior |
|----------|------|----------|
| Exit with command code | `'e'` | Stop and return command's exit code |
| Exit with custom code | `'ec'` | Stop and return plan's custom_exit_code |
| Run next command | `'n'` | Continue to next line regardless of error |

**Defined**: `cetmix_tower_server/models/cx_tower_plan.py:64-75`

Individual lines can override this with plan line actions.

### Conditional Execution

Each line can have a condition that determines if it executes:

**Line Field**: `condition` (Char)

**Example Conditions**:
```python
{{ odoo_version }} == '16.0'
{{ env_type }} == 'production'
{{ needs_restart }} == True
```

Conditions are evaluated using `safe_eval()` after variable rendering.

**Check Method**: `line._is_executable_line(server, variable_values)`
**Location**: `cetmix_tower_server/models/cx_tower_plan_line.py:207-246`

### Variable Sharing

Commands in a flight plan can share data via `custom_values` dictionary:

**In Python Commands**:
```python
# Set value
custom_values['deployment_id'] = '12345'

# Get value
deployment_id = custom_values.get('deployment_id')
```

**In Plan**:
- Values pass from command to command
- Available in execution context
- Stored in plan log
- Not persisted to database after execution

### Nested Flight Plans

Flight plans can execute other flight plans:

- Create command with `action='plan'`
- Set `flight_plan_id` to target plan
- Include in parent plan's lines
- Context key `from_command=True` skips compatibility check

**Recursion Prevention**: Context key `prevent_plan_recursion=True` prevents infinite loops.

**Check**: `_check_recursive_plan()` validates no circular references
**Location**: `cetmix_tower_server/models/cx_tower_plan_line.py:150-166`

## Status Codes

Flight plan execution returns status codes:

| Code | Constant | Meaning |
|------|----------|---------|
| `0` | Success | Plan completed successfully |
| `-301` | ANOTHER_PLAN_RUNNING | Another instance already running |
| `-302` | PLAN_IS_EMPTY | Plan has no executable lines |
| `-303` | PLAN_NOT_ASSIGNED | Plan reference missing in log |
| `-304` | PLAN_LINE_NOT_ASSIGNED | Plan line reference missing |
| `-306` | PLAN_NOT_COMPATIBLE_WITH_SERVER | Plan incompatible with server |
| `-308` | PLAN_STOPPED | Plan stopped by user |

**Defined**: `cetmix_tower_server/models/constants.py:43-64`

## Access Control

### Plan Access Levels

Flight plans inherit access control from `cx.tower.access.role.mixin`:

- Define who can view the plan
- Control who can execute the plan
- Set manager permissions

### User Assignment

- `user_ids`: Users who can execute the plan
- `manager_ids`: Users who can manage the plan

### Command Access Validation

Plans validate that commands don't have higher access levels than the plan itself.

**Warning**: `access_level_warn_msg` shows if command access exceeds plan access.

**Compute**: `_compute_command_access_level()`
**Location**: `cetmix_tower_server/models/cx_tower_plan.py:93-114`

## Best Practices

### For Administrators

1. **Plan Logically**: Group related operations
2. **Name Clearly**: Use descriptive plan and line names
3. **Use Conditions**: Make plans adaptable with conditions
4. **Handle Errors**: Configure appropriate error strategies
5. **Test Thoroughly**: Verify plans on test servers first
6. **Document Steps**: Add notes explaining complex logic
7. **Use Variables**: Make plans reusable with variables
8. **Set Compatibility**: Limit plans to compatible servers

### For Developers

1. **Pass Data via custom_values**: Share data between commands
2. **Validate Conditions**: Ensure condition syntax is correct
3. **Log Appropriately**: Use meaningful messages in Python commands
4. **Handle Edge Cases**: Consider what happens if a line fails
5. **Avoid Recursion**: Be careful with nested plans
6. **Clean Up Resources**: Ensure proper cleanup in all scenarios

## Performance Considerations

### Execution Time

- Plan duration = sum of all command durations
- Long-running plans should use appropriate timeouts
- Consider splitting very long plans

### Resource Usage

- Each line creates a command log
- Variables are rendered for each command
- SSH connections can be reused within plan
- Consider server load for concurrent plans

### Parallel Execution Control

**Field**: `allow_parallel_run` (Boolean)

- `False`: Only one instance per server (default for safety)
- `True`: Multiple instances allowed (use carefully)

## Monitoring and Debugging

### Plan Logs

View plan execution:
- Start and finish times
- Current execution status
- Currently executing line
- All command logs

### Command Logs

Each command in the plan creates a log:
- Links back to plan log
- Shows command-specific results
- Tracks individual command duration

### Stop Running Plans

Plans can be stopped during execution:
- Click "Stop" button on plan log
- Status becomes PLAN_STOPPED (-308)
- Currently running command is also stopped

**Method**: `plan_log.stop()`
**Location**: `cetmix_tower_server/models/cx_tower_plan_log.py:203-221`

## Troubleshooting

### Plan Not Running

**Check**:
- Plan compatibility with server
- Access permissions
- Another instance already running
- Plan has executable lines

### Line Skipped

**Causes**:
- Condition evaluated to False
- Check condition syntax
- Verify variable values
- Review condition logic

### Plan Failed

**Debug**:
- Check which line failed
- Review command log for that line
- Verify variable values were correct
- Check error handling strategy

### Unexpected Behavior

**Verify**:
- Line execution order (sequence field)
- Plan line actions are correct
- Condition evaluation is as expected
- Variable values are being set/passed correctly

## Related Features

- **[Command Execution](../command-execution/README.md)**: Commands executed in plans
- **[Server Management](../server-management/README.md)**: Servers where plans run
- **[Variable Management](../variable-management/README.md)**: Variables used in plans
- **[Logging & Monitoring](../logging-monitoring/README.md)**: Plan execution tracking
- **[Scheduling & Automation](../scheduling-automation/README.md)**: Schedule plan execution

## Next Steps

- Read [Flight Plan Basics](./01-flight-plan-basics.md) for detailed instructions
- Learn about [Command Types](../command-execution/01-command-types.md) to understand available commands
- Review [Variable Management](../variable-management/README.md) for dynamic plans
- Explore [Scheduling & Automation](../scheduling-automation/README.md) to automate plan execution

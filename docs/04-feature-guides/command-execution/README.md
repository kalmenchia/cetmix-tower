# Command Execution

Command execution is a core feature of Cetmix Tower that enables you to run various types of operations on remote servers. This feature provides flexible command types and execution modes to handle different use cases.

## Overview

The command execution feature provides:

- **Multiple Command Types**: SSH commands, Python code, file templates, flight plans
- **Flexible Execution**: Single server or multiple servers
- **Variable Support**: Dynamic command rendering with variables
- **Access Control**: Fine-grained permissions for command execution
- **Logging**: Complete execution history and results
- **Error Handling**: Configurable error handling strategies

## Core Concepts

### Command (cx.tower.command)

A command is a reusable operation that can be executed on one or more servers. Commands have:
- An action type (SSH, Python, file template, or flight plan)
- Code or template to execute
- Optional default path
- Variable support
- Server compatibility rules
- Access control settings

**Location**: `cetmix_tower_server/models/cx_tower_command.py`

### Command Types

Cetmix Tower supports four command types (defined in `_selection_action()` at line 217):

1. **ssh_command**: Execute shell commands via SSH
2. **python_code**: Execute Python code within Odoo environment
3. **file_using_template**: Create/update files from templates
4. **plan**: Execute a flight plan

### Command Log (cx.tower.command.log)

Every command execution creates a log record containing:
- Start and finish timestamps
- Execution status (exit code)
- Command output (response)
- Error messages
- Duration
- Variable values used

**Location**: `cetmix_tower_server/models/cx_tower_command_log.py`

## Feature Guides

### [01. Command Types](./01-command-types.md)

Comprehensive guide covering:
- Detailed explanation of each command type
- When to use each type
- Configuration options
- Execution context and available libraries
- Code examples for each type

**Audiences**: Administrators, End Users, Developers

## Key Models and Files

### Models

- **cx.tower.command**: Main command model
  File: `cetmix_tower_server/models/cx_tower_command.py`

- **cx.tower.command.log**: Command execution logs
  File: `cetmix_tower_server/models/cx_tower_command_log.py`

### Command Runners

Command execution is handled by runner methods in the server model:

- `_command_runner()`: Main router
  Location: `cetmix_tower_server/models/cx_tower_server.py:1098-1186`

- `_command_runner_ssh()`: SSH command execution
  Location: `cetmix_tower_server/models/cx_tower_server.py:1318-1369`

- `_command_runner_python_code()`: Python code execution
  Location: `cetmix_tower_server/models/cx_tower_server.py:1427-1466`

- `_command_runner_file_using_template()`: File template execution
  Location: `cetmix_tower_server/models/cx_tower_server.py:1215-1316`

- `_command_runner_flight_plan()`: Flight plan execution
  Location: `cetmix_tower_server/models/cx_tower_server.py:1371-1425`

## Common Use Cases

### For Administrators

1. **Create Standard Commands**: Define reusable operations
2. **Configure Access**: Control who can execute specific commands
3. **Set Server Compatibility**: Limit commands to specific servers/OSes
4. **Organize Commands**: Use tags for categorization
5. **Monitor Execution**: Review command logs and performance

### For End Users

1. **Execute Commands**: Run pre-configured commands on servers
2. **Provide Variables**: Supply values for command variables
3. **Monitor Progress**: Track command execution status
4. **Review Results**: Check command output and errors

### For Developers

1. **Programmatic Execution**: Run commands from code
2. **Custom Command Types**: Extend command functionality
3. **Integration**: Integrate commands with external systems
4. **Automation**: Trigger commands from workflows

## Technical Overview

### Command Execution Flow

```
1. User triggers command execution
   ↓
2. server.run_command(command, **kwargs)
   ↓
3. Check server compatibility
   ↓
4. Check parallel run restriction
   ↓
5. Render command code with variables
   ↓
6. Create command log (or skip if no_command_log=True)
   ↓
7. _command_runner_wrapper()
   ↓
8. Type-specific runner method
   ↓
9. Update command log with results
   ↓
10. Update server status (if configured)
```

### Variable Rendering

Commands can use variables in their code and paths:

```bash
# Command code with variables
cd {{ project_path }} && git pull
systemctl restart {{ service_name }}
```

Variables are rendered using `_render_command()` method:
- Extracts variables from code and path
- Gets variable values for the server
- Applies custom variable values (if user has write access)
- Renders code and path with values

**Location**: `cetmix_tower_server/models/cx_tower_server.py:940-1009`

### Server Compatibility

Commands can be limited to specific servers or operating systems:

- `server_ids`: Specific servers where command can run
- `os_ids`: Operating systems compatible with command
- Empty fields mean command is universally compatible

Compatibility is checked via `_check_server_compatibility()`:
**Location**: `cetmix_tower_server/models/cx_tower_command.py:309-318`

### Parallel Execution Control

The `allow_parallel_run` field controls whether multiple instances of the same command can run simultaneously on the same server.

- `allow_parallel_run=False`: Only one instance at a time
- `allow_parallel_run=True`: Multiple instances allowed

Status code `-201` (ANOTHER_COMMAND_RUNNING) is returned if blocked.

**Check location**: `cetmix_tower_server/models/cx_tower_server.py:877-903`

## Access Control

### Command Access Levels

Commands inherit access control from `cx.tower.access.mixin`:
- Public: Anyone can execute
- User: Specific users can execute
- Manager: Only managers can execute

### User Assignment

- `user_ids`: Users who can execute the command
- `manager_ids`: Users who can manage the command

### Execution Permissions

To execute a command, users need:
1. Access to the command itself
2. Access to the server (at least read)
3. Command must be compatible with server

## Configuration Options

### Common Fields

```python
# Basic
name = fields.Char()           # Command name
action = fields.Selection()    # Command type
code = fields.Text()           # Command code
path = fields.Char()           # Default execution path

# Access & Compatibility
server_ids = fields.Many2many('cx.tower.server')  # Compatible servers
os_ids = fields.Many2many('cx.tower.os')          # Compatible OS
tag_ids = fields.Many2many('cx.tower.tag')        # Tags

# Execution
allow_parallel_run = fields.Boolean()  # Allow concurrent execution
server_status = fields.Selection()     # Status to set on success
no_split_for_sudo = fields.Boolean()   # Don't split commands for sudo
```

### Type-Specific Fields

```python
# For file_using_template
file_template_id = fields.Many2one('cx.tower.file.template')
if_file_exists = fields.Selection([('skip', 'Skip'), ('overwrite', 'Overwrite'), ('raise', 'Raise Error')])
disconnect_file = fields.Boolean()  # Disconnect file from template after creation

# For plan
flight_plan_id = fields.Many2one('cx.tower.plan')
```

## Error Handling

### Status Codes

Commands return integer status codes:
- `0`: Success
- `-100`: General error
- `-201`: Another command running
- `-202`: No command runner found
- `-203`: Python command error
- `-206`: Command timed out
- `-207`: Command not compatible with server
- `-208`: Command stopped by user
- `503`: SSH connection error

**Defined**: `cetmix_tower_server/models/constants.py`

### Error Responses

Command execution returns a dictionary:
```python
{
    'status': int,      # Exit code
    'response': str,    # Command output
    'error': str        # Error message
}
```

### Timeout Handling

Commands running longer than configured timeout are automatically terminated:
- Configure via system parameter: `cetmix_tower_server.command_timeout`
- Zombie commands are marked with status `-206`
- Checked by `_check_zombie_commands()` method

## Best Practices

### For Administrators

1. **Name Commands Clearly**: Use descriptive, action-oriented names
2. **Use Tags**: Organize commands by purpose or project
3. **Set Server Compatibility**: Prevent execution on incompatible servers
4. **Test Commands**: Verify on test servers before production use
5. **Document Variables**: Use clear variable names with descriptions
6. **Enable Parallel Run Carefully**: Consider resource implications
7. **Set Status Updates**: Use server_status to track server lifecycle

### For Developers

1. **Use Context Keys**: Control behavior with context
2. **Handle SSH Connections**: Reuse connections for efficiency
3. **Validate Input**: Check variables before execution
4. **Log Appropriately**: Use command logs for auditing
5. **Handle Errors Gracefully**: Return meaningful error messages
6. **Clean Up Resources**: Ensure SSH disconnection
7. **Test Edge Cases**: Handle timeouts, network failures, etc.

## Security Considerations

### Command Code Security

- Python code executes in a safe evaluation context
- Limited libraries and functions available
- User input should be validated
- Avoid exposing sensitive data in command output

### SSH Command Security

- Commands run with configured user's permissions
- Sudo usage is controlled per server
- SSH keys more secure than passwords
- Host key verification prevents MITM attacks

### Secret Management

- Use inline secrets for sensitive values: `{{ KEY:secret_ref }}`
- Secrets are replaced with spoilers in logs
- Never hardcode passwords in command code
- Use variable system for sensitive configuration

## Performance Optimization

### Connection Reuse

For multiple commands on the same server:
```python
# Get connection once
ssh_client = server._get_ssh_client(raise_on_error=True)

# Reuse for multiple commands
server.run_command(cmd1, ssh_connection=ssh_client)
server.run_command(cmd2, ssh_connection=ssh_client)
server.run_command(cmd3, ssh_connection=ssh_client)
```

### Parallel Execution

For independent operations:
- Enable `allow_parallel_run=True`
- Use job queue module for background execution
- Consider server resource limits

### Variable Caching

Variables are cached during rendering:
- Minimize variable lookups
- Use local variables in Python commands
- Batch variable updates

## Troubleshooting

### Command Not Appearing for Server

**Causes**:
- Command limited to specific servers (check `server_ids`)
- OS incompatibility (check `os_ids`)
- Access control restrictions
- Command archived/inactive

### Command Execution Fails

**Check**:
- SSH connectivity (run `test_ssh_connection()`)
- Variable values are set correctly
- Command code syntax
- Server permissions (sudo configuration)
- Timeout settings

### Python Command Errors

**Common issues**:
- Syntax errors in Python code
- Undefined variables or objects
- Limited library access
- Safe eval restrictions

### File Template Errors

**Check**:
- Template code syntax
- Variable values
- Remote path accessibility
- File permissions on server

## Related Features

- **[Server Management](../server-management/README.md)**: Manage servers where commands execute
- **[Flight Plans](../flight-plans/README.md)**: Orchestrate multiple commands
- **[Variable Management](../variable-management/README.md)**: Use variables in commands
- **[Secret Management](../secret-management/README.md)**: Handle sensitive data
- **[File Management](../file-management/README.md)**: File templates and operations
- **[Logging & Monitoring](../logging-monitoring/README.md)**: Track command execution

## Next Steps

- Read [Command Types](./01-command-types.md) for detailed command type documentation
- Learn about [Flight Plans](../flight-plans/README.md) for multi-command orchestration
- Review [Variable Management](../variable-management/README.md) for dynamic commands
- Explore [Secret Management](../secret-management/README.md) for secure credential handling

# Server Management

Server management is the foundation of Cetmix Tower. This feature allows you to manage remote servers, configure SSH connections, organize servers, and execute operations across your infrastructure.

## Overview

The server management feature provides:

- **Server Creation & Configuration**: Add servers with SSH credentials
- **Connection Management**: Test and manage SSH connections
- **Server Organization**: Group servers with tags, partners, and templates
- **Lifecycle Management**: Track server status through various states
- **Access Control**: Define who can view, manage, and execute commands on servers

## Core Concepts

### Server (cx.tower.server)

The central model representing a remote server. Each server record contains:
- Connection details (IP address, SSH port, credentials)
- Authentication method (password or key-based)
- Operating system information
- Tags and organizational metadata
- Associated commands, flight plans, and files

### Server Templates (cx.tower.server.template)

Reusable configurations for creating multiple servers with similar settings:
- Pre-configured variables
- Default settings
- Standardized server logs

### Server Status

Servers can have different statuses representing their lifecycle:
- `stopped`: Server is stopped
- `starting`: Server is being started
- `running`: Server is active and running
- `stopping`: Server is being stopped
- `restarting`: Server is being restarted
- `deleting`: Server deletion in progress
- `delete_error`: Server deletion failed

## Feature Guides

### [01. Server Basics](./01-server-basics.md)

Comprehensive guide covering:
- Creating and configuring servers
- SSH connection setup (password and key-based)
- Server organization with tags and partners
- Server templates
- Server status lifecycle
- Running commands and flight plans

**Audiences**: Administrators, End Users, Developers

## Key Models and Files

### Models

- **cx.tower.server**: Main server model
  File: `cetmix_tower_server/models/cx_tower_server.py`

- **cx.tower.server.template**: Server template model
  File: `cetmix_tower_server/models/cx_tower_server_template.py`

- **cx.tower.server.log**: Server logs
  File: `cetmix_tower_server/models/cx_tower_server_log.py`

### SSH Implementation

- **SSH Connection & Manager**: SSH client implementation
  File: `cetmix_tower_server/ssh/ssh.py`

## Common Use Cases

### For Administrators

1. **Initial Server Setup**: Configure new servers with SSH credentials
2. **Template Creation**: Define reusable server configurations
3. **Access Management**: Control who can access and manage servers
4. **Server Organization**: Use tags and partners to categorize servers
5. **Connection Testing**: Verify SSH connectivity and troubleshoot issues

### For End Users

1. **View Server Information**: Check server details and status
2. **Run Commands**: Execute commands on accessible servers
3. **Monitor Operations**: Track command and flight plan execution
4. **Access Logs**: Review command execution history

### For Developers

1. **Programmatic Server Creation**: Create servers via code
2. **Custom SSH Operations**: Implement custom connection logic
3. **Server Lifecycle Hooks**: Extend server creation/deletion flows
4. **Integration**: Connect servers with external systems

## Related Features

- **[Command Execution](../command-execution/README.md)**: Execute commands on servers
- **[Flight Plans](../flight-plans/README.md)**: Orchestrate multi-step operations
- **[Variable Management](../variable-management/README.md)**: Use variables in server operations
- **[File Management](../file-management/README.md)**: Manage files on servers
- **[Secret Management](../secret-management/README.md)**: Store SSH keys and passwords securely
- **[Logging & Monitoring](../logging-monitoring/README.md)**: Track server operations

## Technical Overview

### Server Model Structure

The `cx.tower.server` model inherits from:
- `cx.tower.access.role.mixin`: Access control
- `cx.tower.variable.mixin`: Variable management
- `cx.tower.reference.mixin`: Reference tracking
- `mail.thread`: Odoo messaging
- `mail.activity.mixin`: Activity tracking
- `cx.tower.vault.mixin`: Secure storage

### Key Methods

- `run_command(command, path, sudo, ssh_connection, **kwargs)`: Execute commands
- `run_flight_plan(flight_plan, **kwargs)`: Execute flight plans
- `test_ssh_connection()`: Test SSH connectivity
- `upload_file(data, remote_path)`: Upload files to server
- `download_file(remote_path)`: Download files from server
- `_get_ssh_client()`: Get SSH connection instance

### SSH Connection Flow

```
Server Record → _get_ssh_client() → SSHConnection → SSHManager
                                   ↓
                            Paramiko SSH Client
                                   ↓
                            Remote Server
```

## Security Considerations

### SSH Key Storage

- Private SSH keys are stored securely using the vault mixin
- Keys are accessible only to users with appropriate permissions
- Host keys are verified to prevent MITM attacks (unless `skip_host_key` is enabled)

### Password Storage

- SSH passwords are stored encrypted
- Passwords are only accessible via secure methods (`_get_secret_value()`)
- Password access is logged and auditable

### Access Control

- Server access is controlled via access levels and user assignments
- Operations require appropriate permissions
- Sudo usage is configurable per server

## Configuration Options

### SSH Settings

- `ip_v4_address` / `ip_v6_address`: Server IP address
- `ssh_port`: SSH port (default: 22)
- `ssh_username`: SSH user account
- `ssh_auth_mode`: Authentication method ('p' for password, 'k' for key)
- `use_sudo`: Sudo configuration ('n' for no password, 'p' for password)
- `skip_host_key`: Skip host key verification (use with caution)

### Organization

- `partner_id`: Link server to a partner/customer
- `tag_ids`: Categorize with tags
- `os_id`: Operating system
- `server_template_id`: Source template

## Best Practices

### For Administrators

1. **Use Key-Based Authentication**: More secure than passwords
2. **Verify Host Keys**: Prevents man-in-the-middle attacks
3. **Use Server Templates**: Standardize server configurations
4. **Tag Servers Appropriately**: Makes filtering and searching easier
5. **Test Connections**: Verify SSH access after configuration
6. **Use Sudo Wisely**: Configure based on actual needs
7. **Regular Cleanup**: Archive or delete unused servers

### For Developers

1. **Use Context Keys**: Control behavior (e.g., `skip_ssh_settings_check`)
2. **Handle SSH Disconnections**: Use `@ensure_ssh_disconnect` decorator
3. **Validate Before Operations**: Check server compatibility
4. **Log Important Operations**: Use server logs for auditing
5. **Reuse SSH Connections**: Pass `ssh_connection` parameter when running multiple commands

## Troubleshooting

### Common Issues

**SSH Connection Failed**
- Verify IP address and port
- Check firewall rules
- Verify SSH credentials
- Test with `test_ssh_connection()`

**Host Key Verification Failed**
- Update host key using "Show Host Key" action
- Or enable `skip_host_key` (not recommended for production)

**Permission Denied**
- Verify SSH username and credentials
- Check server-side SSH configuration
- Ensure user has appropriate permissions

**Sudo Command Failed**
- Verify sudo configuration
- Check if user has sudo privileges
- Verify sudo password if required

## Next Steps

- Read [Server Basics](./01-server-basics.md) for detailed instructions
- Learn about [Command Execution](../command-execution/README.md)
- Explore [Flight Plans](../flight-plans/README.md) for orchestration
- Review [Secret Management](../secret-management/README.md) for SSH key management

---
title: "Bug Reporting Guide"
version: "1.0"
last_updated: "2025-11-16"
maintained_by: "E-Global SCM Development Team"
status: "Active"
audience: "All"
---

# Bug Reporting Guide

This guide helps you report bugs effectively to ensure quick resolution.

## Before Reporting

### 1. Verify It's a Bug

- **Read Documentation**: Check feature guides to ensure expected behavior
- **Check Version**: Verify you're using a supported version (Odoo 17.0)
- **Test in Clean Environment**: Try to reproduce without customizations
- **Review Logs**: Check command logs, plan logs, and Odoo logs

### 2. Search Existing Issues

Visit [GitHub Issues](https://github.com/cetmix/cetmix-tower/issues) and search for:
- Similar error messages
- Related functionality
- Your module/feature area

### 3. Gather Information

Collect the following before reporting:

#### System Information
- Odoo version: `17.0.x.x`
- Tower module versions (from Apps menu)
- PostgreSQL version
- Operating system
- Python version

#### Bug Details
- Exact steps to reproduce
- Expected behavior
- Actual behavior
- Error messages (full stack traces)
- Screenshots if applicable
- Relevant log entries

---

## How to Report a Bug

### GitHub Issues (Recommended)

**Step 1**: Go to [Cetmix Tower Issues](https://github.com/cetmix/cetmix-tower/issues)

**Step 2**: Click "New Issue"

**Step 3**: Use the bug report template below

---

## Bug Report Template

```markdown
### Bug Description
[Clear, concise description of the bug]

### Steps to Reproduce
1. Navigate to [location]
2. Click [button/action]
3. Enter [data]
4. Observe [issue]

### Expected Behavior
[What should happen]

### Actual Behavior
[What actually happens]

### Environment
- **Odoo Version**: 17.0.x.x
- **Tower Modules**: cetmix_tower_server 17.0.2.0.5, cetmix_tower_git 17.0.1.0.5
- **PostgreSQL**: 14.x
- **OS**: Ubuntu 22.04 / Windows 10 / macOS
- **Python**: 3.10.x

### Error Messages
```
[Paste full error message or stack trace]
```

### Logs
```
[Paste relevant log entries from command logs, plan logs, or Odoo logs]
```

### Screenshots
[Attach screenshots if helpful]

### Additional Context
[Any other relevant information]

### Workaround
[If you found a temporary workaround, describe it here]
```

---

## Severity Levels

Help us prioritize by selecting appropriate severity:

### 🔴 Critical
- **Impact**: System unusable, data loss, security vulnerability
- **Examples**:
  - Cannot connect to any servers
  - Vault data corruption
  - Security breach
- **Response Time**: Immediate

### 🟠 High
- **Impact**: Major functionality broken, no workaround available
- **Examples**:
  - Flight plans fail to execute
  - File sync not working
  - Commands timeout always
- **Response Time**: 1-2 business days

### 🟡 Medium
- **Impact**: Functionality impaired, workaround available
- **Examples**:
  - Specific command type fails
  - UI display issue
  - Performance degradation
- **Response Time**: 1 week

### 🟢 Low
- **Impact**: Minor inconvenience, cosmetic issue
- **Examples**:
  - Typo in UI
  - Minor formatting issue
  - Feature request
- **Response Time**: As time permits

---

## Bug Categories

### Performance Issues

**Template Additions**:
```markdown
### Performance Metrics
- **Duration**: [Current execution time]
- **Expected Duration**: [Expected execution time]
- **Server Load**: [CPU/Memory usage during issue]
- **Database Queries**: [Number of queries if known]

### Performance Test Results
[Include any benchmarking or profiling data]
```

### Data Issues

**Template Additions**:
```markdown
### Data Affected
- **Model**: cx.tower.server / cx.tower.command / etc.
- **Records Affected**: [Number or IDs]
- **Data State**: [Current incorrect state]
- **Expected Data State**: [What data should be]

### SQL Query
```sql
-- Query to identify affected records
SELECT id, name, [fields]
FROM [table]
WHERE [condition];
```
```

### UI/Display Issues

**Template Additions**:
```markdown
### Browser Information
- **Browser**: Chrome 120 / Firefox 121 / Safari 17
- **Screen Resolution**: 1920x1080
- **Zoom Level**: 100%

### Screenshots
[Attach before/after or comparison screenshots]
```

### Integration Issues

**Template Additions**:
```markdown
### External System
- **Type**: AWS / OVH / Git / Webhook / Other
- **Version**: [External system version]
- **API Endpoint**: [If applicable]
- **Authentication Method**: [Key/password/token]

### Integration Logs
[Paste logs from external system if available]
```

---

## Command/Plan Execution Failures

For command or flight plan failures, include:

### Command Log Details
1. Navigate to **Tower → Logs → Command Logs**
2. Find the failed execution
3. Include:
   - Command name and code
   - Server name
   - Start/finish date
   - Exit code
   - Response output
   - Error output

**Example**:
```
Command: Deploy Application
Server: production-web-01
Exit Code: -201
Error: Another command is already running
Duration: 0.1 seconds
```

### Flight Plan Log Details
1. Navigate to **Tower → Logs → Flight Plan Logs**
2. Find the failed execution
3. Include:
   - Flight plan name
   - Server name
   - Which line failed
   - Command logs from all executed lines

---

## Security Issues

⚠️ **IMPORTANT**: DO NOT report security vulnerabilities publicly on GitHub!

For security issues:

1. **Email**: security@cetmix.com
2. **Include**: All standard bug information
3. **Encrypt**: Use PGP if possible
4. **Responsible Disclosure**: Allow time for patch before public disclosure

---

## What Happens After Reporting

### 1. Initial Response
- Team acknowledges issue within 1-2 business days
- Severity and priority assigned
- Additional information requested if needed

### 2. Investigation
- Issue reproduced in development environment
- Root cause identified
- Solution designed

### 3. Resolution
- Fix implemented and tested
- Patch released in next version
- GitHub issue updated with resolution

### 4. Verification
- Reporter asked to verify fix
- Issue closed when confirmed resolved

---

## Best Practices

### ✅ DO

- **Be Specific**: Exact steps, not "it doesn't work"
- **Include Context**: Environment, versions, configuration
- **Provide Examples**: Sample data, commands, screenshots
- **Follow Up**: Respond to questions from developers
- **Test Workarounds**: Try suggested temporary fixes
- **Update**: Comment if you find new information

### ❌ DON'T

- **Don't Duplicate**: Search first, comment on existing issues
- **Don't Demand**: Be respectful and professional
- **Don't Omit Info**: Include all requested details
- **Don't Mix Issues**: One bug per report
- **Don't Ignore Requests**: Respond to developer questions

---

## Common Issues & Quick Fixes

Before reporting, try these common fixes:

### "SSH Connection Failed"
1. Verify server IP and port
2. Test SSH manually: `ssh user@server -p 22`
3. Check host key verification settings
4. Verify firewall allows connection

### "Command Timeout"
1. Increase timeout in Settings
2. Check if command is hanging
3. Test command manually via SSH
4. Review server load

### "File Sync Failed"
1. Verify file permissions on server
2. Check file path exists
3. Test file operations manually
4. Review server logs

### "Variable Not Substituted"
1. Verify variable reference format: `{{ var_name }}`
2. Check variable is defined
3. Verify variable value is set
4. Review variable scope (global vs server-specific)

### "Access Denied"
1. Check user access level
2. Verify record ownership
3. Review access groups
4. Check manager permissions

---

## Feature Requests

Feature requests are welcome! Use the same process but:

1. **Label**: Use "enhancement" or "feature request" label
2. **Describe Use Case**: Explain why you need it
3. **Provide Examples**: Show how it would work
4. **Consider Alternatives**: Mention if existing features almost work

**Template**:
```markdown
### Feature Description
[Clear description of desired feature]

### Use Case
[Why you need this feature, what problem it solves]

### Proposed Solution
[How you envision it working]

### Alternatives Considered
[Other approaches you've tried]

### Additional Context
[Mockups, examples, references]
```

---

## Getting Community Help

For questions and discussions:

- **GitHub Discussions**: [Repository discussions](https://github.com/cetmix/cetmix-tower/discussions)
- **Stack Overflow**: Tag with `odoo` and `cetmix-tower`
- **Odoo Community**: [Odoo forums](https://www.odoo.com/forum)

---

## Related Documentation

- [Troubleshooting Guide](../09-deployment-operations/03-troubleshooting.md)
- [Common Tasks](../02-getting-started/04-common-tasks.md)
- [Technical Support](../01-overview/01-project-overview.md#getting-help)
- [Development Workflows](../08-development-workflows/README.md)

---

**Last Updated**: 2025-11-16
**Next Review**: 2026-02-16
**Maintained By**: E-Global SCM Development Team

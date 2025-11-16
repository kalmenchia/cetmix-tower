---
title: "Technical Guides"
section: "07-technical-guides"
audience: "Developers, Technical Consultants"
version: "17.0"
last_updated: "2025-11-16"
---

# Technical Guides

Welcome to the Cetmix Tower Technical Guides. This section provides in-depth technical documentation for developers, technical consultants, and system architects working with Cetmix Tower.

## Target Audience

These guides are designed for:

- **Odoo Developers**: Building custom modules or extending Tower functionality
- **Technical Consultants**: Implementing and customizing Tower for clients
- **System Architects**: Designing Tower-based solutions
- **DevOps Engineers**: Integrating Tower with CI/CD pipelines and infrastructure
- **Module Contributors**: Contributing to Cetmix Tower development

**Not your role?**
- **End Users**: See [User Guides](../03-user-guides/README.md)
- **System Administrators**: See [Getting Started](../02-getting-started/README.md) and [Deployment](../09-deployment-operations/README.md)
- **Module Documentation**: See [Module References](../05-module-references/README.md)

---

## Guide Organization

The technical guides are organized by technical domain, progressing from data layer to presentation layer:

### 1. [Models and ORM](./01-models-and-orm.md)

Deep dive into Cetmix Tower's data model architecture.

**Topics Covered:**
- Complete model hierarchy and inheritance
- Model relationships and dependencies
- Mixin architecture (Reference, Access, Vault, Variable, Template, Key)
- ORM patterns and best practices
- Database schema considerations
- Performance optimization techniques

**Who should read this:**
- Developers extending Tower models
- Architects designing Tower integrations
- Contributors adding new features

**Key Learnings:**
- How to leverage Tower's mixin system
- Understanding the vault security mechanism
- Working with references and access control
- Model dependency management

---

### 2. [Views and UI](./02-views-and-ui.md)

Comprehensive guide to Tower's user interface architecture.

**Topics Covered:**
- View architecture (form, tree, kanban, calendar)
- OWL (Odoo Web Library) components
- Custom widgets and fields
- JavaScript customizations
- SCSS styling and theming
- Asset bundles and loading

**Who should read this:**
- Frontend developers
- UI/UX customization specialists
- Developers creating custom widgets

**Key Learnings:**
- Tower's custom UI components
- ACE editor integration for code editing
- Server status field implementation
- Variable autocomplete functionality

---

### 3. [Security and Access Control](./03-security-and-access.md)

Complete security model documentation.

**Topics Covered:**
- Security architecture overview
- Role-based access control (User, Manager, Root)
- Access mixins implementation
- Record rules and field-level security
- Vault system for secrets management
- SSH security best practices
- Security testing and validation

**Who should read this:**
- Security consultants
- Developers implementing access control
- System administrators
- Compliance teams

**Key Learnings:**
- Three-tier access level system
- How the vault protects sensitive data
- Record-level permission implementation
- Best practices for securing Tower installations

---

## Prerequisites for Developers

Before working with Cetmix Tower, you should be familiar with:

### Required Knowledge

**Odoo Framework:**
- ✅ Odoo 17.0 architecture
- ✅ ORM (Object-Relational Mapping)
- ✅ Model inheritance patterns
- ✅ View definitions (XML)
- ✅ Security (groups, rules, access rights)
- ✅ Python programming

**Web Technologies:**
- ✅ JavaScript (ES6+)
- ✅ OWL (Odoo Web Library)
- ✅ XML for views
- ✅ SCSS/CSS
- ✅ REST APIs

**Infrastructure:**
- ✅ SSH protocol and authentication
- ✅ Linux/Unix systems
- ✅ Shell scripting
- ✅ Git version control

### Recommended Knowledge

**Python Libraries:**
- 📚 Paramiko (SSH implementation)
- 📚 DNSPython (DNS operations)
- 📚 tldextract (domain parsing)

**DevOps Concepts:**
- 📚 CI/CD pipelines
- 📚 Infrastructure as Code
- 📚 Configuration management
- 📚 Monitoring and logging

---

## Development Environment Setup

### Required Dependencies

Cetmix Tower requires these external Python dependencies:

```python
# From cetmix_tower_server/__manifest__.py
"external_dependencies": {
    "python": [
        "paramiko<4",      # SSH connectivity
        "tldextract",      # Domain extraction
        "dnspython",       # DNS operations
    ],
}
```

**Installation:**

```bash
# Install Python dependencies
pip install 'paramiko<4' tldextract dnspython

# For development
pip install -r requirements.txt  # If available
```

### Module Dependencies

Cetmix Tower depends on these Odoo modules:

```python
"depends": [
    "mail",          # Messaging and activity tracking
    "rpc_helper",    # RPC utilities
]
```

---

## Architecture Overview

### Module Structure

Cetmix Tower consists of multiple modules:

**Core Modules:**
- `cetmix_tower`: Base module and main menu
- `cetmix_tower_server`: Core server management functionality

**Optional Modules:**
- `cetmix_tower_yaml`: YAML-based configuration
- `cetmix_tower_server_queue`: Queue job integration
- `cetmix_tower_server_notify_backend`: Backend notifications
- `cetmix_tower_git`: Git integration
- `cetmix_tower_webhook`: Webhook support
- `cetmix_tower_aws`: AWS integration
- `cetmix_tower_ovh`: OVH cloud integration

### Core Concepts

**1. Servers**
- Physical or virtual machines managed by Tower
- SSH connectivity for command execution
- Variable values storage
- Status tracking

**2. Commands**
- Shell scripts or Python code
- Variable substitution
- OS compatibility checks
- Timeout and error handling

**3. Flight Plans**
- Multi-command orchestration
- Conditional execution
- Error handling strategies
- Sequential or parallel execution

**4. Files**
- Configuration file management
- Template-based generation
- Bi-directional sync (Tower ↔ Server)
- Auto-sync capabilities

**5. Variables**
- Parameterized commands and files
- Type system (String, Options)
- Validation and transformation
- Hierarchical value resolution

---

## Code Organization

### File Paths Reference

All file paths in this documentation are absolute paths from the repository root:

```
/home/user/cetmix-tower/
├── cetmix_tower_server/
│   ├── models/
│   │   ├── cx_tower_server.py           # Server model
│   │   ├── cx_tower_command.py          # Command model
│   │   ├── cx_tower_plan.py             # Flight plan model
│   │   ├── cx_tower_file.py             # File model
│   │   ├── cx_tower_variable.py         # Variable model
│   │   ├── cx_tower_reference_mixin.py  # Reference mixin
│   │   ├── cx_tower_access_mixin.py     # Access control mixin
│   │   ├── cx_tower_vault_mixin.py      # Vault security mixin
│   │   └── ...
│   ├── views/
│   │   ├── cx_tower_server_view.xml
│   │   ├── cx_tower_command_view.xml
│   │   └── ...
│   ├── security/
│   │   ├── cetmix_tower_server_groups.xml
│   │   ├── ir.model.access.csv
│   │   └── cx_tower_*_security.xml
│   ├── static/src/
│   │   ├── components/
│   │   │   ├── ace_variables/
│   │   │   └── server_status/
│   │   └── utils/
│   ├── ssh/
│   │   └── ssh.py                       # SSH manager
│   └── ...
└── ...
```

---

## Development Workflow

### Typical Development Tasks

**1. Extending Models**
```python
# Inherit existing model
class CxTowerServer(models.Model):
    _inherit = 'cx.tower.server'

    custom_field = fields.Char()
```

**2. Adding Custom Commands**
```python
# Add new command type
class CxTowerCommand(models.Model):
    _inherit = 'cx.tower.command'

    def _execute_custom_type(self, server):
        # Custom implementation
        pass
```

**3. Creating Custom Widgets**
```javascript
// OWL component for custom field
import { Component } from "@odoo/owl";

class CustomWidget extends Component {
    // Implementation
}
```

**4. Adding Security Rules**
```xml
<!-- Record rule for custom access -->
<record id="custom_access_rule" model="ir.rule">
    <field name="name">Custom Access</field>
    <field name="model_id" ref="model_cx_tower_custom"/>
    <field name="domain_force">...</field>
</record>
```

---

## Best Practices

### Code Standards

**Python:**
- Follow PEP 8 style guide
- Use Odoo conventions (OCA guidelines)
- Document complex methods
- Add docstrings to all public methods
- Use type hints where beneficial

**JavaScript:**
- Follow Odoo OWL conventions
- Use ES6+ features
- Component-based architecture
- Proper event handling

**XML:**
- Consistent indentation (4 spaces)
- Meaningful record IDs
- Group related records
- Comment complex structures

### Security Considerations

⚠️ **Critical Security Rules:**

1. **Never store secrets in plain text fields**
   - Always use vault mixin for sensitive data
   - Use `SECRET_FIELDS` attribute

2. **Validate user input**
   - Check command parameters
   - Sanitize file paths
   - Validate variable values

3. **Respect access levels**
   - Don't bypass access checks
   - Use `sudo()` carefully
   - Log security-relevant operations

4. **SSH safety**
   - Always disconnect SSH connections
   - Use connection timeouts
   - Handle connection errors gracefully

---

## Testing

### Test Categories

**1. Unit Tests**
- Test individual model methods
- Mock external dependencies
- Fast execution

**2. Integration Tests**
- Test SSH connectivity
- Test command execution
- Test file synchronization

**3. Security Tests**
- Verify access control
- Test record rules
- Validate vault security

### Running Tests

```bash
# Run all tests
odoo-bin -d test_db -i cetmix_tower_server --test-enable --stop-after-init

# Run specific test
odoo-bin -d test_db --test-tags cetmix_tower_server.test_server
```

---

## Performance Optimization

### Key Performance Considerations

**1. Database Queries**
- Use `read()` for batch operations
- Minimize search queries in loops
- Use `search_count()` when appropriate
- Leverage database indexes

**2. SSH Connections**
- Reuse connections when possible
- Close connections properly (use decorator)
- Set appropriate timeouts
- Pool connections for parallel operations

**3. File Operations**
- Stream large files
- Use batching for multiple files
- Implement progress tracking
- Handle failures gracefully

**4. Caching**
- Cache command templates
- Cache variable values
- Use `@ormcache` appropriately
- Clear caches when data changes

---

## Debugging

### Debugging Tools

**1. Odoo Logging**
```python
import logging
_logger = logging.getLogger(__name__)

_logger.debug("Debug message")
_logger.info("Info message")
_logger.warning("Warning message")
_logger.error("Error message")
```

**2. PDB Debugger**
```python
import pdb
pdb.set_trace()  # Breakpoint
```

**3. Browser DevTools**
- Console for JavaScript debugging
- Network tab for API calls
- Elements for DOM inspection

### Common Issues

See [Troubleshooting Guide](../09-deployment-operations/04-troubleshooting.md) for common issues and solutions.

---

## Contributing

### Contribution Guidelines

1. **Fork and Branch**
   - Fork the repository
   - Create feature branch
   - Make changes
   - Test thoroughly

2. **Code Review**
   - Follow coding standards
   - Add tests
   - Update documentation
   - Submit pull request

3. **Documentation**
   - Document new features
   - Update API references
   - Add code examples
   - Update changelog

---

## Resources

### Official Documentation

- [Odoo Documentation](https://www.odoo.com/documentation/17.0/)
- [OWL Documentation](https://github.com/odoo/owl)
- [Paramiko Documentation](https://www.paramiko.org/)

### Community

- [Cetmix Tower Website](https://tower.cetmix.com)
- [GitHub Repository](https://github.com/cetmix/cetmix-tower)
- [OCA Guidelines](https://github.com/OCA/odoo-community.org)

---

## Quick Navigation

### Technical Guides

| Guide | Focus | Audience |
|-------|-------|----------|
| [Models and ORM](./01-models-and-orm.md) | Data layer, models, mixins | Backend developers |
| [Views and UI](./02-views-and-ui.md) | Interface, components, widgets | Frontend developers |
| [Security](./03-security-and-access.md) | Access control, vault, permissions | Security specialists |

### Related Documentation

- [Module References](../05-module-references/README.md): Detailed module documentation
- [API References](../06-api-references/README.md): API documentation
- [Development Workflows](../08-development-workflows/README.md): Development processes

---

**Navigation:**
- [← Back to Documentation Home](../README.md)
- [Next: Models and ORM →](./01-models-and-orm.md)

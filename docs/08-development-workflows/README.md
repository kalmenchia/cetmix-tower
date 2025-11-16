---
title: Development Workflows
description: Development practices, standards, and workflows for Cetmix Tower
category: development-workflows
order: 1
---

# Development Workflows

This section covers development practices, coding standards, testing guidelines, and Git workflows for contributing to Cetmix Tower.

## Overview

Cetmix Tower follows Odoo Community Association (OCA) development standards with additional project-specific conventions. This ensures code quality, consistency, and maintainability across the project.

## Development Environment

### Requirements

- **Odoo**: Version 17.0
- **Python**: 3.10+
- **PostgreSQL**: 12+
- **Node.js**: 16.17.0 (for linting tools)
- **Git**: For version control

### Setup

```bash
# Clone repository
git clone https://github.com/cetmix/cetmix-tower.git
cd cetmix-tower

# Create Python virtual environment
python3 -m venv venv
source venv/bin/activate  # or venv\Scripts\activate on Windows

# Install dependencies
pip install -r requirements.txt

# Install pre-commit hooks
pre-commit install
```

### Project Structure

```
cetmix-tower/
├── cetmix_tower/              # Base module (deprecated, dependencies only)
├── cetmix_tower_server/       # Core server management module
├── cetmix_tower_yaml/         # YAML import/export functionality
├── cetmix_tower_webhook/      # Webhook integration
├── cetmix_tower_git/          # Git integration
├── cetmix_tower_aws/          # AWS integration
├── cetmix_tower_ovh/          # OVH integration
├── cetmix_tower_server_queue/ # Job queue integration
├── cetmix_tower_server_notify_backend/ # Notification backend
├── docs/                      # Documentation
├── .pre-commit-config.yaml    # Pre-commit hooks configuration
├── .pylintrc                  # Pylint configuration
├── .pylintrc-mandatory        # Mandatory Pylint checks
├── .ruff.toml                 # Ruff linter configuration
├── .eslintrc.yml              # ESLint configuration
└── requirements.txt           # Python dependencies
```

## Quick Links

### [Coding Standards](01-coding-standards.md)

Learn about:
- Python coding standards (PEP 8, Odoo guidelines)
- Naming conventions
- Documentation requirements
- XML/view standards
- JavaScript/OWL standards
- Pre-commit hooks and linting

### [Testing Guidelines](02-testing-guidelines.md)

Learn about:
- Testing philosophy
- Unit testing with pytest
- Integration testing
- Test coverage requirements
- Running tests
- CI/CD testing

### [Git Workflow](03-git-workflow.md)

Learn about:
- Branching strategy
- Commit message conventions
- Pull request process
- Code review guidelines
- Release management

## Development Tools

### Pre-commit Hooks

Cetmix Tower uses pre-commit hooks to enforce code quality:

```bash
# Install hooks
pre-commit install

# Run manually
pre-commit run --all-files

# Update hooks
pre-commit autoupdate
```

**Configured hooks** (from `.pre-commit-config.yaml`):
- `prettier` - Code formatting (JS, XML, JSON, YAML)
- `eslint` - JavaScript linting
- `ruff` - Python linting and formatting
- `pylint-odoo` - Odoo-specific Python checks
- `oca-checks` - OCA module validation

### Linting Tools

#### Python - Ruff

Fast Python linter and formatter:

```bash
# Check code
ruff check .

# Fix auto-fixable issues
ruff check --fix .

# Format code
ruff format .
```

Configuration: `.ruff.toml`

#### Python - Pylint

Odoo-specific linting:

```bash
# Run pylint with optional checks
pylint --rcfile=.pylintrc cetmix_tower_server/

# Run mandatory checks only
pylint --rcfile=.pylintrc-mandatory cetmix_tower_server/
```

#### JavaScript - ESLint

```bash
# Check JavaScript files
eslint --color --fix static/src/**/*.js
```

Configuration: `.eslintrc.yml`

#### XML/JSON/YAML - Prettier

```bash
# Format files
prettier --write "**/*.{xml,json,yaml,yml}"
```

## Module Development

### Creating a New Module

```bash
# Create module directory
mkdir cetmix_tower_custom

# Create __manifest__.py
cat > cetmix_tower_custom/__manifest__.py << 'EOF'
{
    "name": "Cetmix Tower Custom",
    "version": "17.0.1.0.0",
    "category": "Tools",
    "summary": "Custom extensions for Cetmix Tower",
    "author": "Your Name",
    "website": "https://example.com",
    "license": "AGPL-3",
    "depends": ["cetmix_tower_server"],
    "data": [],
    "installable": True,
    "application": False,
}
EOF

# Create __init__.py
mkdir cetmix_tower_custom/models
touch cetmix_tower_custom/__init__.py
touch cetmix_tower_custom/models/__init__.py
```

### Module Structure

```
cetmix_tower_custom/
├── __init__.py
├── __manifest__.py
├── models/
│   ├── __init__.py
│   └── custom_model.py
├── views/
│   └── custom_views.xml
├── security/
│   └── ir.model.access.csv
├── data/
│   └── data.xml
├── static/
│   └── src/
│       └── js/
│           └── custom.js
├── tests/
│   ├── __init__.py
│   └── test_custom.py
└── README.md
```

## Running Odoo

### Development Mode

```bash
# Start Odoo in development mode
odoo-bin -c /etc/odoo/odoo.conf \
  --dev=all \
  --log-level=debug \
  --limit-time-cpu=99999 \
  --limit-time-real=99999
```

### With Module Updates

```bash
# Update module on startup
odoo-bin -c /etc/odoo/odoo.conf \
  -u cetmix_tower_server \
  --dev=all
```

### Running Tests

```bash
# Run tests for specific module
odoo-bin -c /etc/odoo/odoo.conf \
  -i cetmix_tower_server \
  --test-enable \
  --stop-after-init \
  --log-level=test

# Run specific test
odoo-bin -c /etc/odoo/odoo.conf \
  --test-tags cetmix_tower_server.tests.test_server \
  --test-enable \
  --stop-after-init
```

## Debugging

### Python Debugger

```python
# Add breakpoint in code
import pdb; pdb.set_trace()

# Or use built-in breakpoint() (Python 3.7+)
breakpoint()
```

### Odoo Shell

```bash
# Start Odoo shell
odoo-bin shell -d your-database -c /etc/odoo/odoo.conf

# In shell
>>> server = env['cx.tower.server'].browse(1)
>>> server.name
'Production Server'
```

### Log Debugging

```python
import logging
_logger = logging.getLogger(__name__)

_logger.info('Info message')
_logger.warning('Warning message')
_logger.error('Error message')
_logger.debug('Debug message')  # Only with --log-level=debug
```

## Contributing

### Before Contributing

1. Read [Coding Standards](01-coding-standards.md)
2. Set up pre-commit hooks
3. Review [Git Workflow](03-git-workflow.md)
4. Check [Testing Guidelines](02-testing-guidelines.md)

### Contribution Process

1. **Fork** the repository
2. **Create branch** from main/17.0
3. **Make changes** following coding standards
4. **Write tests** for new functionality
5. **Run linters** and fix issues
6. **Commit** with conventional commit messages
7. **Push** to your fork
8. **Create Pull Request** with description
9. **Address review** comments
10. **Merge** after approval

### Getting Help

- **Documentation**: Check docs/ directory
- **Issues**: [GitHub Issues](https://github.com/cetmix/cetmix-tower/issues)
- **Discussions**: [GitHub Discussions](https://github.com/cetmix/cetmix-tower/discussions)
- **Support**: support@cetmix.com

## Best Practices

### Code Quality

1. **Follow OCA guidelines** - Use standard Odoo patterns
2. **Write docstrings** - Document all public methods
3. **Use type hints** - Where applicable
4. **Keep functions small** - Single responsibility principle
5. **Avoid duplication** - DRY principle

### Performance

1. **Optimize searches** - Use appropriate domains
2. **Batch operations** - Avoid loops with ORM operations
3. **Use SQL wisely** - For complex aggregations
4. **Profile code** - Identify bottlenecks
5. **Cache when appropriate** - Use `@ormcache`

### Security

1. **Validate input** - Always validate user input
2. **Use access rules** - Implement proper security
3. **Avoid SQL injection** - Use parameterized queries
4. **Secure secrets** - Use vault for sensitive data
5. **Check permissions** - Verify access rights

### Documentation

1. **Update README** - Keep module README current
2. **Document APIs** - Include examples
3. **Write changelog** - Document changes
4. **Add inline comments** - For complex logic
5. **Create user guides** - For new features

## Resources

### Odoo Documentation

- [Odoo 17 Documentation](https://www.odoo.com/documentation/17.0/)
- [ORM API](https://www.odoo.com/documentation/17.0/developer/reference/backend/orm.html)
- [Security](https://www.odoo.com/documentation/17.0/developer/reference/backend/security.html)
- [Views](https://www.odoo.com/documentation/17.0/developer/reference/backend/views.html)

### OCA Guidelines

- [OCA Guidelines](https://github.com/OCA/odoo-community.org/blob/master/website/Contribution/CONTRIBUTING.rst)
- [Module Structure](https://github.com/OCA/maintainer-tools/blob/master/CONTRIBUTING.md)
- [Pylint Plugin](https://github.com/OCA/pylint-odoo)

### Tower Resources

- [Cetmix Tower Docs](https://tower.cetmix.com/documentation)
- [GitHub Repository](https://github.com/cetmix/cetmix-tower)
- [Issue Tracker](https://github.com/cetmix/cetmix-tower/issues)

## License

Cetmix Tower is licensed under [AGPL-3.0](https://www.gnu.org/licenses/agpl-3.0.en.html).

All contributions must be compatible with this license.

## Next Steps

- [Coding Standards →](01-coding-standards.md)
- [Testing Guidelines →](02-testing-guidelines.md)
- [Git Workflow →](03-git-workflow.md)

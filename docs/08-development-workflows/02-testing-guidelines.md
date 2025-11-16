---
title: Testing Guidelines
description: Testing strategies and guidelines for Cetmix Tower
category: development-workflows
order: 3
---

# Testing Guidelines

Comprehensive testing guidelines for Cetmix Tower development.

## Testing Philosophy

- **Test Early**: Write tests alongside code
- **Test Often**: Run tests frequently during development
- **Test Coverage**: Aim for high coverage of critical paths
- **Test Quality**: Focus on meaningful tests, not just coverage numbers

## Testing Stack

- **Framework**: Odoo's built-in test framework (based on unittest)
- **Test Runner**: Odoo test runner
- **Additional Tools**: pytest (optional), coverage.py

## Test Structure

### Directory Layout

```
cetmix_tower_server/
├── models/
│   └── cx_tower_server.py
├── tests/
│   ├── __init__.py
│   ├── common.py              # Shared test utilities
│   ├── test_server.py         # Server model tests
│   ├── test_command.py        # Command model tests
│   └── test_integration.py    # Integration tests
```

### Test File Structure

```python
# tests/test_server.py
from odoo.tests import tagged
from odoo.tests.common import TransactionCase
from odoo.exceptions import ValidationError

from .common import TowerTestCase


@tagged('post_install', '-at_install')
class TestCxTowerServer(TowerTestCase):
    """Test cx.tower.server model."""

    @classmethod
    def setUpClass(cls):
        """Set up test data."""
        super().setUpClass()
        cls.Server = cls.env['cx.tower.server']
        cls.test_server = cls._create_test_server()

    def test_create_server(self):
        """Test server creation."""
        server = self.Server.create({
            'name': 'Test Server',
            'ip_v4_address': '192.168.1.100',
            'ssh_username': 'ubuntu',
            'ssh_port': 22,
            'ssh_auth_mode': 'p',
            'ssh_password': 'test123',
        })
        self.assertTrue(server.id)
        self.assertEqual(server.name, 'Test Server')

    def test_ssh_connection_validation(self):
        """Test SSH connection validation."""
        with self.assertRaises(ValidationError):
            self.Server.create({
                'name': 'Invalid Server',
                # Missing IP address
                'ssh_username': 'ubuntu',
            })
```

## Test Types

### 1. Unit Tests

Test individual methods in isolation.

```python
def test_render_command(self):
    """Test command rendering with variables."""
    command = self.env['cx.tower.command'].create({
        'name': 'Test Command',
        'action': 'ssh_command',
        'code': 'echo {{ version }}',
    })

    rendered = self.test_server._render_command(
        command,
        custom_variable_values={'version': '1.2.3'}
    )

    self.assertEqual(
        rendered['rendered_code'],
        'echo 1.2.3'
    )
```

### 2. Integration Tests

Test component interactions.

```python
def test_run_command_flow(self):
    """Test complete command execution flow."""
    # Create command
    command = self.env['cx.tower.command'].create({
        'name': 'Echo Test',
        'action': 'ssh_command',
        'code': 'echo "test"',
    })

    # Mock SSH connection
    with self.mock_ssh_connection():
        # Run command
        self.test_server.run_command(command)

    # Verify log created
    log = self.env['cx.tower.command.log'].search([
        ('server_id', '=', self.test_server.id),
        ('command_id', '=', command.id),
    ], limit=1)

    self.assertTrue(log)
    self.assertEqual(log.command_status, 0)
```

### 3. Functional Tests

Test end-to-end workflows.

```python
@tagged('post_install', '-at_install', 'functional')
class TestDeploymentWorkflow(TowerTestCase):
    """Test deployment workflow."""

    def test_complete_deployment(self):
        """Test complete deployment process."""
        # Create server
        server = self._create_test_server()

        # Create deployment plan
        plan = self._create_deployment_plan()

        # Mock SSH and external calls
        with self.mock_ssh_connection(), \
             self.mock_external_api():

            # Run deployment
            plan_log = server.run_flight_plan(plan)

            # Wait for completion
            self._wait_for_plan_completion(plan_log)

        # Verify deployment succeeded
        self.assertEqual(plan_log.plan_status, 0)
```

## Common Test Utilities

### Base Test Class

```python
# tests/common.py
from odoo.tests.common import TransactionCase
from unittest.mock import patch, MagicMock


class TowerTestCase(TransactionCase):
    """Base test class for Tower tests."""

    @classmethod
    def setUpClass(cls):
        super().setUpClass()
        cls.env = cls.env(context=dict(
            cls.env.context,
            tracking_disable=True,  # Disable tracking for tests
            test_queue_job_no_delay=True,  # Run jobs immediately
        ))

    @classmethod
    def _create_test_server(cls, **kwargs):
        """Create a test server."""
        values = {
            'name': 'Test Server',
            'ip_v4_address': '192.168.1.100',
            'ssh_username': 'test',
            'ssh_port': 22,
            'ssh_auth_mode': 'p',
            'ssh_password': 'test123',
            'skip_host_key': True,
        }
        values.update(kwargs)
        return cls.env['cx.tower.server'].with_context(
            skip_ssh_settings_check=True
        ).create(values)

    def mock_ssh_connection(self):
        """Mock SSH connection."""
        mock_connection = MagicMock()
        mock_connection.command_executor.exec_command.return_value = (
            0, ['test output'], []
        )
        return patch(
            'odoo.addons.cetmix_tower_server.models.cx_tower_server.'
            'CxTowerServer._get_ssh_client',
            return_value=mock_connection
        )
```

## Running Tests

### Run All Tests

```bash
# Run all Tower tests
odoo-bin -c /etc/odoo/odoo.conf \
    --test-enable \
    --stop-after-init \
    --test-tags cetmix_tower_server
```

### Run Specific Test File

```bash
odoo-bin -c /etc/odoo/odoo.conf \
    --test-enable \
    --stop-after-init \
    --test-tags cetmix_tower_server.tests.test_server
```

### Run Specific Test Class

```bash
odoo-bin -c /etc/odoo/odoo.conf \
    --test-enable \
    --stop-after-init \
    --test-tags cetmix_tower_server.tests.test_server.TestCxTowerServer
```

### Run with Coverage

```bash
# Install coverage
pip install coverage

# Run with coverage
coverage run --source=cetmix_tower_server \
    odoo-bin -c /etc/odoo/odoo.conf \
    --test-enable \
    --stop-after-init \
    --test-tags cetmix_tower_server

# Generate report
coverage report
coverage html  # HTML report in htmlcov/
```

## Test Best Practices

### 1. Use Descriptive Names

```python
# Good
def test_server_creation_requires_ip_address(self):
    """Test that server creation fails without IP address."""

# Bad
def test_1(self):
    """Test."""
```

### 2. One Assertion Per Test (Guideline)

```python
# Prefer
def test_server_has_correct_name(self):
    self.assertEqual(self.server.name, 'Test Server')

def test_server_has_correct_ip(self):
    self.assertEqual(self.server.ip_v4_address, '192.168.1.100')

# Over
def test_server_properties(self):
    self.assertEqual(self.server.name, 'Test Server')
    self.assertEqual(self.server.ip_v4_address, '192.168.1.100')
    self.assertEqual(self.server.ssh_port, 22)
```

### 3. Use setUp and tearDown Appropriately

```python
def setUp(self):
    """Set up before each test."""
    super().setUp()
    self.test_server = self._create_test_server()

def tearDown(self):
    """Clean up after each test."""
    # Cleanup if needed
    super().tearDown()
```

### 4. Mock External Dependencies

```python
def test_api_call(self):
    """Test external API call."""
    with patch('requests.post') as mock_post:
        mock_post.return_value.status_code = 200
        mock_post.return_value.json.return_value = {'status': 'ok'}

        result = self.server.notify_external_system()

        self.assertTrue(result)
        mock_post.assert_called_once()
```

### 5. Test Error Cases

```python
def test_invalid_ssh_credentials(self):
    """Test that invalid credentials raise error."""
    server = self._create_test_server(
        ssh_password='wrong_password'
    )

    with self.assertRaises(ValidationError):
        server.test_ssh_connection(raise_on_error=True)
```

## Test Coverage Goals

- **Critical Code**: 90%+ coverage
- **Business Logic**: 80%+ coverage
- **Overall Module**: 70%+ coverage

Focus on meaningful coverage, not just numbers.

## CI/CD Testing

Tests run automatically on:
- Pull requests
- Merges to main branch
- Release branches

See `.github/workflows/` for CI configuration.

## Related Documentation

- [Coding Standards](01-coding-standards.md)
- [Git Workflow](03-git-workflow.md)
- [Development Workflows](README.md)

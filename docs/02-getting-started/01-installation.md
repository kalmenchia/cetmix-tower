---
title: "Installation Guide"
version: "1.0"
last_updated: "2025-11-16"
maintained_by: "E-Global SCM Development Team"
status: "Active"
audience: "System Administrators, Developers"
---

# Installation Guide

This guide covers installing Cetmix Tower in various environments and scenarios.

---

## Table of Contents

- [For System Administrators](#for-system-administrators)
  - [Prerequisites](#prerequisites)
  - [System Requirements](#system-requirements)
  - [Python Dependencies Installation](#python-dependencies-installation)
  - [Module Installation](#module-installation)
  - [Installation Verification](#installation-verification)
  - [Troubleshooting](#troubleshooting)
- [For Developers](#for-developers)
  - [Development Environment Setup](#development-environment-setup)
  - [Installing from Source](#installing-from-source)
  - [Database Initialization](#database-initialization)
  - [Running in Development Mode](#running-in-development-mode)
- [Installation Scenarios](#installation-scenarios)
  - [Minimal Installation](#minimal-installation-core-only)
  - [Full Installation](#full-installation-with-all-features)
  - [Docker Installation](#docker-installation)

---

## For System Administrators

### Prerequisites

Before installing Cetmix Tower, ensure your system meets these requirements:

#### Software Requirements

| Component | Version | Purpose | Required |
|-----------|---------|---------|----------|
| **Odoo** | 17.0 | Base platform | Yes |
| **Python** | 3.10+ | Runtime environment | Yes |
| **PostgreSQL** | 12+ | Database server | Yes |
| **Git** | 2.x+ | Version control | Yes |
| **pip** | Latest | Python package manager | Yes |

#### Python Package Dependencies

**Core Dependencies (Required)**:
- `paramiko<4` - SSH connection library
- `tldextract` - Domain name extraction
- `dnspython` - DNS operations

**Optional Dependencies**:
- `pyyaml` - YAML export/import (for `cetmix_tower_yaml` module)
- `boto3` - AWS EC2 integration (for `cetmix_tower_aws` module)
- `ovh` - OVH cloud integration (for `cetmix_tower_ovh` module)

#### Odoo Module Dependencies

**Core Dependencies**:
- `mail` - Odoo mail module
- `rpc_helper` - RPC helper utilities

**Optional Dependencies**:
- `queue_job` - For asynchronous task execution (recommended)

### System Requirements

#### Hardware Requirements

**Minimum Configuration**:
- **CPU**: 2 cores
- **RAM**: 4 GB
- **Disk**: 10 GB free space
- **Network**: Stable internet connection

**Recommended Configuration**:
- **CPU**: 4+ cores
- **RAM**: 8+ GB
- **Disk**: 20+ GB free space (SSD preferred)
- **Network**: High-speed internet, low latency to managed servers

#### Network Requirements

- **Outbound SSH** (port 22 or custom): Access to managed servers
- **Outbound HTTPS** (port 443): For cloud API integrations (AWS, OVH)
- **Firewall**: Allow outbound connections to managed servers

#### Operating System

Cetmix Tower is tested on:
- Ubuntu 20.04 LTS, 22.04 LTS
- Debian 10, 11
- CentOS 7, 8
- Red Hat Enterprise Linux 8+
- macOS (development only)

### Python Dependencies Installation

#### Step 1: Update pip

```bash
python3 -m pip install --upgrade pip
```

#### Step 2: Install Core Dependencies

```bash
pip install "paramiko<4" tldextract dnspython
```

**Expected Output**:
```
Successfully installed paramiko-3.x.x tldextract-x.x.x dnspython-x.x.x
```

#### Step 3: Install Optional Dependencies

**For YAML Support**:
```bash
pip install pyyaml
```

**For AWS Integration**:
```bash
pip install boto3
```

**For OVH Integration**:
```bash
pip install ovh
```

**For Full Installation** (all dependencies):
```bash
pip install "paramiko<4" tldextract dnspython pyyaml boto3 ovh
```

#### Step 4: Verify Installation

```bash
python3 -c "import paramiko, tldextract, dns.resolver; print('Core dependencies installed successfully')"
```

**Troubleshooting Dependencies**:
- If `paramiko` fails: Install `libffi-dev` and `python3-dev` packages
- If `dnspython` fails: Try `python3-dnspython` system package
- For permission errors: Use `pip install --user` or virtual environment

### Module Installation

#### Method 1: Via Odoo Apps Menu (Recommended)

1. **Login to Odoo**
   - Open your Odoo instance in a web browser
   - Login with administrator credentials

2. **Navigate to Apps**
   - Click on the **Apps** menu in the top navigation
   - Remove the "Apps" filter (click the X on the search bar)

3. **Update Apps List**
   - Click **Update Apps List** button
   - Confirm the update dialog

4. **Search for Cetmix Tower**
   - In the search bar, type: `cetmix tower`
   - You should see multiple Cetmix Tower modules

5. **Install Modules**

   **For Standard Installation**:
   - Find **Cetmix Tower** (the main module)
   - Click **Install** button
   - Wait for installation to complete (this will install dependencies)

   **For Minimal Installation**:
   - Install **Cetmix Tower Server** only
   - Skip the main bundle

   **For Full Installation**:
   - Install **Cetmix Tower** (main module)
   - Install **Cetmix Tower AWS** (if needed)
   - Install **Cetmix Tower OVH** (if needed)
   - Install **Cetmix Tower YAML** (if needed)

6. **Verify Installation**
   - After installation, a new **Tower** menu should appear in the top navigation
   - Click Tower → Servers to verify access

#### Method 2: Manual Installation via Command Line

1. **Clone or Download Modules**

   ```bash
   cd /path/to/odoo/addons
   git clone https://github.com/cetmix/cetmix-tower.git
   ```

2. **Update Addons Path** (if needed)

   Edit your Odoo configuration file (`odoo.conf`):
   ```ini
   [options]
   addons_path = /usr/lib/python3/dist-packages/odoo/addons,/path/to/cetmix-tower
   ```

3. **Restart Odoo**

   ```bash
   sudo systemctl restart odoo
   # OR
   sudo service odoo restart
   ```

4. **Update Apps List**
   - Login to Odoo as administrator
   - Go to Apps → Update Apps List

5. **Install via Command Line** (alternative)

   ```bash
   odoo-bin -c /etc/odoo/odoo.conf -d your_database -i cetmix_tower --stop-after-init
   ```

   Or for minimal installation:
   ```bash
   odoo-bin -c /etc/odoo/odoo.conf -d your_database -i cetmix_tower_server --stop-after-init
   ```

#### Method 3: Using OCA Module Tools

If you use OCA's `odoo-module-migrator` or similar tools:

```bash
# Install using pip from git
pip install git+https://github.com/cetmix/cetmix-tower.git@17.0
```

### Installation Verification

#### Step 1: Check Module Installation

1. Go to **Settings → Apps**
2. Remove the "Apps" filter
3. Search for "cetmix tower"
4. Verify the following modules are installed:

**Standard Installation**:
- ✓ Cetmix Tower
- ✓ Cetmix Tower Server
- ✓ Cetmix Tower Server Queue
- ✓ Cetmix Tower Server Notify Backend
- ✓ Cetmix Tower Git
- ✓ Cetmix Tower Webhook

**Check installation status**: Each module should show "Installed" badge

#### Step 2: Verify Menu Access

1. Check that the **Tower** menu appears in the top navigation
2. Click Tower and verify these sub-menus are accessible:
   - Servers
   - Commands
   - Flight Plans
   - Files
   - Variables
   - Keys
   - Scheduled Tasks
   - Configuration

#### Step 3: Check User Access Rights

1. Go to **Settings → Users & Companies → Users**
2. Select your user
3. Scroll to **Access Rights**
4. Find **Cetmix Tower** section
5. Verify you have appropriate access level (Root for administrators)

#### Step 4: Test Basic Functionality

1. Navigate to **Tower → Servers**
2. Click **Create** button
3. If the form loads without errors, installation is successful
4. Cancel and don't save

#### Step 5: Check System Logs

```bash
# Check Odoo logs for errors
tail -f /var/log/odoo/odoo-server.log | grep -i "cetmix\|tower"
```

**Look for**:
- Module installation messages
- No error messages
- Successful model loading

### Troubleshooting

#### Issue: Module Not Found After Installation

**Symptom**: Cetmix Tower modules don't appear in Apps menu

**Solution**:
1. Verify addons path in `odoo.conf`:
   ```bash
   grep addons_path /etc/odoo/odoo.conf
   ```
2. Ensure cetmix-tower directory is in the addons path
3. Restart Odoo service
4. Update Apps List again

#### Issue: Import Error for paramiko

**Symptom**:
```
ImportError: No module named 'paramiko'
```

**Solution**:
```bash
# Install for the correct Python version
python3 -m pip install "paramiko<4"

# If using system packages
sudo apt-get install python3-paramiko

# Verify installation
python3 -c "import paramiko; print(paramiko.__version__)"
```

#### Issue: Database Migration Errors

**Symptom**: Errors during module installation or upgrade

**Solution**:
1. Backup your database first
2. Check PostgreSQL logs: `/var/log/postgresql/postgresql-*.log`
3. Try upgrading with debug mode:
   ```bash
   odoo-bin -c /etc/odoo/odoo.conf -d your_database -u cetmix_tower --log-level=debug
   ```
4. Review error messages and fix data issues

#### Issue: Access Denied Errors

**Symptom**: Cannot access Tower menu or features

**Solution**:
1. Go to Settings → Users & Companies → Users
2. Edit your user
3. Under Access Rights → Cetmix Tower, select appropriate level:
   - **Root** for full access
   - **Manager** for standard use
   - **User** for basic operations
4. Save and refresh browser

#### Issue: SSH Connection Errors

**Symptom**: Cannot connect to servers via SSH

**Solution**:
1. Verify paramiko is installed: `python3 -c "import paramiko"`
2. Check firewall rules allow outbound SSH (port 22)
3. Test SSH manually: `ssh user@server_ip`
4. Verify Odoo user has network access
5. Check server SSH configuration allows the connection

#### Issue: Missing Dependencies After Installation

**Symptom**:
```
ModuleNotFoundError: No module named 'tldextract'
```

**Solution**:
```bash
# Install missing dependency
pip install tldextract

# Restart Odoo
sudo systemctl restart odoo
```

#### Issue: Performance Issues After Installation

**Symptom**: Slow Odoo performance after installing Cetmix Tower

**Solution**:
1. Check PostgreSQL is properly tuned
2. Ensure sufficient RAM (min 4GB, recommended 8GB+)
3. Enable worker processes in odoo.conf:
   ```ini
   workers = 4
   max_cron_threads = 2
   ```
4. Configure database vacuum and analyze
5. Consider using queue_job for async operations

#### Issue: Queue Job Module Not Found

**Symptom**:
```
Module queue_job not found
```

**Solution**:
1. Install OCA queue_job module:
   ```bash
   # Clone OCA queue repository
   cd /path/to/addons
   git clone -b 17.0 https://github.com/OCA/queue.git
   ```
2. Restart Odoo
3. Install queue_job module from Apps menu
4. Upgrade cetmix_tower_server_queue module

---

## For Developers

### Development Environment Setup

#### Prerequisites

1. **Development Tools**:
   ```bash
   sudo apt-get install git python3-dev python3-pip python3-venv
   sudo apt-get install libpq-dev libxml2-dev libxslt1-dev libldap2-dev
   sudo apt-get install libsasl2-dev libffi-dev libjpeg-dev zlib1g-dev
   ```

2. **Node.js and npm** (for frontend development):
   ```bash
   curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
   sudo apt-get install -y nodejs
   ```

3. **PostgreSQL Development**:
   ```bash
   sudo apt-get install postgresql postgresql-contrib
   ```

#### Create Virtual Environment

```bash
# Create virtual environment
python3 -m venv odoo-venv

# Activate virtual environment
source odoo-venv/bin/activate

# Upgrade pip
pip install --upgrade pip setuptools wheel
```

#### Install Odoo from Source

```bash
# Clone Odoo repository
git clone --depth 1 --branch 17.0 https://github.com/odoo/odoo.git odoo17

# Install Odoo dependencies
cd odoo17
pip install -r requirements.txt
```

### Installing from Source

#### Step 1: Clone Cetmix Tower Repository

```bash
cd /path/to/your/projects
git clone https://github.com/cetmix/cetmix-tower.git
cd cetmix-tower
git checkout 17.0
```

#### Step 2: Install Python Dependencies

```bash
# Activate virtual environment
source /path/to/odoo-venv/bin/activate

# Install core dependencies
pip install "paramiko<4" tldextract dnspython

# Install optional dependencies for development
pip install pyyaml boto3 ovh

# Install development tools
pip install black isort pylint flake8 pre-commit
```

#### Step 3: Install Pre-commit Hooks (Optional)

```bash
cd /path/to/cetmix-tower
pre-commit install
```

#### Step 4: Configure Odoo for Development

Create `odoo-dev.conf`:

```ini
[options]
addons_path = /path/to/odoo17/addons,/path/to/cetmix-tower
admin_passwd = admin
db_host = localhost
db_port = 5432
db_user = odoo
db_password = odoo
http_port = 8069
logfile = /tmp/odoo-dev.log
log_level = debug
workers = 0
```

### Database Initialization

#### Create PostgreSQL User and Database

```bash
# Create PostgreSQL user
sudo -u postgres createuser -s odoo

# Set password for odoo user
sudo -u postgres psql -c "ALTER USER odoo WITH PASSWORD 'odoo';"

# Create development database
sudo -u postgres createdb -O odoo cetmix_tower_dev
```

#### Initialize Database with Modules

```bash
# Activate virtual environment
source /path/to/odoo-venv/bin/activate

# Initialize database with Cetmix Tower
cd /path/to/odoo17
./odoo-bin -c /path/to/odoo-dev.conf -d cetmix_tower_dev -i cetmix_tower --stop-after-init

# Or for minimal installation
./odoo-bin -c /path/to/odoo-dev.conf -d cetmix_tower_dev -i cetmix_tower_server --stop-after-init
```

### Running in Development Mode

#### Start Odoo in Development Mode

```bash
# Activate virtual environment
source /path/to/odoo-venv/bin/activate

# Start Odoo with dev mode
cd /path/to/odoo17
./odoo-bin -c /path/to/odoo-dev.conf -d cetmix_tower_dev --dev=all
```

**Development Mode Options**:
- `--dev=all` - Enable all development features
- `--dev=reload` - Auto-reload on code changes
- `--dev=qweb` - QWeb template debugging
- `--dev=xml` - XML debugging

#### Access Development Instance

- **URL**: http://localhost:8069
- **Database**: cetmix_tower_dev
- **Email**: admin
- **Password**: admin (or what you set in odoo.conf)

#### Running Tests

```bash
# Run all tests for Cetmix Tower modules
./odoo-bin -c /path/to/odoo-dev.conf -d cetmix_tower_test -i cetmix_tower --test-enable --stop-after-init --log-level=test

# Run specific module tests
./odoo-bin -c /path/to/odoo-dev.conf -d cetmix_tower_test -u cetmix_tower_server --test-enable --stop-after-init --log-level=test
```

#### Enable Debug Mode in Browser

1. Navigate to http://localhost:8069
2. Login as administrator
3. Add `?debug=1` to the URL
4. Or activate from developer menu

#### Live Reload Setup

For automatic browser reload on code changes:

```bash
# Install watchdog
pip install watchdog

# Run Odoo with auto-reload
./odoo-bin -c /path/to/odoo-dev.conf -d cetmix_tower_dev --dev=reload
```

---

## Installation Scenarios

### Minimal Installation (Core Only)

**Use Case**: Testing, small deployments, or basic server management

**Modules to Install**:
1. `cetmix_tower_server` - Core functionality

**Installation Command**:
```bash
odoo-bin -c /etc/odoo/odoo.conf -d your_database -i cetmix_tower_server --stop-after-init
```

**Python Dependencies**:
```bash
pip install "paramiko<4" tldextract dnspython
```

**Features Included**:
- Server management
- SSH command execution
- File management
- Variables and secrets
- Basic logging

**Features NOT Included**:
- Git integration
- Webhook support
- YAML export/import
- Cloud provider integrations
- Asynchronous task execution

### Full Installation (With All Features)

**Use Case**: Production deployments, multi-cloud environments, comprehensive automation

**Modules to Install**:
1. `cetmix_tower` - Main application bundle (includes dependencies)
2. `cetmix_tower_aws` - AWS EC2 integration
3. `cetmix_tower_ovh` - OVH cloud integration
4. `cetmix_tower_yaml` - YAML export/import

**Installation Commands**:
```bash
# Install all Python dependencies
pip install "paramiko<4" tldextract dnspython pyyaml boto3 ovh

# Install all modules
odoo-bin -c /etc/odoo/odoo.conf -d your_database -i cetmix_tower,cetmix_tower_aws,cetmix_tower_ovh,cetmix_tower_yaml --stop-after-init
```

**Features Included**:
- All minimal installation features
- Git repository management
- Webhook integrations
- AWS EC2 instance management
- OVH cloud management
- YAML configuration export/import
- Asynchronous task execution
- Backend notifications

### Docker Installation

**Use Case**: Containerized deployments, quick testing, isolated environments

#### Docker Compose Setup

Create `docker-compose.yml`:

```yaml
version: '3.8'

services:
  db:
    image: postgres:15
    environment:
      - POSTGRES_USER=odoo
      - POSTGRES_PASSWORD=odoo
      - POSTGRES_DB=postgres
    volumes:
      - odoo-db-data:/var/lib/postgresql/data
    networks:
      - odoo-network

  odoo:
    image: odoo:17.0
    depends_on:
      - db
    ports:
      - "8069:8069"
    environment:
      - HOST=db
      - USER=odoo
      - PASSWORD=odoo
    volumes:
      - odoo-data:/var/lib/odoo
      - ./cetmix-tower:/mnt/extra-addons/cetmix-tower
      - ./odoo.conf:/etc/odoo/odoo.conf
    networks:
      - odoo-network
    command: --dev=all

volumes:
  odoo-db-data:
  odoo-data:

networks:
  odoo-network:
    driver: bridge
```

Create `odoo.conf`:

```ini
[options]
addons_path = /usr/lib/python3/dist-packages/odoo/addons,/mnt/extra-addons/cetmix-tower
admin_passwd = admin
db_host = db
db_port = 5432
db_user = odoo
db_password = odoo
```

#### Build and Run

```bash
# Clone Cetmix Tower
git clone https://github.com/cetmix/cetmix-tower.git

# Install Python dependencies in Docker
docker-compose run --rm odoo pip install "paramiko<4" tldextract dnspython pyyaml

# Start services
docker-compose up -d

# Check logs
docker-compose logs -f odoo

# Initialize database
docker-compose run --rm odoo odoo -i cetmix_tower --stop-after-init
```

#### Access Containerized Instance

- **URL**: http://localhost:8069
- **Database**: Select or create database
- **Email**: admin
- **Password**: admin

#### Custom Dockerfile (Advanced)

Create `Dockerfile`:

```dockerfile
FROM odoo:17.0

USER root

# Install Python dependencies
RUN pip3 install --no-cache-dir \
    "paramiko<4" \
    tldextract \
    dnspython \
    pyyaml \
    boto3 \
    ovh

# Copy Cetmix Tower modules
COPY ./cetmix-tower /mnt/extra-addons/cetmix-tower

# Set permissions
RUN chown -R odoo:odoo /mnt/extra-addons/cetmix-tower

USER odoo
```

Build and run:

```bash
docker build -t cetmix-tower:17.0 .
docker run -d -p 8069:8069 --name cetmix-tower-instance cetmix-tower:17.0
```

---

## Post-Installation Steps

After successful installation:

1. **Configure Settings** - See [Configuration Guide](02-configuration.md)
2. **Set Up Users** - Configure access rights for team members
3. **Add First Server** - See [First Steps](03-first-steps.md)
4. **Test Connectivity** - Verify SSH connections work
5. **Create Test Command** - Run a simple command to verify functionality

---

## Upgrade Guide

### Upgrading Existing Installation

```bash
# Backup database first
pg_dump -U odoo your_database > backup_before_upgrade.sql

# Update module code
cd /path/to/cetmix-tower
git pull origin 17.0

# Restart Odoo
sudo systemctl restart odoo

# Upgrade modules
odoo-bin -c /etc/odoo/odoo.conf -d your_database -u cetmix_tower --stop-after-init
```

### Migration from Older Versions

Migration between major versions requires careful planning. Contact Cetmix support for migration assistance.

---

## Next Steps

- [Configure Cetmix Tower →](02-configuration.md)
- [Take Your First Steps →](03-first-steps.md)
- [Return to Getting Started Overview](README.md)

---

**Need Help?**
- Check [Troubleshooting](#troubleshooting) section above
- Visit [Issues & Bugs](../12-issues-bugs/README.md)
- Review [Technical Guides](../07-technical-guides/README.md)

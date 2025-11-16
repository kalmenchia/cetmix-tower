---
title: Comprehensive Deployment Guide
description: Complete guide to deploying Cetmix Tower in all environments
author: Cetmix
date: 2025-11-16
version: 1.0
module: cetmix_tower_server
category: deployment
tags: [deployment, docker, cloud, aws, ovh, production, staging]
---

# Comprehensive Deployment Guide

This guide covers deploying Cetmix Tower across different environments and platforms, from development to production.

## Table of Contents

- [Pre-Deployment Planning](#pre-deployment-planning)
- [Development Deployment](#development-deployment)
- [Staging Environment](#staging-environment)
- [Production Deployment](#production-deployment)
- [Docker Deployment](#docker-deployment)
- [Cloud Deployments](#cloud-deployments)
- [Configuration Management](#configuration-management)
- [Database Management](#database-management)
- [Backup and Restore](#backup-and-restore)
- [Version Upgrades](#version-upgrades)
- [Troubleshooting](#troubleshooting)

## Pre-Deployment Planning

### Infrastructure Requirements

#### Minimum Requirements (Development)

- **CPU**: 2 cores
- **RAM**: 4 GB
- **Disk**: 20 GB SSD
- **OS**: Ubuntu 20.04+ / Debian 11+ / RHEL 8+
- **Python**: 3.10+
- **PostgreSQL**: 12+

#### Recommended (Production)

- **CPU**: 4+ cores
- **RAM**: 8+ GB
- **Disk**: 50+ GB SSD
- **OS**: Ubuntu 22.04 LTS / Debian 12
- **Python**: 3.11
- **PostgreSQL**: 15+
- **Load Balancer**: Nginx/HAProxy
- **SSL**: Let's Encrypt / Commercial certificate

#### Network Requirements

- **Ports**:
  - 8069 (Odoo HTTP)
  - 8072 (Odoo Longpolling)
  - 22 (SSH to managed servers)
  - 443 (HTTPS via reverse proxy)
  - 5432 (PostgreSQL - internal only)

- **Firewall Rules**:
  - Allow outbound SSH (port 22) to managed servers
  - Allow HTTPS (443) from users
  - Restrict PostgreSQL (5432) to localhost
  - Allow HTTP (8069) from reverse proxy only

### Planning Checklist

- [ ] Define deployment environment (dev/staging/prod)
- [ ] Choose deployment method (Docker/traditional)
- [ ] Plan database strategy
- [ ] Configure network and firewall
- [ ] Obtain SSL certificates
- [ ] Plan backup strategy
- [ ] Define monitoring requirements
- [ ] Document architecture

## Development Deployment

### Local Development Setup

#### 1. Install Dependencies (Ubuntu/Debian)

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install PostgreSQL
sudo apt install postgresql postgresql-client -y

# Install Python and pip
sudo apt install python3 python3-pip python3-dev -y

# Install system dependencies
sudo apt install libxml2-dev libxslt1-dev libldap2-dev \
  libsasl2-dev libtiff5-dev libjpeg8-dev libopenjp2-7-dev \
  zlib1g-dev libfreetype6-dev liblcms2-dev libwebp-dev \
  libharfbuzz-dev libfribidi-dev libxcb1-dev libpq-dev \
  build-essential git wget nodejs npm -y

# Install wkhtmltopdf
wget https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6.1-2/wkhtmltox_0.12.6.1-2.jammy_amd64.deb
sudo dpkg -i wkhtmltox_0.12.6.1-2.jammy_amd64.deb
sudo apt-get install -f -y
```

#### 2. Install Odoo 17.0

```bash
# Create Odoo user
sudo useradd -m -U -r -d /opt/odoo -s /bin/bash odoo

# Clone Odoo
sudo su - odoo
git clone https://github.com/odoo/odoo.git --depth 1 --branch 17.0 /opt/odoo/odoo

# Install Python dependencies
cd /opt/odoo/odoo
pip3 install -r requirements.txt
```

#### 3. Install Tower Modules

```bash
# Clone Tower repository
cd /opt/odoo
git clone https://github.com/cetmix/cetmix-tower.git

# Install Tower dependencies
pip3 install -r cetmix-tower/requirements.txt
```

#### 4. Configure PostgreSQL

```bash
# Create database user
sudo su - postgres
createuser -s odoo

# Set password for odoo user
psql
ALTER USER odoo WITH PASSWORD 'secure_password';
\q
exit
```

#### 5. Create Odoo Configuration

```bash
# Create config directory
sudo mkdir -p /etc/odoo
sudo chown odoo:odoo /etc/odoo

# Create configuration file
sudo nano /etc/odoo/odoo.conf
```

```ini
[options]
# Paths
addons_path = /opt/odoo/odoo/addons,/opt/odoo/cetmix-tower
data_dir = /var/lib/odoo

# Database
db_host = localhost
db_port = 5432
db_user = odoo
db_password = secure_password

# Logging
logfile = /var/log/odoo/odoo.log
log_level = info

# Development settings
dev_mode = reload,qweb,werkzeug,xml

# Server
http_port = 8069
workers = 0  # 0 for development
```

#### 6. Start Odoo

```bash
# Create log directory
sudo mkdir -p /var/log/odoo
sudo chown odoo:odoo /var/log/odoo

# Start Odoo
sudo su - odoo
cd /opt/odoo/odoo
python3 odoo-bin -c /etc/odoo/odoo.conf -d dev_database -i cetmix_tower_server
```

Access Odoo at http://localhost:8069

## Staging Environment

### Staging Setup

Staging mirrors production but uses separate infrastructure:

#### Key Differences from Production

- Smaller server resources
- Separate database
- Test data instead of production data
- More verbose logging
- No customer data

#### Configuration

```ini
# /etc/odoo/staging.conf
[options]
addons_path = /opt/odoo/odoo/addons,/opt/odoo/cetmix-tower,/opt/odoo/custom_modules
data_dir = /var/lib/odoo/staging

db_host = localhost
db_name = staging_database
db_user = odoo
db_password = staging_secure_password

logfile = /var/log/odoo/staging.log
log_level = info

http_port = 8069
workers = 2
max_cron_threads = 1

# Enable developer mode
dev_mode = reload,qweb
```

## Production Deployment

### Production Architecture

```
┌─────────────────┐
│  Load Balancer  │ (Nginx/HAProxy)
│   (SSL/TLS)     │
└────────┬────────┘
         │
    ┌────┴─────┐
    │          │
┌───▼────┐ ┌──▼─────┐
│ Odoo 1 │ │ Odoo 2 │ (Workers)
└───┬────┘ └──┬─────┘
    │         │
    └────┬────┘
         │
  ┌──────▼────────┐
  │  PostgreSQL   │
  │   (Primary)   │
  └───────────────┘
```

### 1. PostgreSQL Setup (Production)

```bash
# Install PostgreSQL 15
sudo apt install postgresql-15 postgresql-contrib-15 -y

# Configure PostgreSQL
sudo nano /etc/postgresql/15/main/postgresql.conf
```

```ini
# PostgreSQL configuration for production
max_connections = 200
shared_buffers = 2GB
effective_cache_size = 6GB
maintenance_work_mem = 512MB
checkpoint_completion_target = 0.9
wal_buffers = 16MB
default_statistics_target = 100
random_page_cost = 1.1
effective_io_concurrency = 200
work_mem = 10MB
min_wal_size = 1GB
max_wal_size = 4GB
```

```bash
# Restart PostgreSQL
sudo systemctl restart postgresql

# Create production database
sudo su - postgres
createdb production_database
createuser odoo_prod
psql
ALTER USER odoo_prod WITH PASSWORD 'very_secure_password';
GRANT ALL PRIVILEGES ON DATABASE production_database TO odoo_prod;
\q
exit
```

### 2. Odoo Production Configuration

```ini
# /etc/odoo/production.conf
[options]
# Paths
addons_path = /opt/odoo/odoo/addons,/opt/odoo/cetmix-tower,/opt/odoo/enterprise,/opt/odoo/custom_modules
data_dir = /var/lib/odoo/production

# Database
db_host = localhost
db_port = 5432
db_user = odoo_prod
db_password = very_secure_password
db_name = production_database
db_maxconn = 64
db_template = template0

# Logging
logfile = /var/log/odoo/production.log
log_level = warn
log_handler = :WARNING,werkzeug:WARNING

# Server
http_port = 8069
longpolling_port = 8072
proxy_mode = True

# Workers (adjust based on CPU cores)
workers = 4
max_cron_threads = 2

# Limits
limit_memory_hard = 2684354560  # 2.5 GB
limit_memory_soft = 2147483648  # 2 GB
limit_request = 8192
limit_time_cpu = 600
limit_time_real = 1200
limit_time_real_cron = 300

# Security
admin_passwd = very_secure_admin_password
list_db = False
```

### 3. Systemd Service

```bash
# Create systemd service
sudo nano /etc/systemd/system/odoo.service
```

```ini
[Unit]
Description=Odoo 17 Server
Documentation=https://www.odoo.com
After=network.target postgresql.service

[Service]
Type=simple
User=odoo
Group=odoo
ExecStart=/opt/odoo/odoo/odoo-bin -c /etc/odoo/production.conf
StandardOutput=journal+console
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
```

```bash
# Enable and start service
sudo systemctl daemon-reload
sudo systemctl enable odoo
sudo systemctl start odoo
sudo systemctl status odoo
```

### 4. Nginx Reverse Proxy with SSL

```bash
# Install Nginx
sudo apt install nginx certbot python3-certbot-nginx -y

# Configure Nginx
sudo nano /etc/nginx/sites-available/tower.example.com
```

```nginx
# Upstream Odoo servers
upstream odoo {
    server 127.0.0.1:8069;
}

upstream odoochat {
    server 127.0.0.1:8072;
}

# HTTP redirect to HTTPS
server {
    listen 80;
    server_name tower.example.com;
    return 301 https://$server_name$request_uri;
}

# HTTPS server
server {
    listen 443 ssl http2;
    server_name tower.example.com;

    # SSL configuration
    ssl_certificate /etc/letsencrypt/live/tower.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/tower.example.com/privkey.pem;
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:10m;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;

    # Security headers
    add_header Strict-Transport-Security "max-age=31536000" always;
    add_header X-Content-Type-Options nosniff;
    add_header X-Frame-Options SAMEORIGIN;
    add_header X-XSS-Protection "1; mode=block";

    # Logging
    access_log /var/log/nginx/tower_access.log;
    error_log /var/log/nginx/tower_error.log;

    # Proxy settings
    proxy_read_timeout 720s;
    proxy_connect_timeout 720s;
    proxy_send_timeout 720s;
    proxy_set_header X-Forwarded-Host $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Real-IP $remote_addr;

    # Handle longpolling
    location /longpolling {
        proxy_pass http://odoochat;
    }

    # Handle web requests
    location / {
        proxy_redirect off;
        proxy_pass http://odoo;

        # Increase buffer size
        proxy_buffer_size 128k;
        proxy_buffers 4 256k;
        proxy_busy_buffers_size 256k;
    }

    # Cache static files
    location ~* /web/static/ {
        proxy_cache_valid 200 60m;
        proxy_buffering on;
        expires 864000;
        proxy_pass http://odoo;
    }

    # Upload size limit
    client_max_body_size 100M;
}
```

```bash
# Enable site
sudo ln -s /etc/nginx/sites-available/tower.example.com /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx

# Obtain SSL certificate
sudo certbot --nginx -d tower.example.com
```

## Docker Deployment

### Docker Compose Setup

```yaml
# docker-compose.yml
version: '3.8'

services:
  web:
    image: odoo:17.0
    container_name: tower_web
    depends_on:
      - db
    ports:
      - "8069:8069"
      - "8072:8072"
    volumes:
      # Custom addons
      - ./cetmix-tower:/mnt/extra-addons/cetmix-tower:ro
      - ./custom_modules:/mnt/extra-addons/custom:ro
      # Data persistence
      - odoo-data:/var/lib/odoo
      # Configuration
      - ./config/odoo.conf:/etc/odoo/odoo.conf:ro
      # Logs
      - ./logs:/var/log/odoo
    environment:
      - HOST=db
      - USER=odoo
      - PASSWORD=odoo_password
    command: -- --config=/etc/odoo/odoo.conf

  db:
    image: postgres:15-alpine
    container_name: tower_db
    environment:
      - POSTGRES_DB=postgres
      - POSTGRES_USER=odoo
      - POSTGRES_PASSWORD=odoo_password
      - PGDATA=/var/lib/postgresql/data/pgdata
    volumes:
      - db-data:/var/lib/postgresql/data/pgdata
    ports:
      - "5432:5432"  # For backup purposes only

volumes:
  odoo-data:
  db-data:

networks:
  default:
    name: tower_network
```

```ini
# config/odoo.conf
[options]
addons_path = /mnt/extra-addons/cetmix-tower,/mnt/extra-addons/custom
admin_passwd = secure_admin_password
db_host = db
db_port = 5432
db_user = odoo
db_password = odoo_password
logfile = /var/log/odoo/odoo.log
log_level = info
workers = 2
proxy_mode = True
```

```bash
# Start services
docker-compose up -d

# View logs
docker-compose logs -f web

# Stop services
docker-compose down

# Backup
docker-compose exec db pg_dump -U odoo postgres | gzip > backup.sql.gz
```

## Cloud Deployments

### AWS Deployment

#### 1. EC2 Instance Setup

```bash
# Launch EC2 instance
# - Instance type: t3.medium (for production: t3.large or better)
# - AMI: Ubuntu 22.04 LTS
# - Storage: 50 GB gp3
# - Security Group:
#   - SSH (22) from your IP
#   - HTTPS (443) from anywhere
#   - HTTP (80) from anywhere (for Let's Encrypt)
```

#### 2. RDS PostgreSQL Setup

```bash
# Create RDS PostgreSQL instance
# - Engine: PostgreSQL 15
# - Instance class: db.t3.medium
# - Storage: 100 GB gp3
# - Multi-AZ: Yes (for production)
# - Backup retention: 7 days
```

#### 3. Deploy Tower

```bash
# SSH to EC2
ssh -i keypair.pem ubuntu@ec2-xxx.amazonaws.com

# Follow production deployment steps
# Update database connection to RDS endpoint
```

### OVH Deployment

For OVH cloud integration, use `cetmix_tower_ovh` module:

```bash
# Install OVH module
pip3 install ovh

# Configure Tower
# Add OVH API credentials in Odoo
# Use OVH-specific commands for server management
```

## Configuration Management

### Environment Variables

```bash
# .env file (for Docker)
ODOO_VERSION=17.0
DB_HOST=db
DB_PORT=5432
DB_USER=odoo
DB_PASSWORD=secure_password
ADMIN_PASSWORD=admin_secure_password
WORKERS=4
```

### Secrets Management

```bash
# Use environment variables or vault
export DB_PASSWORD=$(cat /run/secrets/db_password)
export ADMIN_PASSWD=$(cat /run/secrets/admin_password)
```

## Database Management

### Database Initialization

```bash
# Initialize database with Tower modules
odoo-bin -c /etc/odoo/odoo.conf -d production_database \
  -i cetmix_tower_server,cetmix_tower_yaml,cetmix_tower_git \
  --stop-after-init
```

### Database Upgrade

```bash
# Upgrade modules
odoo-bin -c /etc/odoo/odoo.conf -d production_database \
  -u cetmix_tower_server \
  --stop-after-init
```

## Backup and Restore

### Automated Backup Script

```bash
#!/bin/bash
# /opt/scripts/backup_tower.sh

BACKUP_DIR="/backups/tower"
DATE=$(date +%Y%m%d_%H%M%S)
DB_NAME="production_database"

# Create backup directory
mkdir -p $BACKUP_DIR

# Database backup
pg_dump -U odoo $DB_NAME | gzip > $BACKUP_DIR/db_$DATE.sql.gz

# Filestore backup
rsync -av /var/lib/odoo/filestore/$DB_NAME/ $BACKUP_DIR/filestore_$DATE/

# Cleanup old backups (keep 30 days)
find $BACKUP_DIR -name "db_*.sql.gz" -mtime +30 -delete
find $BACKUP_DIR -name "filestore_*" -mtime +30 -exec rm -rf {} \;

# Upload to S3 (optional)
#aws s3 sync $BACKUP_DIR s3://my-tower-backups/
```

```bash
# Make executable
chmod +x /opt/scripts/backup_tower.sh

# Add to crontab
crontab -e
# Daily backup at 2 AM
0 2 * * * /opt/scripts/backup_tower.sh
```

### Restore

```bash
# Restore database
gunzip < backup.sql.gz | psql -U odoo production_database

# Restore filestore
rsync -av filestore_backup/ /var/lib/odoo/filestore/production_database/

# Restart Odoo
sudo systemctl restart odoo
```

## Version Upgrades

See [Version Upgrades Guide](../13-potential-upgrade/01-odoo-version-upgrades.md) for detailed instructions.

## Troubleshooting

### Common Issues

#### 1. Odoo Won't Start

```bash
# Check logs
sudo journalctl -u odoo -f

# Check configuration
odoo-bin -c /etc/odoo/odoo.conf --test-enable

# Check PostgreSQL
sudo systemctl status postgresql
```

#### 2. Database Connection Issues

```bash
# Test PostgreSQL connection
psql -h localhost -U odoo -d production_database

# Check pg_hba.conf
sudo nano /etc/postgresql/15/main/pg_hba.conf
```

#### 3. Performance Issues

```bash
# Check worker configuration
# Recommended: workers = (CPU cores * 2) + 1

# Monitor resources
htop
sudo iotop
```

## Next Steps

- Set up [Monitoring](../10-security-compliance/)
- Review [Security Best Practices](../10-security-compliance/)
- Configure [Custom Extensions](../11-customization-extensions/)
- Plan [Version Upgrades](../13-potential-upgrade/)

---

**Remember**: Always test deployments in staging before production!

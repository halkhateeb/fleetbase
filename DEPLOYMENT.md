# Deployment Guide

This guide covers deploying Fleetbase in various environments, from development to production.

## Table of Contents

- [Deployment Options](#deployment-options)
- [Docker Deployment](#docker-deployment)
- [AWS Deployment](#aws-deployment)
- [Kubernetes Deployment](#kubernetes-deployment)
- [Manual Deployment](#manual-deployment)
- [Configuration](#configuration)
- [Security Checklist](#security-checklist)
- [Monitoring](#monitoring)
- [Backup and Recovery](#backup-and-recovery)
- [Troubleshooting](#troubleshooting)

## Deployment Options

Fleetbase can be deployed in multiple ways:

| Method | Use Case | Complexity | Scalability |
|--------|----------|------------|-------------|
| Docker Compose | Development, Small Production | Low | Limited |
| AWS (One-Click) | Production, Enterprise | Low | High |
| Kubernetes | Large-Scale Production | High | Very High |
| Manual | Custom Infrastructure | Medium | Variable |

## Docker Deployment

### Prerequisites

- Docker Engine 20.10+
- Docker Compose 2.0+
- At least 4GB RAM
- 20GB disk space

### Quick Start

```bash
# Clone the repository
git clone https://github.com/fleetbase/fleetbase.git
cd fleetbase

# Run installation script
./scripts/docker-install.sh

# Services will be available at:
# - Console: http://localhost:4200
# - API: http://localhost:8000
# - Socket: http://localhost:8001
```

### Custom Configuration

Create a `docker-compose.override.yml` file:

```yaml
version: "3.8"

services:
  application:
    environment:
      # Console Configuration
      CONSOLE_HOST: http://your-domain.com:4200
      
      # Database Configuration
      DB_CONNECTION: mysql
      DB_HOST: mysql
      DB_PORT: 3306
      DB_DATABASE: fleetbase
      DB_USERNAME: fleetbase
      DB_PASSWORD: your-secure-password
      
      # Redis Configuration
      REDIS_HOST: redis
      REDIS_PORT: 6379
      
      # Mail Configuration
      MAIL_MAILER: smtp
      MAIL_HOST: smtp.mailtrap.io
      MAIL_PORT: 2525
      MAIL_USERNAME: your-username
      MAIL_PASSWORD: your-password
      MAIL_ENCRYPTION: tls
      MAIL_FROM_ADDRESS: noreply@fleetbase.io
      MAIL_FROM_NAME: Fleetbase
      
      # AWS S3 Configuration
      FILESYSTEM_DRIVER: s3
      AWS_ACCESS_KEY_ID: your-access-key
      AWS_SECRET_ACCESS_KEY: your-secret-key
      AWS_DEFAULT_REGION: us-east-1
      AWS_BUCKET: your-bucket-name
      
      # External Services
      IPINFO_API_KEY: your-ipinfo-key
      GOOGLE_MAPS_API_KEY: your-google-maps-key
      GOOGLE_MAPS_LOCALE: us
      
      # Twilio (SMS)
      TWILIO_SID: your-twilio-sid
      TWILIO_TOKEN: your-twilio-token
      TWILIO_FROM: +1234567890
      
      # OSRM Routing
      OSRM_HOST: https://router.project-osrm.org
      
      # Application Settings
      APP_ENV: production
      APP_DEBUG: false
      APP_KEY: base64:your-generated-key
      
  socket:
    environment:
      # WebSocket Configuration
      # IMPORTANT: Update origins for production
      SOCKETCLUSTER_OPTIONS: '{"origins":"https://your-domain.com:*,wss://your-domain.com:*"}'
      
  mysql:
    environment:
      MYSQL_ROOT_PASSWORD: your-root-password
      MYSQL_DATABASE: fleetbase
      MYSQL_USER: fleetbase
      MYSQL_PASSWORD: your-secure-password
    volumes:
      - mysql-data:/var/lib/mysql
      
  redis:
    volumes:
      - redis-data:/data

volumes:
  mysql-data:
  redis-data:
```

### Production Docker Setup

1. **Generate Application Key:**

```bash
docker-compose exec application php artisan key:generate --show
```

Copy the generated key to `APP_KEY` in your override file.

2. **Run Migrations:**

```bash
docker-compose exec application php artisan migrate
```

3. **Configure Reverse Proxy (Nginx):**

Create `nginx.conf`:

```nginx
upstream api {
    server localhost:8000;
}

upstream console {
    server localhost:4200;
}

upstream socket {
    server localhost:8001;
}

server {
    listen 80;
    server_name your-domain.com;
    
    # Redirect to HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name your-domain.com;
    
    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;
    
    # Console
    location / {
        proxy_pass http://console;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
    
    # API
    location /api/ {
        proxy_pass http://api;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
    
    # WebSocket
    location /socket/ {
        proxy_pass http://socket;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

4. **SSL Certificate:**

```bash
# Using Let's Encrypt
sudo certbot --nginx -d your-domain.com
```

### Docker Management Commands

```bash
# Start services
docker-compose up -d

# Stop services
docker-compose down

# View logs
docker-compose logs -f

# Restart specific service
docker-compose restart application

# Update images
docker-compose pull
docker-compose up -d --build

# Access container shell
docker-compose exec application bash

# Database backup
docker-compose exec mysql mysqldump -u fleetbase -p fleetbase > backup.sql

# Database restore
docker-compose exec -T mysql mysql -u fleetbase -p fleetbase < backup.sql
```

## AWS Deployment

### One-Click Deployment

Deploy Fleetbase on AWS with pre-configured infrastructure:

[![Deploy to AWS](https://img.shields.io/badge/Deploy%20to%20AWS-FF9900?style=for-the-badge&logo=amazon-aws&logoColor=white)](https://console.fleetbase.io/aws-marketplace)

### What Gets Deployed

The one-click deployment creates:

- **Compute**: ECS Fargate cluster with auto-scaling
- **Database**: RDS MySQL 8.0 (Multi-AZ)
- **Cache**: ElastiCache Redis
- **Storage**: S3 bucket with CloudFront CDN
- **Networking**: VPC, subnets, NAT gateways, security groups
- **Load Balancer**: Application Load Balancer with SSL
- **Monitoring**: CloudWatch logs and dashboards
- **Queue**: SQS for background jobs

### Manual AWS Setup

If you prefer manual setup:

#### 1. Infrastructure Setup

```bash
# Clone infrastructure repository
git clone https://github.com/fleetbase/fleetbase-aws-infra.git
cd fleetbase-aws-infra

# Configure AWS credentials
aws configure

# Initialize Terraform
terraform init

# Plan infrastructure
terraform plan

# Apply infrastructure
terraform apply
```

#### 2. Application Deployment

```bash
# Build and push Docker images
./scripts/build-and-push.sh

# Update ECS service
aws ecs update-service \
  --cluster fleetbase-cluster \
  --service fleetbase-api \
  --force-new-deployment
```

#### 3. Configure Environment Variables

Update environment variables in ECS Task Definition:

```json
{
  "name": "application",
  "environment": [
    {"name": "APP_ENV", "value": "production"},
    {"name": "APP_KEY", "value": "base64:..."},
    {"name": "DB_HOST", "value": "fleetbase-db.xxx.rds.amazonaws.com"},
    {"name": "REDIS_HOST", "value": "fleetbase-cache.xxx.cache.amazonaws.com"},
    {"name": "AWS_BUCKET", "value": "fleetbase-storage"},
    {"name": "FILESYSTEM_DRIVER", "value": "s3"}
  ]
}
```

### AWS Best Practices

1. **Use Secrets Manager** for sensitive data
2. **Enable Multi-AZ** for RDS and ElastiCache
3. **Configure Auto Scaling** based on CPU/memory
4. **Set up CloudWatch Alarms** for monitoring
5. **Enable RDS automated backups**
6. **Use CloudFront** for static assets
7. **Configure WAF** for API protection

## Kubernetes Deployment

### Prerequisites

- Kubernetes cluster (1.20+)
- kubectl configured
- Helm 3.0+

### Helm Chart Installation

```bash
# Add Fleetbase Helm repository
helm repo add fleetbase https://charts.fleetbase.io
helm repo update

# Install Fleetbase
helm install fleetbase fleetbase/fleetbase \
  --set api.image.tag=latest \
  --set console.image.tag=latest \
  --set ingress.enabled=true \
  --set ingress.host=fleetbase.example.com \
  --set mysql.persistence.enabled=true \
  --set redis.persistence.enabled=true
```

### Custom values.yaml

```yaml
# values.yaml
replicaCount:
  api: 3
  console: 2
  socket: 2

image:
  api:
    repository: fleetbase/api
    tag: latest
  console:
    repository: fleetbase/console
    tag: latest
  socket:
    repository: fleetbase/socket
    tag: latest

env:
  APP_ENV: production
  APP_DEBUG: false
  
mysql:
  enabled: true
  auth:
    database: fleetbase
    username: fleetbase
    password: your-password
  primary:
    persistence:
      enabled: true
      size: 50Gi

redis:
  enabled: true
  auth:
    password: your-redis-password
  master:
    persistence:
      enabled: true
      size: 10Gi

ingress:
  enabled: true
  className: nginx
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
  hosts:
    - host: fleetbase.example.com
      paths:
        - path: /
          pathType: Prefix
          backend: console
        - path: /api
          pathType: Prefix
          backend: api
  tls:
    - secretName: fleetbase-tls
      hosts:
        - fleetbase.example.com

autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70

resources:
  api:
    limits:
      cpu: 1000m
      memory: 1Gi
    requests:
      cpu: 500m
      memory: 512Mi
  console:
    limits:
      cpu: 500m
      memory: 512Mi
    requests:
      cpu: 250m
      memory: 256Mi
```

### Deploy with Custom Values

```bash
helm install fleetbase fleetbase/fleetbase -f values.yaml
```

### Kubernetes Management

```bash
# Check deployment status
kubectl get pods -n fleetbase

# View logs
kubectl logs -f deployment/fleetbase-api -n fleetbase

# Scale deployment
kubectl scale deployment fleetbase-api --replicas=5 -n fleetbase

# Update deployment
helm upgrade fleetbase fleetbase/fleetbase -f values.yaml

# Rollback
helm rollback fleetbase

# Uninstall
helm uninstall fleetbase
```

## Manual Deployment

### Server Requirements

- **OS**: Ubuntu 20.04 LTS or later
- **CPU**: 4+ cores
- **RAM**: 8GB minimum, 16GB recommended
- **Storage**: 50GB+ SSD
- **PHP**: 8.1+
- **Node.js**: 18+
- **MySQL**: 8.0+
- **Redis**: 6.0+
- **Nginx/Apache**: Latest stable

### Installation Steps

#### 1. Install Dependencies

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install PHP
sudo apt install php8.1-fpm php8.1-cli php8.1-mysql php8.1-redis \
  php8.1-mbstring php8.1-xml php8.1-curl php8.1-zip php8.1-gd -y

# Install Composer
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer

# Install Node.js
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install nodejs -y

# Install MySQL
sudo apt install mysql-server -y

# Install Redis
sudo apt install redis-server -y

# Install Nginx
sudo apt install nginx -y
```

#### 2. Clone and Setup Application

```bash
# Clone repository
cd /var/www
sudo git clone https://github.com/fleetbase/fleetbase.git
sudo chown -R www-data:www-data fleetbase

# Setup API
cd /var/www/fleetbase/api
sudo -u www-data composer install --no-dev --optimize-autoloader
sudo -u www-data cp .env.example .env
sudo -u www-data php artisan key:generate
sudo -u www-data php artisan migrate --force

# Setup Console
cd /var/www/fleetbase/console
npm install
npm run build:prod

# Setup Socket Server
cd /var/www/fleetbase/socket
npm install
```

#### 3. Configure Nginx

```nginx
# /etc/nginx/sites-available/fleetbase
server {
    listen 80;
    server_name your-domain.com;
    
    root /var/www/fleetbase/console/dist;
    index index.html;
    
    # Console (SPA)
    location / {
        try_files $uri $uri/ /index.html;
    }
    
    # API
    location /api {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
    
    # WebSocket
    location /socket {
        proxy_pass http://127.0.0.1:8001;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

Enable the site:

```bash
sudo ln -s /etc/nginx/sites-available/fleetbase /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

#### 4. Setup Systemd Services

**API Service:**

```ini
# /etc/systemd/system/fleetbase-api.service
[Unit]
Description=Fleetbase API
After=network.target mysql.service redis.service

[Service]
Type=simple
User=www-data
WorkingDirectory=/var/www/fleetbase/api
ExecStart=/usr/bin/php artisan serve --host=127.0.0.1 --port=8000
Restart=always

[Install]
WantedBy=multi-user.target
```

**Socket Service:**

```ini
# /etc/systemd/system/fleetbase-socket.service
[Unit]
Description=Fleetbase Socket Server
After=network.target

[Service]
Type=simple
User=www-data
WorkingDirectory=/var/www/fleetbase/socket
ExecStart=/usr/bin/node server.js
Restart=always

[Install]
WantedBy=multi-user.target
```

**Queue Worker:**

```ini
# /etc/systemd/system/fleetbase-worker.service
[Unit]
Description=Fleetbase Queue Worker
After=network.target redis.service

[Service]
Type=simple
User=www-data
WorkingDirectory=/var/www/fleetbase/api
ExecStart=/usr/bin/php artisan queue:work --sleep=3 --tries=3
Restart=always

[Install]
WantedBy=multi-user.target
```

Enable and start services:

```bash
sudo systemctl daemon-reload
sudo systemctl enable fleetbase-api fleetbase-socket fleetbase-worker
sudo systemctl start fleetbase-api fleetbase-socket fleetbase-worker
```

## Configuration

### Environment Variables

Key environment variables to configure:

```bash
# Application
APP_ENV=production
APP_DEBUG=false
APP_KEY=base64:...

# Database
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=fleetbase
DB_USERNAME=fleetbase
DB_PASSWORD=secure_password

# Redis
REDIS_HOST=127.0.0.1
REDIS_PASSWORD=redis_password
REDIS_PORT=6379

# Mail
MAIL_MAILER=smtp
MAIL_HOST=smtp.mailgun.org
MAIL_PORT=587
MAIL_USERNAME=your_username
MAIL_PASSWORD=your_password

# AWS S3
FILESYSTEM_DRIVER=s3
AWS_ACCESS_KEY_ID=your_key
AWS_SECRET_ACCESS_KEY=your_secret
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=your_bucket

# Services
GOOGLE_MAPS_API_KEY=your_key
TWILIO_SID=your_sid
TWILIO_TOKEN=your_token
IPINFO_API_KEY=your_key

# Console
CONSOLE_HOST=https://your-domain.com
```

## Security Checklist

- [ ] Change all default passwords
- [ ] Generate strong APP_KEY
- [ ] Enable HTTPS/SSL
- [ ] Configure firewall rules
- [ ] Set up proper CORS origins
- [ ] Enable rate limiting
- [ ] Configure WebSocket origin restrictions
- [ ] Use secrets management for sensitive data
- [ ] Enable database encryption at rest
- [ ] Set up regular security updates
- [ ] Configure backup retention
- [ ] Enable audit logging
- [ ] Implement IP whitelisting where appropriate
- [ ] Set strong session timeouts
- [ ] Enable two-factor authentication
- [ ] Regular security audits

## Monitoring

### CloudWatch (AWS)

```bash
# Install CloudWatch agent
wget https://s3.amazonaws.com/amazoncloudwatch-agent/ubuntu/amd64/latest/amazon-cloudwatch-agent.deb
sudo dpkg -i amazon-cloudwatch-agent.deb

# Configure monitoring
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard
```

### Application Monitoring

```bash
# Install Laravel Telescope (development)
composer require laravel/telescope
php artisan telescope:install
php artisan migrate

# Access at https://your-domain.com/telescope
```

### Log Monitoring

```bash
# View application logs
tail -f /var/www/fleetbase/api/storage/logs/laravel.log

# View Nginx logs
tail -f /var/log/nginx/access.log
tail -f /var/log/nginx/error.log

# View systemd service logs
journalctl -u fleetbase-api -f
```

## Backup and Recovery

### Database Backup

```bash
# Manual backup
mysqldump -u fleetbase -p fleetbase > backup-$(date +%Y%m%d).sql

# Automated daily backup (crontab)
0 2 * * * /usr/bin/mysqldump -u fleetbase -p'password' fleetbase | gzip > /backups/fleetbase-$(date +\%Y\%m\%d).sql.gz
```

### File Backup

```bash
# Backup uploads and storage
tar -czf storage-backup-$(date +%Y%m%d).tar.gz /var/www/fleetbase/api/storage

# S3 sync (if using S3)
aws s3 sync s3://your-bucket /local/backup/path
```

### Recovery

```bash
# Restore database
mysql -u fleetbase -p fleetbase < backup-20240101.sql

# Restore files
tar -xzf storage-backup-20240101.tar.gz -C /var/www/fleetbase/api/
```

## Troubleshooting

### Common Issues

**Database Connection Failed:**
```bash
# Check MySQL is running
sudo systemctl status mysql

# Test connection
mysql -u fleetbase -p -h 127.0.0.1

# Check firewall
sudo ufw status
```

**Permission Denied:**
```bash
# Fix ownership
sudo chown -R www-data:www-data /var/www/fleetbase

# Fix permissions
sudo chmod -R 755 /var/www/fleetbase
sudo chmod -R 775 /var/www/fleetbase/api/storage
```

**Service Not Starting:**
```bash
# Check service status
sudo systemctl status fleetbase-api

# View logs
sudo journalctl -u fleetbase-api -n 50

# Restart service
sudo systemctl restart fleetbase-api
```

## Support

- **Documentation**: https://docs.fleetbase.io
- **GitHub Issues**: https://github.com/fleetbase/fleetbase/issues
- **Discord**: https://discord.gg/V7RVWRQ2Wm
- **Email**: support@fleetbase.io

# OTT Platform - DevOps Documentation

## Table of Contents
1. [Project Overview](#project-overview)
2. [Architecture](#architecture)
3. [Development Setup](#development-setup)
4. [Deployment](#deployment)
5. [Docker Configuration](#docker-configuration)
6. [Environment Variables](#environment-variables)
7. [Continuous Integration/Deployment](#continuous-integrationdeployment)
8. [Monitoring and Maintenance](#monitoring-and-maintenance)
9. [Security Considerations](#security-considerations)
10. [Backup and Recovery](#backup-and-recovery)

## Project Overview
A Node.js-based OTT (Over-The-Top) platform with features including user authentication, content streaming, and admin management. The application uses Express.js for the backend, EJS for templating, and MongoDB Atlas for the database.

## Architecture
```
├── Application (Node.js/Express)
│   └── Running on Port 3000
├── Nginx Reverse Proxy
│   └── Ports 80/443
└── MongoDB Atlas
    └── Cloud Database
```

## Development Setup

### Prerequisites
- Node.js (v18 or higher)
- Docker and Docker Compose
- Git

### Local Development
1. Clone the repository:
```bash
git clone https://github.com/zabhitak/OTT.git
cd OTT
```

2. Install dependencies:
```bash
npm install
```

3. Create `.env` file:
```env
MONGODB_URI=your_mongodb_connection_string
EMAIL_ID=your_email_service_account
PASSWORD=your_email_service_password
ADMIN_ID=admin_email
ADMIN_PASSWORD=admin_password
```

4. Start development server:
```bash
npm run start
```

## Deployment

### Server Requirements
- Ubuntu 20.04 LTS or higher
- 2GB RAM minimum
- Docker and Docker Compose installed

### Deployment Process

1. **Server Setup**
```bash
./deploy.sh setup
```
This will:
- Update system packages
- Install Docker and dependencies
- Configure firewall
- Setup fail2ban
- Configure swap space

2. **Application Deployment**
```bash
./deploy.sh deploy
```

3. **Performance Optimization**
```bash
./deploy.sh optimize
```

4. **Maintenance**
```bash
./deploy.sh maintenance
```

## Docker Configuration

### Docker Compose Structure
```yaml
services:
  app:
    build: .
    ports:
      - "3000:3000"
    env_file:
      - .env
    restart: unless-stopped

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    depends_on:
      - app
```

### Dockerfile
```dockerfile
FROM node:18-slim
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "index.js"]
```

## Environment Variables

Create a `.env` file with the following structure:
```env
MONGODB_URI=mongodb+srv://<username>:<password>@<cluster>.mongodb.net/<database>
EMAIL_ID=your_email_service_account
PASSWORD=your_email_app_password
ADMIN_ID=admin@example.com
ADMIN_PASSWORD=admin_secure_password
```

⚠️ Never commit the actual `.env` file to version control.

## Continuous Integration/Deployment

### Manual Deployment
The project includes a comprehensive deployment script (`deploy.sh`) with the following capabilities:

```bash
./deploy.sh [command]

Commands:
  setup       - Initial server setup
  deploy      - Deploy application
  optimize    - Apply performance optimizations
  maintenance - Run maintenance tasks
  backup      - Create backup
```

### Automated CI/CD (Future Implementation)
Recommended CI/CD pipeline using GitHub Actions:

```yaml
name: Deploy to Production
on:
  push:
    branches: [ main ]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Deploy to Server
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USERNAME }}
          key: ${{ secrets.SSH_KEY }}
          script: |
            cd /root/test_project3
            git pull
            docker-compose up -d --build
```

## Monitoring and Maintenance

### Regular Maintenance Tasks
```bash
# View application logs
docker-compose logs --tail=100 app

# Monitor system resources
htop

# Check disk usage
ncdu

# View nginx access logs
docker-compose logs nginx
```

### Automated Maintenance
- Daily cleanup of Docker resources
- Log rotation
- System updates
- Database backups

## Security Considerations

1. **Firewall Configuration**
   - Only ports 22, 80, and 443 should be open
   - Use UFW for firewall management

2. **MongoDB Atlas Security**
   - IP Whitelist
   - Strong passwords
   - Regular security audits

3. **Application Security**
   - Session management using MongoDB store
   - Rate limiting
   - HTTP Security headers

4. **Server Security**
   - Regular updates
   - fail2ban for SSH protection
   - SSL/TLS certificates

## Backup and Recovery

### Automated Backups
```bash
./deploy.sh backup
```

### Backup Strategy
1. Daily application data backups
2. Database backups (MongoDB Atlas)
3. Configuration backups
4. Docker volume backups

### Recovery Process
1. Restore MongoDB data
2. Deploy application code
3. Restore configuration files
4. Verify system functionality

## Troubleshooting

### Common Issues

1. **MongoDB Connection Issues**
   - Check IP whitelist in MongoDB Atlas
   - Verify connection string in .env
   - Check network connectivity

2. **Docker Issues**
   ```bash
   # Rebuild containers
   docker-compose up -d --build --force-recreate
   
   # Check container logs
   docker-compose logs -f
   ```

3. **Nginx Issues**
   - Check nginx configuration
   - Verify SSL certificates
   - Check proxy settings

### Performance Optimization

1. **Node.js Optimization**
   - PM2 for process management
   - Memory limits configuration
   - Garbage collection optimization

2. **Nginx Optimization**
   - Caching configuration
   - Gzip compression
   - Worker processes optimization

3. **MongoDB Optimization**
   - Proper indexing
   - Connection pooling
   - Query optimization

## Bash Scripts

### 1. Main Deployment Script (deploy.sh)
```bash
#!/bin/bash

# Configuration
REMOTE_HOST="your_server_ip"
REMOTE_USER="root"
REMOTE_DIR="/root/test_project3"
DOCKER_COMPOSE_FILE="docker-compose.yml"

# Colors for output
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
RED='\033[0;31m'
NC='\033[0m' # No Color

# Function to print section headers
print_section() {
    echo -e "\n${BLUE}=== $1 ===${NC}\n"
}

# Function to check command status
check_status() {
    if [ $? -eq 0 ]; then
        echo -e "${GREEN}✓ $1${NC}"
    else
        echo -e "${RED}✗ $2${NC}"
        exit 1
    fi
}

# Function to setup server environment
setup_server() {
    print_section "Setting up server environment"
    
    ssh ${REMOTE_USER}@${REMOTE_HOST} "bash -s" << 'ENDSSH'
    # Update system packages
    apt-get update && apt-get upgrade -y
    
    # Install essential tools
    apt-get install -y \
        apt-transport-https \
        ca-certificates \
        curl \
        gnupg \
        lsb-release \
        htop \
        git \
        ufw \
        fail2ban \
        ncdu \
        net-tools

    # Install Docker
    curl -fsSL https://get.docker.com -o get-docker.sh
    sh get-docker.sh
    systemctl enable docker
    systemctl start docker

    # Install Docker Compose
    curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" \
        -o /usr/local/bin/docker-compose
    chmod +x /usr/local/bin/docker-compose

    # Setup firewall
    ufw allow OpenSSH
    ufw allow 80/tcp
    ufw allow 443/tcp
    ufw --force enable

    # Configure fail2ban
    cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
    systemctl enable fail2ban
    systemctl start fail2ban
ENDSSH
}

# Function to perform system maintenance
perform_maintenance() {
    print_section "Performing system maintenance"
    
    ssh ${REMOTE_USER}@${REMOTE_HOST} "bash -s" << 'ENDSSH'
    # Clean up old Docker resources
    docker system prune -af --volumes
    
    # Clean package manager cache
    apt-get clean
    apt-get autoremove -y
    
    # Clean old log files
    find /var/log -type f -name "*.log" -mtime +30 -delete
    find /var/log -type f -name "*.gz" -mtime +30 -delete
ENDSSH
}

# Function to deploy application
deploy_app() {
    print_section "Deploying application"
    
    # Create deployment directory
    ssh ${REMOTE_USER}@${REMOTE_HOST} "mkdir -p ${REMOTE_DIR}"
    
    # Copy necessary files
    scp docker-compose.yml Dockerfile .env ${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_DIR}/
    scp -r nginx public ${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_DIR}/
    scp package*.json ${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_DIR}/
    
    # Deploy application
    ssh ${REMOTE_USER}@${REMOTE_HOST} "cd ${REMOTE_DIR} && \
        docker-compose down && \
        docker-compose up -d --build"
    
    check_status "Deployment successful" "Deployment failed"
}

# Function to create backup
create_backup() {
    print_section "Creating backup"
    
    ssh ${REMOTE_USER}@${REMOTE_HOST} "bash -s" << 'ENDSSH'
    BACKUP_DIR="/root/test_project3/backups"
    BACKUP_DATE=$(date +%Y%m%d_%H%M%S)
    
    # Create backup directory
    mkdir -p $BACKUP_DIR
    
    # Stop containers
    cd /root/test_project3
    docker-compose down
    
    # Create backup
    tar -czf ${BACKUP_DIR}/backup_${BACKUP_DATE}.tar.gz \
        --exclude='node_modules' \
        --exclude='.git' \
        /root/test_project3
    
    # Restart containers
    docker-compose up -d
    
    # Remove backups older than 7 days
    find $BACKUP_DIR -type f -name "backup_*.tar.gz" -mtime +7 -delete
ENDSSH
}

# Function to monitor application
monitor_app() {
    print_section "Monitoring application"
    
    ssh ${REMOTE_USER}@${REMOTE_HOST} "bash -s" << 'ENDSSH'
    # Check Docker container status
    docker ps
    
    # Check container logs
    docker-compose logs --tail=50 app
    
    # Check system resources
    echo -e "\nDisk Usage:"
    df -h
    
    echo -e "\nMemory Usage:"
    free -h
    
    echo -e "\nTop Processes:"
    top -b -n 1 | head -n 20
ENDSSH
}

# Main script execution
case "$1" in
    "setup")
        setup_server
        ;;
    "deploy")
        deploy_app
        ;;
    "maintenance")
        perform_maintenance
        ;;
    "backup")
        create_backup
        ;;
    "monitor")
        monitor_app
        ;;
    *)
        echo "Usage: $0 {setup|deploy|maintenance|backup|monitor}"
        echo "  setup       - Set up initial server environment"
        echo "  deploy      - Deploy application"
        echo "  maintenance - Perform system maintenance tasks"
        echo "  backup      - Create application backup"
        echo "  monitor     - Monitor application status"
        exit 1
        ;;
esac
```

### 2. Database Backup Script (db_backup.sh)
```bash
#!/bin/bash

# Configuration
MONGODB_URI="your_mongodb_uri"
BACKUP_DIR="/root/test_project3/db_backups"
RETENTION_DAYS=7

# Create backup directory
mkdir -p $BACKUP_DIR

# Set backup filename
BACKUP_FILE="$BACKUP_DIR/mongodb_$(date +%Y%m%d_%H%M%S).gz"

# Create backup
mongodump --uri="$MONGODB_URI" --gzip --archive="$BACKUP_FILE"

# Remove old backups
find $BACKUP_DIR -type f -name "mongodb_*.gz" -mtime +$RETENTION_DAYS -delete
```

### 3. SSL Certificate Renewal Script (renew_ssl.sh)
```bash
#!/bin/bash

# Configuration
DOMAINS="your-domain.com www.your-domain.com"
EMAIL="admin@your-domain.com"

# Stop nginx
docker-compose stop nginx

# Renew certificates
certbot renew --non-interactive --agree-tos --email $EMAIL

# Start nginx
docker-compose start nginx

# Check status
docker-compose ps nginx
```

### 4. Log Rotation Script (rotate_logs.sh)
```bash
#!/bin/bash

# Configuration
LOG_DIR="/root/test_project3/logs"
MAX_SIZE="100M"
RETAIN_DAYS=30

# Rotate application logs
find $LOG_DIR -type f -name "*.log" -size +$MAX_SIZE | while read file; do
    timestamp=$(date +%Y%m%d_%H%M%S)
    mv "$file" "${file}.${timestamp}"
    gzip "${file}.${timestamp}"
done

# Remove old logs
find $LOG_DIR -type f -name "*.gz" -mtime +$RETAIN_DAYS -delete
```

### 5. Monitoring Script (monitor.sh)
```bash
#!/bin/bash

# Configuration
ALERT_EMAIL="admin@your-domain.com"
DISK_THRESHOLD=90
MEMORY_THRESHOLD=90

# Check disk usage
check_disk() {
    disk_usage=$(df -h | awk '$NF=="/"{print $5}' | sed 's/%//g')
    if [ $disk_usage -gt $DISK_THRESHOLD ]; then
        echo "ALERT: Disk usage is at ${disk_usage}%" | mail -s "High Disk Usage Alert" $ALERT_EMAIL
    fi
}

# Check memory usage
check_memory() {
    memory_usage=$(free | awk '/Mem/{printf("%.0f"), $3/$2*100}')
    if [ $memory_usage -gt $MEMORY_THRESHOLD ]; then
        echo "ALERT: Memory usage is at ${memory_usage}%" | mail -s "High Memory Usage Alert" $ALERT_EMAIL
    fi
}

# Check container health
check_containers() {
    unhealthy_containers=$(docker ps --filter health=unhealthy --format "{{.Names}}")
    if [ ! -z "$unhealthy_containers" ]; then
        echo "ALERT: Unhealthy containers found: $unhealthy_containers" | \
            mail -s "Container Health Alert" $ALERT_EMAIL
    fi
}

# Run checks
check_disk
check_memory
check_containers
```

## Additional Resources

- [Node.js Best Practices](https://github.com/goldbergyoni/nodebestpractices)
- [Docker Documentation](https://docs.docker.com/)
- [MongoDB Atlas Documentation](https://docs.atlas.mongodb.com/)
- [Nginx Documentation](https://nginx.org/en/docs/)

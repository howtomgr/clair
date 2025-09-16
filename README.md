# clair Installation Guide

clair is a free and open-source container vulnerability scanner. Clair provides static analysis of vulnerabilities in containers

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Supported Operating Systems](#supported-operating-systems)
3. [Installation](#installation)
4. [Configuration](#configuration)
5. [Service Management](#service-management)
6. [Troubleshooting](#troubleshooting)
7. [Security Considerations](#security-considerations)
8. [Performance Tuning](#performance-tuning)
9. [Backup and Restore](#backup-and-restore)
10. [System Requirements](#system-requirements)
11. [Support](#support)
12. [Contributing](#contributing)
13. [License](#license)
14. [Acknowledgments](#acknowledgments)
15. [Version History](#version-history)
16. [Appendices](#appendices)

## 1. Prerequisites

- **Hardware Requirements**:
  - CPU: 2+ cores
  - RAM: 2GB minimum
  - Storage: 10GB for DB
  - Network: HTTP/gRPC
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 6060 (default clair port)
  - API on 6061
- **Dependencies**:
  - See official documentation for specific requirements
- **System Access**: root or sudo privileges required


## 2. Supported Operating Systems

This guide supports installation on:
- RHEL 8/9 and derivatives (CentOS Stream, Rocky Linux, AlmaLinux)
- Debian 11/12
- Ubuntu 20.04/22.04/24.04 LTS
- Arch Linux (rolling release)
- Alpine Linux 3.18+
- openSUSE Leap 15.5+ / Tumbleweed
- SUSE Linux Enterprise Server (SLES) 15+
- macOS 12+ (Monterey and later) 
- FreeBSD 13+
- Windows 10/11/Server 2019+ (where applicable)

## 3. Installation

### RHEL/CentOS/Rocky Linux/AlmaLinux

```bash
# Install EPEL repository if needed
sudo dnf install -y epel-release

# Install clair
sudo dnf install -y clair

# Enable and start service
sudo systemctl enable --now clair

# Configure firewall
sudo firewall-cmd --permanent --add-port=6060/tcp
sudo firewall-cmd --reload

# Verify installation
clair --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install clair
sudo apt install -y clair

# Enable and start service
sudo systemctl enable --now clair

# Configure firewall
sudo ufw allow 6060

# Verify installation
clair --version
```

### Arch Linux

```bash
# Install clair
sudo pacman -S clair

# Enable and start service
sudo systemctl enable --now clair

# Verify installation
clair --version
```

### Alpine Linux

```bash
# Install clair
apk add --no-cache clair

# Enable and start service
rc-update add clair default
rc-service clair start

# Verify installation
clair --version
```

### openSUSE/SLES

```bash
# Install clair
sudo zypper install -y clair

# Enable and start service
sudo systemctl enable --now clair

# Configure firewall
sudo firewall-cmd --permanent --add-port=6060/tcp
sudo firewall-cmd --reload

# Verify installation
clair --version
```

### macOS

```bash
# Using Homebrew
brew install clair

# Start service
brew services start clair

# Verify installation
clair --version
```

### FreeBSD

```bash
# Using pkg
pkg install clair

# Enable in rc.conf
echo 'clair_enable="YES"' >> /etc/rc.conf

# Start service
service clair start

# Verify installation
clair --version
```

### Windows

```bash
# Using Chocolatey
choco install clair

# Or using Scoop
scoop install clair

# Verify installation
clair --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/clair

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
clair --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable clair

# Start service
sudo systemctl start clair

# Stop service
sudo systemctl stop clair

# Restart service
sudo systemctl restart clair

# Check status
sudo systemctl status clair

# View logs
sudo journalctl -u clair -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add clair default

# Start service
rc-service clair start

# Stop service
rc-service clair stop

# Restart service
rc-service clair restart

# Check status
rc-service clair status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'clair_enable="YES"' >> /etc/rc.conf

# Start service
service clair start

# Stop service
service clair stop

# Restart service
service clair restart

# Check status
service clair status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start clair
brew services stop clair
brew services restart clair

# Check status
brew services list | grep clair
```

### Windows Service Manager

```powershell
# Start service
net start clair

# Stop service
net stop clair

# Using PowerShell
Start-Service clair
Stop-Service clair
Restart-Service clair

# Check status
Get-Service clair
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream clair_backend {
    server 127.0.0.1:6060;
}

server {
    listen 80;
    server_name clair.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name clair.example.com;

    ssl_certificate /etc/ssl/certs/clair.example.com.crt;
    ssl_certificate_key /etc/ssl/private/clair.example.com.key;

    location / {
        proxy_pass http://clair_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName clair.example.com
    Redirect permanent / https://clair.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName clair.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/clair.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/clair.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:6060/
    ProxyPassReverse / http://127.0.0.1:6060/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend clair_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/clair.pem
    redirect scheme https if !{ ssl_fc }
    default_backend clair_backend

backend clair_backend
    balance roundrobin
    server clair1 127.0.0.1:6060 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R clair:clair /etc/clair
sudo chmod 750 /etc/clair

# Configure firewall
sudo firewall-cmd --permanent --add-port=6060/tcp
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on
```

## Database Setup

See official documentation for database configuration requirements.

## Performance Optimization

### System Tuning

```bash
# Basic system tuning
echo 'net.core.somaxconn = 65535' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv4.tcp_max_syn_backlog = 65535' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

## Monitoring

### Basic Monitoring

```bash
# Check service status
sudo systemctl status clair

# View logs
sudo journalctl -u clair -f

# Monitor resource usage
top -p $(pgrep clair)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/clair"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/clair-backup-$DATE.tar.gz" /etc/clair /var/lib/clair

echo "Backup completed: $BACKUP_DIR/clair-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop clair

# Restore from backup
tar -xzf /backup/clair/clair-backup-*.tar.gz -C /

# Start service
sudo systemctl start clair
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u clair -n 100
sudo tail -f /var/log/clair/clair.log

# Check configuration
clair --version

# Check permissions
ls -la /etc/clair
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 6060

# Test connectivity
telnet localhost 6060

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep clair)

# Check disk I/O
iotop -p $(pgrep clair)

# Check connections
ss -an | grep 6060
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  clair:
    image: clair:latest
    ports:
      - "6060:6060"
    volumes:
      - ./config:/etc/clair
      - ./data:/var/lib/clair
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update clair

# Debian/Ubuntu
sudo apt update && sudo apt upgrade clair

# Arch Linux
sudo pacman -Syu clair

# Alpine Linux
apk update && apk upgrade clair

# openSUSE
sudo zypper update clair

# FreeBSD
pkg update && pkg upgrade clair

# Always backup before updates
tar -czf /backup/clair-pre-update-$(date +%Y%m%d).tar.gz /etc/clair

# Restart after updates
sudo systemctl restart clair
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/clair

# Clean old logs
find /var/log/clair -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/clair
```

## Additional Resources

- Official Documentation: https://docs.clair.org/
- GitHub Repository: https://github.com/clair/clair
- Community Forum: https://forum.clair.org/
- Best Practices Guide: https://docs.clair.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.

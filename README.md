# prtg Installation Guide

prtg is a free and open-source network monitor. PRTG provides comprehensive network monitoring solution

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
  - RAM: 4GB minimum
  - Storage: 20GB for data
  - Network: HTTP/SNMP
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 80 (default prtg port)
  - Probe on 23560
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

# Install prtg
sudo dnf install -y prtg

# Enable and start service
sudo systemctl enable --now prtg

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
prtg --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install prtg
sudo apt install -y prtg

# Enable and start service
sudo systemctl enable --now prtg

# Configure firewall
sudo ufw allow 80

# Verify installation
prtg --version
```

### Arch Linux

```bash
# Install prtg
sudo pacman -S prtg

# Enable and start service
sudo systemctl enable --now prtg

# Verify installation
prtg --version
```

### Alpine Linux

```bash
# Install prtg
apk add --no-cache prtg

# Enable and start service
rc-update add prtg default
rc-service prtg start

# Verify installation
prtg --version
```

### openSUSE/SLES

```bash
# Install prtg
sudo zypper install -y prtg

# Enable and start service
sudo systemctl enable --now prtg

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
prtg --version
```

### macOS

```bash
# Using Homebrew
brew install prtg

# Start service
brew services start prtg

# Verify installation
prtg --version
```

### FreeBSD

```bash
# Using pkg
pkg install prtg

# Enable in rc.conf
echo 'prtg_enable="YES"' >> /etc/rc.conf

# Start service
service prtg start

# Verify installation
prtg --version
```

### Windows

```bash
# Using Chocolatey
choco install prtg

# Or using Scoop
scoop install prtg

# Verify installation
prtg --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/prtg

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
prtg --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable prtg

# Start service
sudo systemctl start prtg

# Stop service
sudo systemctl stop prtg

# Restart service
sudo systemctl restart prtg

# Check status
sudo systemctl status prtg

# View logs
sudo journalctl -u prtg -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add prtg default

# Start service
rc-service prtg start

# Stop service
rc-service prtg stop

# Restart service
rc-service prtg restart

# Check status
rc-service prtg status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'prtg_enable="YES"' >> /etc/rc.conf

# Start service
service prtg start

# Stop service
service prtg stop

# Restart service
service prtg restart

# Check status
service prtg status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start prtg
brew services stop prtg
brew services restart prtg

# Check status
brew services list | grep prtg
```

### Windows Service Manager

```powershell
# Start service
net start prtg

# Stop service
net stop prtg

# Using PowerShell
Start-Service prtg
Stop-Service prtg
Restart-Service prtg

# Check status
Get-Service prtg
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream prtg_backend {
    server 127.0.0.1:80;
}

server {
    listen 80;
    server_name prtg.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name prtg.example.com;

    ssl_certificate /etc/ssl/certs/prtg.example.com.crt;
    ssl_certificate_key /etc/ssl/private/prtg.example.com.key;

    location / {
        proxy_pass http://prtg_backend;
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
    ServerName prtg.example.com
    Redirect permanent / https://prtg.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName prtg.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/prtg.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/prtg.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:80/
    ProxyPassReverse / http://127.0.0.1:80/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend prtg_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/prtg.pem
    redirect scheme https if !{ ssl_fc }
    default_backend prtg_backend

backend prtg_backend
    balance roundrobin
    server prtg1 127.0.0.1:80 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R prtg:prtg /etc/prtg
sudo chmod 750 /etc/prtg

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
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
sudo systemctl status prtg

# View logs
sudo journalctl -u prtg -f

# Monitor resource usage
top -p $(pgrep prtg)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/prtg"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/prtg-backup-$DATE.tar.gz" /etc/prtg /var/lib/prtg

echo "Backup completed: $BACKUP_DIR/prtg-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop prtg

# Restore from backup
tar -xzf /backup/prtg/prtg-backup-*.tar.gz -C /

# Start service
sudo systemctl start prtg
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u prtg -n 100
sudo tail -f /var/log/prtg/prtg.log

# Check configuration
prtg --version

# Check permissions
ls -la /etc/prtg
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 80

# Test connectivity
telnet localhost 80

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep prtg)

# Check disk I/O
iotop -p $(pgrep prtg)

# Check connections
ss -an | grep 80
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  prtg:
    image: prtg:latest
    ports:
      - "80:80"
    volumes:
      - ./config:/etc/prtg
      - ./data:/var/lib/prtg
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update prtg

# Debian/Ubuntu
sudo apt update && sudo apt upgrade prtg

# Arch Linux
sudo pacman -Syu prtg

# Alpine Linux
apk update && apk upgrade prtg

# openSUSE
sudo zypper update prtg

# FreeBSD
pkg update && pkg upgrade prtg

# Always backup before updates
tar -czf /backup/prtg-pre-update-$(date +%Y%m%d).tar.gz /etc/prtg

# Restart after updates
sudo systemctl restart prtg
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/prtg

# Clean old logs
find /var/log/prtg -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/prtg
```

## Additional Resources

- Official Documentation: https://docs.prtg.org/
- GitHub Repository: https://github.com/prtg/prtg
- Community Forum: https://forum.prtg.org/
- Best Practices Guide: https://docs.prtg.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.

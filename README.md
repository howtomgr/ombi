# ombi Installation Guide

ombi is a free and open-source media request system. Ombi provides a self-hosted web application for users to request media for Plex/Emby/Jellyfin

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
  - CPU: 1 core minimum
  - RAM: 512MB minimum
  - Storage: 500MB for app
  - Network: HTTP/HTTPS access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 3579 (default ombi port)
  - None
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

# Install ombi
sudo dnf install -y ombi

# Enable and start service
sudo systemctl enable --now ombi

# Configure firewall
sudo firewall-cmd --permanent --add-port=3579/tcp
sudo firewall-cmd --reload

# Verify installation
ombi --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install ombi
sudo apt install -y ombi

# Enable and start service
sudo systemctl enable --now ombi

# Configure firewall
sudo ufw allow 3579

# Verify installation
ombi --version
```

### Arch Linux

```bash
# Install ombi
sudo pacman -S ombi

# Enable and start service
sudo systemctl enable --now ombi

# Verify installation
ombi --version
```

### Alpine Linux

```bash
# Install ombi
apk add --no-cache ombi

# Enable and start service
rc-update add ombi default
rc-service ombi start

# Verify installation
ombi --version
```

### openSUSE/SLES

```bash
# Install ombi
sudo zypper install -y ombi

# Enable and start service
sudo systemctl enable --now ombi

# Configure firewall
sudo firewall-cmd --permanent --add-port=3579/tcp
sudo firewall-cmd --reload

# Verify installation
ombi --version
```

### macOS

```bash
# Using Homebrew
brew install ombi

# Start service
brew services start ombi

# Verify installation
ombi --version
```

### FreeBSD

```bash
# Using pkg
pkg install ombi

# Enable in rc.conf
echo 'ombi_enable="YES"' >> /etc/rc.conf

# Start service
service ombi start

# Verify installation
ombi --version
```

### Windows

```bash
# Using Chocolatey
choco install ombi

# Or using Scoop
scoop install ombi

# Verify installation
ombi --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/ombi

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
ombi --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable ombi

# Start service
sudo systemctl start ombi

# Stop service
sudo systemctl stop ombi

# Restart service
sudo systemctl restart ombi

# Check status
sudo systemctl status ombi

# View logs
sudo journalctl -u ombi -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add ombi default

# Start service
rc-service ombi start

# Stop service
rc-service ombi stop

# Restart service
rc-service ombi restart

# Check status
rc-service ombi status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'ombi_enable="YES"' >> /etc/rc.conf

# Start service
service ombi start

# Stop service
service ombi stop

# Restart service
service ombi restart

# Check status
service ombi status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start ombi
brew services stop ombi
brew services restart ombi

# Check status
brew services list | grep ombi
```

### Windows Service Manager

```powershell
# Start service
net start ombi

# Stop service
net stop ombi

# Using PowerShell
Start-Service ombi
Stop-Service ombi
Restart-Service ombi

# Check status
Get-Service ombi
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream ombi_backend {
    server 127.0.0.1:3579;
}

server {
    listen 80;
    server_name ombi.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name ombi.example.com;

    ssl_certificate /etc/ssl/certs/ombi.example.com.crt;
    ssl_certificate_key /etc/ssl/private/ombi.example.com.key;

    location / {
        proxy_pass http://ombi_backend;
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
    ServerName ombi.example.com
    Redirect permanent / https://ombi.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName ombi.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/ombi.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/ombi.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:3579/
    ProxyPassReverse / http://127.0.0.1:3579/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend ombi_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/ombi.pem
    redirect scheme https if !{ ssl_fc }
    default_backend ombi_backend

backend ombi_backend
    balance roundrobin
    server ombi1 127.0.0.1:3579 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R ombi:ombi /etc/ombi
sudo chmod 750 /etc/ombi

# Configure firewall
sudo firewall-cmd --permanent --add-port=3579/tcp
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
sudo systemctl status ombi

# View logs
sudo journalctl -u ombi -f

# Monitor resource usage
top -p $(pgrep ombi)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/ombi"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/ombi-backup-$DATE.tar.gz" /etc/ombi /var/lib/ombi

echo "Backup completed: $BACKUP_DIR/ombi-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop ombi

# Restore from backup
tar -xzf /backup/ombi/ombi-backup-*.tar.gz -C /

# Start service
sudo systemctl start ombi
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u ombi -n 100
sudo tail -f /var/log/ombi/ombi.log

# Check configuration
ombi --version

# Check permissions
ls -la /etc/ombi
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 3579

# Test connectivity
telnet localhost 3579

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep ombi)

# Check disk I/O
iotop -p $(pgrep ombi)

# Check connections
ss -an | grep 3579
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  ombi:
    image: ombi:latest
    ports:
      - "3579:3579"
    volumes:
      - ./config:/etc/ombi
      - ./data:/var/lib/ombi
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update ombi

# Debian/Ubuntu
sudo apt update && sudo apt upgrade ombi

# Arch Linux
sudo pacman -Syu ombi

# Alpine Linux
apk update && apk upgrade ombi

# openSUSE
sudo zypper update ombi

# FreeBSD
pkg update && pkg upgrade ombi

# Always backup before updates
tar -czf /backup/ombi-pre-update-$(date +%Y%m%d).tar.gz /etc/ombi

# Restart after updates
sudo systemctl restart ombi
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/ombi

# Clean old logs
find /var/log/ombi -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/ombi
```

## Additional Resources

- Official Documentation: https://docs.ombi.org/
- GitHub Repository: https://github.com/ombi/ombi
- Community Forum: https://forum.ombi.org/
- Best Practices Guide: https://docs.ombi.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.

# fluentd Installation Guide

fluentd is a free and open-source data collector. Fluentd provides unified logging layer for collecting and consuming data

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
  - Storage: 10GB for buffers
  - Network: Various inputs
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 24224 (default fluentd port)
  - HTTP on 9880
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

# Install fluentd
sudo dnf install -y fluentd

# Enable and start service
sudo systemctl enable --now fluentd

# Configure firewall
sudo firewall-cmd --permanent --add-port=24224/tcp
sudo firewall-cmd --reload

# Verify installation
fluentd --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install fluentd
sudo apt install -y fluentd

# Enable and start service
sudo systemctl enable --now fluentd

# Configure firewall
sudo ufw allow 24224

# Verify installation
fluentd --version
```

### Arch Linux

```bash
# Install fluentd
sudo pacman -S fluentd

# Enable and start service
sudo systemctl enable --now fluentd

# Verify installation
fluentd --version
```

### Alpine Linux

```bash
# Install fluentd
apk add --no-cache fluentd

# Enable and start service
rc-update add fluentd default
rc-service fluentd start

# Verify installation
fluentd --version
```

### openSUSE/SLES

```bash
# Install fluentd
sudo zypper install -y fluentd

# Enable and start service
sudo systemctl enable --now fluentd

# Configure firewall
sudo firewall-cmd --permanent --add-port=24224/tcp
sudo firewall-cmd --reload

# Verify installation
fluentd --version
```

### macOS

```bash
# Using Homebrew
brew install fluentd

# Start service
brew services start fluentd

# Verify installation
fluentd --version
```

### FreeBSD

```bash
# Using pkg
pkg install fluentd

# Enable in rc.conf
echo 'fluentd_enable="YES"' >> /etc/rc.conf

# Start service
service fluentd start

# Verify installation
fluentd --version
```

### Windows

```bash
# Using Chocolatey
choco install fluentd

# Or using Scoop
scoop install fluentd

# Verify installation
fluentd --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/fluentd

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
fluentd --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable fluentd

# Start service
sudo systemctl start fluentd

# Stop service
sudo systemctl stop fluentd

# Restart service
sudo systemctl restart fluentd

# Check status
sudo systemctl status fluentd

# View logs
sudo journalctl -u fluentd -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add fluentd default

# Start service
rc-service fluentd start

# Stop service
rc-service fluentd stop

# Restart service
rc-service fluentd restart

# Check status
rc-service fluentd status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'fluentd_enable="YES"' >> /etc/rc.conf

# Start service
service fluentd start

# Stop service
service fluentd stop

# Restart service
service fluentd restart

# Check status
service fluentd status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start fluentd
brew services stop fluentd
brew services restart fluentd

# Check status
brew services list | grep fluentd
```

### Windows Service Manager

```powershell
# Start service
net start fluentd

# Stop service
net stop fluentd

# Using PowerShell
Start-Service fluentd
Stop-Service fluentd
Restart-Service fluentd

# Check status
Get-Service fluentd
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream fluentd_backend {
    server 127.0.0.1:24224;
}

server {
    listen 80;
    server_name fluentd.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name fluentd.example.com;

    ssl_certificate /etc/ssl/certs/fluentd.example.com.crt;
    ssl_certificate_key /etc/ssl/private/fluentd.example.com.key;

    location / {
        proxy_pass http://fluentd_backend;
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
    ServerName fluentd.example.com
    Redirect permanent / https://fluentd.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName fluentd.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/fluentd.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/fluentd.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:24224/
    ProxyPassReverse / http://127.0.0.1:24224/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend fluentd_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/fluentd.pem
    redirect scheme https if !{ ssl_fc }
    default_backend fluentd_backend

backend fluentd_backend
    balance roundrobin
    server fluentd1 127.0.0.1:24224 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R fluentd:fluentd /etc/fluentd
sudo chmod 750 /etc/fluentd

# Configure firewall
sudo firewall-cmd --permanent --add-port=24224/tcp
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
sudo systemctl status fluentd

# View logs
sudo journalctl -u fluentd -f

# Monitor resource usage
top -p $(pgrep fluentd)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/fluentd"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/fluentd-backup-$DATE.tar.gz" /etc/fluentd /var/lib/fluentd

echo "Backup completed: $BACKUP_DIR/fluentd-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop fluentd

# Restore from backup
tar -xzf /backup/fluentd/fluentd-backup-*.tar.gz -C /

# Start service
sudo systemctl start fluentd
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u fluentd -n 100
sudo tail -f /var/log/fluentd/fluentd.log

# Check configuration
fluentd --version

# Check permissions
ls -la /etc/fluentd
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 24224

# Test connectivity
telnet localhost 24224

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep fluentd)

# Check disk I/O
iotop -p $(pgrep fluentd)

# Check connections
ss -an | grep 24224
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  fluentd:
    image: fluentd:latest
    ports:
      - "24224:24224"
    volumes:
      - ./config:/etc/fluentd
      - ./data:/var/lib/fluentd
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update fluentd

# Debian/Ubuntu
sudo apt update && sudo apt upgrade fluentd

# Arch Linux
sudo pacman -Syu fluentd

# Alpine Linux
apk update && apk upgrade fluentd

# openSUSE
sudo zypper update fluentd

# FreeBSD
pkg update && pkg upgrade fluentd

# Always backup before updates
tar -czf /backup/fluentd-pre-update-$(date +%Y%m%d).tar.gz /etc/fluentd

# Restart after updates
sudo systemctl restart fluentd
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/fluentd

# Clean old logs
find /var/log/fluentd -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/fluentd
```

## Additional Resources

- Official Documentation: https://docs.fluentd.org/
- GitHub Repository: https://github.com/fluentd/fluentd
- Community Forum: https://forum.fluentd.org/
- Best Practices Guide: https://docs.fluentd.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.

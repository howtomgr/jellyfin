# jellyfin Installation Guide

jellyfin is a free and open-source free software media system. Jellyfin is a free and open-source media server for hosting and streaming your media library, serving as an alternative to Plex, Emby, or commercial streaming solutions

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
  - CPU: 2+ cores (4+ for transcoding)
  - RAM: 2GB minimum (4GB+ recommended)
  - Storage: 100GB+ for media storage
  - Network: Stable network for streaming
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 8096 (default jellyfin port)
  - Port 8920 for HTTPS
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

# Install jellyfin
sudo dnf install -y jellyfin

# Enable and start service
sudo systemctl enable --now jellyfin

# Configure firewall
sudo firewall-cmd --permanent --add-port=8096/tcp
sudo firewall-cmd --reload

# Verify installation
jellyfin --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install jellyfin
sudo apt install -y jellyfin

# Enable and start service
sudo systemctl enable --now jellyfin

# Configure firewall
sudo ufw allow 8096

# Verify installation
jellyfin --version
```

### Arch Linux

```bash
# Install jellyfin
sudo pacman -S jellyfin

# Enable and start service
sudo systemctl enable --now jellyfin

# Verify installation
jellyfin --version
```

### Alpine Linux

```bash
# Install jellyfin
apk add --no-cache jellyfin

# Enable and start service
rc-update add jellyfin default
rc-service jellyfin start

# Verify installation
jellyfin --version
```

### openSUSE/SLES

```bash
# Install jellyfin
sudo zypper install -y jellyfin

# Enable and start service
sudo systemctl enable --now jellyfin

# Configure firewall
sudo firewall-cmd --permanent --add-port=8096/tcp
sudo firewall-cmd --reload

# Verify installation
jellyfin --version
```

### macOS

```bash
# Using Homebrew
brew install jellyfin

# Start service
brew services start jellyfin

# Verify installation
jellyfin --version
```

### FreeBSD

```bash
# Using pkg
pkg install jellyfin

# Enable in rc.conf
echo 'jellyfin_enable="YES"' >> /etc/rc.conf

# Start service
service jellyfin start

# Verify installation
jellyfin --version
```

### Windows

```bash
# Using Chocolatey
choco install jellyfin

# Or using Scoop
scoop install jellyfin

# Verify installation
jellyfin --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/jellyfin

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
jellyfin --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable jellyfin

# Start service
sudo systemctl start jellyfin

# Stop service
sudo systemctl stop jellyfin

# Restart service
sudo systemctl restart jellyfin

# Check status
sudo systemctl status jellyfin

# View logs
sudo journalctl -u jellyfin -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add jellyfin default

# Start service
rc-service jellyfin start

# Stop service
rc-service jellyfin stop

# Restart service
rc-service jellyfin restart

# Check status
rc-service jellyfin status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'jellyfin_enable="YES"' >> /etc/rc.conf

# Start service
service jellyfin start

# Stop service
service jellyfin stop

# Restart service
service jellyfin restart

# Check status
service jellyfin status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start jellyfin
brew services stop jellyfin
brew services restart jellyfin

# Check status
brew services list | grep jellyfin
```

### Windows Service Manager

```powershell
# Start service
net start jellyfin

# Stop service
net stop jellyfin

# Using PowerShell
Start-Service jellyfin
Stop-Service jellyfin
Restart-Service jellyfin

# Check status
Get-Service jellyfin
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream jellyfin_backend {
    server 127.0.0.1:8096;
}

server {
    listen 80;
    server_name jellyfin.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name jellyfin.example.com;

    ssl_certificate /etc/ssl/certs/jellyfin.example.com.crt;
    ssl_certificate_key /etc/ssl/private/jellyfin.example.com.key;

    location / {
        proxy_pass http://jellyfin_backend;
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
    ServerName jellyfin.example.com
    Redirect permanent / https://jellyfin.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName jellyfin.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/jellyfin.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/jellyfin.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:8096/
    ProxyPassReverse / http://127.0.0.1:8096/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend jellyfin_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/jellyfin.pem
    redirect scheme https if !{ ssl_fc }
    default_backend jellyfin_backend

backend jellyfin_backend
    balance roundrobin
    server jellyfin1 127.0.0.1:8096 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R jellyfin:jellyfin /etc/jellyfin
sudo chmod 750 /etc/jellyfin

# Configure firewall
sudo firewall-cmd --permanent --add-port=8096/tcp
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
sudo systemctl status jellyfin

# View logs
sudo journalctl -u jellyfin -f

# Monitor resource usage
top -p $(pgrep jellyfin)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/jellyfin"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/jellyfin-backup-$DATE.tar.gz" /etc/jellyfin /var/lib/jellyfin

echo "Backup completed: $BACKUP_DIR/jellyfin-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop jellyfin

# Restore from backup
tar -xzf /backup/jellyfin/jellyfin-backup-*.tar.gz -C /

# Start service
sudo systemctl start jellyfin
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u jellyfin -n 100
sudo tail -f /var/log/jellyfin/jellyfin.log

# Check configuration
jellyfin --version

# Check permissions
ls -la /etc/jellyfin
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 8096

# Test connectivity
telnet localhost 8096

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep jellyfin)

# Check disk I/O
iotop -p $(pgrep jellyfin)

# Check connections
ss -an | grep 8096
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  jellyfin:
    image: jellyfin:latest
    ports:
      - "8096:8096"
    volumes:
      - ./config:/etc/jellyfin
      - ./data:/var/lib/jellyfin
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update jellyfin

# Debian/Ubuntu
sudo apt update && sudo apt upgrade jellyfin

# Arch Linux
sudo pacman -Syu jellyfin

# Alpine Linux
apk update && apk upgrade jellyfin

# openSUSE
sudo zypper update jellyfin

# FreeBSD
pkg update && pkg upgrade jellyfin

# Always backup before updates
tar -czf /backup/jellyfin-pre-update-$(date +%Y%m%d).tar.gz /etc/jellyfin

# Restart after updates
sudo systemctl restart jellyfin
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/jellyfin

# Clean old logs
find /var/log/jellyfin -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/jellyfin
```

## Additional Resources

- Official Documentation: https://docs.jellyfin.org/
- GitHub Repository: https://github.com/jellyfin/jellyfin
- Community Forum: https://forum.jellyfin.org/
- Best Practices Guide: https://docs.jellyfin.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.

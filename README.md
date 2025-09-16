# Memcached Installation Guide

Memcached is a free and open-source, high-performance, distributed memory object caching system. Originally developed by Danga Interactive for LiveJournal, it's now used by many high-traffic websites as an in-memory key-value store that significantly reduces database load and improves application response times. It serves as a FOSS alternative to commercial caching solutions like Redis Enterprise or AWS ElastiCache.

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
  - CPU: 1 core minimum (2+ cores recommended for high traffic)
  - RAM: 64MB minimum (1GB+ recommended for production)
  - Storage: 100MB for installation
- **Operating System**: Linux, BSD, macOS, or Windows (via WSL2)
- **Network Requirements**:
  - Port 11211 (default, configurable)
  - TCP and UDP protocols
  - Low latency network for distributed setups
- **Dependencies**:
  - libevent 2.0+ (for event notification)
  - SASL libraries (optional, for authentication)
  - C compiler (for building from source)
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
# RHEL/CentOS 7
sudo yum install -y epel-release
sudo yum install -y memcached

# RHEL/CentOS/Rocky/Alma 8+
sudo dnf install -y epel-release
sudo dnf install -y memcached

# Install development tools (optional, for building from source)
sudo dnf groupinstall -y "Development Tools"
sudo dnf install -y libevent-devel
```

### Debian/Ubuntu

```bash
# Update package list
sudo apt update

# Install memcached
sudo apt install -y memcached libmemcached-tools

# Install development libraries (optional)
sudo apt install -y build-essential libevent-dev libsasl2-dev
```

### Arch Linux

```bash
# Install memcached from official repositories
sudo pacman -S memcached

# Install development tools (optional)
sudo pacman -S base-devel libevent

# For additional tools
sudo pacman -S libmemcached

# Enable and start service
sudo systemctl enable --now memcached
```

### Alpine Linux

```bash
# Install memcached
apk add --no-cache memcached

# Install development tools (optional)
apk add --no-cache build-base libevent-dev

# Create memcached user if not exists
adduser -D -H -s /sbin/nologin memcached
```

### openSUSE/SLES

```bash
# openSUSE Leap/Tumbleweed
sudo zypper install -y memcached

# Install development tools (optional)
sudo zypper install -y gcc make libevent-devel

# SLES (may require additional repositories)
sudo SUSEConnect -p sle-module-web-scripting/15.5/x86_64
sudo zypper install -y memcached

# Enable and start service
sudo systemctl enable --now memcached
```

### macOS

```bash
# Using Homebrew
brew install memcached

# Start as service
brew services start memcached

# Or run manually
/usr/local/opt/memcached/bin/memcached
```

### Windows

```powershell
# Using Chocolatey
choco install memcached

# Using manual installation
# 1. Download from: http://downloads.northscale.com/memcached-1.4.5-amd64.zip
# 2. Extract to C:\memcached
# 3. Install as service:
C:\memcached\memcached.exe -d install

# Start service
net start memcached
```

### Build from Source (All Platforms)

```bash
# Download latest version
wget https://memcached.org/latest
tar -zxvf memcached-*.tar.gz
cd memcached-*

# Configure and build
./configure --prefix=/usr/local
make
sudo make install

# Create systemd service (Linux)
sudo nano /etc/systemd/system/memcached.service
```

## Initial Configuration

### First-Run Setup

1. **Create dedicated user** (if not created by package):
```bash
# Linux systems
sudo useradd -r -s /sbin/nologin memcached

# Verify user exists
id memcached
```

2. **Default configuration locations**:
- RHEL/CentOS/Rocky/AlmaLinux: `/etc/sysconfig/memcached`
- Debian/Ubuntu: `/etc/memcached.conf`
- Arch Linux: `/etc/conf.d/memcached`
- Alpine Linux: `/etc/conf.d/memcached`
- openSUSE/SLES: `/etc/sysconfig/memcached`
- macOS: `~/Library/LaunchAgents/homebrew.mxcl.memcached.plist`
- FreeBSD: `/usr/local/etc/memcached.conf`

3. **Essential settings to change**:

```bash
# Set appropriate memory limit (default is often 64MB)
-m 256    # 256MB cache size

# Bind to specific interface (security)
-l 127.0.0.1    # localhost only
# OR for specific network
-l 192.168.1.100

# Change default port if needed
-p 11211

# Set max connections
-c 1024
```

**WARNING:** Never expose memcached to the public internet without authentication!

### Testing Initial Setup

```bash
# Test memcached is running
echo "stats" | nc localhost 11211

# Check version
echo "version" | nc localhost 11211

# Test set/get operation
(echo "set test 0 60 5"; echo "hello"; echo "get test") | nc localhost 11211
```

## Advanced Configuration

### RHEL/CentOS Configuration

Edit `/etc/sysconfig/memcached`:
```bash
# Port
PORT="11211"

# User
USER="memcached"

# Max connections
MAXCONN="1024"

# Cache size in MB
CACHESIZE="64"

# Listening IP (empty for all interfaces)
OPTIONS="-l 127.0.0.1"

# For network access
# OPTIONS="-l 0.0.0.0"

# With SASL authentication
# OPTIONS="-l 127.0.0.1 -S"
```

### Debian/Ubuntu Configuration

Edit `/etc/memcached.conf`:
```bash
# Memory cache size in MB
-m 64

# Port
-p 11211

# User to run daemon
-u memcache

# Listen on localhost only
-l 127.0.0.1

# Max simultaneous connections
-c 1024

# Run as daemon
-d

# Log file
logfile /var/log/memcached.log

# Verbose logging (remove for production)
# -v

# Very verbose (debugging)
# -vv
```

### Alpine Linux Configuration

Edit `/etc/conf.d/memcached`:
```bash
# Memcached options
MEMCACHED_USER="memcached"
MEMCACHED_PORT="11211"
MEMCACHED_MAX_MEMORY="64"
MEMCACHED_MAX_CONNECTIONS="1024"
MEMCACHED_LISTEN="127.0.0.1"
MEMCACHED_OPTS=""
```

### macOS Configuration

Create `~/Library/LaunchAgents/homebrew.mxcl.memcached.plist`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>homebrew.mxcl.memcached</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/local/opt/memcached/bin/memcached</string>
        <string>-l</string>
        <string>127.0.0.1</string>
        <string>-m</string>
        <string>64</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
</dict>
</plist>
```

### Windows Configuration

Create `C:\memcached\memcached.conf`:
```
# Memory limit in MB
-m 64

# Port number
-p 11211

# IP address to listen on
-l 127.0.0.1

# Maximum connections
-c 1024
```

## 5. Service Management

### RHEL/CentOS/Debian/Ubuntu (systemd)

```bash
# Enable on boot
sudo systemctl enable memcached

# Start service
sudo systemctl start memcached

# Stop service
sudo systemctl stop memcached

# Restart service
sudo systemctl restart memcached

# Check status
sudo systemctl status memcached

# View logs
sudo journalctl -u memcached -f
```

### Alpine Linux (OpenRC)

```bash
# Enable on boot
rc-update add memcached default

# Start service
rc-service memcached start

# Stop service
rc-service memcached stop

# Restart service
rc-service memcached restart

# Check status
rc-service memcached status
```

### macOS

```bash
# Start service
brew services start memcached

# Stop service
brew services stop memcached

# Restart service
brew services restart memcached

# Check if running
brew services list | grep memcached
```

### Windows

```powershell
# Start service
net start memcached

# Stop service
net stop memcached

# Restart service
net stop memcached && net start memcached

# Check status
sc query memcached

# Configure service
sc config memcached start= auto
```

## Security Configuration

### Enable SASL Authentication

**RHEL/CentOS/Debian/Ubuntu:**
```bash
# Install SASL
# RHEL/CentOS
sudo yum install -y cyrus-sasl cyrus-sasl-devel cyrus-sasl-plain

# Debian/Ubuntu
sudo apt install -y sasl2-bin libsasl2-2 libsasl2-dev libsasl2-modules

# Create SASL configuration
sudo mkdir -p /etc/sasl2
sudo nano /etc/sasl2/memcached.conf
```

Add to `/etc/sasl2/memcached.conf`:
```
mech_list: plain
log_level: 5
sasldb_path: /etc/sasl2/memcached-sasldb2
```

Create SASL user:
```bash
sudo saslpasswd2 -a memcached -c -f /etc/sasl2/memcached-sasldb2 myuser
sudo chown memcached:memcached /etc/sasl2/memcached-sasldb2
```

Enable SASL in memcached:
```bash
# RHEL/CentOS - Edit /etc/sysconfig/memcached
OPTIONS="-l 127.0.0.1 -S"

# Debian/Ubuntu - Edit /etc/memcached.conf
# Add line:
-S

# Restart service
sudo systemctl restart memcached
```

### Firewall Configuration

**RHEL/CentOS (firewalld):**
```bash
# Add service
sudo firewall-cmd --permanent --add-service=memcache
# Or specific port
sudo firewall-cmd --permanent --add-port=11211/tcp

# Reload firewall
sudo firewall-cmd --reload
```

**Debian/Ubuntu (ufw):**
```bash
# Allow from specific IP
sudo ufw allow from 192.168.1.100 to any port 11211

# Allow from subnet
sudo ufw allow from 192.168.1.0/24 to any port 11211
```

**Alpine (iptables):**
```bash
# Add rule
iptables -A INPUT -p tcp --dport 11211 -s 192.168.1.0/24 -j ACCEPT

# Save rules
/etc/init.d/iptables save
```

## 8. Performance Tuning

### Linux Kernel Parameters

Add to `/etc/sysctl.conf`:
```bash
# Increase max connections
net.core.somaxconn = 1024
net.ipv4.tcp_max_syn_backlog = 1024

# TCP memory
net.ipv4.tcp_mem = 786432 1048576 26777216
net.ipv4.tcp_rmem = 4096 87380 134217728
net.ipv4.tcp_wmem = 4096 65536 134217728

# Apply changes
sudo sysctl -p
```

### Connection Pooling

Configure for high-traffic scenarios:
```bash
# Edit memcached config
# RHEL/CentOS: /etc/sysconfig/memcached
# Debian/Ubuntu: /etc/memcached.conf

# Increase connection limit
-c 10000

# Use multiple threads (CPU cores)
-t 4

# Disable CAS (Compare-And-Swap) if not needed
-C

# Large memory pages (Linux)
-L
```

### Memory Optimization

```bash
# Calculate slab sizes
memcached -vv

# Custom slab configuration
memcached -f 1.25 -n 48

# Monitor slab usage
echo "stats slabs" | nc localhost 11211
```

## Monitoring

### Built-in Statistics

```bash
# Basic stats
echo "stats" | nc localhost 11211

# Slab statistics
echo "stats slabs" | nc localhost 11211

# Item statistics
echo "stats items" | nc localhost 11211

# Connection stats
echo "stats conns" | nc localhost 11211

# Settings
echo "stats settings" | nc localhost 11211
```

### Monitoring Scripts

Create `/usr/local/bin/memcached-stats.sh`:
```bash
#!/bin/bash

echo "=== Memcached Statistics ==="
echo "stats" | nc localhost 11211 | grep -E "STAT (bytes|curr_items|get_hits|get_misses|evictions)"

HITS=$(echo "stats" | nc localhost 11211 | grep "get_hits" | awk '{print $3}')
MISSES=$(echo "stats" | nc localhost 11211 | grep "get_misses" | awk '{print $3}')

if [ $HITS -gt 0 ]; then
    RATIO=$(echo "scale=2; $HITS * 100 / ($HITS + $MISSES)" | bc)
    echo "Hit Ratio: ${RATIO}%"
fi
```

### Nagios/Monitoring Plugin

```bash
#!/bin/bash
# check_memcached.sh

HOST=${1:-localhost}
PORT=${2:-11211}
WARNING=${3:-80}
CRITICAL=${4:-90}

STATS=$(echo "stats" | nc $HOST $PORT)
USED=$(echo "$STATS" | grep "bytes" | head -1 | awk '{print $3}')
LIMIT=$(echo "$STATS" | grep "limit_maxbytes" | awk '{print $3}')

PERCENT=$(echo "scale=2; $USED * 100 / $LIMIT" | bc | cut -d. -f1)

if [ $PERCENT -ge $CRITICAL ]; then
    echo "CRITICAL - Memory usage at ${PERCENT}%"
    exit 2
elif [ $PERCENT -ge $WARNING ]; then
    echo "WARNING - Memory usage at ${PERCENT}%"
    exit 1
else
    echo "OK - Memory usage at ${PERCENT}%"
    exit 0
fi
```

## Client Configuration Examples

### PHP

```php
// Install: pecl install memcached
$memcached = new Memcached();
$memcached->addServer('localhost', 11211);

// With SASL
$memcached->setOption(Memcached::OPT_BINARY_PROTOCOL, true);
$memcached->setSaslAuthData('username', 'password');

// Basic usage
$memcached->set('key', 'value', 3600);
$value = $memcached->get('key');
```

### Python

```python
# Install: pip install python-memcached
import memcache

mc = memcache.Client(['127.0.0.1:11211'], debug=0)
mc.set("key", "value", time=3600)
value = mc.get("key")

# With connection pooling
# Install: pip install pymemcache
from pymemcache.client import base
client = base.Client(('localhost', 11211))
client.set('key', 'value', expire=3600)
```

### Node.js

```javascript
// Install: npm install memcached
const Memcached = require('memcached');
const memcached = new Memcached('localhost:11211');

memcached.set('key', 'value', 3600, (err) => {
    if (err) console.error(err);
});

memcached.get('key', (err, data) => {
    if (err) console.error(err);
    console.log(data);
});
```

### Ruby

```ruby
# Install: gem install dalli
require 'dalli'

dc = Dalli::Client.new('localhost:11211')
dc.set('key', 'value', 3600)
value = dc.get('key')
```

## 6. Troubleshooting

### Common Issues

1. **Connection refused**:
```bash
# Check if service is running
sudo systemctl status memcached

# Check if listening on correct interface
sudo netstat -tlnp | grep 11211

# Test connection
telnet localhost 11211
```

2. **Out of memory errors**:
```bash
# Check current usage
echo "stats" | nc localhost 11211 | grep bytes

# Increase memory limit in configuration
# Restart service after changes
```

3. **Permission denied**:
```bash
# Check user permissions
ps aux | grep memcached

# Fix permissions
sudo chown memcached:memcached /var/run/memcached
```

### Debug Mode

```bash
# Run in foreground with verbose output
memcached -vv -p 11211 -U 0 -l 127.0.0.1

# Maximum verbosity
memcached -vvv
```

## Best Practices

1. **Memory Allocation**
   - Set memory limit based on available RAM
   - Leave enough memory for OS and other services
   - Monitor eviction rates

2. **Security**
   - Bind to localhost unless network access needed
   - Use SASL authentication for network access
   - Implement firewall rules
   - Never expose to public internet

3. **Key Design**
   - Use consistent naming conventions
   - Implement namespacing
   - Keep keys under 250 bytes
   - Set appropriate TTLs

4. **Monitoring**
   - Track hit/miss ratios
   - Monitor evictions
   - Watch connection counts
   - Set up alerts for service failures

## 9. Backup and Restore

### What to Backup

Memcached is an in-memory cache, so there's no persistent data to backup by default. However, you should backup:

1. **Configuration files**:
```bash
# Create backup directory
sudo mkdir -p /backup/memcached/configs

# Backup configurations
sudo cp /etc/sysconfig/memcached /backup/memcached/configs/  # RHEL-based
sudo cp /etc/memcached.conf /backup/memcached/configs/       # Debian-based
sudo cp /etc/conf.d/memcached /backup/memcached/configs/     # Alpine/Arch
```

2. **Service files** (if customized):
```bash
sudo cp /etc/systemd/system/memcached.service /backup/memcached/configs/
```

### Backup Script

```bash
#!/bin/bash
# backup-memcached-config.sh

BACKUP_DIR="/backup/memcached/$(date +%Y%m%d_%H%M%S)"
mkdir -p "$BACKUP_DIR"

# Detect distribution and backup appropriate config
if [ -f /etc/sysconfig/memcached ]; then
    cp /etc/sysconfig/memcached "$BACKUP_DIR/"
elif [ -f /etc/memcached.conf ]; then
    cp /etc/memcached.conf "$BACKUP_DIR/"
elif [ -f /etc/conf.d/memcached ]; then
    cp /etc/conf.d/memcached "$BACKUP_DIR/"
fi

# Backup custom service files
if [ -f /etc/systemd/system/memcached.service ]; then
    cp /etc/systemd/system/memcached.service "$BACKUP_DIR/"
fi

# Save current memcached stats for reference
echo "stats" | nc localhost 11211 > "$BACKUP_DIR/stats.txt"

echo "Configuration backed up to: $BACKUP_DIR"
```

### Restore Procedure

```bash
#!/bin/bash
# restore-memcached-config.sh

BACKUP_DIR="$1"
if [ -z "$BACKUP_DIR" ]; then
    echo "Usage: $0 <backup-directory>"
    exit 1
fi

# Stop memcached
sudo systemctl stop memcached

# Restore configuration
if [ -f "$BACKUP_DIR/memcached" ]; then
    # Detect where to restore
    if [ -d /etc/sysconfig ]; then
        sudo cp "$BACKUP_DIR/memcached" /etc/sysconfig/
    elif [ -d /etc/conf.d ]; then
        sudo cp "$BACKUP_DIR/memcached" /etc/conf.d/
    fi
elif [ -f "$BACKUP_DIR/memcached.conf" ]; then
    sudo cp "$BACKUP_DIR/memcached.conf" /etc/
fi

# Restore service file if exists
if [ -f "$BACKUP_DIR/memcached.service" ]; then
    sudo cp "$BACKUP_DIR/memcached.service" /etc/systemd/system/
    sudo systemctl daemon-reload
fi

# Start memcached
sudo systemctl start memcached

echo "Configuration restored from: $BACKUP_DIR"
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update memcached

# Debian/Ubuntu
sudo apt update && sudo apt upgrade memcached

# Arch Linux
sudo pacman -Syu memcached

# Alpine Linux
apk upgrade memcached

# openSUSE
sudo zypper update memcached
```

### Version Upgrades

1. **Check current version**:
```bash
memcached -V
```

2. **Plan upgrade**:
- Review changelog for breaking changes
- Test in non-production environment
- Plan for cache warming after restart

3. **Perform upgrade**:
```bash
# Backup configuration
./backup-memcached-config.sh

# Upgrade package
sudo apt update && sudo apt upgrade memcached  # Debian/Ubuntu

# Restart service
sudo systemctl restart memcached

# Verify new version
memcached -V
```

### Log Rotation

Configure log rotation for memcached logs:

```bash
# Create /etc/logrotate.d/memcached
sudo tee /etc/logrotate.d/memcached <<EOF
/var/log/memcached.log {
    daily
    rotate 7
    compress
    delaycompress
    missingok
    notifempty
    create 640 memcached memcached
    postrotate
        systemctl reload memcached > /dev/null 2>&1 || true
    endscript
}
EOF
```

### Cleanup Procedures

```bash
# Clear all cache data (WARNING: This removes all cached items!)
echo "flush_all" | nc localhost 11211

# Remove old log files
find /var/log -name "memcached.log.*" -mtime +30 -delete

# Clean up temporary files
rm -f /tmp/memcached.sock.*
```

## Integration Examples

### PHP Integration

```php
<?php
// Using PECL memcached extension
$memcached = new Memcached();
$memcached->addServer('localhost', 11211);

// Connection pooling
$memcached->setOption(Memcached::OPT_TCP_NODELAY, true);
$memcached->setOption(Memcached::OPT_NO_BLOCK, true);

// Example caching function
function getCachedData($key, $callback, $ttl = 3600) {
    global $memcached;
    
    $data = $memcached->get($key);
    if ($memcached->getResultCode() === Memcached::RES_NOTFOUND) {
        $data = $callback();
        $memcached->set($key, $data, $ttl);
    }
    return $data;
}

// Usage
$users = getCachedData('all_users', function() {
    return db_query("SELECT * FROM users");
}, 300);
```

### Python Integration

```python
import memcache
import functools
import time

# Create client with multiple servers
mc = memcache.Client(['127.0.0.1:11211', '127.0.0.1:11212'], debug=0)

# Decorator for caching
def cache_result(expiration=3600):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            cache_key = f"{func.__name__}:{str(args)}:{str(kwargs)}"
            result = mc.get(cache_key)
            
            if result is None:
                result = func(*args, **kwargs)
                mc.set(cache_key, result, time=expiration)
            
            return result
        return wrapper
    return decorator

# Usage
@cache_result(expiration=300)
def expensive_calculation(x, y):
    time.sleep(2)  # Simulate expensive operation
    return x * y + sum(range(1000000))
```

### Node.js Integration

```javascript
const Memcached = require('memcached');

// Create client with options
const memcached = new Memcached('localhost:11211', {
    retries: 10,
    retry: 10000,
    remove: true,
    failOverServers: ['192.168.1.100:11211']
});

// Promisified wrapper
const cache = {
    get: (key) => new Promise((resolve, reject) => {
        memcached.get(key, (err, data) => {
            if (err) reject(err);
            else resolve(data);
        });
    }),
    
    set: (key, value, ttl = 3600) => new Promise((resolve, reject) => {
        memcached.set(key, value, ttl, (err) => {
            if (err) reject(err);
            else resolve(true);
        });
    }),
    
    delete: (key) => new Promise((resolve, reject) => {
        memcached.del(key, (err) => {
            if (err) reject(err);
            else resolve(true);
        });
    })
};

// Usage with async/await
async function getCachedUser(userId) {
    const cacheKey = `user:${userId}`;
    
    try {
        // Try cache first
        let user = await cache.get(cacheKey);
        
        if (!user) {
            // Cache miss - fetch from database
            user = await db.query('SELECT * FROM users WHERE id = ?', [userId]);
            await cache.set(cacheKey, user, 300); // Cache for 5 minutes
        }
        
        return user;
    } catch (error) {
        console.error('Cache error:', error);
        // Fallback to database
        return await db.query('SELECT * FROM users WHERE id = ?', [userId]);
    }
}
```

### Ruby on Rails Integration

```ruby
# config/environments/production.rb
config.cache_store = :mem_cache_store, 
  'localhost:11211', 
  { 
    namespace: 'myapp',
    expires_in: 1.hour,
    compress: true,
    pool_size: 5,
    pool_timeout: 5
  }

# app/models/user.rb
class User < ApplicationRecord
  def expensive_calculation
    Rails.cache.fetch("user_#{id}_calculation", expires_in: 12.hours) do
      # Expensive calculation here
      sleep 2
      posts.count * comments.count
    end
  end
end

# app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
  around_action :cache_control
  
  private
  
  def cache_control
    if user_signed_in?
      yield
    else
      # Cache pages for non-authenticated users
      expires_in 5.minutes, public: true
      yield
    end
  end
end
```

## Additional Resources

- [Official Documentation](https://memcached.org/)
- [GitHub Repository](https://github.com/memcached/memcached)
- [Protocol Specification](https://github.com/memcached/memcached/blob/master/doc/protocol.txt)
- [Best Practices Guide](https://github.com/memcached/memcached/wiki/Programming)
- [Performance Tuning](https://github.com/memcached/memcached/wiki/Performance)

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
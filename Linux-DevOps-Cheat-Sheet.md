# Linux for DevOps – High Retrieval & Teaching Cheat Sheet

This document is designed for **DevOps use-cases** (servers, deployments, troubleshooting) with focus on **fast recall and teaching**. It assumes you already know Linux but need quick retrieval for interviews, KT sessions, or explaining under pressure. Each section follows a fixed pattern with commands, examples, and ready-made speaking lines.

---

## Table of Contents

1. [Left-Side Topic Map (Overview)](#left-side-topic-map-overview)
2. [Filesystem Basics](#filesystem-basics)
3. [File Operations](#file-operations)
4. [Viewing Files](#viewing-files)
5. [Find & Search](#find--search)
6. [Permissions & Ownership](#permissions--ownership)
7. [Users & Groups](#users--groups)
8. [Process Management](#process-management)
9. [Systemd & Services](#systemd--services)
10. [Logs](#logs)
11. [Disk Usage & Filesystem](#disk-usage--filesystem)
12. [Networking Basics](#networking-basics)
13. [Ports & Connections](#ports--connections)
14. [SSH & Key-based Auth](#ssh--key-based-auth)
15. [Packaging & Updates](#packaging--updates)
16. [Environment Variables](#environment-variables)
17. [Shell Scripting Basics](#shell-scripting-basics)
18. [Cron & Scheduling](#cron--scheduling)
19. [Archiving & Compression](#archiving--compression)
20. [Text Processing](#text-processing)
21. [cURL & Wget](#curl--wget)
22. [System Monitoring](#system-monitoring)
23. [Resource Limits](#resource-limits)
24. [Common Troubleshooting Flow](#common-troubleshooting-flow)
25. [Closing Notes](#closing-notes)

---

## Left-Side Topic Map (Overview)

| Topic | Trigger Idea (1 line) | Most Used Command |
|-------|----------------------|-------------------|
| Filesystem Basics | Navigate directories and check current location | `ls -lah` |
| File Operations | Copy, move, delete files and directories | `cp -r`, `mv`, `rm -rf` |
| Viewing Files | Read file contents in different ways | `cat`, `less`, `tail -f` |
| Find & Search | Locate files and search content in files | `find`, `grep -r` |
| Permissions & Ownership | Control file access and ownership | `chmod`, `chown` |
| Users & Groups | Manage system users and group memberships | `useradd`, `usermod`, `id` |
| Process Management | View, monitor, and control running processes | `ps aux`, `top`, `kill` |
| Systemd & Services | Start, stop, manage system services | `systemctl status` |
| Logs | View system and application logs | `journalctl -u`, `tail -f` |
| Disk Usage & Filesystem | Check disk space and mounted filesystems | `df -h`, `du -sh` |
| Networking Basics | Check network interfaces and connectivity | `ip addr`, `ping`, `ss` |
| Ports & Connections | See listening ports and active connections | `ss -tulpn`, `netstat`, `lsof` |
| SSH & Key-based Auth | Secure remote access to servers | `ssh user@host`, `ssh-keygen` |
| Packaging & Updates | Install and update software packages | `apt install`, `yum install` |
| Environment Variables | Set and view environment configuration | `export VAR=value`, `env` |
| Shell Scripting Basics | Automate tasks with bash scripts | `#!/bin/bash`, variables, loops |
| Cron & Scheduling | Schedule automated tasks | `crontab -e`, `cron syntax` |
| Archiving & Compression | Backup and compress files | `tar -czf`, `tar -xzf` |
| Text Processing | Parse and transform text data | `grep`, `sed`, `awk` |
| cURL & Wget | Make HTTP requests and download files | `curl`, `wget` |
| System Monitoring | Check system health and resource usage | `uptime`, `free -h`, `vmstat` |
| Resource Limits | Control process resource consumption | `ulimit`, `/etc/security/limits.conf` |
| Common Troubleshooting Flow | Systematic approach to debug issues | Check logs → processes → network |

---

## Filesystem Basics

🔑 **Primary Trigger:** "Navigate the filesystem, list files, and know where you are – foundation of all server work."

### 1. What it is (Simple Definition)

- Basic commands to move around directories and view contents
- Every server task starts here: navigating to logs, configs, apps
- Essential for any SSH session or troubleshooting

### 2. Why / When DevOps Uses This

- SSH into server: first thing is `pwd` to see where you are
- Navigate to application directories (`cd /opt/myapp`)
- List files to find configs, logs, or binaries
- Check file sizes, permissions, and timestamps
- Locate hidden files (dotfiles) with `ls -a`

### 3. Key Commands / Patterns

- `pwd` - print working directory
- `cd <path>` - change directory
- `cd -` - go to previous directory
- `ls` - list directory contents
- `ls -lah` - detailed list with hidden files and human-readable sizes
- `ls -lt` - sort by modification time (newest first)
- `tree` - show directory tree structure

### 4. Minimal Examples

```bash
# Check current location
pwd

# List all files including hidden, with details
ls -lah

# Navigate to logs directory
cd /var/log

# Go back to previous directory
cd -

# List files sorted by time (newest first)
ls -lt

# Show directory structure
tree -L 2 /etc/nginx

# List only directories
ls -d */
```

### 5. Example Scenario

- SSH into production web server
- Run `pwd` to confirm location (likely `/home/username`)
- Navigate to application: `cd /opt/webapp`
- List files to find config: `ls -lah`
- Check log directory: `cd logs && ls -lt` (newest logs first)
- Return to app root: `cd ..`

### 6. Common KT / Interview Line

"Every troubleshooting session starts with pwd to orient myself, then ls -lah to see what files exist before diving deeper."

### 7. Closing Line Trigger

"In short, these basic navigation commands are muscle memory for any DevOps engineer working on Linux servers."

---

## File Operations

🔑 **Primary Trigger:** "Copy, move, delete files – essential for deploying, backing up, and cleaning up servers."

### 1. What it is (Simple Definition)

- Commands to manipulate files and directories
- Core operations for deployments, backups, and maintenance
- Understanding recursive operations and safety flags

### 2. Why / When DevOps Uses This

- Deploy application: copy files to `/opt/app` or `/var/www`
- Backup configs before changes: `cp config.yml config.yml.bak`
- Move old logs to archive location
- Clean up old deployment artifacts: `rm -rf old-release/`
- Create directory structures for new applications

### 3. Key Commands / Patterns

- `cp <src> <dest>` - copy file
- `cp -r <dir> <dest>` - copy directory recursively
- `mv <src> <dest>` - move/rename file or directory
- `rm <file>` - remove file
- `rm -rf <dir>` - remove directory recursively (force)
- `mkdir -p <path>` - create directory with parents
- `touch <file>` - create empty file or update timestamp

### 4. Minimal Examples

```bash
# Backup config file before editing
cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak

# Copy entire application directory
cp -r /opt/app /opt/app-backup

# Move old logs to archive
mv /var/log/app.log.1 /var/log/archive/

# Rename file
mv old-name.txt new-name.txt

# Create directory structure
mkdir -p /opt/myapp/{logs,config,data}

# Remove old deployment
rm -rf /opt/myapp/releases/v1.0.0

# Create empty file
touch /var/log/myapp/app.log
```

### 5. Example Scenario

- Deploying new application version
- Create directory: `mkdir -p /opt/myapp/v2.0.0`
- Copy artifacts: `cp -r dist/* /opt/myapp/v2.0.0/`
- Backup old version: `mv /opt/myapp/current /opt/myapp/v1.0.0`
- Update symlink: `ln -sf /opt/myapp/v2.0.0 /opt/myapp/current`
- Remove ancient releases: `rm -rf /opt/myapp/v0.*`

### 6. Common KT / Interview Line

"Before any production change, I always backup the original file with .bak extension so rollback is instant if needed."

### 7. Closing Line Trigger

"In short, file operations are the building blocks of deployments, backups, and server maintenance."

---

## Viewing Files

🔑 **Primary Trigger:** "Read file contents – configs, logs, scripts – using the right tool for the job."

### 1. What it is (Simple Definition)

- Commands to view file contents without editing
- Different tools for different needs: full view, pagination, live tail
- Essential for reading configs and monitoring logs

### 2. Why / When DevOps Uses This

- Check configuration files before editing
- View application logs to debug issues
- Follow logs in real-time during deployments
- Quickly see first/last lines of large log files
- Search through logs while viewing them

### 3. Key Commands / Patterns

- `cat <file>` - print entire file to stdout
- `less <file>` - paginated viewing (space/arrows to navigate)
- `head -n 20 <file>` - show first 20 lines
- `tail -n 50 <file>` - show last 50 lines
- `tail -f <file>` - follow file in real-time (live tail)
- `tail -f <file> | grep ERROR` - live tail with filtering

### 4. Minimal Examples

```bash
# View entire config file
cat /etc/nginx/nginx.conf

# Paginated view of large file (space to scroll, q to quit)
less /var/log/syslog

# See first 20 lines
head -n 20 /var/log/application.log

# See last 50 lines
tail -n 50 /var/log/application.log

# Follow log in real-time
tail -f /var/log/myapp/app.log

# Follow log and filter for errors
tail -f /var/log/myapp/app.log | grep ERROR

# Follow multiple files
tail -f /var/log/nginx/access.log /var/log/nginx/error.log

# Show last 100 lines and then follow
tail -n 100 -f /var/log/app.log
```

### 5. Example Scenario

- Application deployment in progress
- Open new terminal window
- Run `tail -f /var/log/myapp/app.log` to watch logs live
- Watch for startup messages, errors, or warnings
- In another window, test API endpoints
- Logs show request processing in real-time
- Filter for errors: `tail -f app.log | grep -i error`

### 6. Common KT / Interview Line

"During any deployment or troubleshooting, I always have tail -f running on the application log to see what's happening in real time."

### 7. Closing Line Trigger

"In short, viewing commands let us read configs and monitor logs without editing, keeping operations safe and visible."

---

## Find & Search

🔑 **Primary Trigger:** "Locate files and search content – find where things are and what's inside them."

### 1. What it is (Simple Definition)

- `find` locates files by name, type, size, or modification time
- `grep` searches for text patterns within files
- Combined, they're powerful for hunting configs, logs, or issues

### 2. Why / When DevOps Uses This

- Find all config files: `find /etc -name "*.conf"`
- Search logs for errors: `grep ERROR /var/log/app.log`
- Locate files modified in last hour (recent changes)
- Find large files consuming disk space
- Search recursively through all application logs

### 3. Key Commands / Patterns

- `find <path> -name <pattern>` - find by filename
- `find <path> -type f/d` - find files or directories
- `find <path> -mtime -1` - modified in last 24 hours
- `find <path> -size +100M` - files larger than 100MB
- `grep <pattern> <file>` - search pattern in file
- `grep -r <pattern> <dir>` - recursive search in directory
- `grep -i <pattern>` - case-insensitive search
- `grep -v <pattern>` - invert match (exclude pattern)

### 4. Minimal Examples

```bash
# Find all .conf files in /etc
find /etc -name "*.conf"

# Find files modified in last hour
find /var/log -mtime -1

# Find large files (>100MB)
find /opt -type f -size +100M

# Find empty directories
find /tmp -type d -empty

# Search for ERROR in log file
grep "ERROR" /var/log/application.log

# Search recursively, case-insensitive
grep -ri "database connection failed" /var/log/

# Search with line numbers
grep -n "ERROR" app.log

# Search excluding pattern
grep -v "INFO" app.log

# Count occurrences
grep -c "ERROR" app.log

# Combine find and grep
find /var/log -name "*.log" -exec grep -l "ERROR" {} \;

# Find files and show their size
find /opt/app -type f -exec du -h {} \; | sort -rh | head -20
```

### 5. Example Scenario

- Application throwing 500 errors
- Don't know which log file contains the error
- Search all logs: `grep -r "500 Internal Server Error" /var/log/myapp/`
- Find specific error log: `find /var/log/myapp -name "error*.log"`
- Count error occurrences today: `grep -c "ERROR" error.log`
- Check when error file was last modified: `find /var/log -name "error.log" -ls`

### 6. Common KT / Interview Line

"When hunting down issues, I combine find to locate files and grep to search content – essential tools for log analysis and debugging."

### 7. Closing Line Trigger

"In short, find and grep are your search engine for the Linux filesystem."

---

## Permissions & Ownership

🔑 **Primary Trigger:** "Control who can read, write, or execute files – critical for security and application functionality."

### 1. What it is (Simple Definition)

- File permissions control read (r), write (w), execute (x) access
- Ownership defines user and group that own the file
- Proper permissions prevent security issues and access errors

### 2. Why / When DevOps Uses This

- Make scripts executable: `chmod +x deploy.sh`
- Secure config files: `chmod 600 /etc/app/secrets.conf`
- Fix web server file ownership: `chown -R nginx:nginx /var/www`
- Ensure log files are writable by application
- Troubleshoot permission denied errors

### 3. Key Commands / Patterns

- `chmod <mode> <file>` - change permissions
- `chmod 755 file` - rwxr-xr-x (common for executables)
- `chmod 644 file` - rw-r--r-- (common for files)
- `chmod 600 file` - rw------- (secure, owner only)
- `chmod +x file` - add execute permission
- `chown user:group file` - change owner and group
- `chown -R user:group dir` - recursive ownership change
- `chgrp group file` - change group only

### 4. Minimal Examples

```bash
# Make script executable
chmod +x deploy.sh

# Secure config file (owner read/write only)
chmod 600 /etc/myapp/database.conf

# Standard file permissions (owner rw, others read)
chmod 644 /etc/nginx/nginx.conf

# Standard directory permissions
chmod 755 /opt/myapp

# Change owner and group recursively
chown -R appuser:appgroup /opt/myapp

# Change only group
chgrp nginx /var/log/nginx/access.log

# Remove write permissions for group and others
chmod go-w /etc/important.conf

# View permissions in detail
ls -lah /opt/myapp

# Find files with specific permissions
find /opt -perm 777
```

### 5. Example Scenario

- Deploy application as `appuser`
- Application can't write to log file (Permission denied)
- Check ownership: `ls -l /var/log/myapp/app.log`
- File owned by root, not appuser
- Fix ownership: `chown appuser:appuser /var/log/myapp/app.log`
- Ensure writable: `chmod 644 /var/log/myapp/app.log`
- Application now logs successfully

### 6. Common KT / Interview Line

"Most Permission denied errors are fixed with proper chown and chmod – ensuring the right user owns files with correct access permissions."

### 7. Closing Line Trigger

"In short, permissions and ownership control access and are crucial for both security and functionality."

---

## Users & Groups

🔑 **Primary Trigger:** "Manage system users and groups – essential for multi-user environments and service accounts."

### 1. What it is (Simple Definition)

- Users are accounts that can log in or run processes
- Groups organize users for shared permissions
- Service accounts run applications without login access

### 2. Why / When DevOps Uses This

- Create service accounts for applications (nginx, postgres)
- Add users to groups for shared access
- Set up deployment user with sudo privileges
- Check which user/group is running a process
- Manage SSH access for team members

### 3. Key Commands / Patterns

- `useradd <username>` - create user
- `useradd -m -s /bin/bash <user>` - create with home and shell
- `passwd <username>` - set password
- `usermod -aG <group> <user>` - add user to group
- `userdel <username>` - delete user
- `groupadd <groupname>` - create group
- `id <username>` - show user ID and groups
- `whoami` - show current user
- `who` or `w` - show logged-in users

### 4. Minimal Examples

```bash
# Create application user
sudo useradd -m -s /bin/bash appuser

# Set password
sudo passwd appuser

# Create user without home directory (service account)
sudo useradd -r -s /usr/sbin/nologin nginx

# Add user to group
sudo usermod -aG docker jenkins

# Add user to sudo group
sudo usermod -aG sudo deployuser

# Check user information
id appuser

# Check current user
whoami

# Show all groups for user
groups appuser

# Create group
sudo groupadd developers

# Delete user
sudo userdel -r olduser

# See who's logged in
who
w
```

### 5. Example Scenario

- Setting up Jenkins server
- Create jenkins user: `sudo useradd -m -s /bin/bash jenkins`
- Add to docker group: `sudo usermod -aG docker jenkins`
- Set ownership of Jenkins directory: `sudo chown -R jenkins:jenkins /var/lib/jenkins`
- Jenkins can now run docker commands without sudo
- Verify: `sudo -u jenkins docker ps`

### 6. Common KT / Interview Line

"Creating dedicated service accounts with appropriate group memberships is standard practice for running applications securely."

### 7. Closing Line Trigger

"In short, user and group management enables secure multi-user environments and proper application isolation."

---

## Process Management

🔑 **Primary Trigger:** "View and control running processes – essential for debugging performance and stopping runaway processes."

### 1. What it is (Simple Definition)

- Processes are running programs on the system
- Each process has a PID (Process ID), owner, and resource usage
- Commands to view, monitor, and control process execution

### 2. Why / When DevOps Uses This

- Check what processes are consuming CPU/memory
- Find PID of application to restart it
- Kill stuck or runaway processes
- Monitor resource usage during load tests
- Identify which user is running a process

### 3. Key Commands / Patterns

- `ps aux` - list all processes
- `ps aux | grep <name>` - find specific process
- `top` - interactive process monitor (press q to quit)
- `htop` - enhanced interactive monitor (install separately)
- `kill <PID>` - gracefully terminate process (SIGTERM)
- `kill -9 <PID>` - force kill process (SIGKILL)
- `killall <name>` - kill all processes by name
- `pkill <pattern>` - kill processes matching pattern
- `pgrep <pattern>` - find PIDs by pattern

### 4. Minimal Examples

```bash
# List all processes
ps aux

# Find specific process
ps aux | grep nginx

# Show process tree
ps auxf

# Interactive monitoring (top processes by CPU)
top

# Enhanced monitoring (if installed)
htop

# Find PID
pgrep nginx

# Kill process gracefully
kill 12345

# Force kill process
kill -9 12345

# Kill all processes by name
killall -9 java

# Kill by pattern
pkill -f "myapp.jar"

# Show only specific user's processes
ps -u appuser

# Sort by memory usage
ps aux --sort=-%mem | head -20

# Show threads for a process
ps -T -p 12345
```

### 5. Example Scenario

- Application server becomes unresponsive
- Run `top` to see CPU/memory usage
- Java process at 100% CPU
- Find PID: `ps aux | grep java` shows PID 8432
- Try graceful shutdown: `kill 8432`
- Wait 30 seconds, check if still running: `ps -p 8432`
- If still running, force kill: `kill -9 8432`
- Restart service: `systemctl start myapp`

### 6. Common KT / Interview Line

"When troubleshooting performance issues, top and ps aux are my first tools to identify which processes are consuming resources."

### 7. Closing Line Trigger

"In short, process management gives visibility and control over everything running on the system."

---

## Systemd & Services

🔑 **Primary Trigger:** "Manage system services with systemctl – start, stop, enable, and check status of applications."

### 1. What it is (Simple Definition)

- Systemd is modern init system managing services on Linux
- Services are background processes (daemons) managed by systemd
- `systemctl` is the command to control services

### 2. Why / When DevOps Uses This

- Start/stop applications: `systemctl start nginx`
- Enable services to start on boot
- Check if service is running and healthy
- Restart services after configuration changes
- View service logs with journalctl integration

### 3. Key Commands / Patterns

- `systemctl status <service>` - check service status
- `systemctl start <service>` - start service
- `systemctl stop <service>` - stop service
- `systemctl restart <service>` - restart service
- `systemctl reload <service>` - reload config without restart
- `systemctl enable <service>` - enable at boot
- `systemctl disable <service>` - disable at boot
- `systemctl is-active <service>` - check if running
- `systemctl list-units --type=service` - list all services

### 4. Minimal Examples

```bash
# Check service status
systemctl status nginx

# Start service
sudo systemctl start nginx

# Stop service
sudo systemctl stop nginx

# Restart service
sudo systemctl restart nginx

# Reload configuration
sudo systemctl reload nginx

# Enable service to start on boot
sudo systemctl enable nginx

# Disable service from starting on boot
sudo systemctl disable nginx

# Check if service is running
systemctl is-active nginx

# Check if service is enabled
systemctl is-enabled nginx

# List all services
systemctl list-units --type=service

# List failed services
systemctl --failed

# Show service configuration
systemctl cat nginx

# Edit service unit file
sudo systemctl edit nginx

# Reload systemd after unit file changes
sudo systemctl daemon-reload
```

### 5. Example Scenario

- Deploy new version of application
- Stop service: `sudo systemctl stop myapp`
- Update application files
- Reload systemd: `sudo systemctl daemon-reload`
- Start service: `sudo systemctl start myapp`
- Check status: `systemctl status myapp`
- If running, check logs: `journalctl -u myapp -n 50`
- Ensure enabled at boot: `sudo systemctl enable myapp`

### 6. Common KT / Interview Line

"Systemctl is how we manage all services in modern Linux – checking status, restarting after deployments, and ensuring services start on boot."

### 7. Closing Line Trigger

"In short, systemd and systemctl are central to service management in modern Linux distributions."

---

## Logs

🔑 **Primary Trigger:** "View system and application logs – first place to look when troubleshooting issues."

### 1. What it is (Simple Definition)

- Logs record system events, application output, and errors
- Systemd logs managed by journalctl
- Traditional logs in /var/log/ directory
- Essential for debugging and auditing

### 2. Why / When DevOps Uses This

- Debug application errors and crashes
- Check service startup issues
- Monitor authentication attempts
- Track system events and kernel messages
- Analyze performance issues and timeouts

### 3. Key Commands / Patterns

- `journalctl` - view systemd logs
- `journalctl -u <service>` - logs for specific service
- `journalctl -f` - follow logs in real-time
- `journalctl -n 100` - show last 100 lines
- `journalctl --since "1 hour ago"` - time-filtered logs
- `tail -f /var/log/syslog` - follow traditional syslog
- `less /var/log/messages` - view system messages

### 4. Minimal Examples

```bash
# View all systemd logs
journalctl

# View logs for specific service
journalctl -u nginx

# Follow logs in real-time
journalctl -u myapp -f

# Show last 50 lines
journalctl -u myapp -n 50

# Logs from last hour
journalctl --since "1 hour ago"

# Logs from today
journalctl --since today

# Logs between time range
journalctl --since "2024-01-29 10:00:00" --until "2024-01-29 12:00:00"

# Show only error messages
journalctl -p err

# Logs for specific user
journalctl _UID=1000

# Traditional syslog
tail -f /var/log/syslog

# Apache/nginx access logs
tail -f /var/log/nginx/access.log

# Application-specific logs
tail -f /var/log/myapp/application.log

# Search logs
journalctl -u nginx | grep ERROR

# Boot logs
journalctl -b
```

### 5. Example Scenario

- Web application returning 500 errors
- Check nginx logs: `journalctl -u nginx -n 100`
- No errors in nginx, check application
- View app logs: `journalctl -u myapp -f`
- See "Database connection refused" errors
- Check database service: `systemctl status postgresql`
- Database stopped, start it: `sudo systemctl start postgresql`
- Monitor app logs: errors stop appearing

### 6. Common KT / Interview Line

"Whenever troubleshooting, I start with journalctl -u <service> to see what the application is logging, then work backwards from there."

### 7. Closing Line Trigger

"In short, logs are the primary source of truth for debugging – always check logs first."

---

## Disk Usage & Filesystem

🔑 **Primary Trigger:** "Check disk space and mounted filesystems – prevent disk full issues that break applications."

### 1. What it is (Simple Definition)

- Monitor available disk space and usage
- View mounted filesystems and mount points
- Identify large files and directories consuming space

### 2. Why / When DevOps Uses This

- Prevent "disk full" errors that crash applications
- Find what's consuming disk space (logs, caches)
- Verify NFS or external storage mounts
- Check before deploying large applications
- Monitor disk usage alerts from monitoring systems

### 3. Key Commands / Patterns

- `df -h` - show filesystem disk usage (human-readable)
- `df -i` - show inode usage
- `du -sh <dir>` - show directory size
- `du -h <dir> | sort -rh | head -20` - largest items in directory
- `mount` - show mounted filesystems
- `mount <device> <mountpoint>` - mount filesystem
- `umount <mountpoint>` - unmount filesystem

### 4. Minimal Examples

```bash
# Check disk usage for all filesystems
df -h

# Check inode usage (sometimes runs out even with space left)
df -i

# Check size of specific directory
du -sh /var/log

# Find largest directories
du -h /opt | sort -rh | head -20

# Find largest files in /var
find /var -type f -exec du -h {} \; | sort -rh | head -20

# Check what's using space in current directory
du -h --max-depth=1 . | sort -rh

# Show all mounted filesystems
mount

# Show mounted filesystems (cleaner output)
findmnt

# Check specific mount point
df -h /var/log

# Mount NFS share
sudo mount -t nfs server:/share /mnt/nfs

# Unmount
sudo umount /mnt/nfs

# Show disk I/O statistics
iostat -x 1
```

### 5. Example Scenario

- Application stops writing logs (disk full error)
- Check disk space: `df -h` shows /var at 100%
- Find what's consuming space: `du -sh /var/*`
- `/var/log` is 50GB
- Check largest logs: `du -sh /var/log/* | sort -rh | head -10`
- Old application logs not rotated: 30GB
- Remove old logs: `sudo rm /var/log/myapp/*.log.old`
- Verify space: `df -h` now shows 70% usage
- Application resumes logging

### 6. Common KT / Interview Line

"Running out of disk space is a common production issue – df -h and du -sh help identify the problem quickly before it impacts users."

### 7. Closing Line Trigger

"In short, monitoring disk usage prevents critical failures and helps manage storage resources efficiently."

---

## Networking Basics

🔑 **Primary Trigger:** "Check network interfaces, connectivity, and basic network troubleshooting."

### 1. What it is (Simple Definition)

- Commands to view network configuration and test connectivity
- Essential for troubleshooting network issues
- Verify server can reach external services

### 2. Why / When DevOps Uses This

- Check if server has network connectivity
- Verify correct IP address configuration
- Test connectivity to database, APIs, or external services
- Troubleshoot "connection refused" or "timeout" errors
- Verify DNS resolution

### 3. Key Commands / Patterns

- `ip addr` or `ip a` - show network interfaces
- `ip route` - show routing table
- `ifconfig` - legacy command to show interfaces
- `ping <host>` - test connectivity
- `ping -c 4 <host>` - send only 4 packets
- `traceroute <host>` - trace route to destination
- `nslookup <domain>` - DNS lookup
- `dig <domain>` - detailed DNS query
- `curl -I <url>` - test HTTP connectivity

### 4. Minimal Examples

```bash
# Show network interfaces and IP addresses
ip addr show

# Show only specific interface
ip addr show eth0

# Show routing table
ip route

# Test connectivity
ping google.com

# Send only 4 ping packets
ping -c 4 8.8.8.8

# Trace route to destination
traceroute google.com

# DNS lookup
nslookup google.com

# Detailed DNS query
dig google.com

# Check if port is reachable
curl -v telnet://db-server:5432

# Test HTTP endpoint
curl -I https://api.example.com

# Show DNS servers
cat /etc/resolv.conf

# Legacy interface view
ifconfig

# Bring interface up/down (requires root)
sudo ip link set eth0 up
sudo ip link set eth0 down
```

### 5. Example Scenario

- Application can't connect to database
- Check network interface: `ip addr` shows eth0 has IP
- Test connectivity to DB server: `ping db-server.local`
- Ping works, so network is up
- Test specific port: `curl -v telnet://db-server.local:5432`
- Connection refused – database likely not running
- SSH to DB server and check: `systemctl status postgresql`
- Database stopped, start it to resolve issue

### 6. Common KT / Interview Line

"When debugging connectivity issues, I use ping to verify basic network, then test specific ports with curl or telnet."

### 7. Closing Line Trigger

"In short, basic networking commands help isolate where connectivity breaks down."

---

## Ports & Connections

🔑 **Primary Trigger:** "See listening ports and active connections – essential for troubleshooting network services."

### 1. What it is (Simple Definition)

- Commands to show which ports are listening
- View active network connections
- Identify which process is using a port

### 2. Why / When DevOps Uses This

- Verify service is listening on expected port
- Check for port conflicts before starting service
- Identify established connections to database or API
- Troubleshoot "port already in use" errors
- Security auditing of open ports

### 3. Key Commands / Patterns

- `ss -tulpn` - show listening TCP/UDP ports (modern)
- `netstat -tulpn` - show listening ports (legacy)
- `lsof -i :8080` - show what's using port 8080
- `lsof -i -P -n` - show all network connections
- `ss -tnp` - show established TCP connections
- `netstat -an | grep LISTEN` - show listening ports

### 4. Minimal Examples

```bash
# Show all listening TCP/UDP ports with process info
sudo ss -tulpn

# Show established connections
ss -tnp

# Show all network connections
sudo lsof -i -P -n

# Check what's using specific port
sudo lsof -i :8080

# Check if port is listening
ss -tulpn | grep :8080

# Show connections to specific host
ss -tnp | grep 192.168.1.100

# Count connections by state
ss -tan | awk '{print $1}' | sort | uniq -c

# Show TCP connections with process names
sudo netstat -tulpn

# Show only listening ports
ss -tln

# Show established connections with process info
sudo ss -tp

# Find which process is using port 3306
sudo lsof -i TCP:3306
```

### 5. Example Scenario

- Starting nginx returns "port 80 already in use"
- Check what's using port 80: `sudo lsof -i :80`
- Shows Apache is running on port 80
- Stop Apache: `sudo systemctl stop apache2`
- Verify port is free: `sudo lsof -i :80` returns nothing
- Start nginx: `sudo systemctl start nginx`
- Confirm nginx listening: `sudo ss -tulpn | grep :80`

### 6. Common KT / Interview Line

"When troubleshooting network services, ss -tulpn shows exactly what's listening on which ports, helping identify conflicts or verify services are up."

### 7. Closing Line Trigger

"In short, port and connection commands reveal the state of network services and active connections."

---

## SSH & Key-based Auth

🔑 **Primary Trigger:** "Secure remote access to servers using SSH keys – standard practice for DevOps."

### 1. What it is (Simple Definition)

- SSH provides secure remote shell access to servers
- Key-based authentication is more secure than passwords
- Essential for accessing cloud servers and automation

### 2. Why / When DevOps Uses This

- Connect to production servers securely
- Automate deployments with passwordless SSH
- Set up CI/CD pipelines to deploy to servers
- Manage multiple servers with SSH config
- Troubleshoot and maintain remote systems

### 3. Key Commands / Patterns

- `ssh user@host` - connect to remote server
- `ssh-keygen` - generate SSH key pair
- `ssh-copy-id user@host` - copy public key to server
- `ssh -i <keyfile> user@host` - connect with specific key
- `scp <file> user@host:<path>` - copy file to remote
- `~/.ssh/config` - SSH client configuration

### 4. Minimal Examples

```bash
# Connect to server
ssh ubuntu@server.example.com

# Generate SSH key pair
ssh-keygen -t rsa -b 4096 -C "your-email@example.com"

# Generate ED25519 key (modern, recommended)
ssh-keygen -t ed25519 -C "your-email@example.com"

# Copy public key to server
ssh-copy-id ubuntu@server.example.com

# Connect with specific key
ssh -i ~/.ssh/deploy_key ubuntu@server.example.com

# Copy file to remote server
scp app.tar.gz ubuntu@server:/opt/app/

# Copy from remote to local
scp ubuntu@server:/var/log/app.log ./

# Copy directory recursively
scp -r dist/ ubuntu@server:/opt/app/

# Execute command on remote server
ssh ubuntu@server "systemctl status nginx"

# SSH with port forwarding
ssh -L 8080:localhost:80 ubuntu@server

# SSH config (~/.ssh/config)
# Host prod
#   HostName server.example.com
#   User ubuntu
#   IdentityFile ~/.ssh/prod_key
# Then connect with: ssh prod
```

### 5. Example Scenario

- Need to deploy to production servers
- Generate SSH key: `ssh-keygen -t ed25519`
- Copy to servers: `ssh-copy-id deploy@prod1.example.com`
- Test connection: `ssh deploy@prod1`
- Works without password prompt
- Add to CI/CD: store private key as secret
- CI/CD can now deploy using SSH key
- No passwords stored anywhere

### 6. Common KT / Interview Line

"SSH key-based authentication is standard in DevOps – it's more secure than passwords and enables automated deployments without manual logins."

### 7. Closing Line Trigger

"In short, SSH with key-based auth is the secure foundation for all remote server management."

---

## Packaging & Updates

🔑 **Primary Trigger:** "Install, update, and manage software packages using package managers."

### 1. What it is (Simple Definition)

- Package managers install and update software
- Debian/Ubuntu use `apt`, Red Hat/CentOS use `yum`/`dnf`
- Manage dependencies automatically

### 2. Why / When DevOps Uses This

- Install required software (nginx, docker, python)
- Update packages for security patches
- Search for available packages
- Remove unwanted software
- Set up new servers with necessary tools

### 3. Key Commands / Patterns

**APT (Debian/Ubuntu):**
- `apt update` - update package lists
- `apt upgrade` - upgrade packages
- `apt install <package>` - install package
- `apt remove <package>` - remove package
- `apt search <keyword>` - search packages

**YUM/DNF (Red Hat/CentOS):**
- `yum update` or `dnf update` - update packages
- `yum install <package>` - install package
- `yum remove <package>` - remove package
- `yum search <keyword>` - search packages

### 4. Minimal Examples

```bash
# --- APT (Debian/Ubuntu) ---

# Update package lists
sudo apt update

# Upgrade all packages
sudo apt upgrade -y

# Install package
sudo apt install nginx -y

# Install multiple packages
sudo apt install git curl wget -y

# Remove package
sudo apt remove apache2 -y

# Remove package and config files
sudo apt purge apache2 -y

# Search for package
apt search docker

# Show package info
apt show nginx

# List installed packages
apt list --installed

# Clean package cache
sudo apt clean

# --- YUM/DNF (Red Hat/CentOS) ---

# Update packages
sudo yum update -y

# Install package
sudo yum install nginx -y

# Remove package
sudo yum remove apache2 -y

# Search for package
yum search docker

# List installed packages
yum list installed

# Clean cache
sudo yum clean all

# --- DNF (modern Red Hat) ---
sudo dnf install nginx -y
sudo dnf update -y
```

### 5. Example Scenario

- Setting up new Ubuntu server
- Update package lists: `sudo apt update`
- Upgrade existing packages: `sudo apt upgrade -y`
- Install required tools: `sudo apt install nginx docker.io python3-pip git -y`
- Verify installation: `nginx -v`, `docker --version`
- Server ready with all necessary software
- Schedule regular updates via cron or automation

### 6. Common KT / Interview Line

"Before installing any software, always run apt update to refresh package lists, then apt install with -y flag for non-interactive installation."

### 7. Closing Line Trigger

"In short, package managers simplify software installation and keep systems updated with security patches."

---

## Environment Variables

🔑 **Primary Trigger:** "Set configuration values in environment – used by applications for runtime settings."

### 1. What it is (Simple Definition)

- Environment variables store configuration values
- Available to processes and shells
- Common for API keys, paths, application settings

### 2. Why / When DevOps Uses This

- Configure applications without hardcoding values
- Set different configs per environment (dev/staging/prod)
- Store temporary credentials or tokens
- Modify PATH for custom binaries
- Pass configuration to containers and scripts

### 3. Key Commands / Patterns

- `export VAR=value` - set variable in current shell
- `env` - list all environment variables
- `echo $VAR` - print variable value
- `printenv VAR` - print specific variable
- `unset VAR` - remove variable
- `~/.bashrc` or `~/.bash_profile` - persistent variables
- `/etc/environment` - system-wide variables

### 4. Minimal Examples

```bash
# Set environment variable
export DATABASE_URL="postgres://localhost:5432/mydb"

# View variable
echo $DATABASE_URL

# Set multiple variables
export API_KEY="abc123"
export ENV="production"

# List all environment variables
env

# List sorted
env | sort

# Search for specific variable
env | grep DATABASE

# Print specific variable
printenv DATABASE_URL

# Unset variable
unset DATABASE_URL

# Set variable temporarily for one command
DATABASE_URL="postgres://test:5432/db" node app.js

# Make variable persistent (add to ~/.bashrc)
echo 'export PATH="$PATH:/opt/myapp/bin"' >> ~/.bashrc
source ~/.bashrc

# System-wide variables (/etc/environment)
# Add: JAVA_HOME="/usr/lib/jvm/java-11"

# Check if variable is set
if [ -z "$DATABASE_URL" ]; then
  echo "DATABASE_URL not set"
fi
```

### 5. Example Scenario

- Application needs database connection string
- Set in environment: `export DATABASE_URL="postgres://db.local:5432/prod"`
- Application reads from environment: `os.environ.get('DATABASE_URL')`
- For production, add to systemd service file
- Or set in `/etc/environment` for system-wide access
- Different values per environment without code changes

### 6. Common KT / Interview Line

"Environment variables let us externalize configuration, making the same application code work across dev, staging, and production with different settings."

### 7. Closing Line Trigger

"In short, environment variables are the standard way to configure applications without hardcoding values."

---

## Shell Scripting Basics

🔑 **Primary Trigger:** "Automate repetitive tasks with bash scripts – essential for DevOps automation."

### 1. What it is (Simple Definition)

- Shell scripts are executable text files containing commands
- Automate multi-step processes and repetitive tasks
- Bash is the most common shell for scripting

### 2. Why / When DevOps Uses This

- Automate deployment steps
- Create backup scripts
- Perform health checks
- Process logs or data files
- Set up new servers with initialization scripts

### 3. Key Commands / Patterns

- `#!/bin/bash` - shebang line (first line)
- Variables: `VAR="value"`
- Command substitution: `OUTPUT=$(command)`
- Conditionals: `if [ condition ]; then ... fi`
- Loops: `for`, `while`
- Functions: `function_name() { ... }`
- Exit codes: `$?`, `exit 0`
- Arguments: `$1`, `$2`, `$@`

### 4. Minimal Examples

```bash
#!/bin/bash
# Basic deployment script

# Variables
APP_NAME="myapp"
DEPLOY_DIR="/opt/$APP_NAME"
BACKUP_DIR="/backup/$APP_NAME"

# Exit on any error
set -e

# Print with timestamp
echo "[$(date '+%Y-%m-%d %H:%M:%S')] Starting deployment..."

# Backup existing version
if [ -d "$DEPLOY_DIR" ]; then
  echo "Backing up existing version..."
  cp -r "$DEPLOY_DIR" "$BACKUP_DIR/backup-$(date +%Y%m%d-%H%M%S)"
fi

# Deploy new version
echo "Deploying new version..."
mkdir -p "$DEPLOY_DIR"
cp -r dist/* "$DEPLOY_DIR/"

# Restart service
echo "Restarting service..."
systemctl restart "$APP_NAME"

# Health check
sleep 5
if systemctl is-active --quiet "$APP_NAME"; then
  echo "Deployment successful!"
  exit 0
else
  echo "Deployment failed - service not running"
  exit 1
fi
```

**Common patterns:**
```bash
# Check if command succeeded
if [ $? -eq 0 ]; then
  echo "Success"
else
  echo "Failed"
fi

# Loop through files
for file in /var/log/*.log; do
  echo "Processing $file"
done

# Loop through array
SERVERS=("web1" "web2" "web3")
for server in "${SERVERS[@]}"; do
  ssh "$server" "uptime"
done

# Function
function backup() {
  local source=$1
  local dest=$2
  cp -r "$source" "$dest"
}

# Check if file exists
if [ -f "/etc/config.conf" ]; then
  echo "Config exists"
fi

# Check if directory exists
if [ -d "/opt/app" ]; then
  echo "App directory exists"
fi
```

### 5. Example Scenario

- Need to deploy application to 5 servers
- Write deployment script: checks backup, copies files, restarts service
- Add health check at end to verify success
- Make executable: `chmod +x deploy.sh`
- Run on each server: `./deploy.sh`
- Script handles all steps consistently
- Reduces human error and speeds up deployments

### 6. Common KT / Interview Line

"Shell scripts automate manual processes – I write scripts for deployments, backups, and health checks to ensure consistency and reduce errors."

### 7. Closing Line Trigger

"In short, bash scripting turns repetitive manual tasks into automated, reliable processes."

---

## Cron & Scheduling

🔑 **Primary Trigger:** "Schedule automated tasks with cron – backups, cleanups, and periodic checks."

### 1. What it is (Simple Definition)

- Cron runs commands on a schedule (minutes, hours, days)
- Crontab is per-user schedule configuration
- Essential for automated maintenance tasks

### 2. Why / When DevOps Uses This

- Schedule nightly database backups
- Rotate logs automatically
- Run health checks every 5 minutes
- Clean up old files weekly
- Generate daily reports

### 3. Key Commands / Patterns

- `crontab -e` - edit current user's crontab
- `crontab -l` - list crontab entries
- `crontab -r` - remove crontab
- Format: `minute hour day month weekday command`
- `* * * * *` - every minute
- `/var/log/cron` - cron execution logs

### 4. Minimal Examples

```bash
# Edit crontab
crontab -e

# List current crontab
crontab -l

# Crontab syntax: minute hour day month weekday command
# * * * * * command
# | | | | |
# | | | | +-- Day of week (0-7, 0 and 7 = Sunday)
# | | | +---- Month (1-12)
# | | +------ Day of month (1-31)
# | +-------- Hour (0-23)
# +---------- Minute (0-59)

# Example crontab entries:

# Every minute
* * * * * /opt/scripts/health-check.sh

# Every 5 minutes
*/5 * * * * /opt/scripts/monitor.sh

# Every hour at minute 30
30 * * * * /opt/scripts/cleanup.sh

# Every day at 2:30 AM
30 2 * * * /opt/scripts/backup.sh

# Every Sunday at 3 AM
0 3 * * 0 /opt/scripts/weekly-backup.sh

# First day of every month at 1 AM
0 1 1 * * /opt/scripts/monthly-report.sh

# Every weekday (Mon-Fri) at 9 AM
0 9 * * 1-5 /opt/scripts/workday-task.sh

# Redirect output to log file
0 2 * * * /opt/scripts/backup.sh >> /var/log/backup.log 2>&1

# Run as specific user (system crontab /etc/crontab)
0 2 * * * root /opt/scripts/backup.sh

# Set environment variables
PATH=/usr/local/bin:/usr/bin:/bin
0 * * * * /opt/scripts/task.sh
```

**Common cron patterns:**
```bash
# Every minute:        * * * * *
# Every 5 minutes:     */5 * * * *
# Every hour:          0 * * * *
# Every day at 2 AM:   0 2 * * *
# Every Monday at 3 AM: 0 3 * * 1
# Every month 1st:     0 0 1 * *
```

### 5. Example Scenario

- Need daily database backups at 2 AM
- Edit crontab: `crontab -e`
- Add entry: `0 2 * * * /opt/scripts/db-backup.sh >> /var/log/db-backup.log 2>&1`
- Save and exit
- Cron runs backup every day at 2 AM
- Check logs: `tail -f /var/log/db-backup.log`
- Verify backups are being created

### 6. Common KT / Interview Line

"Cron is how we schedule automated tasks in Linux – I use it for backups, log rotation, and periodic health checks without manual intervention."

### 7. Closing Line Trigger

"In short, cron enables hands-off automation for time-based recurring tasks."

---

## Archiving & Compression

🔑 **Primary Trigger:** "Create and extract archives with tar – essential for backups and deployments."

### 1. What it is (Simple Definition)

- Tar archives multiple files into single file
- Compression reduces archive size (gzip, bzip2)
- Standard format for backups and distributing code

### 2. Why / When DevOps Uses This

- Create backups of application directories
- Package application for deployment
- Extract deployment artifacts
- Transfer multiple files efficiently
- Archive logs before deletion

### 3. Key Commands / Patterns

- `tar -czf archive.tar.gz <files>` - create gzip archive
- `tar -xzf archive.tar.gz` - extract gzip archive
- `tar -tzf archive.tar.gz` - list archive contents
- `tar -cjf archive.tar.bz2 <files>` - create bzip2 archive
- `gzip <file>` - compress file
- `gunzip <file.gz>` - decompress file
- `zip -r archive.zip <dir>` - create zip archive
- `unzip archive.zip` - extract zip archive

### 4. Minimal Examples

```bash
# Create tar.gz archive
tar -czf backup.tar.gz /opt/myapp

# Extract tar.gz archive
tar -xzf backup.tar.gz

# Extract to specific directory
tar -xzf backup.tar.gz -C /tmp/restore

# List archive contents without extracting
tar -tzf backup.tar.gz

# Create archive with verbose output
tar -czvf backup.tar.gz /opt/myapp

# Exclude files from archive
tar -czf backup.tar.gz --exclude='*.log' /opt/myapp

# Create bzip2 archive (better compression, slower)
tar -cjf backup.tar.bz2 /opt/myapp

# Extract bzip2 archive
tar -xjf backup.tar.bz2

# Gzip single file
gzip large-file.log
# Creates large-file.log.gz, removes original

# Decompress
gunzip large-file.log.gz

# Keep original when compressing
gzip -k large-file.log

# Zip archive (cross-platform)
zip -r backup.zip /opt/myapp

# Unzip
unzip backup.zip

# List zip contents
unzip -l backup.zip

# Archive with date in filename
tar -czf backup-$(date +%Y%m%d).tar.gz /opt/myapp
```

### 5. Example Scenario

- Need to backup application before deployment
- Create archive: `tar -czf myapp-backup-$(date +%Y%m%d).tar.gz /opt/myapp`
- Deploy fails, need to rollback
- Extract backup: `tar -xzf myapp-backup-20240129.tar.gz -C /opt/`
- Application restored from archive
- Deployment can be retried after fixing issues

### 6. Common KT / Interview Line

"Tar with gzip compression is the standard for creating backups and packaging applications – I use it daily for deployments and archiving."

### 7. Closing Line Trigger

"In short, archiving and compression are essential for backups, transfers, and efficient storage."

---

## Text Processing

🔑 **Primary Trigger:** "Parse and transform text with grep, sed, awk – powerful tools for log analysis and data processing."

### 1. What it is (Simple Definition)

- Grep searches for patterns in text
- Sed edits text streams (find and replace)
- Awk processes structured text (columns, calculations)

### 2. Why / When DevOps Uses This

- Extract errors from logs
- Parse structured log files
- Find and replace configuration values
- Count occurrences of events
- Process CSV or JSON data in logs

### 3. Key Commands / Patterns

**Grep:**
- `grep <pattern> <file>` - search pattern
- `grep -r <pattern> <dir>` - recursive search
- `grep -i <pattern>` - case-insensitive
- `grep -v <pattern>` - invert match
- `grep -c <pattern>` - count matches

**Sed:**
- `sed 's/old/new/' <file>` - replace first occurrence
- `sed 's/old/new/g' <file>` - replace all occurrences
- `sed -i 's/old/new/g' <file>` - edit file in-place

**Awk:**
- `awk '{print $1}' <file>` - print first column
- `awk -F',' '{print $2}' <file>` - CSV parsing
- `awk '/pattern/ {print $0}' <file>` - conditional print

### 4. Minimal Examples

```bash
# --- GREP ---

# Search for ERROR in log
grep "ERROR" /var/log/app.log

# Case-insensitive search
grep -i "error" /var/log/app.log

# Search recursively in directory
grep -r "database connection" /var/log/

# Show line numbers
grep -n "ERROR" app.log

# Show 3 lines before and after match (context)
grep -C 3 "ERROR" app.log

# Count occurrences
grep -c "ERROR" app.log

# Invert match (exclude pattern)
grep -v "INFO" app.log

# Multiple patterns
grep -E "ERROR|FATAL" app.log

# --- SED ---

# Replace first occurrence per line
sed 's/old-value/new-value/' config.txt

# Replace all occurrences (global)
sed 's/old-value/new-value/g' config.txt

# Edit file in-place
sed -i 's/localhost/production-db/g' config.conf

# Delete lines matching pattern
sed '/debug-log/d' config.conf

# Print specific lines (10-20)
sed -n '10,20p' app.log

# Replace only on lines matching pattern
sed '/database/s/localhost/prod-db/g' config.conf

# --- AWK ---

# Print first column
awk '{print $1}' access.log

# Print multiple columns
awk '{print $1, $3}' access.log

# CSV parsing (comma delimiter)
awk -F',' '{print $2}' data.csv

# Print lines where column 3 > 100
awk '$3 > 100' data.txt

# Sum values in column
awk '{sum += $1} END {print sum}' numbers.txt

# Count lines
awk 'END {print NR}' file.txt

# Print lines matching pattern
awk '/ERROR/ {print $0}' app.log

# Calculate average
awk '{sum += $1; count++} END {print sum/count}' numbers.txt

# --- COMBINED ---

# Extract unique IPs from access log
awk '{print $1}' access.log | sort | uniq

# Count HTTP status codes
awk '{print $9}' access.log | sort | uniq -c | sort -rn

# Find errors and extract timestamp
grep "ERROR" app.log | awk '{print $1, $2}'

# Replace in file and count changes
sed 's/old/new/g' file.txt | grep -c "new"
```

### 5. Example Scenario

- Analyze nginx access logs for 500 errors
- Extract 500 errors: `grep " 500 " /var/log/nginx/access.log`
- Get unique IPs: `grep " 500 " access.log | awk '{print $1}' | sort | uniq -c | sort -rn`
- Shows which IPs are causing most errors
- Extract timestamps: `grep " 500 " access.log | awk '{print $4}' | sed 's/\[//'`
- Identify when errors started occurring

### 6. Common KT / Interview Line

"Grep, sed, and awk are my go-to tools for log analysis – they let me quickly find patterns, extract data, and process large text files efficiently."

### 7. Closing Line Trigger

"In short, text processing tools turn raw logs into actionable insights without custom scripts."

---

## cURL & Wget

🔑 **Primary Trigger:** "Make HTTP requests and download files from command line – essential for testing APIs and deployments."

### 1. What it is (Simple Definition)

- cURL transfers data with URLs (HTTP, FTP, etc.)
- Wget downloads files from web
- Both essential for API testing and downloading artifacts

### 2. Why / When DevOps Uses This

- Test API endpoints and health checks
- Download deployment artifacts
- Verify service responses
- Call webhooks for notifications
- Download configuration or scripts

### 3. Key Commands / Patterns

**cURL:**
- `curl <url>` - GET request
- `curl -I <url>` - show headers only
- `curl -X POST <url>` - POST request
- `curl -d "data" <url>` - send data
- `curl -H "Header: value" <url>` - custom header
- `curl -o <file> <url>` - save to file

**Wget:**
- `wget <url>` - download file
- `wget -O <file> <url>` - save with custom name
- `wget -c <url>` - continue interrupted download
- `wget -r <url>` - recursive download

### 4. Minimal Examples

```bash
# --- CURL ---

# Simple GET request
curl https://api.example.com/status

# Show response headers
curl -I https://api.example.com

# Verbose output (debugging)
curl -v https://api.example.com

# Follow redirects
curl -L https://short.url

# POST request with data
curl -X POST https://api.example.com/users \
  -H "Content-Type: application/json" \
  -d '{"name":"John","email":"john@example.com"}'

# POST with file
curl -X POST https://api.example.com/upload \
  -F "file=@document.pdf"

# Custom headers
curl -H "Authorization: Bearer token123" \
  https://api.example.com/protected

# Save response to file
curl -o response.json https://api.example.com/data

# Silent mode (no progress bar)
curl -s https://api.example.com/status

# Show only HTTP status code
curl -o /dev/null -s -w "%{http_code}\n" https://api.example.com

# Test endpoint with timeout
curl --max-time 5 https://api.example.com

# --- WGET ---

# Download file
wget https://example.com/file.tar.gz

# Save with different name
wget -O myfile.tar.gz https://example.com/file.tar.gz

# Continue interrupted download
wget -c https://example.com/large-file.iso

# Download in background
wget -b https://example.com/large-file.tar.gz

# Download multiple files
wget -i urls.txt

# Mirror website
wget -m https://example.com

# Download with custom user agent
wget --user-agent="Mozilla/5.0" https://example.com/file.zip

# --- COMMON DEVOPS USES ---

# Health check
curl -f https://myapp.example.com/health || echo "Health check failed"

# Test API endpoint
curl -X GET https://api.example.com/users | jq .

# Download latest release
wget https://github.com/user/repo/releases/latest/download/app.tar.gz

# Send Slack notification
curl -X POST https://hooks.slack.com/services/TOKEN \
  -H "Content-Type: application/json" \
  -d '{"text":"Deployment completed"}'
```

### 5. Example Scenario

- After deployment, verify application is responding
- Check health endpoint: `curl https://myapp.example.com/health`
- Returns 200 OK with JSON response
- Test API endpoint: `curl https://myapp.example.com/api/users | jq .`
- API working correctly
- Download logs for analysis: `curl https://myapp.example.com/logs -o app-logs.txt`
- Deployment verified successful

### 6. Common KT / Interview Line

"Curl is essential for testing HTTP endpoints – I use it constantly to verify APIs are responding correctly after deployments."

### 7. Closing Line Trigger

"In short, curl and wget provide command-line HTTP capabilities for testing and downloading from anywhere."

---

## System Monitoring

🔑 **Primary Trigger:** "Check system health and resource usage – uptime, load, memory, CPU statistics."

### 1. What it is (Simple Definition)

- Commands to monitor system performance and health
- Check load average, memory usage, I/O statistics
- Essential for performance troubleshooting

### 2. Why / When DevOps Uses This

- Check system load before deployments
- Investigate performance issues
- Monitor during load tests
- Verify adequate resources available
- Troubleshoot slow applications

### 3. Key Commands / Patterns

- `uptime` - system uptime and load average
- `free -h` - memory usage
- `vmstat 1` - virtual memory statistics
- `iostat -x 1` - I/O statistics
- `mpstat 1` - CPU statistics per core
- `sar` - system activity report (requires sysstat)
- `dmesg` - kernel messages
- `nmon` - interactive performance monitor

### 4. Minimal Examples

```bash
# Check uptime and load average
uptime

# Check memory usage (human-readable)
free -h

# Detailed memory info
cat /proc/meminfo

# Virtual memory statistics (every 1 second)
vmstat 1

# Extended vmstat
vmstat -S M 1

# I/O statistics
iostat

# Extended I/O stats with details
iostat -x 1

# CPU statistics per core
mpstat -P ALL 1

# Check disk I/O by device
iostat -dx 1

# System activity report (if installed)
sar -u 1 5   # CPU usage, 1 sec interval, 5 times
sar -r 1 5   # Memory usage
sar -b 1 5   # I/O statistics

# Kernel messages
dmesg | tail -50

# Check for errors in kernel log
dmesg | grep -i error

# Load average explanation
# uptime output: load average: 1.50, 1.20, 0.80
# Last 1 min, 5 min, 15 min averages
# Values < CPU count = good
# Values > CPU count = system overloaded

# Check CPU count
nproc

# CPU info
lscpu

# System information
uname -a

# Check running kernel
uname -r

# Hardware info
lshw -short

# PCI devices
lspci

# USB devices
lsusb
```

### 5. Example Scenario

- Application responding slowly
- Check load: `uptime` shows load average 8.5 on 4-core system (overloaded)
- Check memory: `free -h` shows 95% memory usage
- Check processes: `top` shows Java process consuming 80% CPU
- Check I/O: `iostat -x 1` shows disk at 100% utilization
- Identify root cause: memory leak causing swapping, high I/O
- Need to restart application or add resources

### 6. Common KT / Interview Line

"When investigating performance issues, I check uptime for load average, free for memory, and iostat for disk I/O to identify bottlenecks."

### 7. Closing Line Trigger

"In short, system monitoring commands provide quick visibility into resource utilization and performance."

---

## Resource Limits

🔑 **Primary Trigger:** "Control process resource consumption with ulimit – prevent runaway processes."

### 1. What it is (Simple Definition)

- Ulimit sets limits on resources processes can use
- Controls max files, memory, CPU time, processes
- Prevents single process from consuming all resources

### 2. Why / When DevOps Uses This

- Prevent applications from opening too many files
- Limit memory usage per process
- Control maximum number of processes
- Protect system from resource exhaustion
- Configure for high-performance applications

### 3. Key Commands / Patterns

- `ulimit -a` - show all limits
- `ulimit -n` - show max open files
- `ulimit -n 4096` - set max open files
- `ulimit -u` - max user processes
- `/etc/security/limits.conf` - persistent limits

### 4. Minimal Examples

```bash
# Show all resource limits
ulimit -a

# Show max open files
ulimit -n

# Set max open files (for current shell)
ulimit -n 65536

# Show max user processes
ulimit -u

# Set max user processes
ulimit -u 4096

# Show max file size
ulimit -f

# Show max memory size
ulimit -m

# Show stack size
ulimit -s

# Set unlimited (if allowed)
ulimit -n unlimited

# Persistent limits: /etc/security/limits.conf
# Format: <domain> <type> <item> <value>
# Examples:
# *         soft    nofile      4096
# *         hard    nofile      65536
# username  soft    nproc       1024
# username  hard    nproc       2048

# Check limits for running process
cat /proc/<PID>/limits

# Set limits in systemd service
# In /etc/systemd/system/myapp.service:
# [Service]
# LimitNOFILE=65536
# LimitNPROC=4096
```

### 5. Example Scenario

- Application crashes with "Too many open files" error
- Check current limit: `ulimit -n` shows 1024
- Check what app needs: documentation says 4096
- Set temporarily: `ulimit -n 4096`
- Restart application: works fine
- Make permanent: add to `/etc/security/limits.conf`:
  ```
  appuser soft nofile 4096
  appuser hard nofile 8192
  ```
- Or in systemd service file: `LimitNOFILE=4096`

### 6. Common KT / Interview Line

"When applications hit 'too many open files' errors, increasing the ulimit for file descriptors usually resolves it immediately."

### 7. Closing Line Trigger

"In short, resource limits prevent processes from exhausting system resources and protect system stability."

---

## Common Troubleshooting Flow

🔑 **Primary Trigger:** "Systematic approach to debug Linux server issues – logs, processes, network, disk."

### 1. What it is (Simple Definition)

- Structured methodology to troubleshoot server issues
- Check logs first, then processes, network, and resources
- Follow consistent pattern to find root cause faster

### 2. Why / When DevOps Uses This

- Application not responding
- Service won't start
- Performance degradation
- Connection errors
- Deployment failures

### 3. Key Commands / Patterns

**1. Check Logs:**
- `journalctl -u <service> -n 100`
- `tail -f /var/log/<app>/error.log`

**2. Check Processes:**
- `systemctl status <service>`
- `ps aux | grep <app>`
- `top` or `htop`

**3. Check Resources:**
- `df -h` (disk space)
- `free -h` (memory)
- `uptime` (load average)

**4. Check Network:**
- `ss -tulpn` (listening ports)
- `ping <host>` (connectivity)
- `curl <url>` (endpoint test)

### 4. Minimal Examples

```bash
# --- TROUBLESHOOTING WORKFLOW ---

# STEP 1: Check service status
systemctl status nginx

# STEP 2: Check recent logs
journalctl -u nginx -n 100

# STEP 3: Check if process is running
ps aux | grep nginx

# STEP 4: Check listening ports
sudo ss -tulpn | grep nginx

# STEP 5: Check disk space
df -h

# STEP 6: Check memory
free -h

# STEP 7: Check system load
uptime

# STEP 8: Test connectivity
curl http://localhost

# --- SPECIFIC SCENARIOS ---

# Service won't start:
# 1. Check status:
systemctl status myapp
# 2. Check logs:
journalctl -u myapp -n 50
# 3. Check config file syntax:
nginx -t   # for nginx
java -jar app.jar --validate  # validate config
# 4. Check permissions:
ls -la /opt/myapp
# 5. Try starting manually:
/opt/myapp/bin/start.sh

# Application slow:
# 1. Check load:
uptime
# 2. Check processes:
top
# 3. Check memory:
free -h
# 4. Check disk I/O:
iostat -x 1
# 5. Check logs for errors:
tail -f /var/log/myapp/app.log | grep ERROR

# Can't connect to service:
# 1. Check if listening:
sudo ss -tulpn | grep :8080
# 2. Check firewall:
sudo iptables -L
sudo firewall-cmd --list-all
# 3. Test locally:
curl http://localhost:8080
# 4. Test externally:
curl http://server-ip:8080
# 5. Check DNS:
nslookup server-hostname

# Disk full:
# 1. Check usage:
df -h
# 2. Find large directories:
du -sh /* | sort -rh | head -10
# 3. Find large files:
find / -type f -size +1G -exec ls -lh {} \;
# 4. Check logs:
du -sh /var/log/*
# 5. Clean up:
sudo journalctl --vacuum-time=7d
sudo rm /var/log/*.old
```

### 5. Example Scenario

- Production API not responding (500 errors)
- **Step 1 - Logs**: `journalctl -u api-service -n 100`
  - See "Database connection timeout" errors
- **Step 2 - Check DB**: `systemctl status postgresql`
  - Database is running
- **Step 3 - Test DB connection**: `psql -h localhost -U dbuser`
  - Connection works fine
- **Step 4 - Check resources**: `free -h`
  - Memory at 98%, likely cause
- **Step 5 - Check processes**: `top`
  - API process consuming 8GB (memory leak)
- **Solution**: Restart API service to clear memory
- **Follow-up**: Investigate memory leak for permanent fix

### 6. Common KT / Interview Line

"When troubleshooting, I follow a systematic approach: logs first to see errors, then processes to check if running, then resources, then network connectivity."

### 7. Closing Line Trigger

"In short, a structured troubleshooting flow helps identify root causes faster than random commands."

---

## Closing Notes

This cheat sheet covers **Linux for DevOps** with focus on fast retrieval and teaching. Each section provides commands, examples, and spoken explanations for real-world scenarios.

**How to Use This Document:**

- **Before interviews**: Read primary triggers to activate memory
- **During troubleshooting**: Reference relevant sections quickly
- **Teaching juniors**: Use examples and scenarios verbatim
- **Building scripts**: Copy command patterns for automation

**Practice Tips:**

- Run these commands in a test VM or container
- Create a personal "lab environment" to practice
- Build muscle memory for common command sequences
- Practice explaining concepts aloud using the KT lines

**DevOps Command Priorities:**

**Daily Use:**
- `systemctl status`, `journalctl -u`, `tail -f`
- `ps aux | grep`, `top`, `htop`
- `df -h`, `du -sh`, `free -h`
- `ss -tulpn`, `curl`, `ping`

**Troubleshooting:**
- `grep -r`, `find`, `journalctl`
- `strace`, `lsof`, `netstat`
- `dmesg | grep error`

**Automation:**
- Shell scripting, cron
- `ssh`, `scp`, `rsync`
- `tar`, `gzip`

**Remember:**
- **Logs first** - most issues show up in logs
- **Check resources** - CPU, memory, disk
- **Test connectivity** - ping, curl, ss
- **Understand permissions** - chmod, chown
- **Use systemctl** - modern service management

---

**End of Linux for DevOps – High Retrieval & Teaching Cheat Sheet**

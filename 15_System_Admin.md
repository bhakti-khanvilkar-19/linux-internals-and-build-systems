# Linux System Administration — Logs, Cron, Backup, and Security

## Objective
Master system administration: manage logs, schedule tasks, perform backups, monitor system health, and implement basic security.

---

## 1) System Logs

### Log Locations

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                          IMPORTANT LOG FILES                                        │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   Log File                     │ Contents                                           │
│   ─────────────────────────────┼──────────────────────────────────────────          │
│   /var/log/syslog              │ General system messages (Ubuntu/Debian)            │
│   /var/log/messages            │ General system messages (RHEL/CentOS)              │
│   /var/log/auth.log            │ Authentication (login, sudo, SSH)                  │
│   /var/log/kern.log            │ Kernel messages                                    │
│   /var/log/dmesg               │ Boot messages (use dmesg command)                  │
│   /var/log/boot.log            │ Boot process                                       │
│   /var/log/apache2/            │ Apache web server                                  │
│   /var/log/nginx/              │ Nginx web server                                   │
│   /var/log/mysql/              │ MySQL database                                     │
│   /var/log/apt/                │ Package management                                 │
│   /var/log/cron.log            │ Cron job output                                    │
│   /var/log/fail2ban.log        │ Fail2ban intrusion prevention                      │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### Viewing Logs

```bash
# Traditional log viewing
cat /var/log/syslog
tail -f /var/log/syslog                  # Follow in real-time
tail -n 100 /var/log/auth.log            # Last 100 lines
less /var/log/syslog                     # Paginated view

# Grep for specific entries
grep "error" /var/log/syslog
grep -i "failed" /var/log/auth.log
zgrep "error" /var/log/syslog.*.gz       # Search compressed logs

# Kernel messages
dmesg                                    # All kernel messages
dmesg -T                                 # Human-readable timestamps
dmesg | tail -20                         # Recent messages
dmesg -l err,warn                        # Errors and warnings
```

### journalctl (systemd)

```bash
# View all logs
journalctl

# Recent logs (follow)
journalctl -f

# Current boot only
journalctl -b

# Previous boot
journalctl -b -1

# By time
journalctl --since "1 hour ago"
journalctl --since "2026-02-24 10:00:00"
journalctl --since today
journalctl --since yesterday --until today

# By service
journalctl -u nginx
journalctl -u ssh -f

# By priority
journalctl -p err                        # Errors and worse
journalctl -p warning                    # Warnings and worse
# Levels: emerg, alert, crit, err, warning, notice, info, debug

# Kernel messages
journalctl -k
journalctl --dmesg

# Output formats
journalctl -o json-pretty               # JSON format
journalctl -o verbose                   # All fields

# Disk usage
journalctl --disk-usage

# Clean old logs
sudo journalctl --vacuum-time=7d         # Delete older than 7 days
sudo journalctl --vacuum-size=500M       # Keep max 500MB
```

### Log Rotation

```bash
# logrotate config
cat /etc/logrotate.conf
ls /etc/logrotate.d/

# Example: /etc/logrotate.d/myapp
/var/log/myapp/*.log {
    daily
    rotate 7
    compress
    delaycompress
    missingok
    notifempty
    create 640 root adm
    postrotate
        systemctl reload myapp > /dev/null 2>&1 || true
    endscript
}

# Test configuration
sudo logrotate -d /etc/logrotate.conf    # Debug (dry run)
sudo logrotate -f /etc/logrotate.conf    # Force rotation
```

---

## 2) Scheduled Tasks (Cron)

### Crontab Format

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                          CRONTAB FORMAT                                             │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   ┌───────────── minute (0-59)                                                      │
│   │ ┌─────────── hour (0-23)                                                        │
│   │ │ ┌───────── day of month (1-31)                                                │
│   │ │ │ ┌─────── month (1-12)                                                       │
│   │ │ │ │ ┌───── day of week (0-7, 0&7=Sunday)                                      │
│   │ │ │ │ │                                                                         │
│   * * * * * command                                                                 │
│                                                                                     │
│   Special values:                                                                   │
│   *     any value                                                                   │
│   ,     list: 1,3,5                                                                 │
│   -     range: 1-5                                                                  │
│   /     step: */15 (every 15)                                                       │
│                                                                                     │
│   Special strings:                                                                  │
│   @reboot    Run once at startup                                                    │
│   @yearly    0 0 1 1 *                                                              │
│   @monthly   0 0 1 * *                                                              │
│   @weekly    0 0 * * 0                                                              │
│   @daily     0 0 * * *                                                              │
│   @hourly    0 * * * *                                                              │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### Managing Crontab

```bash
# Edit current user's crontab
crontab -e

# View current crontab
crontab -l

# Remove crontab
crontab -r

# Edit another user's crontab (as root)
sudo crontab -u username -e

# System-wide cron locations
ls /etc/crontab              # System crontab
ls /etc/cron.d/              # Drop-in cron files
ls /etc/cron.daily/          # Daily scripts
ls /etc/cron.hourly/         # Hourly scripts
ls /etc/cron.weekly/         # Weekly scripts
ls /etc/cron.monthly/        # Monthly scripts
```

### Crontab Examples

```bash
# Every 5 minutes
*/5 * * * * /path/to/script.sh

# Every hour at minute 30
30 * * * * /path/to/script.sh

# Every day at 2:30 AM
30 2 * * * /path/to/script.sh

# Every Monday at 9 AM
0 9 * * 1 /path/to/script.sh

# First day of month at midnight
0 0 1 * * /path/to/script.sh

# Every weekday (Mon-Fri) at 6 PM
0 18 * * 1-5 /path/to/script.sh

# At startup
@reboot /path/to/script.sh

# With logging
*/5 * * * * /path/to/script.sh >> /var/log/myscript.log 2>&1

# With environment
PATH=/usr/local/bin:/usr/bin:/bin
0 * * * * /path/to/script.sh
```

### Cron Troubleshooting

```bash
# Check cron service
systemctl status cron

# View cron log
grep CRON /var/log/syslog
journalctl -u cron

# Common issues:
# 1. No PATH - use full paths in scripts
# 2. Not executable - chmod +x script.sh
# 3. Output not captured - redirect stdout/stderr
# 4. Environment differ from shell - set env in crontab
```

---

## 3) System Monitoring

### System Information

```bash
# OS info
cat /etc/os-release
uname -a
hostnamectl

# Hardware
lscpu                                    # CPU info
free -h                                  # Memory
lsblk                                    # Block devices
df -h                                    # Disk space
lspci                                    # PCI devices
lsusb                                    # USB devices
```

### Real-time Monitoring

```bash
# top - process monitor
top
# Press: M (sort by memory), P (CPU), k (kill), q (quit)

# htop - better process monitor (install: apt install htop)
htop

# CPU usage
mpstat 1 5                               # Every second, 5 times
vmstat 1 5                               # Virtual memory stats

# Memory
free -h
cat /proc/meminfo

# Disk I/O
iostat -x 1 5
iotop                                    # Interactive I/O monitor

# Network
iftop                                    # Network by connection
nethogs                                  # Network by process
```

### Resource Monitoring Commands

```bash
# System uptime and load
uptime
# Load average: 1min, 5min, 15min average
# Rule: load < number of CPUs = OK

# Who is logged in
who
w
last                                     # Login history
lastb                                    # Failed logins

# Process information
ps aux                                   # All processes
ps aux --sort=-%mem | head               # Top memory users
ps aux --sort=-%cpu | head               # Top CPU users
pstree                                   # Process tree

# Open files/connections
lsof                                     # All open files
lsof -i :80                              # Who's using port 80
lsof -u username                         # Files by user
```

---

## 4) Backup Strategies

### Backup Tools

```bash
# tar - Archive files
tar -cvzf backup.tar.gz /path/to/dir         # Create
tar -tvzf backup.tar.gz                       # List contents
tar -xvzf backup.tar.gz                       # Extract
tar -xvzf backup.tar.gz -C /target/           # Extract to location

# Incremental backup with tar
tar -cvzf full-backup.tar.gz -g snapshot.snar /data/
tar -cvzf incr-backup.tar.gz -g snapshot.snar /data/

# rsync - Efficient sync
rsync -avz /source/ /destination/             # Local sync
rsync -avz /source/ user@host:/dest/          # Remote sync
rsync -avz --delete /source/ /dest/           # Mirror (delete extra)
rsync -avzn /source/ /dest/                   # Dry run

# rsync options
-a    Archive mode (preserves permissions, etc.)
-v    Verbose
-z    Compress during transfer
-P    Show progress
--exclude='*.log'    Exclude pattern
--delete             Delete files not in source
```

### Backup Script Example

```bash
#!/bin/bash
# backup.sh - Daily backup script

set -euo pipefail

# Configuration
BACKUP_SRC="/home /etc /var/www"
BACKUP_DST="/backup"
RETENTION_DAYS=30
DATE=$(date +%Y%m%d)
HOSTNAME=$(hostname)

# Create backup
BACKUP_FILE="${BACKUP_DST}/${HOSTNAME}-${DATE}.tar.gz"

echo "Starting backup: $BACKUP_FILE"
tar -czpf "$BACKUP_FILE" $BACKUP_SRC 2>/dev/null

# Verify
if [[ -f "$BACKUP_FILE" ]]; then
    SIZE=$(du -h "$BACKUP_FILE" | cut -f1)
    echo "Backup complete: $SIZE"
else
    echo "Backup failed!"
    exit 1
fi

# Cleanup old backups
find "$BACKUP_DST" -name "*.tar.gz" -mtime +$RETENTION_DAYS -delete
echo "Cleaned up backups older than $RETENTION_DAYS days"
```

### Remote Backup

```bash
# rsync to remote server
rsync -avz --progress /data/ user@backup.example.com:/backups/data/

# With SSH key
rsync -avz -e "ssh -i ~/.ssh/backup_key" /data/ user@host:/backup/

# Scheduled in cron
0 2 * * * rsync -az /data/ user@backup:/backups/ >> /var/log/backup.log 2>&1
```

---

## 5) Security Basics

### User Security

```bash
# Password policies
sudo nano /etc/login.defs
# PASS_MAX_DAYS   90
# PASS_MIN_DAYS   7
# PASS_WARN_AGE   14

# Check password expiry
chage -l username

# Force password change
sudo chage -d 0 username

# Lock/unlock user
sudo usermod -L username                 # Lock
sudo usermod -U username                 # Unlock

# Check sudo access
sudo -l
```

### SSH Security

```bash
# /etc/ssh/sshd_config
PermitRootLogin no                       # Disable root login
PasswordAuthentication no                # Key-only auth
PubkeyAuthentication yes
Port 2222                                # Change default port
MaxAuthTries 3
AllowUsers user1 user2                   # Whitelist users

# Restart after changes
sudo systemctl restart sshd
```

### Firewall (UFW)

```bash
# Enable/status
sudo ufw enable
sudo ufw status verbose

# Default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow services
sudo ufw allow ssh
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Allow from specific IP
sudo ufw allow from 192.168.1.100

# Deny
sudo ufw deny 23

# Remove rule
sudo ufw delete allow 80/tcp
```

### Fail2ban

```bash
# Install
sudo apt install fail2ban

# Config
sudo nano /etc/fail2ban/jail.local

[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600
findtime = 600

# Manage
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
sudo fail2ban-client status
sudo fail2ban-client status sshd
sudo fail2ban-client unban IP_ADDRESS
```

### File Permissions Security

```bash
# Find world-writable files
find / -perm -o+w -type f 2>/dev/null

# Find SUID/SGID files
find / -perm -4000 -type f 2>/dev/null   # SUID
find / -perm -2000 -type f 2>/dev/null   # SGID

# Find files with no owner
find / -nouser -o -nogroup 2>/dev/null

# Secure important files
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub
chmod 700 ~/.ssh
```

### Security Auditing

```bash
# Check listening ports
sudo ss -tulpn
sudo netstat -tulpn

# Check active connections
sudo ss -tan

# Check running services
systemctl list-units --type=service --state=running

# Check scheduled tasks
crontab -l
sudo ls -la /etc/cron.*

# Check login history
last
lastb                                    # Failed logins
```

---

## 6) System Maintenance

### Package Updates

```bash
# Update system (Ubuntu/Debian)
sudo apt update
sudo apt upgrade -y
sudo apt full-upgrade -y
sudo apt autoremove -y

# Unattended upgrades
sudo apt install unattended-upgrades
sudo dpkg-reconfigure unattended-upgrades
```

### Disk Maintenance

```bash
# Check filesystem
sudo fsck -n /dev/sda1                   # Check only

# Monitor disk health
sudo smartctl -a /dev/sda
sudo smartctl -t short /dev/sda          # Run test

# Clean up
sudo apt clean                           # Package cache
sudo journalctl --vacuum-time=7d         # Old logs
find /tmp -type f -atime +7 -delete      # Old temp files
```

### Performance Tuning

```bash
# Check swappiness
cat /proc/sys/vm/swappiness

# Adjust (temporary)
sudo sysctl vm.swappiness=10

# Permanent
echo "vm.swappiness=10" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

# Filesystem mount options (in /etc/fstab)
# noatime - don't update access times
# nodiratime - don't update directory access times
```

---

## 7) Practical Exercises

### Exercise 1: Set Up Log Monitoring
```bash
# Create a script to check for authentication failures
#!/bin/bash
ERRORS=$(grep -c "Failed password" /var/log/auth.log)
if [[ $ERRORS -gt 10 ]]; then
    echo "Warning: $ERRORS failed login attempts" | mail -s "Security Alert" admin@example.com
fi

# Add to cron (every 5 minutes)
*/5 * * * * /path/to/check_auth.sh
```

### Exercise 2: Create Backup Cron Job
```bash
# Add to crontab
0 2 * * * /usr/local/bin/backup.sh >> /var/log/backup.log 2>&1
```

### Exercise 3: Harden SSH
```bash
# Edit /etc/ssh/sshd_config
sudo sed -i 's/#PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config
sudo sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
sudo systemctl restart sshd
```

---

## 8) Quick Reference

| Task | Command |
|------|---------|
| View system logs | `journalctl -f` |
| View service logs | `journalctl -u service` |
| Edit crontab | `crontab -e` |
| List cron jobs | `crontab -l` |
| System uptime | `uptime` |
| Memory usage | `free -h` |
| Disk usage | `df -h` |
| Top processes | `top` or `htop` |
| Backup with tar | `tar -czvf backup.tar.gz /path` |
| Sync with rsync | `rsync -avz src/ dest/` |
| Enable firewall | `sudo ufw enable` |
| Allow port | `sudo ufw allow 80/tcp` |
| Check open ports | `sudo ss -tulpn` |
| Check failed logins | `lastb` |
| Update system | `sudo apt update && sudo apt upgrade` |

---

## 9) Interview Questions

**Q1: How do you check system logs for SSH login failures?**
> `grep "Failed password" /var/log/auth.log` or `journalctl -u ssh | grep Failed`

**Q2: How do you schedule a task to run every day at 3 AM?**
> Add to crontab: `0 3 * * * /path/to/script.sh`

**Q3: What's the difference between tar and rsync for backups?**
> `tar` creates compressed archives; good for full backups. `rsync` synchronizes files efficiently; only transfers changed data, ideal for incremental backups.

**Q4: How do you check which ports are listening on a server?**
> `sudo ss -tulpn` or `sudo netstat -tulpn`

**Q5: What security measures would you implement on a new server?**
> 1) Disable root SSH login, 2) Use SSH keys instead of passwords, 3) Enable firewall (UFW), 4) Install fail2ban, 5) Keep system updated, 6) Disable unnecessary services.

---

## 10) Checklist: New Server Setup

```
□ Set hostname
□ Create non-root user with sudo
□ Set up SSH keys
□ Disable password authentication
□ Disable root SSH login
□ Configure firewall (UFW)
□ Install and configure fail2ban
□ Set up automatic security updates
□ Configure timezone
□ Set up logging and log rotation
□ Configure backup schedule
□ Install monitoring tools
□ Harden file permissions
□ Remove unnecessary packages/services
```

---

*This completes the Linux System Administration series. Return to [README.md](README.md) for the full index.*

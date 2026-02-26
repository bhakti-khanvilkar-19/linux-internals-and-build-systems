# Linux Services & Systemd — Managing System Services

## Objective
Master systemd: manage services, understand unit files, control boot behavior, and analyze system logs.

> **Interactive Diagram**: See [diagrams/systemd_architecture.drawio](diagrams/systemd_architecture.drawio) for systemd components and flow.

---

## 1) What is systemd?

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                            SYSTEMD OVERVIEW                                         │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   systemd = System and Service Manager                                              │
│                                                                                     │
│   • PID 1 - The first process started by the kernel                                 │
│   • Manages services, mounts, devices, sockets, timers                              │
│   • Parallel service startup (fast boot)                                            │
│   • On-demand activation (socket/bus activation)                                    │
│   • Dependency-based service control                                                │
│   • Unified logging with journald                                                   │
│                                                                                     │
│   ─────────────────────────────────────────────────────────────────────────────     │
│                                                                                     │
│   Pre-systemd Era:                          systemd Era:                            │
│   • SysVinit (init scripts in /etc/init.d)  • Unit files in /lib/systemd/           │
│   • Sequential startup (slow)               • Parallel startup (fast)               │
│   • Shell scripts                           • Declarative configuration             │
│   • No dependency handling                  • Automatic dependencies                │
│                                                                                     │
│   systemd is now standard on: Ubuntu, Debian, Fedora, CentOS/RHEL, Arch, etc.       │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 2) Unit Types

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                            SYSTEMD UNIT TYPES                                       │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   Unit Type    │ Extension  │ Purpose                                               │
│   ─────────────┼────────────┼───────────────────────────────────────────            │
│   Service      │ .service   │ Daemons and processes                                 │
│   Socket       │ .socket    │ Socket-based activation                               │
│   Target       │ .target    │ Group of units (like runlevels)                       │
│   Mount        │ .mount     │ Filesystem mount points                               │
│   Automount    │ .automount │ On-demand mounting                                    │
│   Timer        │ .timer     │ Scheduled activation (like cron)                      │
│   Device       │ .device    │ Device units (auto-generated)                         │
│   Path         │ .path      │ Path-based activation                                 │
│   Slice        │ .slice     │ Resource management (cgroups)                         │
│   Scope        │ .scope     │ Externally created processes                          │
│   Swap         │ .swap      │ Swap space                                            │
│                                                                                     │
│   Most common: .service, .target, .timer                                            │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 3) systemctl — Service Control

### Basic Commands

```bash
# Service management
sudo systemctl start nginx           # Start service
sudo systemctl stop nginx            # Stop service
sudo systemctl restart nginx         # Stop and start
sudo systemctl reload nginx          # Reload config (no downtime)
sudo systemctl reload-or-restart nginx  # Reload if supported, else restart

# Enable/disable at boot
sudo systemctl enable nginx          # Start on boot
sudo systemctl disable nginx         # Don't start on boot
sudo systemctl enable --now nginx    # Enable AND start now
sudo systemctl disable --now nginx   # Disable AND stop now

# Status
systemctl status nginx               # Detailed status
systemctl is-active nginx            # Just "active" or "inactive"
systemctl is-enabled nginx           # "enabled" or "disabled"
systemctl is-failed nginx            # Check if failed

# Mask (completely disable)
sudo systemctl mask nginx            # Cannot be started at all
sudo systemctl unmask nginx          # Remove mask
```

### Listing Units

```bash
# List all units
systemctl list-units                 # Running units
systemctl list-units --all           # All units
systemctl list-units --failed        # Failed units only
systemctl list-units --type=service  # Services only

# List unit files
systemctl list-unit-files            # All unit files
systemctl list-unit-files --type=service
systemctl list-unit-files --state=enabled

# List dependencies
systemctl list-dependencies nginx    # What nginx needs
systemctl list-dependencies nginx --reverse  # What needs nginx
```

### Understanding Status Output

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                      systemctl status OUTPUT                                        │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│  $ systemctl status nginx                                                           │
│                                                                                     │
│  ● nginx.service - A high performance web server                                    │
│       Loaded: loaded (/lib/systemd/system/nginx.service; enabled; preset: enabled)  │
│       Active: active (running) since Mon 2026-02-24 10:30:00 UTC; 2h ago            │
│         Docs: man:nginx(8)                                                          │
│      Process: 1234 ExecStart=/usr/sbin/nginx -g daemon on; ... (code=exited, ...)   │
│     Main PID: 1235 (nginx)                                                          │
│        Tasks: 5 (limit: 4915)                                                       │
│       Memory: 8.5M                                                                  │
│          CPU: 35ms                                                                  │
│       CGroup: /system.slice/nginx.service                                           │
│               ├─1235 "nginx: master process /usr/sbin/nginx"                        │
│               └─1236 "nginx: worker process"                                        │
│                                                                                     │
│  Feb 24 10:30:00 server systemd[1]: Starting A high performance web server...       │
│  Feb 24 10:30:00 server systemd[1]: Started A high performance web server.          │
│                                                                                     │
│  ─────────────────────────────────────────────────────────────────────────────      │
│                                                                                     │
│  Status Icons:                                                                      │
│  ● Green = running     ○ White = inactive     ● Red = failed                        │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 4) Unit File Locations

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                         UNIT FILE LOCATIONS                                         │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   Location                          │ Purpose                │ Priority            │
│   ──────────────────────────────────┼────────────────────────┼──────────────       │
│   /lib/systemd/system/              │ Distribution defaults  │ Lowest              │
│   /run/systemd/system/              │ Runtime units          │ Medium              │
│   /etc/systemd/system/              │ Admin customizations   │ Highest             │
│                                                                                     │
│   Higher priority overrides lower!                                                  │
│                                                                                     │
│   Example:                                                                          │
│   /lib/systemd/system/nginx.service ← Package default                               │
│   /etc/systemd/system/nginx.service.d/override.conf ← Your customizations           │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### Viewing Unit Files

```bash
# Show unit file content
systemctl cat nginx.service

# Show effective configuration (all overrides)
systemctl show nginx.service

# Show specific property
systemctl show nginx -p MainPID

# Find unit file location
systemctl show nginx -p FragmentPath
```

---

## 5) Creating Custom Services

### Basic Service Unit

```bash
# Create service file
sudo nano /etc/systemd/system/myapp.service
```

```ini
[Unit]
Description=My Application
Documentation=https://myapp.example.com/docs
After=network.target
Wants=network-online.target

[Service]
Type=simple
User=myappuser
Group=myappgroup
WorkingDirectory=/opt/myapp
ExecStart=/opt/myapp/bin/myapp --config /etc/myapp/config.yaml
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure
RestartSec=5

# Environment
Environment="NODE_ENV=production"
EnvironmentFile=/etc/myapp/env

# Security
NoNewPrivileges=true
ProtectSystem=strict
ProtectHome=true
ReadWritePaths=/var/lib/myapp

[Install]
WantedBy=multi-user.target
```

### Service Types

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                          SERVICE TYPES                                              │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   Type=simple (default)                                                             │
│   • ExecStart is the main process                                                   │
│   • systemd considers it started immediately                                        │
│   • Use for processes that don't fork                                               │
│                                                                                     │
│   Type=forking                                                                      │
│   • Process forks and parent exits                                                  │
│   • Use PIDFile= to track the daemon                                                │
│   • Traditional Unix daemons                                                        │
│                                                                                     │
│   Type=oneshot                                                                      │
│   • Process exits after doing its job                                               │
│   • Often used with RemainAfterExit=yes                                             │
│   • Good for setup/cleanup scripts                                                  │
│                                                                                     │
│   Type=notify                                                                       │
│   • Process sends notification when ready                                           │
│   • Best for systemd-aware applications                                             │
│                                                                                     │
│   Type=dbus                                                                         │
│   • Ready when D-Bus name is acquired                                               │
│   • For D-Bus services                                                              │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### Activating Custom Service

```bash
# Reload systemd to see new unit
sudo systemctl daemon-reload

# Enable and start
sudo systemctl enable --now myapp

# Check status
systemctl status myapp

# View logs
journalctl -u myapp -f
```

---

## 6) Editing Services

### Edit Override (Recommended)

```bash
# Create drop-in override file
sudo systemctl edit nginx

# This creates /etc/systemd/system/nginx.service.d/override.conf
# Enter your overrides (only what you want to change):

[Service]
LimitNOFILE=65536
Environment="NGINX_LOG_LEVEL=debug"

# Save and exit

# Changes applied after daemon-reload
sudo systemctl daemon-reload
sudo systemctl restart nginx
```

### Edit Full File

```bash
# Edit full unit file (replaces original)
sudo systemctl edit nginx --full

# Or manually copy and edit
sudo cp /lib/systemd/system/nginx.service /etc/systemd/system/
sudo nano /etc/systemd/system/nginx.service
```

### Revert to Default

```bash
# Remove overrides
sudo rm -r /etc/systemd/system/nginx.service.d/
sudo systemctl daemon-reload
sudo systemctl restart nginx
```

---

## 7) Targets (Runlevels)

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                      SYSTEMD TARGETS vs SYSVINIT RUNLEVELS                          │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   Runlevel  │ Target                    │ Description                               │
│   ──────────┼───────────────────────────┼────────────────────────────────           │
│   0         │ poweroff.target           │ Shutdown                                  │
│   1         │ rescue.target             │ Single user / rescue                      │
│   2,3,4     │ multi-user.target         │ Multi-user, no GUI                        │
│   5         │ graphical.target          │ Multi-user with GUI                       │
│   6         │ reboot.target             │ Reboot                                    │
│                                                                                     │
│   Other important targets:                                                          │
│   • network.target - Network is up                                                  │
│   • network-online.target - Network is fully online                                 │
│   • basic.target - Basic system ready                                               │
│   • sysinit.target - System initialization complete                                 │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### Managing Targets

```bash
# Current default target
systemctl get-default

# Change default target
sudo systemctl set-default multi-user.target    # Boot to CLI
sudo systemctl set-default graphical.target     # Boot to GUI

# Switch target now (like changing runlevel)
sudo systemctl isolate multi-user.target        # No reboot needed

# Emergency/rescue modes
sudo systemctl rescue                           # Single user
sudo systemctl emergency                        # Minimal environment
```

---

## 8) journalctl — System Logs

### Basic Queries

```bash
# All logs
journalctl

# Follow (like tail -f)
journalctl -f

# Since boot
journalctl -b                    # Current boot
journalctl -b -1                 # Previous boot
journalctl --list-boots          # List all boots

# By service
journalctl -u nginx              # All nginx logs
journalctl -u nginx -f           # Follow nginx logs
journalctl -u nginx --since "1 hour ago"

# By time
journalctl --since "2026-02-24 10:00:00"
journalctl --since "1 hour ago"
journalctl --since yesterday
journalctl --until "2026-02-24 12:00:00"
journalctl --since "1 hour ago" --until "30 minutes ago"

# By priority
journalctl -p err                # Errors and worse
journalctl -p warning            # Warnings and worse
# Priorities: emerg, alert, crit, err, warning, notice, info, debug
```

### Advanced Queries

```bash
# By process
journalctl _PID=1234

# By user
journalctl _UID=1000

# Kernel messages (dmesg equivalent)
journalctl -k
journalctl --dmesg

# Output formats
journalctl -o json               # JSON format
journalctl -o json-pretty        # Pretty JSON
journalctl -o verbose            # All fields
journalctl -o short-iso          # ISO timestamps

# Disk usage
journalctl --disk-usage

# Vacuum old logs
sudo journalctl --vacuum-time=7d    # Keep 7 days
sudo journalctl --vacuum-size=500M  # Keep 500MB max
```

### Journal Configuration

```bash
# Config file
cat /etc/systemd/journald.conf

# Common settings:
# Storage=persistent          # Keep across reboots
# SystemMaxUse=500M          # Max disk usage
# MaxRetentionSec=1month     # Max age
```

---

## 9) Timers (Cron Alternative)

### Creating a Timer

```bash
# Service to run
sudo nano /etc/systemd/system/backup.service
```

```ini
[Unit]
Description=Daily backup

[Service]
Type=oneshot
ExecStart=/usr/local/bin/backup.sh
```

```bash
# Timer to schedule it
sudo nano /etc/systemd/system/backup.timer
```

```ini
[Unit]
Description=Run backup daily

[Timer]
OnCalendar=daily
# Or: OnCalendar=*-*-* 02:00:00  (every day at 2 AM)
# Or: OnBootSec=10min            (10 min after boot)
# Or: OnUnitActiveSec=1h         (1 hour after last run)
Persistent=true                  # Run missed runs on boot

[Install]
WantedBy=timers.target
```

```bash
# Enable timer
sudo systemctl enable --now backup.timer

# List timers
systemctl list-timers --all

# Test the service manually
sudo systemctl start backup.service
```

### Timer Time Formats

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                         ONCALENDAR EXPRESSIONS                                      │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   Expression              │ Meaning                                                 │
│   ────────────────────────┼────────────────────────────────────────────             │
│   minutely                │ Every minute                                            │
│   hourly                  │ Every hour                                              │
│   daily                   │ Every day at midnight                                   │
│   weekly                  │ Every Monday at midnight                                │
│   monthly                 │ First day of month at midnight                          │
│   yearly                  │ January 1st at midnight                                 │
│   *-*-* 02:00:00          │ Every day at 2 AM                                       │
│   Mon *-*-* 09:00:00      │ Every Monday at 9 AM                                    │
│   *-*-01 00:00:00         │ First of every month at midnight                        │
│   Sat,Sun *-*-* 10:00:00  │ Weekends at 10 AM                                       │
│   *:0/15                  │ Every 15 minutes                                        │
│                                                                                     │
│   Test with: systemd-analyze calendar "Mon *-*-* 09:00:00"                          │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 10) System Control

### Power Management

```bash
# Shutdown
sudo systemctl poweroff
sudo systemctl halt

# Reboot
sudo systemctl reboot

# Suspend/Hibernate
sudo systemctl suspend
sudo systemctl hibernate
sudo systemctl hybrid-sleep
```

### Analysis

```bash
# Boot time analysis
systemd-analyze                      # Total boot time
systemd-analyze blame                # Time per service
systemd-analyze critical-chain       # Critical path
systemd-analyze plot > boot.svg      # Visual graph

# Verify unit files
systemd-analyze verify myapp.service

# Security analysis
systemd-analyze security nginx
```

---

## 11) Practical Exercises

### Exercise 1: Create a Custom Service
```bash
# Create a simple web server service
sudo nano /etc/systemd/system/simple-web.service

# Content:
[Unit]
Description=Simple Python Web Server
After=network.target

[Service]
Type=simple
User=www-data
WorkingDirectory=/var/www/html
ExecStart=/usr/bin/python3 -m http.server 8080
Restart=always

[Install]
WantedBy=multi-user.target

# Enable and start
sudo systemctl daemon-reload
sudo systemctl enable --now simple-web
systemctl status simple-web
```

### Exercise 2: Create a Timer
```bash
# Create a cleanup service and timer
# See section 9 for detailed steps
```

### Exercise 3: Investigate Logs
```bash
# Find recent errors
journalctl -p err --since "1 hour ago"

# Find slow services at boot
systemd-analyze blame | head -10

# Check specific service logs
journalctl -u ssh -n 50
```

---

## 12) Quick Reference

| Task | Command |
|------|---------|
| Start service | `sudo systemctl start name` |
| Stop service | `sudo systemctl stop name` |
| Restart | `sudo systemctl restart name` |
| Reload config | `sudo systemctl reload name` |
| Enable at boot | `sudo systemctl enable name` |
| Disable at boot | `sudo systemctl disable name` |
| Status | `systemctl status name` |
| List all services | `systemctl list-units --type=service` |
| Failed services | `systemctl --failed` |
| Reload daemon | `sudo systemctl daemon-reload` |
| Edit service | `sudo systemctl edit name` |
| View logs | `journalctl -u name` |
| Follow logs | `journalctl -u name -f` |
| Boot time | `systemd-analyze blame` |
| Timers list | `systemctl list-timers` |

---

## 13) Interview Questions

**Q1: What is systemd and why is it PID 1?**
> systemd is the system and service manager that initializes the system. It's PID 1 because the kernel starts it as the first user-space process, and it then starts everything else.

**Q2: Difference between `enable` and `start`?**
> `start` runs the service now. `enable` configures it to start at boot. Use `enable --now` to do both.

**Q3: How do you customize a vendor service file?**
> Use `systemctl edit name` to create a drop-in override in `/etc/systemd/system/name.service.d/override.conf`. Never edit files in `/lib/systemd/system/` directly.

**Q4: What are systemd targets?**
> Targets are groups of units that define system states (like runlevels). `multi-user.target` is CLI-only, `graphical.target` includes the desktop.

**Q5: How do you view logs for a specific service?**
> `journalctl -u servicename`. Add `-f` to follow, `--since` to filter by time, `-p err` for errors only.

---

*Next: [11_Networking.md](11_Networking.md) — Network configuration, IP addressing, and troubleshooting.*

# Linux Processes — Process Management & Monitoring

## Objective
Understand how Linux manages processes: viewing, controlling, prioritizing, and monitoring system processes.

> **Interactive Diagram**: See [diagrams/process_lifecycle.drawio](diagrams/process_lifecycle.drawio) for process states and signals.

---

## 1) What is a Process?

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                          PROCESS FUNDAMENTALS                                       │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   PROCESS = A running instance of a program                                         │
│                                                                                     │
│   Every process has:                                                                │
│   • PID (Process ID) - Unique identifier                                            │
│   • PPID (Parent PID) - Who started it                                              │
│   • UID/GID - Owner user/group                                                      │
│   • State - Running, Sleeping, Stopped, Zombie                                      │
│   • Priority/Nice - Scheduling priority                                             │
│   • Memory space - Virtual address space                                            │
│   • File descriptors - Open files, sockets                                          │
│   • Environment - Variables inherited from parent                                   │
│                                                                                     │
│   ─────────────────────────────────────────────────────────────────────────────     │
│                                                                                     │
│   Process Hierarchy:                                                                │
│   ┌──────────┐                                                                      │
│   │ init (1) │  ← First process (systemd on modern Linux)                           │
│   └────┬─────┘                                                                      │
│        ├─────────────┬──────────────┐                                               │
│        ▼             ▼              ▼                                               │
│   ┌─────────┐   ┌─────────┐   ┌──────────┐                                          │
│   │ sshd    │   │ cron    │   │ systemd- │                                          │
│   │ (123)   │   │ (456)   │   │ journald │                                          │
│   └────┬────┘   └─────────┘   └──────────┘                                          │
│        │                                                                            │
│        ▼                                                                            │
│   ┌─────────┐                                                                       │
│   │ bash    │  ← Your shell session                                                 │
│   │ (789)   │                                                                       │
│   └────┬────┘                                                                       │
│        │                                                                            │
│        ▼                                                                            │
│   ┌─────────┐                                                                       │
│   │ vim     │  ← Program you run                                                    │
│   │ (1000)  │                                                                       │
│   └─────────┘                                                                       │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 2) Process States

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                           PROCESS STATES                                            │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   ┌───────────┐                                                                     │
│   │  Created  │                                                                     │
│   └─────┬─────┘                                                                     │
│         │ fork()                                                                    │
│         ▼                                                                           │
│   ┌───────────┐     schedule()    ┌───────────┐                                     │
│   │   Ready   │ ◄───────────────► │  Running  │──────────────┐                      │
│   │  (queue)  │                   │   (CPU)   │              │                      │
│   └───────────┘                   └─────┬─────┘              │ exit()               │
│         ▲                               │                    │                      │
│         │ I/O complete                  │ I/O wait           ▼                      │
│         │                               ▼              ┌───────────┐                │
│   ┌─────┴─────┐                   ┌───────────┐        │Terminated │                │
│   │ Sleeping  │ ◄──────────────── │  Blocked  │        │ (Zombie)  │                │
│   │   (S/D)   │                   │    (D)    │        └───────────┘                │
│   └───────────┘                   └───────────┘              │                      │
│         │                                                    │ parent wait()        │
│         │ SIGSTOP                                            ▼                      │
│         ▼                                               ┌───────────┐               │
│   ┌───────────┐                                         │  Removed  │               │
│   │  Stopped  │ ── SIGCONT ─────────────────────────►   └───────────┘               │
│   │    (T)    │                                                                     │
│   └───────────┘                                                                     │
│                                                                                     │
│   State Codes in ps/top:                                                            │
│   R = Running or runnable                                                           │
│   S = Sleeping (interruptible)                                                      │
│   D = Uninterruptible sleep (usually I/O)                                           │
│   T = Stopped (by signal or debugger)                                               │
│   Z = Zombie (terminated but not reaped)                                            │
│   I = Idle kernel thread                                                            │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 3) Viewing Processes — ps

### Basic ps Commands

```bash
# Your processes
ps                           # Current shell's processes only

# All processes (BSD style)
ps aux                       # a=all users, u=user format, x=no tty

# All processes (System V style)
ps -ef                       # e=all, f=full format

# Combined useful format
ps auxf                      # With process tree (forest)

# Custom columns
ps -eo pid,ppid,user,%cpu,%mem,stat,cmd --sort=-%mem
```

### Understanding ps aux Output

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                          ps aux OUTPUT EXPLAINED                                    │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│  USER    PID  %CPU %MEM    VSZ   RSS TTY STAT START   TIME COMMAND                  │
│  root      1   0.0  0.1 169396 13436 ?   Ss   Feb20   0:08 /lib/systemd/systemd     │
│  │         │    │    │     │     │   │   │     │       │    │                       │
│  │         │    │    │     │     │   │   │     │       │    └── Command             │
│  │         │    │    │     │     │   │   │     │       └── CPU time used            │
│  │         │    │    │     │     │   │   │     └── Start time                       │
│  │         │    │    │     │     │   │   └── State (S=sleep, s=session leader)      │
│  │         │    │    │     │     │   └── Terminal (? = no terminal)                 │
│  │         │    │    │     │     └── Resident memory (KB)                           │
│  │         │    │    │     └── Virtual memory (KB)                                  │
│  │         │    │    └── Memory %                                                   │
│  │         │    └── CPU %                                                           │
│  │         └── Process ID                                                           │
│  └── Owner                                                                          │
│                                                                                     │
│  STAT Modifiers:                                                                    │
│  < = High priority    N = Low priority    L = Pages locked                          │
│  s = Session leader   l = Multi-threaded  + = Foreground process                    │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### Filtering Processes

```bash
# By name
ps aux | grep nginx
pgrep nginx                  # Returns PIDs
pgrep -l nginx               # With names
pgrep -u root                # By user

# By PID
ps -p 1234                   # Specific PID
ps -p 1234,5678              # Multiple PIDs

# By user
ps -u alice                  # Alice's processes
ps -U root -u root           # Real and effective UID

# Process tree
ps auxf                      # Forest view
pstree                       # Tree diagram
pstree -p                    # With PIDs
pstree -u alice              # User's tree
```

---

## 4) Viewing Processes — top & htop

### top — Built-in Monitor

```bash
top                          # Start top
```

**Interactive Commands in top:**

| Key | Action |
|-----|--------|
| `q` | Quit |
| `h` | Help |
| `k` | Kill process (enter PID) |
| `r` | Renice process |
| `M` | Sort by memory |
| `P` | Sort by CPU |
| `T` | Sort by time |
| `u` | Filter by user |
| `c` | Show full command |
| `1` | Show individual CPUs |
| `m` | Toggle memory display |
| `t` | Toggle task/CPU display |
| `z` | Color/mono toggle |
| `W` | Write config file |

### htop — Better Interactive Monitor

```bash
# Install
sudo apt install htop

# Run
htop
```

**htop Features:**
- Mouse support
- Color-coded
- Scroll through processes
- Tree view built-in
- Easy kill with F9
- Search with F3
- Filter with F4
- Sort with F6

---

## 5) Killing Processes

### Signals

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                           COMMON SIGNALS                                            │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   Signal    │ Number │ Default  │ Description                                       │
│   ──────────┼────────┼──────────┼────────────────────────────────────────           │
│   SIGHUP    │ 1      │ Terminate│ Hangup - often used to reload config              │
│   SIGINT    │ 2      │ Terminate│ Interrupt (Ctrl+C)                                │
│   SIGQUIT   │ 3      │ Core dump│ Quit (Ctrl+\)                                     │
│   SIGKILL   │ 9      │ Terminate│ Force kill (CANNOT be caught!)                    │
│   SIGTERM   │ 15     │ Terminate│ Graceful termination (default)                    │
│   SIGSTOP   │ 19     │ Stop     │ Pause process (CANNOT be caught)                  │
│   SIGCONT   │ 18     │ Continue │ Resume stopped process                            │
│   SIGUSR1   │ 10     │ Terminate│ User-defined signal 1                             │
│   SIGUSR2   │ 12     │ Terminate│ User-defined signal 2                             │
│                                                                                     │
│   Best practice: Try SIGTERM (15) first, then SIGKILL (9) if needed                 │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### kill Command

```bash
# Graceful kill (default SIGTERM)
kill PID
kill 1234

# Force kill
kill -9 PID                  # SIGKILL
kill -KILL PID               # Same as above

# Send specific signal
kill -HUP 1234               # Reload config
kill -STOP 1234              # Pause
kill -CONT 1234              # Resume

# Kill multiple
kill 1234 5678 9012

# List all signals
kill -l
```

### killall and pkill

```bash
# Kill by name
killall nginx                # All processes named nginx
killall -9 nginx             # Force kill

# Kill by pattern
pkill nginx                  # Matches pattern
pkill -9 nginx               # Force kill
pkill -u alice               # Kill user's processes
pkill -t pts/0               # Kill terminal's processes

# Kill older/newer than
pkill -o nginx               # Oldest matching process
pkill -n nginx               # Newest matching process
```

---

## 6) Background & Foreground Jobs

### Job Control

```bash
# Run in background
command &                    # Start in background
./long_script.sh &

# Suspend current process
# Press Ctrl+Z               # Puts in background, stopped

# List jobs
jobs                         # Show background jobs
jobs -l                      # With PIDs

# Bring to foreground
fg                           # Last job
fg %1                        # Job number 1

# Send to background
bg                           # Continue last stopped job in background
bg %1                        # Specific job

# Disown job (prevent SIGHUP on logout)
disown %1                    # Remove from job table
disown -h %1                 # Keep in table but ignore SIGHUP
```

### Running After Logout

```bash
# nohup - Immune to hangups
nohup ./long_task.sh &
# Output goes to nohup.out

# nohup with custom output
nohup ./task.sh > output.log 2>&1 &

# screen - Terminal multiplexer
screen                       # Start session
# Press Ctrl+A, then D        # Detach
screen -r                    # Reattach
screen -ls                   # List sessions

# tmux - Modern alternative
tmux                         # Start session
# Press Ctrl+B, then D        # Detach
tmux attach                  # Reattach
tmux ls                      # List sessions
```

---

## 7) Process Priority & Nice

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                           PROCESS PRIORITY                                          │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   Nice Value: -20 to +19                                                            │
│                                                                                     │
│   -20 ◄────────────────────────────────────────────────────────────► +19            │
│   HIGHEST PRIORITY                                        LOWEST PRIORITY           │
│   (greedy, gets more CPU)                              (nice, yields CPU)           │
│                                                                                     │
│   • Default nice = 0                                                                │
│   • Only root can set negative nice (higher priority)                               │
│   • Any user can increase nice (lower priority)                                     │
│                                                                                     │
│   PR (Priority) shown in top = 20 + nice                                            │
│   Example: nice -5 → PR 15                                                          │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### Setting Priority

```bash
# Start with specified nice value
nice -n 10 ./cpu_heavy_task   # Low priority
nice -n -10 ./important_task  # High priority (needs root)
nice ./task                   # Default: nice 10

# Change running process priority
renice 10 -p PID              # Set nice to 10
renice -10 -p PID             # Set nice to -10 (needs root)
renice 5 -u alice             # All of alice's processes

# In top/htop
# Press 'r', enter PID, enter new nice value
```

---

## 8) /proc Filesystem — Process Info

```bash
# Process directory
ls /proc/1234/                # PID 1234's info

# Useful files per process
cat /proc/1234/status         # Process status
cat /proc/1234/cmdline        # Command line
cat /proc/1234/environ        # Environment variables
cat /proc/1234/fd/            # Open file descriptors
cat /proc/1234/maps           # Memory maps
cat /proc/1234/stat           # Statistics
cat /proc/1234/limits         # Resource limits

# System-wide
cat /proc/loadavg             # Load average
cat /proc/uptime              # System uptime
cat /proc/meminfo             # Memory info
cat /proc/cpuinfo             # CPU info
```

---

## 9) Monitoring Tools

### System Load

```bash
# Load average
uptime
# 10:30:00 up 5 days, 3:20, 2 users, load average: 0.52, 0.48, 0.45
#                                                  │     │     │
#                                                  │     │     └── 15 min avg
#                                                  │     └── 5 min avg
#                                                  └── 1 min avg
# Rule: Load > number of CPUs = system is overloaded

# Number of CPUs
nproc
cat /proc/cpuinfo | grep processor | wc -l

# Watch load in real-time
watch -n 1 uptime
```

### Memory

```bash
# Memory usage
free -h                       # Human-readable
free -h -s 2                  # Update every 2 seconds

# Understanding free output:
#               total    used    free  shared  buff/cache   available
# Mem:           16Gi   8.0Gi   2.0Gi   512Mi      6.0Gi       7.5Gi
#
# "available" is what matters - memory than can be used for new apps
# (includes reclaimable cache)
```

### vmstat, iostat

```bash
# Virtual memory stats
vmstat 1                      # Update every 1 second
vmstat 1 5                    # 5 updates, 1 second apart

# I/O stats
iostat                        # Basic I/O statistics
iostat -x 1                   # Extended, every 1 second
```

### Other Tools

```bash
# iotop - I/O by process
sudo apt install iotop
sudo iotop

# iftop - Network by process
sudo apt install iftop
sudo iftop

# nethogs - Network per process
sudo apt install nethogs
sudo nethogs

# dstat - All-in-one
sudo apt install dstat
dstat
```

---

## 10) Resource Limits

```bash
# View limits
ulimit -a                     # All limits
ulimit -n                     # Open files limit
ulimit -u                     # Max user processes

# Set limits (session)
ulimit -n 65536               # Increase open files

# Permanent limits: /etc/security/limits.conf
# alice  hard  nofile  65536
# alice  soft  nofile  65536
# @developers hard nproc 10000
```

---

## 11) Practical Exercises

### Exercise 1: Process Investigation
```bash
# Find highest CPU process
ps aux --sort=-%cpu | head -5

# Find highest memory process
ps aux --sort=-%mem | head -5

# Count processes by user
ps -eo user | sort | uniq -c | sort -rn

# Find process using a port
sudo lsof -i :80
sudo ss -tlnp | grep :80
```

### Exercise 2: Job Control
```bash
# Start a long-running process
sleep 300 &

# Check jobs
jobs -l

# Bring to foreground
fg %1

# Suspend it (Ctrl+Z)
# Send back to background
bg %1

# Kill it
kill %1
```

### Exercise 3: Process Tree
```bash
# Find your shell's process tree
pstree -s $$                  # $$ = current shell PID

# Find all children of a process
pstree -p 1                   # Children of init
```

---

## 12) Quick Reference

| Task | Command |
|------|---------|
| All processes | `ps aux` |
| Process tree | `ps auxf` or `pstree` |
| By name | `pgrep -l name` |
| Interactive | `htop` |
| Graceful kill | `kill PID` |
| Force kill | `kill -9 PID` |
| Kill by name | `killall name` |
| Background | `command &` |
| Jobs list | `jobs -l` |
| Foreground | `fg %n` |
| Background cont. | `bg %n` |
| Set priority | `nice -n N cmd` |
| Change priority | `renice N -p PID` |
| Load average | `uptime` |
| Memory | `free -h` |

---

## 13) Interview Questions

**Q1: What's the difference between SIGTERM and SIGKILL?**
> SIGTERM (15) asks a process to terminate gracefully - the process can catch it and clean up. SIGKILL (9) forces immediate termination - cannot be caught or ignored.

**Q2: What is a zombie process and how do you fix it?**
> A zombie is a process that has finished but its parent hasn't called wait() to read its exit status. The parent process needs to be fixed or killed to clean up zombies.

**Q3: What does load average mean?**
> The average number of processes waiting for CPU over 1, 5, and 15 minutes. A load of 1.0 on a single CPU means it's fully utilized. Load > number of CPUs indicates overload.

**Q4: How do you run a process that survives logout?**
> Use `nohup command &`, `screen`, or `tmux`. These prevent the process from receiving SIGHUP when the terminal closes.

**Q5: What's the difference between nice and renice?**
> `nice` sets the priority when starting a process. `renice` changes the priority of an already running process.

---

*Next: [10_Services_Systemd.md](10_Services_Systemd.md) — Managing services with systemd.*

# Linux File System — Directory Structure, Paths & Navigation

## Objective
Master Linux filesystem concepts: directory hierarchy, absolute vs relative paths, navigation commands, and file types.

> **Interactive Diagram**: Open [diagrams/file_system_navigation.drawio](diagrams/file_system_navigation.drawio) for visual path navigation flows.

---

## 1) Everything is a File

In Linux, **everything is represented as a file**:

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                         EVERYTHING IS A FILE                                        │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   Regular files        →  Documents, images, executables                            │
│   Directories          →  Special files containing other files                      │
│   Device files         →  /dev/sda (disk), /dev/tty (terminal)                      │
│   Symbolic links       →  Pointers to other files                                   │
│   Sockets              →  Inter-process communication                               │
│   Named pipes (FIFO)   →  One-way data channel                                      │
│                                                                                     │
│   This unified model makes Linux powerful and flexible!                             │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### File Types (shown by `ls -l`)

| Symbol | Type | Example |
|--------|------|---------|
| `-` | Regular file | `-rw-r--r-- file.txt` |
| `d` | Directory | `drwxr-xr-x Documents/` |
| `l` | Symbolic link | `lrwxrwxrwx link -> target` |
| `c` | Character device | `crw-rw---- /dev/tty` |
| `b` | Block device | `brw-rw---- /dev/sda` |
| `s` | Socket | `srwxrwxrwx /var/run/docker.sock` |
| `p` | Named pipe | `prw-r--r-- mypipe` |

---

## 2) The Filesystem Hierarchy (FHS)

```
/                           Root - The top of everything
│
├── bin/                    Essential user binaries (ls, cp, cat)
│                           Modern: symlink to /usr/bin
│
├── boot/                   Boot files
│   ├── vmlinuz-*          Kernel images
│   ├── initrd.img-*       Initial RAM disks
│   └── grub/              Bootloader config
│
├── dev/                    Device files (kernel-generated)
│   ├── sda, sda1          Block devices (disks)
│   ├── tty*               Terminal devices
│   ├── null               Discard output
│   ├── zero               Infinite zeros
│   └── random             Random numbers
│
├── etc/                    System configuration
│   ├── passwd             User accounts
│   ├── shadow             Encrypted passwords
│   ├── fstab              Filesystem mount table
│   ├── hosts              Static hostname mapping
│   ├── hostname           System hostname
│   └── apt/               Package manager config
│
├── home/                   User home directories
│   ├── alice/
│   │   ├── Documents/
│   │   ├── Downloads/
│   │   ├── .bashrc        Shell config
│   │   └── .config/       App configs (XDG)
│   └── bob/
│
├── lib/, lib64/            Essential libraries
│                           Modern: symlink to /usr/lib
│
├── media/                  Removable media (auto-mounted)
│   └── username/
│       └── USB_DRIVE/
│
├── mnt/                    Temporary mount points (manual)
│
├── opt/                    Optional/third-party software
│   ├── google/chrome/
│   └── vscode/
│
├── proc/                   Process information (virtual)
│   ├── 1/                 PID 1 (init) info
│   ├── cpuinfo            CPU details
│   ├── meminfo            Memory stats
│   └── cmdline            Kernel parameters
│
├── root/                   Root user's home directory
│
├── run/                    Runtime data (tmpfs, cleared on boot)
│   ├── user/1000/         Per-user runtime
│   └── systemd/           Systemd runtime
│
├── sbin/                   System binaries (admin tools)
│                           Modern: symlink to /usr/sbin
│
├── srv/                    Service data (web, FTP)
│   └── www/               Web server files
│
├── sys/                    Kernel/hardware info (virtual)
│   ├── class/             Device classes
│   └── devices/           Device tree
│
├── tmp/                    Temporary files (anyone can write)
│                           Cleared on boot
│
├── usr/                    User programs (largest directory)
│   ├── bin/               User binaries
│   ├── sbin/              System binaries
│   ├── lib/               Libraries
│   ├── local/             Locally installed software
│   ├── share/             Architecture-independent data
│   │   ├── man/           Manual pages
│   │   └── doc/           Documentation
│   └── include/           C header files
│
└── var/                    Variable data
    ├── log/               System logs
    │   ├── syslog
    │   ├── auth.log
    │   └── kern.log
    ├── cache/             Caches
    ├── lib/               Application state
    ├── spool/             Queues (mail, print)
    └── tmp/               Persistent temp files
```

---

## 3) Paths: Absolute vs Relative

### Absolute Paths
Start from root `/`, always same location regardless of current directory:

```bash
/home/alice/Documents/report.txt
/etc/passwd
/usr/bin/python3
```

### Relative Paths
Start from current directory:

```bash
Documents/report.txt        # From home directory
../bob/file.txt             # Go up one, into bob
./script.sh                 # Current directory
../../etc/passwd            # Go up two levels
```

### Special Directory Symbols

| Symbol | Meaning | Example |
|--------|---------|---------|
| `/` | Root directory | `cd /` |
| `.` | Current directory | `./script.sh` |
| `..` | Parent directory | `cd ..` |
| `~` | Home directory | `cd ~` or `cd ~/Documents` |
| `~user` | Another user's home | `cd ~bob` |
| `-` | Previous directory | `cd -` |

### Path Navigation Visual

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                           PATH NAVIGATION                                           │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   Current location: /home/alice/Documents                                           │
│                                                                                     │
│   /                                                                                 │
│   └── home/                                                                         │
│       ├── alice/           ← ~  (home)                                              │
│       │   ├── Documents/   ← .  (current)                                           │
│       │   │   └── report.txt                                                        │
│       │   ├── Downloads/   ← ../Downloads (sibling)                                 │
│       │   └── .bashrc                                                               │
│       └── bob/             ← ../../bob or ~bob                                      │
│           └── file.txt                                                              │
│                                                                                     │
│   From /home/alice/Documents:                                                       │
│   ─────────────────────────────────────────────────────────────────────────────     │
│   report.txt          →  ./report.txt (same dir)                                    │
│   ../Downloads        →  /home/alice/Downloads                                      │
│   ../../bob           →  /home/bob                                                  │
│   ~                   →  /home/alice                                                │
│   /etc/passwd         →  Always /etc/passwd (absolute)                              │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 4) Navigation Commands

### pwd — Print Working Directory

```bash
pwd
# /home/alice/Documents

pwd -P    # Physical path (resolve symlinks)
pwd -L    # Logical path (keep symlinks) - default
```

### cd — Change Directory

```bash
cd /etc                     # Go to /etc
cd                          # Go to home (~)
cd ~                        # Go to home (explicit)
cd -                        # Go to previous directory
cd ..                       # Go up one level
cd ../..                    # Go up two levels
cd ~/Documents              # Go to Documents in home
cd "path with spaces"       # Paths with spaces need quotes
```

### ls — List Directory Contents

```bash
ls                          # Basic listing
ls -l                       # Long format (permissions, size, date)
ls -la                      # Include hidden files (start with .)
ls -lh                      # Human-readable sizes (KB, MB, GB)
ls -lt                      # Sort by modification time
ls -lS                      # Sort by size (largest first)
ls -lR                      # Recursive (list subdirectories)
ls -ld /etc                 # Show directory itself, not contents
ls -li                      # Show inode numbers
```

### Understanding `ls -l` Output

```
drwxr-xr-x  5  alice  staff  4096  Feb 24 10:30  Documents
│├──┬──┤   │  │      │      │     │             │
││  │  │   │  │      │      │     │             └── Name
││  │  │   │  │      │      │     └── Modification time
││  │  │   │  │      │      └── Size (bytes)
││  │  │   │  │      └── Group
││  │  │   │  └── Owner
││  │  │   └── Number of hard links
││  │  └── Others permissions (r-x = read, execute)
││  └── Group permissions (r-x)
│└── Owner permissions (rwx = read, write, execute)
└── File type (d = directory)
```

### tree — Display Directory Tree

```bash
tree                        # Show current directory tree
tree -L 2                   # Limit depth to 2 levels
tree -d                     # Directories only
tree -a                     # Include hidden files
tree -h                     # Human-readable sizes
tree /etc -L 1              # Show /etc, 1 level deep

# Install if not present:
sudo apt install tree
```

---

## 5) File Operations

### Viewing Files

```bash
cat file.txt                # Print entire file
less file.txt               # Paginated viewer (q to quit)
more file.txt               # Simple pager
head file.txt               # First 10 lines
head -n 20 file.txt         # First 20 lines
tail file.txt               # Last 10 lines
tail -n 20 file.txt         # Last 20 lines
tail -f /var/log/syslog     # Follow (live updates)
```

### Creating Files and Directories

```bash
touch newfile.txt           # Create empty file (or update timestamp)
mkdir newdir                # Create directory
mkdir -p path/to/deep/dir   # Create parent directories as needed
mkdir -m 755 mydir          # Create with specific permissions
```

### Copying

```bash
cp source dest              # Copy file
cp -r source_dir dest_dir   # Copy directory recursively
cp -i source dest           # Interactive (ask before overwrite)
cp -v source dest           # Verbose (show what's being copied)
cp -p source dest           # Preserve permissions and timestamps
cp -a source dest           # Archive mode (preserve everything)
```

### Moving/Renaming

```bash
mv oldname newname          # Rename file
mv file /path/to/dest/      # Move file
mv -i source dest           # Interactive (ask before overwrite)
mv -v source dest           # Verbose
```

### Deleting

```bash
rm file.txt                 # Remove file
rm -r directory/            # Remove directory recursively
rm -f file.txt              # Force (no confirmation)
rm -rf directory/           # Force remove directory (DANGEROUS!)
rm -i file.txt              # Interactive (ask before each)
rmdir emptydir              # Remove empty directory only
```

### Linking

```bash
# Symbolic (soft) link - like a shortcut
ln -s /path/to/target linkname

# Hard link - same inode, same data
ln /path/to/target linkname

# Example:
ln -s /var/log/syslog ~/syslog-link
ls -l ~/syslog-link
# lrwxrwxrwx 1 alice alice 15 Feb 24 10:30 syslog-link -> /var/log/syslog
```

### Symbolic vs Hard Links

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                      SYMBOLIC vs HARD LINKS                                         │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   SYMBOLIC LINK (Soft Link)          HARD LINK                                      │
│   ─────────────────────────          ─────────                                      │
│                                                                                     │
│   link → target                      link ──┐                                       │
│     │                                       │                                       │
│     └──→ /path/to/target                    ├──→ [inode] → [data]                   │
│              │                              │                                       │
│              └──→ [inode] → [data]   target ┘                                       │
│                                                                                     │
│   • Can link to directories         • Cannot link to directories                    │
│   • Can cross filesystems           • Must be same filesystem                       │
│   • Breaks if target deleted        • Data exists until ALL links deleted           │
│   • Different inode                 • Same inode                                    │
│   • Small file (stores path)        • No extra space                                │
│                                                                                     │
│   Use case: Shortcuts, aliases      Use case: Backup, prevent deletion              │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 6) Finding Files

### find — Search by Criteria

```bash
# By name
find /home -name "*.txt"              # Find .txt files
find . -iname "readme*"               # Case-insensitive

# By type
find /var -type f                     # Files only
find /var -type d                     # Directories only
find /dev -type l                     # Symbolic links

# By size
find . -size +100M                    # Larger than 100MB
find . -size -1k                      # Smaller than 1KB

# By time
find . -mtime -7                      # Modified in last 7 days
find . -atime +30                     # Accessed more than 30 days ago

# By permission
find . -perm 755                      # Exact permission
find . -perm -644                     # At least these permissions

# By owner
find /home -user alice                # Owned by alice
find /var -group www-data             # Owned by group

# Execute action
find . -name "*.tmp" -delete          # Delete matching files
find . -name "*.sh" -exec chmod +x {} \;  # Make executable
find . -name "*.log" -exec ls -l {} \;    # List details
```

### locate — Fast Database Search

```bash
locate filename                        # Fast search (uses database)
locate -i readme                       # Case-insensitive
sudo updatedb                          # Update the database

# Note: locate is faster but may not find recently created files
```

### which / whereis / type

```bash
which python3                          # Find executable in PATH
# /usr/bin/python3

whereis python3                        # Find binary, source, and man pages
# python3: /usr/bin/python3 /usr/lib/python3 /usr/share/man/man1/python3.1.gz

type ls                                # Show what a command is
# ls is aliased to 'ls --color=auto'
type -a python                         # Show all locations
```

---

## 7) File Information

### stat — Detailed File Info

```bash
stat file.txt
#   File: file.txt
#   Size: 1234       Blocks: 8          IO Block: 4096   regular file
# Device: 259,2      Inode: 123456      Links: 1
# Access: (0644/-rw-r--r--)  Uid: (1000/alice)   Gid: (1000/alice)
# Access: 2026-02-24 10:30:00.000000000 +0530
# Modify: 2026-02-24 09:15:00.000000000 +0530
# Change: 2026-02-24 09:15:00.000000000 +0530
#  Birth: 2026-02-20 14:00:00.000000000 +0530
```

### file — Determine File Type

```bash
file document.pdf
# document.pdf: PDF document, version 1.4

file /bin/ls
# /bin/ls: ELF 64-bit LSB pie executable, x86-64

file script.sh
# script.sh: Bourne-Again shell script, ASCII text executable
```

### du — Disk Usage

```bash
du -h file.txt                         # File size
du -sh directory/                      # Total directory size
du -sh */                              # Size of each subdirectory
du -ah /home/alice | sort -h | tail -10  # Top 10 largest
```

### df — Filesystem Disk Space

```bash
df -h                                  # Human-readable
df -h /                                # Specific mount point
df -T                                  # Show filesystem type
df -i                                  # Show inode usage
```

---

## 8) Wildcards (Globbing)

```bash
# * — Matches any characters (zero or more)
ls *.txt                               # All .txt files
ls file*                               # Files starting with "file"
ls *report*                            # Files containing "report"

# ? — Matches exactly one character
ls file?.txt                           # file1.txt, fileA.txt (not file10.txt)
ls ???.txt                             # Exactly 3-char name + .txt

# [] — Matches any character in brackets
ls file[123].txt                       # file1.txt, file2.txt, file3.txt
ls file[a-z].txt                       # filea.txt through filez.txt
ls file[!0-9].txt                      # NOT a digit

# {} — Brace expansion
cp file.{txt,bak}                      # cp file.txt file.bak
mkdir -p project/{src,bin,doc}         # Create multiple dirs
touch file{1..5}.txt                   # file1.txt through file5.txt
echo {A..Z}                            # A B C ... Z
```

---

## 9) Hidden Files

Files starting with `.` are hidden by default:

```bash
ls -a                                  # Show all (including hidden)
ls -la                                 # Long listing with hidden

# Common hidden files:
~/.bashrc                              # Bash configuration
~/.bash_history                        # Command history
~/.profile                             # Login shell config
~/.config/                             # Application configs (XDG)
~/.local/                              # User-local data
~/.ssh/                                # SSH keys and config
~/.gitconfig                           # Git configuration
```

---

## 10) Practical Examples

### Example 1: Organize Downloads

```bash
cd ~/Downloads

# Create folders
mkdir -p Documents Images Videos Archives

# Move files by extension
mv *.pdf *.docx *.txt Documents/ 2>/dev/null
mv *.jpg *.png *.gif Images/ 2>/dev/null
mv *.mp4 *.mkv Videos/ 2>/dev/null
mv *.zip *.tar.gz *.7z Archives/ 2>/dev/null

# Verify
tree -L 1
```

### Example 2: Find Large Files

```bash
# Find files larger than 100MB
find ~ -type f -size +100M 2>/dev/null

# Top 10 largest files in home
du -ah ~ 2>/dev/null | sort -rh | head -10

# Find and list with details
find ~ -type f -size +100M -exec ls -lh {} \; 2>/dev/null
```

### Example 3: Backup Configuration

```bash
# Create backup directory
mkdir -p ~/backups/config-$(date +%Y%m%d)

# Backup important config files
cp -a ~/.bashrc ~/backups/config-$(date +%Y%m%d)/
cp -a ~/.profile ~/backups/config-$(date +%Y%m%d)/
cp -a ~/.ssh ~/backups/config-$(date +%Y%m%d)/ 2>/dev/null

# Verify
ls -la ~/backups/config-$(date +%Y%m%d)/
```

---

## 11) Interview Questions

**Q: What's the difference between `/tmp` and `/var/tmp`?**
A: `/tmp` is cleared on reboot (often tmpfs in RAM), `/var/tmp` persists across reboots.

**Q: What's an inode?**
A: An inode is a data structure storing file metadata (permissions, ownership, timestamps, data block pointers) — everything except filename and content.

**Q: Difference between `rm` and `unlink`?**
A: `rm` can remove directories and multiple files; `unlink` removes only single files. Both decrement the link count.

**Q: How to find files modified in last 24 hours?**
A: `find /path -mtime -1` or `find /path -mmin -1440`

**Q: What does `rm -rf /` do?**
A: Recursively deletes everything from root. NEVER run this! Modern systems have `--preserve-root` protection.

---

## 12) Exercises

1. Navigate to `/var/log` and list files sorted by size
2. Create a directory structure: `project/{src,test,docs}`
3. Find all `.conf` files in `/etc`
4. Create a symbolic link to `/var/log/syslog` in your home
5. Find files larger than 10MB in your home directory
6. Use `tree` to visualize `/usr/share` (2 levels deep)

---

## 13) Quick Reference Card

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                       FILE SYSTEM QUICK REFERENCE                                   │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│  NAVIGATION                                                                         │
│  pwd             Print working directory                                            │
│  cd /path        Change to absolute path                                            │
│  cd ~            Go home                                                            │
│  cd -            Go to previous directory                                           │
│  cd ..           Go up one level                                                    │
│                                                                                     │
│  LISTING                                                                            │
│  ls -la          List all with details                                              │
│  ls -lh          Human-readable sizes                                               │
│  ls -lt          Sort by time                                                       │
│  tree -L 2       Tree view, 2 levels                                                │
│                                                                                     │
│  FILE OPERATIONS                                                                    │
│  touch file      Create empty file                                                  │
│  mkdir -p path   Create directories                                                 │
│  cp -r src dst   Copy recursively                                                   │
│  mv old new      Move/rename                                                        │
│  rm -r dir       Remove directory                                                   │
│  ln -s tgt lnk   Create symlink                                                     │
│                                                                                     │
│  FINDING                                                                            │
│  find . -name    Find by name                                                       │
│  find . -type    Find by type (f/d/l)                                               │
│  find . -size    Find by size (+100M)                                               │
│  locate name     Fast search (database)                                             │
│  which cmd       Find executable                                                    │
│                                                                                     │
│  INFO                                                                               │
│  stat file       Detailed info                                                      │
│  file name       File type                                                          │
│  du -sh dir      Directory size                                                     │
│  df -h           Disk space                                                         │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

---

*Next: [06_Basic_Commands.md](06_Basic_Commands.md) — Essential terminal commands*

# Linux Users & Permissions — Complete Guide

## Objective
Understand Linux user management, groups, file permissions, and access control with practical examples.

> **Interactive Diagram**: Open [diagrams/permissions_chmod.drawio](diagrams/permissions_chmod.drawio) for visual permission calculations.

---

## 1) Users and Groups Concept

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                         USERS AND GROUPS                                            │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   USERS                                  GROUPS                                     │
│   ─────                                  ──────                                     │
│   • Each user has unique UID             • Group users for common access            │
│   • UID 0 = root (superuser)             • Each group has unique GID                │
│   • UIDs 1-999 = system accounts         • User belongs to primary group            │
│   • UIDs 1000+ = regular users           • User can belong to many groups           │
│                                                                                     │
│   ┌────────────┐                         ┌────────────┐                             │
│   │ alice      │ ──primary──────────────▶│ alice (GID)│                             │
│   │ UID: 1000  │       │                 └────────────┘                             │
│   └────────────┘       │                                                            │
│         │              └───supplementary──▶ sudo, docker, www-data                  │
│         │                                                                           │
│         ▼                                                                           │
│   Files created by alice have:                                                      │
│   owner=alice, group=alice (primary group)                                          │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 2) User Information Files

### /etc/passwd — User Accounts

```bash
cat /etc/passwd
# username:x:UID:GID:GECOS:home:shell
# alice:x:1000:1000:Alice Smith:/home/alice:/bin/bash
#   │   │  │    │      │          │           │
#   │   │  │    │      │          │           └── Login shell
#   │   │  │    │      │          └── Home directory
#   │   │  │    │      └── Comment/full name (GECOS)
#   │   │  │    └── Primary GID
#   │   │  └── UID
#   │   └── Password placeholder (actual in /etc/shadow)
#   └── Username
```

### /etc/shadow — Encrypted Passwords

```bash
sudo cat /etc/shadow
# alice:$6$salt$hash:19000:0:99999:7:::
#   │      │           │   │   │   │
#   │      │           │   │   │   └── Days before expiry warning
#   │      │           │   │   └── Max days password valid
#   │      │           │   └── Min days between changes
#   │      │           └── Last password change (days since 1970)
#   │      └── Encrypted password ($6$ = SHA-512)
#   └── Username
```

### /etc/group — Groups

```bash
cat /etc/group
# groupname:x:GID:members
# sudo:x:27:alice,bob
# docker:x:999:alice
```

### /etc/gshadow — Group Passwords

```bash
sudo cat /etc/gshadow     # Group password info (rarely used)
```

---

## 3) User Management Commands

### Creating Users

```bash
# Create user with home directory
sudo useradd -m username

# Create with specific options
sudo useradd -m -s /bin/bash -c "Full Name" -G sudo,docker username

# Options:
# -m           Create home directory
# -d /path     Specify home directory
# -s /bin/bash Specify shell
# -c "Comment" Full name/comment
# -G group1,group2  Add to supplementary groups
# -u 1500      Specify UID
# -g groupname Specify primary group

# Interactive user creation (Debian/Ubuntu)
sudo adduser username      # Prompts for password and info
```

### Modifying Users

```bash
# Change shell
sudo usermod -s /bin/zsh username

# Add to supplementary group (IMPORTANT: use -a to append!)
sudo usermod -aG docker username    # -a = append, -G = groups

# Change home directory
sudo usermod -d /new/home -m username   # -m moves files

# Lock/unlock account
sudo usermod -L username    # Lock
sudo usermod -U username    # Unlock

# Change username
sudo usermod -l newname oldname
```

### Deleting Users

```bash
sudo userdel username           # Delete user (keep home)
sudo userdel -r username        # Delete user AND home directory
sudo deluser username           # Debian/Ubuntu alternative
sudo deluser --remove-home username
```

### Password Management

```bash
sudo passwd username            # Set password for user
passwd                          # Change own password

# Password policies
sudo chage -l username          # View password aging info
sudo chage -M 90 username       # Max days before change required
sudo chage -E 2026-12-31 username  # Account expiry date
sudo passwd -e username         # Force password change on next login
```

---

## 4) Group Management

```bash
# Create group
sudo groupadd groupname
sudo groupadd -g 1500 groupname    # Specific GID

# Delete group
sudo groupdel groupname

# Modify group
sudo groupmod -n newname oldname   # Rename
sudo groupmod -g 2000 groupname    # Change GID

# Add user to group
sudo usermod -aG groupname username
sudo gpasswd -a username groupname    # Alternative

# Remove user from group
sudo gpasswd -d username groupname

# View groups
groups                          # Your groups
groups username                 # User's groups
id                              # Your UID, GID, groups
id username                     # User's info
```

---

## 5) Switching Users

```bash
# Switch to root
sudo -i                         # Root shell (login)
sudo -s                         # Root shell (non-login)
su -                            # Switch to root (needs root password)

# Switch to another user
su - username                   # Switch user (login shell)
su username                     # Switch user (non-login)
sudo -u username command        # Run command as user

# Run command as root
sudo command                    # Run single command as root
sudo !!                         # Run last command with sudo
```

---

## 6) File Permissions — The Basics

### Understanding Permission String

```
-rwxr-xr-- 1 alice developers 4096 Feb 24 10:30 script.sh
│├─┬┤├─┬┤├─┬┤
││ │ │ │ │ │
││ │ │ │ └─┴── Others (everyone else): r-- = read only
││ │ └─┴────── Group (developers): r-x = read, execute
│└─┴────────── Owner (alice): rwx = read, write, execute
└───────────── File type (- = regular file)

Permission bits:
r (read)    = 4
w (write)   = 2
x (execute) = 1

Combinations:
rwx = 7 (4+2+1)
rw- = 6 (4+2)
r-x = 5 (4+1)
r-- = 4
--- = 0
```

### Permission Meaning for Files vs Directories

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                    PERMISSION EFFECTS                                               │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   Permission    For FILE                    For DIRECTORY                           │
│   ──────────────────────────────────────────────────────────────────────            │
│   r (read)      View contents               List contents (ls)                      │
│   w (write)     Modify/delete contents      Create/delete files inside              │
│   x (execute)   Run as program              Enter directory (cd)                    │
│                                                                                     │
│   DIRECTORY SPECIAL CASES:                                                          │
│   ─────────────────────────                                                         │
│   • Need x to cd into directory                                                     │
│   • Need r to list files                                                            │
│   • Need w+x to create/delete files in directory                                    │
│   • Can delete file if you have w on DIRECTORY (not file!)                          │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 7) chmod — Change Permissions

### Symbolic Mode

```bash
# Format: chmod [who][+/-/=][permissions] file

# Who:
# u = user (owner)
# g = group
# o = others
# a = all (ugo)

# Examples:
chmod u+x script.sh             # Add execute for owner
chmod g-w file.txt              # Remove write from group
chmod o=r file.txt              # Set others to read only
chmod a+r file.txt              # Add read for all
chmod ug+rw file.txt            # Add read/write for owner and group
chmod u+x,g+r,o-rwx file.txt    # Multiple changes
chmod -R g+w directory/         # Recursive
```

### Numeric (Octal) Mode

```bash
# Each digit = user, group, others
chmod 755 script.sh             # rwxr-xr-x
chmod 644 file.txt              # rw-r--r--
chmod 700 private/              # rwx------ (owner only)
chmod 777 open.txt              # rwxrwxrwx (dangerous!)
chmod 600 secrets.txt           # rw------- (owner read/write)
chmod 750 shared/               # rwxr-x--- (owner full, group rx)

# Common patterns:
# 755 - Executables, public directories
# 644 - Regular files
# 700 - Private directories
# 600 - Private files, SSH keys
# 775 - Shared directories (group can write)
```

### Permission Calculator

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                      PERMISSION CALCULATOR                                          │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│         Owner (u)         Group (g)         Others (o)                              │
│         ─────────         ─────────         ──────────                              │
│         r  w  x           r  w  x           r  w  x                                 │
│         4  2  1           4  2  1           4  2  1                                 │
│                                                                                     │
│   Example: rw-r--r-- = ?                                                            │
│                                                                                     │
│         r  w  -           r  -  -           r  -  -                                 │
│         4 +2 +0 = 6       4 +0 +0 = 4       4 +0 +0 = 4                             │
│                                                                                     │
│   Answer: 644                                                                       │
│                                                                                     │
│   ─────────────────────────────────────────────────────────────────────────────     │
│                                                                                     │
│   Example: 751 = ?                                                                  │
│                                                                                     │
│         7 = rwx           5 = r-x           1 = --x                                 │
│         (4+2+1)           (4+0+1)           (0+0+1)                                 │
│                                                                                     │
│   Answer: rwxr-x--x                                                                 │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 8) chown — Change Ownership

```bash
# Change owner
sudo chown alice file.txt

# Change owner and group
sudo chown alice:developers file.txt

# Change group only
sudo chown :developers file.txt
sudo chgrp developers file.txt       # Alternative

# Recursive
sudo chown -R alice:alice /home/alice/

# Copy ownership from another file
sudo chown --reference=source.txt target.txt
```

---

## 9) Special Permissions

### SUID (Set User ID) — Run as Owner

```bash
# When executed, runs with owner's permissions
chmod u+s executable           # Add SUID
chmod 4755 executable          # Numeric (4 = SUID)

ls -l /usr/bin/passwd
# -rwsr-xr-x  (note 's' instead of 'x')
# passwd runs as root to modify /etc/shadow
```

### SGID (Set Group ID)

```bash
# For files: Run with group's permissions
# For directories: New files inherit directory's group
chmod g+s directory/           # Add SGID
chmod 2755 directory/          # Numeric (2 = SGID)

# Example: Shared project directory
sudo mkdir /project
sudo chown :developers /project
sudo chmod 2775 /project       # SGID + rwxrwxr-x
# Now files created inside belong to 'developers' group
```

### Sticky Bit — Restrict Deletion

```bash
# Only owner can delete their files (even if dir is writable)
chmod +t directory/            # Add sticky bit
chmod 1777 /tmp/               # Numeric (1 = sticky)

ls -ld /tmp
# drwxrwxrwt  (note 't' instead of 'x')
# Anyone can write to /tmp, but can only delete own files
```

### Special Permissions Summary

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                      SPECIAL PERMISSIONS                                            │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   Bit      Numeric   On File              On Directory                              │
│   ─────────────────────────────────────────────────────────────────                 │
│   SUID     4xxx      Execute as owner     (no effect)                               │
│   SGID     2xxx      Execute as group     New files inherit group                   │
│   Sticky   1xxx      (no effect)          Only owner can delete                     │
│                                                                                     │
│   Display:                                                                          │
│   • SUID: 's' in owner's x position (-rwsr-xr-x)                                    │
│   • SGID: 's' in group's x position (-rwxr-sr-x)                                    │
│   • Sticky: 't' in others' x position (drwxrwxrwt)                                  │
│   • Capital S/T means x is not set                                                  │
│                                                                                     │
│   Example: chmod 4755 = rwsr-xr-x (SUID + 755)                                      │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 10) umask — Default Permissions

```bash
# umask subtracts from default permissions
# Default file: 666 (rw-rw-rw-)
# Default dir:  777 (rwxrwxrwx)

umask                          # Show current umask
# 0022

# With umask 0022:
# Files: 666 - 022 = 644 (rw-r--r--)
# Dirs:  777 - 022 = 755 (rwxr-xr-x)

# Set umask
umask 027                      # More restrictive
# Files: 666 - 027 = 640 (rw-r-----)
# Dirs:  777 - 027 = 750 (rwxr-x---)

# Permanent: Add to ~/.bashrc
echo "umask 027" >> ~/.bashrc
```

---

## 11) ACLs — Access Control Lists

For more granular control beyond owner/group/others:

```bash
# Check filesystem ACL support
mount | grep acl

# View ACLs
getfacl file.txt

# Set ACL for specific user
setfacl -m u:bob:rw file.txt         # bob gets rw

# Set ACL for specific group
setfacl -m g:developers:rx file.txt  # developers get rx

# Set default ACL (for new files in directory)
setfacl -d -m g:developers:rw /project/

# Remove specific ACL
setfacl -x u:bob file.txt

# Remove all ACLs
setfacl -b file.txt

# Recursive
setfacl -R -m g:developers:rx /project/
```

### ACL Example

```bash
# File with ACL shows + in ls -l
-rw-rw-r--+ 1 alice alice 0 Feb 24 10:30 file.txt

getfacl file.txt
# # file: file.txt
# # owner: alice
# # group: alice
# user::rw-
# user:bob:rw-
# group::rw-
# group:developers:r-x
# mask::rwx
# other::r--
```

---

## 12) sudo Configuration

### /etc/sudoers

```bash
# Edit safely (validates syntax)
sudo visudo

# User specifications:
# user  host = (run_as) commands
alice   ALL=(ALL:ALL) ALL       # alice can run any command as anyone
bob     ALL=(root) /usr/bin/apt # bob can only run apt as root
%sudo   ALL=(ALL:ALL) ALL       # sudo group members can do anything

# NOPASSWD - don't require password
alice   ALL=(ALL) NOPASSWD: /usr/bin/apt

# Include directory for modular config
# Files in /etc/sudoers.d/ are included
sudo visudo -f /etc/sudoers.d/custom
```

### Checking sudo Access

```bash
sudo -l                         # List what you can do with sudo
sudo -lU username               # List for specific user (as root)
```

---

## 13) Practical Examples

### Example 1: Set Up Shared Project Directory

```bash
# Create project directory
sudo mkdir -p /projects/webapp

# Create group and add users
sudo groupadd webteam
sudo usermod -aG webteam alice
sudo usermod -aG webteam bob

# Set ownership and permissions
sudo chown root:webteam /projects/webapp
sudo chmod 2775 /projects/webapp    # SGID so files inherit group

# Verify
ls -ld /projects/webapp
# drwxrwsr-x 2 root webteam 4096 Feb 24 10:30 /projects/webapp

# Users need to logout/login for group to take effect
# Or: newgrp webteam
```

### Example 2: Secure SSH Keys

```bash
# Correct SSH permissions
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_rsa         # Private key
chmod 644 ~/.ssh/id_rsa.pub     # Public key
chmod 600 ~/.ssh/config
chmod 600 ~/.ssh/authorized_keys
```

### Example 3: Web Server Permissions

```bash
# Common web directory setup
sudo chown -R www-data:www-data /var/www/html
sudo find /var/www/html -type d -exec chmod 755 {} \;  # Dirs
sudo find /var/www/html -type f -exec chmod 644 {} \;  # Files
```

---

## 14) Interview Questions

**Q: Difference between `su` and `sudo`?**
A: `su` switches to another user (needs their password); `sudo` runs commands as root (needs YOUR password, if configured).

**Q: What's umask 027?**
A: Default permissions become 640 for files, 750 for directories. Removes all permissions from others.

**Q: How to find files with SUID set?**
A: `find / -perm -4000 2>/dev/null`

**Q: Why is /tmp sticky bit important?**
A: Without it, any user could delete any other user's temporary files in /tmp.

**Q: What happens if a directory has write but not execute permission?**
A: You cannot enter (cd) the directory, and most operations fail.

---

## 15) Quick Reference

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                    PERMISSIONS QUICK REFERENCE                                      │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│  USER COMMANDS                       PERMISSIONS                                    │
│  useradd -m user   Create user       r=4, w=2, x=1                                  │
│  usermod -aG grp   Add to group      755 = rwxr-xr-x                                │
│  userdel -r user   Delete + home     644 = rw-r--r--                                │
│  passwd user       Set password      700 = rwx------                                │
│                                                                                     │
│  OWNERSHIP                           SPECIAL                                        │
│  chown user:grp    Change owner      SUID 4xxx (run as owner)                       │
│  chgrp group       Change group      SGID 2xxx (inherit group)                      │
│  chmod 755 file    Change perms      Sticky 1xxx (owner delete)                     │
│                                                                                     │
│  VIEWING                             COMMON PATTERNS                                │
│  id                Your UID/groups   755 - Executables, public dirs                 │
│  groups user       User's groups     644 - Regular files                            │
│  ls -l             Show permissions  600 - Private files, SSH keys                  │
│  getfacl file      View ACLs         777 - AVOID! Too open                          │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

---

*Next: [08_Package_Management.md](08_Package_Management.md) — APT, dpkg, and snap*

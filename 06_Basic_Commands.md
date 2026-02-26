# Linux Basic Commands — Essential Terminal Commands

## Objective
Master essential Linux commands for daily operations: getting help, text viewing, searching, sorting, and system information.

---

## 1) Getting Help

### man — Manual Pages

```bash
man ls                      # Manual for ls
man -k keyword              # Search manuals for keyword
man 5 passwd                # Section 5 (file formats) for passwd

# Man page sections:
# 1 - User commands
# 2 - System calls
# 3 - Library functions
# 4 - Special files (/dev)
# 5 - File formats
# 6 - Games
# 7 - Miscellaneous
# 8 - System admin commands
```

### help and --help

```bash
help cd                     # Bash built-in help
ls --help                   # Quick help for command
info ls                     # Detailed info pages (GNU)
```

### apropos — Search Commands

```bash
apropos copy                # Find commands related to "copy"
apropos -s 1 network        # Search only section 1
```

---

## 2) Text Display Commands

### cat — Concatenate and Display

```bash
cat file.txt                # Display file
cat file1.txt file2.txt     # Display multiple (concatenate)
cat -n file.txt             # Show line numbers
cat -A file.txt             # Show special characters (tabs, newlines)
cat > newfile.txt           # Create file (Ctrl+D to save)
cat >> file.txt             # Append to file
```

### less — Paginated Viewer

```bash
less file.txt               # View file (scrollable)
# Navigation:
#   Space/f     - Page forward
#   b           - Page backward
#   g           - Go to beginning
#   G           - Go to end
#   /pattern    - Search forward
#   ?pattern    - Search backward
#   n           - Next search result
#   N           - Previous result
#   q           - Quit

less +F file.txt            # Follow mode (like tail -f)
```

### head and tail

```bash
head file.txt               # First 10 lines
head -n 20 file.txt         # First 20 lines
head -c 100 file.txt        # First 100 bytes

tail file.txt               # Last 10 lines
tail -n 20 file.txt         # Last 20 lines
tail -f /var/log/syslog     # Follow (live updates)
tail -f -n 50 logfile       # Last 50 + follow
```

### wc — Word Count

```bash
wc file.txt                 # Lines, words, characters
wc -l file.txt              # Lines only
wc -w file.txt              # Words only
wc -c file.txt              # Bytes
wc -m file.txt              # Characters
wc -L file.txt              # Length of longest line
```

---

## 3) Text Processing

### echo — Print Text

```bash
echo "Hello World"          # Print text
echo -n "No newline"        # Without trailing newline
echo -e "Line1\nLine2"      # Enable escape sequences
echo $HOME                  # Print variable
echo "Today is $(date)"     # Command substitution
```

### printf — Formatted Output

```bash
printf "Name: %s, Age: %d\n" "Alice" 25
printf "%10s %5d\n" "Item" 100    # Padded columns
printf "%.2f\n" 3.14159            # 2 decimal places
```

### sort — Sort Lines

```bash
sort file.txt               # Sort alphabetically
sort -n file.txt            # Sort numerically
sort -r file.txt            # Reverse order
sort -k 2 file.txt          # Sort by 2nd field
sort -t: -k 3 -n /etc/passwd  # Sort passwd by UID
sort -u file.txt            # Remove duplicates while sorting
sort -h sizes.txt           # Human-readable (1K, 2M, 3G)
```

### uniq — Remove Duplicates

```bash
sort file.txt | uniq        # Remove consecutive duplicates (sort first!)
uniq -c file.txt            # Count occurrences
uniq -d file.txt            # Show only duplicates
uniq -u file.txt            # Show only unique lines
```

### cut — Extract Columns

```bash
cut -d: -f1 /etc/passwd     # Field 1, delimiter :
cut -d, -f1,3 file.csv      # Fields 1 and 3
cut -c1-10 file.txt         # Characters 1-10
cut -c5- file.txt           # From character 5 to end
```

### paste — Merge Lines

```bash
paste file1.txt file2.txt   # Side by side (tab-separated)
paste -d, file1 file2       # Comma-separated
paste -s file.txt           # Join all lines into one
```

### tr — Translate/Delete Characters

```bash
echo "hello" | tr 'a-z' 'A-Z'    # Convert to uppercase
echo "hello" | tr '[:lower:]' '[:upper:]'  # Same
cat file.txt | tr -d '\r'        # Remove carriage returns
echo "a  b   c" | tr -s ' '      # Squeeze multiple spaces
tr ':' '\n' < /etc/passwd        # Replace : with newline
```

### rev — Reverse Lines

```bash
echo "hello" | rev          # olleh
rev file.txt                # Reverse each line
```

---

## 4) Searching

### grep — Search Text

```bash
grep "pattern" file.txt           # Find lines matching pattern
grep -i "pattern" file            # Case-insensitive
grep -v "pattern" file            # Invert (lines NOT matching)
grep -n "pattern" file            # Show line numbers
grep -c "pattern" file            # Count matches
grep -l "pattern" *.txt           # List files with matches
grep -r "pattern" /path           # Recursive search
grep -w "word" file               # Match whole word only
grep -A 2 "pattern" file          # Show 2 lines after match
grep -B 2 "pattern" file          # Show 2 lines before match
grep -C 2 "pattern" file          # Show 2 lines around match

# Extended regex (-E) or egrep:
grep -E "pattern1|pattern2" file  # OR
grep -E "^start" file             # Lines starting with
grep -E "end$" file               # Lines ending with
grep -E "[0-9]{3}" file           # 3 digits
```

### Common grep Patterns

```bash
# Find error messages
grep -i "error\|fail\|warn" /var/log/syslog

# Find IP addresses
grep -E "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" file

# Find email-like patterns
grep -E "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+" file

# Exclude comments
grep -v "^#" config.conf | grep -v "^$"
```

---

## 5) Input/Output Redirection

### Standard Streams

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                         STANDARD STREAMS                                            │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   Stream                  FD    Default       Symbol                                │
│   ────────────────────────────────────────────────────                              │
│   stdin  (input)          0     keyboard      <                                     │
│   stdout (output)         1     terminal      > or 1>                               │
│   stderr (error)          2     terminal      2>                                    │
│                                                                                     │
│                     ┌──────────┐                                                    │
│   keyboard ──→ stdin │  COMMAND │ stdout ──→ terminal                               │
│                (0)   │          │  (1)                                              │
│                      └────┬─────┘                                                   │
│                           │                                                         │
│                        stderr (2) ──→ terminal                                      │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### Redirection Operators

```bash
# Output redirection
ls > output.txt             # Redirect stdout (overwrite)
ls >> output.txt            # Append stdout
ls 2> errors.txt            # Redirect stderr
ls 2>> errors.txt           # Append stderr
ls > out.txt 2>&1           # Both stdout and stderr to file
ls &> both.txt              # Shorthand for above (bash)
ls > /dev/null 2>&1         # Discard all output

# Input redirection
sort < unsorted.txt         # Read from file
cat << EOF                  # Here document
line 1
line 2
EOF

wc -l <<< "count this"      # Here string
```

### Pipes

```bash
# Connect stdout of one command to stdin of next
ls -la | less               # Paginate listing
cat /etc/passwd | grep alice | cut -d: -f1,6

# Common pipe chains:
ps aux | grep nginx                     # Find nginx process
history | tail -20                      # Last 20 commands
cat access.log | sort | uniq -c | sort -rn | head -10  # Top IPs
find . -name "*.py" | xargs wc -l       # Count lines in Python files
```

### tee — Split Output

```bash
ls | tee output.txt         # Show on screen AND save to file
ls | tee -a output.txt      # Append instead of overwrite
ls | tee file1.txt file2.txt  # Write to multiple files
```

---

## 6) System Information

### uname — System Info

```bash
uname -a                    # All information
uname -s                    # Kernel name (Linux)
uname -r                    # Kernel release (6.8.0-...)
uname -m                    # Machine architecture (x86_64)
uname -n                    # Hostname
```

### hostname

```bash
hostname                    # Show hostname
hostname -I                 # Show IP addresses
hostnamectl                 # Detailed host info
```

### uptime

```bash
uptime                      # System uptime and load
# 10:30:00 up 5 days, 3:45,  2 users,  load average: 0.15, 0.10, 0.08
#                                                    1min  5min  15min
```

### date

```bash
date                        # Current date/time
date +%Y-%m-%d              # Format: 2026-02-24
date +%H:%M:%S              # Format: 10:30:45
date +%s                    # Unix timestamp
date -d "yesterday"         # Yesterday's date
date -d "+2 weeks"          # 2 weeks from now
TZ=UTC date                 # Show UTC time
```

### cal — Calendar

```bash
cal                         # This month
cal 2026                    # Full year
cal 3 2026                  # March 2026
ncal -w                     # Week numbers
```

### Hardware Info

```bash
lscpu                       # CPU information
free -h                     # Memory usage
lsblk                       # Block devices
lspci                       # PCI devices
lsusb                       # USB devices
dmidecode | less            # BIOS/hardware info (needs root)
```

---

## 7) History and Aliases

### Command History

```bash
history                     # Show command history
history 20                  # Last 20 commands
!100                        # Run command #100
!!                          # Run last command
!grep                       # Run last command starting with "grep"
!$                          # Last argument of previous command
!*                          # All arguments of previous command
history -c                  # Clear history
Ctrl+R                      # Reverse search history
```

### Aliases

```bash
alias                       # Show all aliases
alias ll='ls -la'           # Create alias
alias update='sudo apt update && sudo apt upgrade'
unalias ll                  # Remove alias

# Permanent aliases: add to ~/.bashrc
echo "alias ll='ls -la'" >> ~/.bashrc
source ~/.bashrc            # Reload
```

---

## 8) Useful Utilities

### clear and reset

```bash
clear                       # Clear screen
reset                       # Reset terminal (fix garbled display)
Ctrl+L                      # Shortcut to clear
```

### sleep and watch

```bash
sleep 5                     # Wait 5 seconds
sleep 0.5                   # Wait 0.5 seconds

watch df -h                 # Run command every 2 seconds
watch -n 5 date             # Every 5 seconds
watch -d ls -la             # Highlight differences
```

### yes

```bash
yes                         # Output "y" forever
yes | command               # Auto-answer "y" to prompts
yes no | command            # Auto-answer "no"
```

### diff and cmp

```bash
diff file1.txt file2.txt    # Compare files (text)
diff -u file1 file2         # Unified format (like git diff)
diff -y file1 file2         # Side-by-side
diff -r dir1 dir2           # Compare directories

cmp file1 file2             # Compare byte-by-byte
```

### xargs — Build Commands

```bash
find . -name "*.txt" | xargs rm          # Delete found files
find . -name "*.jpg" | xargs -I {} cp {} /backup/
echo "file1 file2 file3" | xargs touch   # Create multiple files
cat urls.txt | xargs wget                # Download URLs
```

---

## 9) Command Chaining

```bash
# Sequential (;) — Run regardless of success
command1 ; command2 ; command3

# AND (&&) — Run next only if previous succeeds
mkdir newdir && cd newdir && touch file.txt

# OR (||) — Run next only if previous fails
cd /nonexistent || echo "Directory not found"

# Combined
mkdir mydir && cd mydir || echo "Failed to create/enter directory"

# Background (&)
long_command &              # Run in background
jobs                        # List background jobs
fg %1                       # Bring job 1 to foreground

# Subshell ()
(cd /tmp && ls)             # Commands run in subshell
pwd                         # Still in original directory

# Command grouping {}
{ echo "start"; date; echo "end"; } > output.txt
```

---

## 10) Practical Examples

### Example 1: Log Analysis

```bash
# Count unique IPs in access log
cat access.log | cut -d' ' -f1 | sort | uniq -c | sort -rn | head -10

# Find error messages in last hour
grep "$(date -d '1 hour ago' +%H:)" /var/log/syslog | grep -i error
```

### Example 2: System Report

```bash
#!/bin/bash
echo "=== System Report ==="
echo "Hostname: $(hostname)"
echo "Date: $(date)"
echo "Uptime: $(uptime -p)"
echo "Users: $(who | wc -l)"
echo "Memory: $(free -h | grep Mem | awk '{print $3 "/" $2}')"
echo "Disk: $(df -h / | tail -1 | awk '{print $3 "/" $2}')"
```

### Example 3: Batch Rename Files

```bash
# Rename all .txt to .bak
for f in *.txt; do mv "$f" "${f%.txt}.bak"; done

# Or using rename command
rename 's/\.txt$/.bak/' *.txt
```

---

## 11) Interview Questions

**Q: What's the difference between `>` and `>>`?**
A: `>` overwrites the file, `>>` appends to it.

**Q: How to count lines in a file?**
A: `wc -l filename` or `cat filename | wc -l`

**Q: Difference between `grep` and `find`?**
A: `grep` searches file contents; `find` searches filenames and directory tree.

**Q: What does `2>&1` mean?**
A: Redirects stderr (2) to the same destination as stdout (1).

**Q: How to run a command in the background?**
A: Append `&` to the command, or use `Ctrl+Z` then `bg`.

---

## 12) Quick Reference Card

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                       BASIC COMMANDS REFERENCE                                      │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│  TEXT DISPLAY                      SEARCHING                                        │
│  cat file        Show file         grep pattern file    Search in file              │
│  less file       Paginated view    grep -r pattern dir  Recursive search            │
│  head -n 20      First 20 lines    grep -i             Case-insensitive             │
│  tail -f file    Follow updates    grep -v             Invert match                 │
│                                                                                     │
│  TEXT PROCESSING                   REDIRECTION                                      │
│  sort            Sort lines        >  file     Overwrite                            │
│  uniq            Remove dups       >> file     Append                               │
│  cut -d: -f1     Extract field     2> file     Stderr only                          │
│  tr 'a' 'b'      Replace chars     &> file     Both stdout+stderr                   │
│  wc -l           Count lines       | cmd       Pipe to command                      │
│                                                                                     │
│  SYSTEM INFO                       CHAINING                                         │
│  uname -a        Kernel info       cmd1 ; cmd2    Run both                          │
│  uptime          Uptime/load       cmd1 && cmd2   Run if success                    │
│  date            Current time      cmd1 || cmd2   Run if failure                    │
│  free -h         Memory usage      cmd &          Background                        │
│                                                                                     │
│  HISTORY                           USEFUL                                           │
│  history         Show history      alias x='cmd'  Create alias                      │
│  !!              Last command      watch cmd      Repeat command                    │
│  !$              Last argument     xargs          Build commands                    │
│  Ctrl+R          Search history    tee file       Save + display                    │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

---

*Next: [07_Users_Permissions.md](07_Users_Permissions.md) — Users, groups, and file permissions*

# Linux Shell Scripting — Bash Automation

## Objective
Master Bash scripting: write scripts for automation, understand syntax, control flow, functions, and best practices.

---

## 1) Script Basics

### Your First Script

```bash
#!/bin/bash
# This is a comment
# Filename: hello.sh

echo "Hello, World!"
```

```bash
# Make executable and run
chmod +x hello.sh
./hello.sh

# Or run with bash directly
bash hello.sh
```

### Shebang Line

```bash
#!/bin/bash        # Use bash
#!/bin/sh          # Use sh (may be dash on Ubuntu)
#!/usr/bin/env bash   # Portable (recommended)
#!/usr/bin/env python3  # For Python scripts
```

### Script Structure

```bash
#!/usr/bin/env bash
#
# Script: backup.sh
# Description: Backup home directory
# Author: Your Name
# Date: 2026-02-24
# Version: 1.0
#

set -e          # Exit on error
set -u          # Error on undefined variables
set -o pipefail # Catch pipe failures

# Constants
readonly BACKUP_DIR="/backup"
readonly LOG_FILE="/var/log/backup.log"

# Functions
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

# Main
main() {
    log "Starting backup..."
    # Your code here
    log "Backup complete"
}

# Run main
main "$@"
```

---

## 2) Variables

### Basic Variables

```bash
# Assignment (NO spaces around =)
name="John"
age=25
path="/home/user"

# Using variables
echo "Name: $name"
echo "Age: ${age}"        # Braces for clarity
echo "Path: ${path}/docs" # Required when concatenating

# Read-only
readonly PI=3.14159
# PI=3.14  # Error!

# Unset variable
unset name
```

### Special Variables

```bash
$0          # Script name
$1, $2, ... # Positional arguments
$#          # Number of arguments
$@          # All arguments (as separate words)
$*          # All arguments (as single string)
$?          # Exit status of last command
$$          # Current process ID
$!          # PID of last background process
$_          # Last argument of previous command
```

### Example: Using Arguments

```bash
#!/bin/bash
# usage.sh

echo "Script name: $0"
echo "First arg: $1"
echo "Second arg: $2"
echo "All args: $@"
echo "Number of args: $#"

# Loop through all arguments
for arg in "$@"; do
    echo "Argument: $arg"
done
```

### String Operations

```bash
str="Hello, World!"

# Length
echo ${#str}              # 13

# Substring
echo ${str:0:5}           # Hello
echo ${str:7}             # World!

# Replace
echo ${str/World/Bash}    # Hello, Bash!
echo ${str//o/_}          # Hell_, W_rld! (all occurrences)

# Remove pattern
file="document.txt.bak"
echo ${file%.bak}         # document.txt (remove suffix)
echo ${file%.*}           # document.txt (remove last extension)
echo ${file%%.*}          # document (remove all extensions)
echo ${file#*/}           # Remove prefix to first /

# Default values
echo ${name:-"default"}   # Use default if unset
echo ${name:="default"}   # Set default if unset
echo ${name:+"set"}       # Use alternate if set
echo ${name:?error}       # Error if unset
```

### Arrays

```bash
# Indexed arrays
fruits=("apple" "banana" "cherry")
echo ${fruits[0]}         # apple
echo ${fruits[@]}         # All elements
echo ${#fruits[@]}        # Length: 3

# Add element
fruits+=("date")

# Loop
for fruit in "${fruits[@]}"; do
    echo "$fruit"
done

# Associative arrays (bash 4+)
declare -A colors
colors[red]="#FF0000"
colors[green]="#00FF00"
colors[blue]="#0000FF"

echo ${colors[red]}       # #FF0000
echo ${!colors[@]}        # Keys: red green blue
echo ${colors[@]}         # Values
```

---

## 3) Input/Output

### Reading Input

```bash
# Basic read
echo "Enter your name:"
read name
echo "Hello, $name"

# Prompt on same line
read -p "Enter username: " username

# Silent input (passwords)
read -sp "Enter password: " password
echo  # Newline

# With timeout
read -t 5 -p "Quick! Enter name: " name

# Read into array
read -a arr <<< "one two three"
echo ${arr[1]}  # two
```

### Command Substitution

```bash
# Modern syntax $(command)
today=$(date +%Y-%m-%d)
files=$(ls -1 | wc -l)
user=$(whoami)

# Legacy syntax `command`
today=`date +%Y-%m-%d`

# Use in strings
echo "Today is $today"
echo "Files in dir: $(ls | wc -l)"
```

### Arithmetic

```bash
# (( )) for arithmetic
a=5
b=3
((sum = a + b))
((product = a * b))
((a++))
((b--))

# Arithmetic expansion
result=$((5 + 3))
result=$((a * b))
result=$((10 / 3))    # Integer division: 3
result=$((10 % 3))    # Modulo: 1

# For floating point, use bc
result=$(echo "scale=2; 10/3" | bc)  # 3.33
```

### Here Documents

```bash
# Multi-line strings
cat << EOF
This is a multi-line
string that can span
multiple lines.
Variable: $HOME
EOF

# No variable expansion
cat << 'EOF'
Variables like $HOME
are NOT expanded here.
EOF

# Here string
grep "pattern" <<< "search in this string"
```

---

## 4) Conditionals

### Test Commands

```bash
# [ ] is test command
[ condition ]

# [[ ]] is modern bash (recommended)
[[ condition ]]

# (( )) for arithmetic conditions
(( arithmetic_expression ))
```

### if/elif/else

```bash
#!/bin/bash

if [[ $1 -gt 10 ]]; then
    echo "Greater than 10"
elif [[ $1 -eq 10 ]]; then
    echo "Equal to 10"
else
    echo "Less than 10"
fi
```

### Comparison Operators

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                        COMPARISON OPERATORS                                         │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│  NUMERIC ( [[ ]] or (( )) )          STRING ( [[ ]] )                               │
│  ─────────────────────────           ────────────────                               │
│  -eq    equal                        =  or ==   equal                               │
│  -ne    not equal                    !=         not equal                           │
│  -lt    less than                    <          less than (lexical)                 │
│  -le    less or equal                >          greater than (lexical)              │
│  -gt    greater than                 -z "$str"  is empty                            │
│  -ge    greater or equal             -n "$str"  is not empty                        │
│                                                                                     │
│  FILE TESTS                                                                         │
│  ──────────                                                                         │
│  -e file    exists                   -w file    writable                            │
│  -f file    regular file             -x file    executable                          │
│  -d file    directory                -s file    size > 0                            │
│  -r file    readable                 -L file    symlink                             │
│                                                                                     │
│  LOGICAL                                                                            │
│  ───────                                                                            │
│  &&    AND                           ||    OR                                       │
│  !     NOT                                                                          │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### Examples

```bash
# Numeric
[[ $age -ge 18 ]] && echo "Adult"
(( age >= 18 )) && echo "Adult"

# String
if [[ "$name" == "John" ]]; then
    echo "Hello John"
fi

# Empty check
if [[ -z "$var" ]]; then
    echo "Variable is empty"
fi

# File tests
if [[ -f "/etc/passwd" ]]; then
    echo "File exists"
fi

if [[ -d "$HOME/Downloads" ]]; then
    echo "Directory exists"
fi

# Compound conditions
if [[ -f "$file" && -r "$file" ]]; then
    echo "File exists and is readable"
fi

# Pattern matching (glob)
if [[ "$filename" == *.txt ]]; then
    echo "Text file"
fi

# Regex matching
if [[ "$email" =~ ^[a-z]+@[a-z]+\.[a-z]+$ ]]; then
    echo "Valid email format"
fi
```

### case Statement

```bash
#!/bin/bash

case $1 in
    start)
        echo "Starting service..."
        ;;
    stop)
        echo "Stopping service..."
        ;;
    restart|reload)
        echo "Restarting..."
        ;;
    *)
        echo "Usage: $0 {start|stop|restart}"
        exit 1
        ;;
esac
```

---

## 5) Loops

### for Loop

```bash
# List iteration
for fruit in apple banana cherry; do
    echo "$fruit"
done

# Array iteration
arr=("one" "two" "three")
for item in "${arr[@]}"; do
    echo "$item"
done

# C-style
for ((i=0; i<5; i++)); do
    echo "Index: $i"
done

# Range
for i in {1..10}; do
    echo "$i"
done

# Range with step
for i in {0..20..5}; do  # 0, 5, 10, 15, 20
    echo "$i"
done

# Command output
for file in *.txt; do
    echo "Processing: $file"
done

# Files in directory
for file in /var/log/*; do
    [[ -f "$file" ]] && echo "$file"
done
```

### while Loop

```bash
# Basic while
count=0
while [[ $count -lt 5 ]]; do
    echo "Count: $count"
    ((count++))
done

# Read file line by line
while IFS= read -r line; do
    echo "Line: $line"
done < input.txt

# Read from command
while read -r pid name; do
    echo "PID: $pid, Name: $name"
done < <(ps -eo pid,comm)

# Infinite loop
while true; do
    echo "Running..."
    sleep 1
done
```

### until Loop

```bash
# Until condition is true
count=0
until [[ $count -ge 5 ]]; do
    echo "Count: $count"
    ((count++))
done
```

### Loop Control

```bash
# break - exit loop
for i in {1..10}; do
    [[ $i -eq 5 ]] && break
    echo "$i"
done

# continue - skip to next iteration
for i in {1..10}; do
    [[ $((i % 2)) -eq 0 ]] && continue
    echo "$i"  # Only odd numbers
done
```

---

## 6) Functions

### Basic Functions

```bash
# Definition
greet() {
    echo "Hello, $1!"
}

# Alternative syntax
function greet {
    echo "Hello, $1!"
}

# Call
greet "World"
greet "Bash"
```

### Function Arguments

```bash
calculate() {
    local a=$1
    local b=$2
    local operation=$3
    
    case $operation in
        add) echo $((a + b)) ;;
        sub) echo $((a - b)) ;;
        mul) echo $((a * b)) ;;
        div) echo $((a / b)) ;;
        *) echo "Unknown operation" ;;
    esac
}

result=$(calculate 10 5 add)
echo "Result: $result"  # 15
```

### Return Values

```bash
# Return status (0-255)
is_even() {
    (( $1 % 2 == 0 )) && return 0 || return 1
}

if is_even 4; then
    echo "Even"
fi

# Return value via stdout
get_sum() {
    echo $(($1 + $2))
}

sum=$(get_sum 5 3)

# Return value via global variable
get_data() {
    result="some data"
}

get_data
echo "$result"
```

### Local Variables

```bash
global_var="I'm global"

my_function() {
    local local_var="I'm local"
    global_var="Modified global"
    echo "$local_var"
}

my_function
echo "$global_var"   # Modified global
echo "$local_var"    # Empty (not accessible)
```

---

## 7) Error Handling

### Exit Codes

```bash
# Check last command status
command
if [[ $? -eq 0 ]]; then
    echo "Success"
else
    echo "Failed"
fi

# One-liner
command && echo "Success" || echo "Failed"

# Set exit code
exit 0    # Success
exit 1    # General error
exit 2    # Misuse of command
```

### Set Options

```bash
set -e              # Exit on error
set -u              # Error on undefined variable
set -o pipefail     # Catch pipe errors
set -x              # Debug mode (print commands)

# Combined
set -euo pipefail

# Disable
set +e              # Don't exit on error
```

### Trap

```bash
# Cleanup on exit
cleanup() {
    echo "Cleaning up..."
    rm -f "$temp_file"
}

trap cleanup EXIT

temp_file=$(mktemp)
# Script continues...
# cleanup runs automatically on exit

# Handle specific signals
trap 'echo "Caught SIGINT"' INT
trap 'echo "Caught SIGTERM"' TERM

# Ignore signal
trap '' INT  # Ignore Ctrl+C
```

---

## 8) Useful Patterns

### Check Root User

```bash
if [[ $EUID -ne 0 ]]; then
    echo "This script must be run as root"
    exit 1
fi
```

### Check Arguments

```bash
if [[ $# -lt 2 ]]; then
    echo "Usage: $0 <arg1> <arg2>"
    exit 1
fi
```

### Check Command Exists

```bash
if ! command -v docker &> /dev/null; then
    echo "Docker is not installed"
    exit 1
fi
```

### Logging Function

```bash
log() {
    local level=$1
    shift
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] [$level] $*" | tee -a "$LOG_FILE"
}

log INFO "Starting script"
log ERROR "Something went wrong"
```

### Process File Safely

```bash
# Read file line by line
while IFS= read -r line || [[ -n "$line" ]]; do
    echo "$line"
done < file.txt

# Process files with spaces in names
find . -name "*.txt" -print0 | while IFS= read -r -d '' file; do
    echo "Processing: $file"
done
```

### Parse Options

```bash
#!/bin/bash

verbose=false
output_file=""

while getopts "vo:" opt; do
    case $opt in
        v) verbose=true ;;
        o) output_file="$OPTARG" ;;
        ?) echo "Usage: $0 [-v] [-o output]"; exit 1 ;;
    esac
done

shift $((OPTIND - 1))

echo "Verbose: $verbose"
echo "Output: $output_file"
echo "Remaining args: $@"
```

---

## 9) Complete Example Script

```bash
#!/usr/bin/env bash
#
# backup.sh - Backup directories to compressed archive
#

set -euo pipefail

# Colors
RED='\033[0;31m'
GREEN='\033[0;32m'
NC='\033[0m' # No Color

# Configuration
BACKUP_DIR="/backup"
DATE=$(date +%Y%m%d_%H%M%S)
LOG_FILE="/var/log/backup.log"

# Functions
log() {
    local level=$1
    shift
    local timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    echo "[$timestamp] [$level] $*" >> "$LOG_FILE"
    case $level in
        INFO)  echo -e "${GREEN}[INFO]${NC} $*" ;;
        ERROR) echo -e "${RED}[ERROR]${NC} $*" >&2 ;;
    esac
}

usage() {
    cat << EOF
Usage: $(basename "$0") [OPTIONS] SOURCE_DIR

Backup a directory to a compressed archive.

Options:
    -o DIR     Output directory (default: /backup)
    -k NUM     Keep NUM backups (default: 7)
    -v         Verbose mode
    -h         Show this help

Example:
    $(basename "$0") -o /mnt/backups -k 5 /home/user
EOF
}

cleanup() {
    log INFO "Cleanup complete"
}

# Parse arguments
keep=7
verbose=false

while getopts "o:k:vh" opt; do
    case $opt in
        o) BACKUP_DIR="$OPTARG" ;;
        k) keep="$OPTARG" ;;
        v) verbose=true ;;
        h) usage; exit 0 ;;
        *) usage; exit 1 ;;
    esac
done

shift $((OPTIND - 1))

# Check arguments
if [[ $# -lt 1 ]]; then
    echo "Error: Source directory required"
    usage
    exit 1
fi

source_dir="$1"

# Validation
if [[ ! -d "$source_dir" ]]; then
    log ERROR "Source directory does not exist: $source_dir"
    exit 1
fi

if [[ ! -d "$BACKUP_DIR" ]]; then
    mkdir -p "$BACKUP_DIR"
fi

# Main
trap cleanup EXIT

log INFO "Starting backup of $source_dir"

backup_name="backup_$(basename "$source_dir")_${DATE}.tar.gz"
backup_path="${BACKUP_DIR}/${backup_name}"

# Create backup
if $verbose; then
    tar -czvf "$backup_path" -C "$(dirname "$source_dir")" "$(basename "$source_dir")"
else
    tar -czf "$backup_path" -C "$(dirname "$source_dir")" "$(basename "$source_dir")"
fi

log INFO "Backup created: $backup_path"

# Rotation - keep only $keep recent backups
cd "$BACKUP_DIR"
ls -t backup_*.tar.gz 2>/dev/null | tail -n +$((keep + 1)) | xargs -r rm -f

log INFO "Backup complete"
```

---

## 10) Quick Reference

| Task | Syntax |
|------|--------|
| Variable | `name="value"` |
| Use variable | `$name` or `${name}` |
| Command substitution | `$(command)` |
| Arithmetic | `$((a + b))` or `((a++))` |
| If statement | `if [[ condition ]]; then ... fi` |
| For loop | `for i in items; do ... done` |
| While loop | `while [[ cond ]]; do ... done` |
| Function | `func() { commands; }` |
| Local variable | `local var="value"` |
| Read input | `read -p "Prompt: " var` |
| Array | `arr=("a" "b" "c")` |
| Array element | `${arr[0]}` |
| All array elements | `${arr[@]}` |
| String length | `${#string}` |
| Exit on error | `set -e` |
| Exit code | `$?` |
| Redirect stderr | `2>&1` |

---

## 11) Interview Questions

**Q1: What does `set -e` do?**
> Causes the script to exit immediately if any command returns a non-zero exit status.

**Q2: Difference between `$@` and `$*`?**
> `$@` treats each argument as a separate word (preferred). `$*` treats all arguments as a single string.

**Q3: How do you read a file line by line?**
> `while IFS= read -r line; do echo "$line"; done < file.txt`

**Q4: How do you check if a file exists?**
> `if [[ -f "$file" ]]; then echo "exists"; fi`

**Q5: What's the difference between `[ ]` and `[[ ]]`?**
> `[[ ]]` is a bash keyword with more features (pattern matching, regex, no word splitting). `[ ]` is POSIX compatible but more limited.

---

*Next: [14_Text_Processing.md](14_Text_Processing.md) — grep, sed, awk, and text manipulation.*

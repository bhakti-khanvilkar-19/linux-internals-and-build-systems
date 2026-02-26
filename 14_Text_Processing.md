# Linux Text Processing — grep, sed, awk, and Pipelines

## Objective
Master text processing: search with grep, transform with sed, analyze with awk, and combine tools with pipelines.

---

## 1) grep — Search Text Patterns

### Basic grep

```bash
# Search for pattern in file
grep "error" logfile.txt

# Case insensitive
grep -i "error" logfile.txt

# Show line numbers
grep -n "error" logfile.txt

# Count matches
grep -c "error" logfile.txt

# Show only matching part
grep -o "error" logfile.txt

# Invert match (lines NOT containing)
grep -v "debug" logfile.txt

# Recursive search
grep -r "TODO" /src/
grep -rn "TODO" /src/          # With line numbers

# Include/exclude files
grep -r --include="*.py" "import" /src/
grep -r --exclude="*.log" "error" /var/
grep -r --exclude-dir=".git" "pattern" .
```

### grep with Context

```bash
# Lines before match
grep -B 3 "error" logfile.txt   # 3 lines before

# Lines after match
grep -A 3 "error" logfile.txt   # 3 lines after

# Lines before and after
grep -C 3 "error" logfile.txt   # 3 lines context
```

### Regular Expressions

```bash
# Basic regex (default)
grep "^error" logfile.txt       # Lines starting with "error"
grep "error$" logfile.txt       # Lines ending with "error"
grep "err.r" logfile.txt        # . matches any char
grep "err*" logfile.txt         # * matches 0+ of previous

# Extended regex (egrep or -E)
grep -E "error|warning" logfile.txt     # OR
grep -E "err(or|ors)" logfile.txt       # Groups
grep -E "[0-9]+" logfile.txt            # One or more digits
grep -E "[0-9]{3}" logfile.txt          # Exactly 3 digits
grep -E "^[A-Z]" logfile.txt            # Starts with uppercase

# Perl regex (-P)
grep -P "\d{3}-\d{3}-\d{4}" contacts.txt  # Phone number
grep -P "(?<=@)\w+" emails.txt            # Domain after @
```

### Regex Quick Reference

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                          REGEX PATTERNS                                             │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   Pattern   │ Meaning                                                               │
│   ──────────┼────────────────────────────────────────────────────────               │
│   .         │ Any single character                                                  │
│   ^         │ Start of line                                                         │
│   $         │ End of line                                                           │
│   *         │ Zero or more of previous                                              │
│   +         │ One or more of previous (extended)                                    │
│   ?         │ Zero or one of previous (extended)                                    │
│   {n}       │ Exactly n occurrences                                                 │
│   {n,m}     │ Between n and m occurrences                                           │
│   [abc]     │ Character class (a, b, or c)                                          │
│   [^abc]    │ Not in class                                                          │
│   [a-z]     │ Range                                                                 │
│   \d        │ Digit (Perl)                                                          │
│   \w        │ Word character (Perl)                                                 │
│   \s        │ Whitespace (Perl)                                                     │
│   (...)     │ Group                                                                 │
│   |         │ OR (extended)                                                         │
│   \b        │ Word boundary                                                         │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### Practical grep Examples

```bash
# Find IP addresses
grep -E "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" access.log

# Find email addresses
grep -E "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}" contacts.txt

# Find failed SSH logins
grep "Failed password" /var/log/auth.log

# Find HTTP status codes
grep -E "HTTP/[0-9.]+ [45][0-9]{2}" access.log

# List files containing pattern
grep -l "error" *.log

# Files NOT containing pattern
grep -L "error" *.log
```

---

## 2) sed — Stream Editor

### Basic sed

```bash
# Substitute first occurrence per line
sed 's/old/new/' file.txt

# Substitute all occurrences (global)
sed 's/old/new/g' file.txt

# Case insensitive
sed 's/old/new/gi' file.txt

# Edit file in place
sed -i 's/old/new/g' file.txt

# Backup before editing
sed -i.bak 's/old/new/g' file.txt

# Multiple substitutions
sed -e 's/old1/new1/g' -e 's/old2/new2/g' file.txt
sed 's/old1/new1/g; s/old2/new2/g' file.txt
```

### Address Selection

```bash
# Specific line
sed '5s/old/new/' file.txt          # Line 5 only

# Range of lines
sed '5,10s/old/new/' file.txt       # Lines 5-10

# From line to end
sed '5,$s/old/new/' file.txt

# Lines matching pattern
sed '/error/s/old/new/' file.txt    # Lines containing "error"

# Between patterns
sed '/start/,/end/s/old/new/' file.txt
```

### Delete Lines

```bash
# Delete specific line
sed '5d' file.txt

# Delete range
sed '5,10d' file.txt

# Delete matching lines
sed '/pattern/d' file.txt

# Delete empty lines
sed '/^$/d' file.txt

# Delete lines starting with #
sed '/^#/d' file.txt
```

### Insert/Append

```bash
# Insert before line 5
sed '5i\New line here' file.txt

# Append after line 5
sed '5a\New line here' file.txt

# Append after matching line
sed '/pattern/a\New line' file.txt
```

### Print

```bash
# Print specific lines (with -n to suppress default output)
sed -n '5p' file.txt                # Line 5
sed -n '5,10p' file.txt             # Lines 5-10
sed -n '/pattern/p' file.txt        # Lines matching pattern
```

### Advanced sed

```bash
# Capture groups
echo "hello world" | sed 's/\(hello\) \(world\)/\2 \1/'
# Output: world hello

# Extended regex
sed -E 's/(hello) (world)/\2 \1/' file.txt

# Delete HTML tags
sed 's/<[^>]*>//g' file.html

# Remove leading whitespace
sed 's/^[ \t]*//' file.txt

# Remove trailing whitespace
sed 's/[ \t]*$//' file.txt

# Remove both
sed 's/^[ \t]*//;s/[ \t]*$//' file.txt

# Replace newlines (use tr instead, or)
sed ':a;N;$!ba;s/\n/ /g' file.txt
```

---

## 3) awk — Pattern Scanning and Processing

### Basic awk

```bash
# Print all lines
awk '{print}' file.txt

# Print specific column (space-separated)
awk '{print $1}' file.txt           # First column
awk '{print $2}' file.txt           # Second column
awk '{print $NF}' file.txt          # Last column
awk '{print $1, $3}' file.txt       # Columns 1 and 3

# Custom field separator
awk -F: '{print $1}' /etc/passwd    # Colon-separated
awk -F',' '{print $2}' data.csv     # Comma-separated

# Output field separator
awk -F: 'BEGIN{OFS=","} {print $1,$3}' /etc/passwd
```

### awk Special Variables

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                          AWK SPECIAL VARIABLES                                      │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   Variable  │ Meaning                                                               │
│   ──────────┼────────────────────────────────────────────                           │
│   $0        │ Entire line                                                           │
│   $1,$2,... │ Field 1, 2, etc.                                                      │
│   NF        │ Number of fields in current line                                      │
│   NR        │ Current record (line) number                                          │
│   FNR       │ Record number in current file                                         │
│   FS        │ Field separator (default: whitespace)                                 │
│   OFS       │ Output field separator                                                │
│   RS        │ Record separator (default: newline)                                   │
│   ORS       │ Output record separator                                               │
│   FILENAME  │ Current filename                                                      │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### Pattern Matching

```bash
# Lines containing pattern
awk '/error/' file.txt

# Lines NOT containing pattern
awk '!/error/' file.txt

# Column matches pattern
awk '$3 ~ /error/' file.txt

# Column doesn't match
awk '$3 !~ /error/' file.txt

# Numeric comparison
awk '$3 > 100' data.txt
awk '$3 >= 50 && $3 <= 100' data.txt

# String comparison
awk '$1 == "admin"' users.txt
```

### BEGIN and END Blocks

```bash
# Header and footer
awk 'BEGIN{print "Start"} {print} END{print "Done"}' file.txt

# Calculate sum
awk '{sum += $1} END{print "Total:", sum}' numbers.txt

# Calculate average
awk '{sum += $1; count++} END{print "Avg:", sum/count}' numbers.txt

# Count lines
awk 'END{print NR}' file.txt
```

### awk Programming

```bash
# If-else
awk '{if ($3 > 100) print $1, "High"; else print $1, "Low"}' data.txt

# For loop
awk '{for (i=1; i<=NF; i++) print $i}' file.txt

# Arrays
awk '{count[$1]++} END{for (k in count) print k, count[k]}' file.txt

# Functions
awk '{print toupper($1)}' file.txt
awk '{print tolower($1)}' file.txt
awk '{print length($1)}' file.txt
awk '{print substr($1, 1, 3)}' file.txt   # First 3 chars

# printf for formatting
awk '{printf "%-10s %5d\n", $1, $2}' file.txt
```

### Practical awk Examples

```bash
# Sum column 3
awk '{sum += $3} END{print sum}' data.txt

# Unique values in column 1
awk '!seen[$1]++' file.txt

# Extract fields from /etc/passwd
awk -F: '{print "User:", $1, "Shell:", $7}' /etc/passwd

# Process CSV
awk -F',' '{print $1, $3}' data.csv

# Log analysis - count HTTP status codes
awk '{count[$9]++} END{for (c in count) print c, count[c]}' access.log

# Print line numbers
awk '{print NR, $0}' file.txt

# Skip header line
awk 'NR>1 {print}' file.txt

# Average response time
awk '{sum+=$NF; count++} END{print "Avg:", sum/count}' access.log
```

---

## 4) Other Text Tools

### cut — Extract Columns

```bash
# By character position
cut -c1-5 file.txt                  # Chars 1-5
cut -c1,3,5 file.txt                # Chars 1, 3, 5

# By field
cut -d':' -f1 /etc/passwd           # Field 1, delimiter :
cut -d',' -f1,3 data.csv            # Fields 1 and 3
cut -d':' -f1-3 /etc/passwd         # Fields 1-3
```

### sort — Sort Lines

```bash
# Alphabetical
sort file.txt

# Reverse
sort -r file.txt

# Numeric
sort -n numbers.txt

# By column
sort -k2 file.txt                   # By 2nd column
sort -k2 -n file.txt                # Numeric sort by 2nd column
sort -t',' -k2 data.csv             # CSV by 2nd column

# Unique
sort -u file.txt

# Ignore case
sort -f file.txt
```

### uniq — Remove Duplicates

```bash
# Remove adjacent duplicates (sort first!)
sort file.txt | uniq

# Count occurrences
sort file.txt | uniq -c

# Only duplicates
sort file.txt | uniq -d

# Only unique lines
sort file.txt | uniq -u

# Ignore case
sort file.txt | uniq -i
```

### tr — Translate Characters

```bash
# Replace characters
echo "hello" | tr 'a-z' 'A-Z'       # HELLO
echo "hello" | tr 'aeiou' '12345'   # h2ll4

# Delete characters
echo "hello 123" | tr -d '0-9'      # hello
echo "hello" | tr -d 'aeiou'        # hll

# Squeeze repeated characters
echo "helllo" | tr -s 'l'           # helo

# Replace newlines with spaces
tr '\n' ' ' < file.txt

# Delete non-printable characters
tr -cd '[:print:]' < file.txt
```

### head and tail

```bash
# First lines
head file.txt                       # First 10 lines
head -n 5 file.txt                  # First 5 lines
head -c 100 file.txt                # First 100 bytes

# Last lines
tail file.txt                       # Last 10 lines
tail -n 5 file.txt                  # Last 5 lines
tail -f file.txt                    # Follow (live updates)
tail -F file.txt                    # Follow with retry

# All but first/last
tail -n +2 file.txt                 # Skip first line
head -n -1 file.txt                 # Skip last line
```

### wc — Word Count

```bash
wc file.txt                         # Lines, words, bytes
wc -l file.txt                      # Lines only
wc -w file.txt                      # Words only
wc -c file.txt                      # Bytes
wc -m file.txt                      # Characters
```

### tee — Write to File and stdout

```bash
# Save output while also displaying
command | tee output.txt

# Append instead of overwrite
command | tee -a output.txt

# Write to multiple files
command | tee file1.txt file2.txt
```

---

## 5) Pipelines and Combinations

### Basic Pipelines

```bash
# Chain commands
cat file.txt | grep "error" | wc -l

# Sort and count unique values
cut -d':' -f7 /etc/passwd | sort | uniq -c | sort -rn

# Find top 10 largest files
du -sh * | sort -rh | head -10

# Find most common words
cat file.txt | tr ' ' '\n' | sort | uniq -c | sort -rn | head -10
```

### Process Pipeline

```bash
# Find processes using most CPU
ps aux --sort=-%cpu | head -10

# Find processes by name
ps aux | grep nginx | grep -v grep

# Kill processes by name
ps aux | grep nginx | awk '{print $2}' | xargs kill

# Or use pgrep/pkill
pgrep nginx
pkill nginx
```

### Log Analysis Pipeline

```bash
# Count errors per hour
grep "ERROR" application.log | \
  cut -d' ' -f1,2 | \
  cut -d':' -f1,2 | \
  sort | \
  uniq -c

# Top IP addresses
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -10

# Response time stats
awk '{print $NF}' access.log | sort -n | \
  awk '{a[NR]=$1} END{print "Min:", a[1], "Max:", a[NR], "Median:", a[int(NR/2)]}'
```

### xargs — Build Command Lines

```bash
# Execute command for each line
cat files.txt | xargs rm

# With placeholder
echo "file1 file2 file3" | xargs -I {} cp {} /backup/

# Parallel execution
cat urls.txt | xargs -n 1 -P 4 wget

# Find and process
find . -name "*.txt" -print0 | xargs -0 grep "pattern"
find . -name "*.log" | xargs rm
```

---

## 6) Practical Examples

### Example 1: Log Analysis

```bash
#!/bin/bash
# Analyze Apache access log

LOG="/var/log/apache2/access.log"

echo "=== Top 10 IPs ==="
awk '{print $1}' "$LOG" | sort | uniq -c | sort -rn | head -10

echo ""
echo "=== HTTP Status Codes ==="
awk '{print $9}' "$LOG" | sort | uniq -c | sort -rn

echo ""
echo "=== Most Requested URLs ==="
awk '{print $7}' "$LOG" | sort | uniq -c | sort -rn | head -10
```

### Example 2: CSV Processing

```bash
# data.csv
# name,age,city
# John,25,NYC
# Jane,30,LA

# Extract columns
awk -F',' 'NR>1 {print $1, $3}' data.csv

# Filter by condition
awk -F',' '$2 > 25' data.csv

# Calculate average age
awk -F',' 'NR>1 {sum+=$2; count++} END{print "Avg age:", sum/count}' data.csv

# Convert to JSON-like format
awk -F',' 'NR>1 {printf "{\"name\":\"%s\",\"age\":%s}\n", $1, $2}' data.csv
```

### Example 3: Config File Editing

```bash
# Update config value
sed -i 's/^port=.*/port=8080/' config.txt

# Add line if not exists
grep -q "^option=" config.txt || echo "option=value" >> config.txt

# Comment out line
sed -i 's/^old_setting/#&/' config.txt

# Uncomment line
sed -i 's/^#new_setting/new_setting/' config.txt
```

### Example 4: System Info

```bash
# Users with bash shell
awk -F: '$7 ~ /bash/ {print $1}' /etc/passwd

# Memory usage by process
ps aux | awk 'NR>1 {mem[$11]+=$6} END{for (p in mem) print mem[p], p}' | sort -rn | head -10

# Disk usage summary
df -h | awk 'NR>1 {print $5, $6}' | sort -rn
```

---

## 7) Quick Reference

| Tool | Purpose | Example |
|------|---------|---------|
| `grep` | Search patterns | `grep -i "error" log.txt` |
| `grep -E` | Extended regex | `grep -E "err|warn" log.txt` |
| `grep -r` | Recursive | `grep -r "TODO" src/` |
| `sed 's/a/b/'` | Substitute | `sed 's/old/new/g' file` |
| `sed -i` | In-place edit | `sed -i 's/old/new/g' file` |
| `sed '/p/d'` | Delete lines | `sed '/error/d' file` |
| `awk '{print $1}'` | Print column | `awk '{print $1}' file` |
| `awk -F:` | Set delimiter | `awk -F: '{print $1}' /etc/passwd` |
| `cut -d: -f1` | Extract field | `cut -d: -f1 /etc/passwd` |
| `sort` | Sort lines | `sort -n numbers.txt` |
| `uniq` | Remove duplicates | `sort \| uniq -c` |
| `tr 'a-z' 'A-Z'` | Translate | `echo "hi" \| tr 'a-z' 'A-Z'` |
| `head -n 5` | First N lines | `head -n 5 file` |
| `tail -f` | Follow file | `tail -f /var/log/syslog` |
| `wc -l` | Count lines | `wc -l file` |

---

## 8) Interview Questions

**Q1: Find all lines containing "error" (case-insensitive) in log files?**
> `grep -ri "error" /var/log/`

**Q2: Replace all occurrences of "foo" with "bar" in a file?**
> `sed -i 's/foo/bar/g' file.txt`

**Q3: Print the 3rd column of a space-separated file?**
> `awk '{print $3}' file.txt` or `cut -d' ' -f3 file.txt`

**Q4: Count unique values in a column?**
> `awk '{print $1}' file | sort | uniq -c | sort -rn`

**Q5: Remove blank lines from a file?**
> `sed '/^$/d' file.txt` or `grep -v '^$' file.txt`

---

*Next: [15_System_Admin.md](15_System_Admin.md) — System administration, logs, cron, and security.*

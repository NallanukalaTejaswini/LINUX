# Day 7 — Shell Scripting I
> Automate everything — your first scripts

---

## 1. Shebang, Script Execution & chmod +x

The **shebang** (`#!`) tells the kernel which interpreter to use for the script.

```bash
#!/bin/bash            # use bash explicitly
#!/usr/bin/env bash    # find bash in $PATH (more portable — preferred)
#!/usr/bin/env python3 # python script
#!/bin/sh              # POSIX sh (maximum portability, fewer features)
```

```bash
# Create and run a script
cat > hello.sh << 'EOF'
#!/usr/bin/env bash
echo "Hello, World!"
EOF

# Make it executable
chmod +x hello.sh

# Run it
./hello.sh             # run from current directory
bash hello.sh          # run without needing execute permission
/full/path/hello.sh    # run with full path

# Why ./hello.sh and not just hello.sh?
# The shell only searches $PATH for commands
# ./ explicitly says "in the current directory"
```

---

## 2. Variables & Quoting Rules

```bash
# Assign variables (NO spaces around =)
name="Alice"
age=30
greeting="Hello"

# Access variable value with $
echo $name
echo "Hello, $name"
echo "Age: ${age}"     # braces: clearer, required in some contexts

# Variable naming rules:
# - Letters, digits, underscores only
# - Cannot start with digit
# - Case-sensitive (name ≠ NAME)

# Quoting — the most important scripting concept
name="Alice Smith"

echo $name             # Alice Smith (works here, but dangerous)
echo "$name"           # Alice Smith (CORRECT — always quote)

# Without quotes, the shell splits on whitespace and expands globs
file="my file.txt"
rm $file               # WRONG: tries to rm "my" and "file.txt" separately
rm "$file"             # CORRECT: treats as one argument

# Single quotes: LITERAL — no expansion whatsoever
echo '$name'           # prints: $name (literally)
echo 'It'\''s fine'    # escape single quote inside single-quoted string

# Double quotes: expand variables and command substitution
echo "$name"           # Alice Smith
echo "$(date)"         # today's date
echo "$((2 + 2))"      # arithmetic expansion: 4

# Command substitution
today=$(date +%Y-%m-%d)    # modern syntax (preferred)
today=`date +%Y-%m-%d`     # legacy backtick syntax (avoid)
echo "Today is $today"

# Variable manipulation
echo ${name:-"default"}    # use "default" if name is unset or empty
echo ${name:="default"}    # assign default if unset
echo ${name:?"error msg"}  # error and exit if unset
echo ${#name}              # length of variable
echo ${name^^}             # uppercase (bash 4+)
echo ${name,,}             # lowercase
```

---

## 3. Environment Variables

```bash
# Environment variables are passed to child processes
export NAME="Alice"        # export to environment
printenv                   # show all environment variables
printenv PATH              # show specific variable
env                        # show environment (similar to printenv)

# Important built-in variables
echo $HOME         # /home/alice — home directory
echo $USER         # alice — current username
echo $PATH         # directories searched for commands
echo $SHELL        # /bin/bash — current shell
echo $PWD          # current working directory
echo $OLDPWD       # previous directory (used by cd -)
echo $HOSTNAME     # machine hostname
echo $EDITOR       # default text editor
echo $LANG         # locale (en_US.UTF-8)
echo $TERM         # terminal type
echo $$            # PID of current shell
echo $!            # PID of last background process
echo $?            # exit code of last command

# Modifying PATH
export PATH="$PATH:/opt/myapp/bin"         # add to end
export PATH="/opt/myapp/bin:$PATH"         # add to beginning (higher priority)

# Startup files (loaded in order):
# Login shell:    /etc/profile → ~/.bash_profile → ~/.bashrc
# Non-login:      ~/.bashrc
# All users:      /etc/environment (simple key=value, no export needed)
# /etc/profile.d/ — drop-in scripts loaded by /etc/profile

# Make changes permanent
echo 'export EDITOR=vim' >> ~/.bashrc
source ~/.bashrc        # reload without restarting shell
```

---

## 4. Conditionals

```bash
# if/elif/else syntax
if [ condition ]; then
    commands
elif [ other_condition ]; then
    commands
else
    commands
fi

# [[ ]] — modern bash conditional (preferred over [ ])
# Advantages: no word splitting, supports =~, &&, ||

# File tests
if [[ -f "$file" ]]; then echo "regular file"; fi
if [[ -d "$dir" ]]; then echo "directory"; fi
if [[ -e "$path" ]]; then echo "exists (any type)"; fi
if [[ -r "$file" ]]; then echo "readable"; fi
if [[ -w "$file" ]]; then echo "writable"; fi
if [[ -x "$file" ]]; then echo "executable"; fi
if [[ -s "$file" ]]; then echo "non-empty file"; fi
if [[ -L "$file" ]]; then echo "symbolic link"; fi

# String tests
if [[ -z "$str" ]]; then echo "empty string"; fi
if [[ -n "$str" ]]; then echo "non-empty string"; fi
if [[ "$a" == "$b" ]]; then echo "equal"; fi
if [[ "$a" != "$b" ]]; then echo "not equal"; fi
if [[ "$str" == *"pattern"* ]]; then echo "contains pattern"; fi
if [[ "$str" =~ ^[0-9]+$ ]]; then echo "is all digits"; fi  # regex

# Numeric comparisons (use -eq not ==)
if [[ $num -eq 5 ]]; then echo "equal"; fi
if [[ $num -ne 5 ]]; then echo "not equal"; fi
if [[ $num -gt 5 ]]; then echo "greater than"; fi
if [[ $num -ge 5 ]]; then echo "greater or equal"; fi
if [[ $num -lt 5 ]]; then echo "less than"; fi
if [[ $num -le 5 ]]; then echo "less or equal"; fi

# Combining conditions
if [[ -f "$file" && -r "$file" ]]; then echo "exists and readable"; fi
if [[ $x -gt 0 || $y -gt 0 ]]; then echo "at least one positive"; fi

# Short-circuit operators (common pattern)
mkdir -p /tmp/mydir || { echo "Failed to create dir"; exit 1; }
[[ -f config.txt ]] || { echo "Config missing"; exit 1; }
```

---

## 5. Loops

```bash
# for loop — iterate over a list
for item in one two three; do
    echo "$item"
done

# for loop — over files
for file in /etc/*.conf; do
    echo "Processing: $file"
done

# for loop — C-style
for ((i=0; i<10; i++)); do
    echo "Count: $i"
done

# for loop — over command output
for user in $(awk -F: '$3 >= 1000 {print $1}' /etc/passwd); do
    echo "User: $user"
done

# while loop — test condition before each iteration
count=0
while [[ $count -lt 5 ]]; do
    echo "Count: $count"
    ((count++))
done

# while loop — read file line by line (CORRECT way)
while IFS= read -r line; do
    echo "Line: $line"
done < /etc/passwd

# read from command output
while IFS= read -r line; do
    echo "$line"
done < <(find /var/log -name "*.log")

# until loop — runs until condition is TRUE (opposite of while)
until [[ -f /tmp/ready ]]; do
    echo "Waiting..."
    sleep 1
done

# break and continue
for i in {1..10}; do
    [[ $i -eq 5 ]] && break     # exit loop at 5
    [[ $((i % 2)) -eq 0 ]] && continue   # skip even numbers
    echo "$i"
done
```

---

## 6. Positional Parameters

```bash
#!/usr/bin/env bash
# Script: greet.sh
# Usage: ./greet.sh Alice 30

echo "Script name: $0"    # ./greet.sh
echo "First arg:   $1"    # Alice
echo "Second arg:  $2"    # 30
echo "All args:    $@"    # Alice 30 (each as separate words — USE THIS)
echo "All args:    $*"    # Alice 30 (all as one string — usually avoid)
echo "Arg count:   $#"    # 2
echo "Process ID:  $$"    # current PID
echo "Last exit:   $?"    # exit code of last command

# Validate arguments
if [[ $# -ne 2 ]]; then
    echo "Usage: $0 <name> <age>"
    exit 1
fi

# Shift — removes first argument, shifts all down
echo "First: $1"    # Alice
shift
echo "First: $1"    # 30 (was $2)

# Processing all arguments
for arg in "$@"; do
    echo "Arg: $arg"
done
```

---

## 7. Functions

```bash
# Define a function
greet() {
    local name="$1"         # local: variable only exists inside function
    local greeting="Hello"
    echo "$greeting, $name!"
}

# Call it
greet "Alice"
greet "Bob"

# Functions return exit codes (0-255), not values
# Use echo to "return" a string value
get_date() {
    echo "$(date +%Y-%m-%d)"
}
today=$(get_date)           # capture "return value"

# Returning status
is_root() {
    [[ $EUID -eq 0 ]]       # returns 0 if root, 1 otherwise
}

if is_root; then
    echo "Running as root"
else
    echo "Not root"
fi

# Full practical example
check_service() {
    local service="$1"
    if systemctl is-active --quiet "$service"; then
        echo "$service is running"
        return 0
    else
        echo "$service is NOT running"
        return 1
    fi
}

check_service nginx || sudo systemctl start nginx
```

---

## A Complete Working Script

```bash
#!/usr/bin/env bash
# backup.sh — back up a directory to a timestamped archive
# Usage: ./backup.sh <source_dir> <dest_dir>

# --- Configuration ---
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

# --- Functions ---
usage() {
    echo "Usage: $0 <source_dir> <dest_dir>"
    echo "Example: $0 /home/alice /backup"
    exit 1
}

log() {
    echo "[$(date '+%H:%M:%S')] $*"
}

# --- Argument validation ---
if [[ $# -ne 2 ]]; then
    usage
fi

SOURCE="$1"
DEST="$2"
ARCHIVE="${DEST}/backup_${TIMESTAMP}.tar.gz"

# --- Checks ---
if [[ ! -d "$SOURCE" ]]; then
    log "ERROR: Source directory '$SOURCE' does not exist"
    exit 1
fi

if [[ ! -d "$DEST" ]]; then
    log "Creating destination directory: $DEST"
    mkdir -p "$DEST" || { log "ERROR: Could not create $DEST"; exit 1; }
fi

# --- Main ---
log "Starting backup: $SOURCE → $ARCHIVE"
tar -czf "$ARCHIVE" "$SOURCE"

if [[ $? -eq 0 ]]; then
    log "Backup complete: $(du -sh "$ARCHIVE" | cut -f1)"
else
    log "ERROR: Backup failed!"
    exit 1
fi
```

---

## Interview Prep

**Q: Write a script to check if a file exists and is non-empty.**

```bash
#!/usr/bin/env bash
check_file() {
    local file="$1"
    if [[ -z "$file" ]]; then
        echo "Usage: $0 <filepath>"
        return 1
    fi
    if [[ ! -e "$file" ]]; then
        echo "ERROR: File does not exist: $file"
        return 1
    fi
    if [[ ! -f "$file" ]]; then
        echo "ERROR: Path exists but is not a regular file: $file"
        return 1
    fi
    if [[ ! -s "$file" ]]; then
        echo "WARNING: File exists but is empty: $file"
        return 2
    fi
    echo "OK: File exists and is non-empty: $file ($(wc -l < "$file") lines)"
    return 0
}

check_file "$1"
```

---

## Common Gotcha

**Always quote your variables: `"$var"` not `$var`.**

Unquoted variables undergo **word splitting** and **glob expansion**:

```bash
filename="my report.txt"

# WRONG — splits on space, rm gets two arguments: "my" and "report.txt"
rm $filename

# CORRECT
rm "$filename"

# Glob expansion danger
files="*.txt"
rm $files      # expands *.txt glob — deletes ALL .txt files!
rm "$files"    # tries to delete a file literally named "*.txt"

# The rule: ALWAYS quote variable references unless you specifically
# want word splitting or glob expansion (rare)
```

---

## Practice Exercises

1. Write a script that accepts a username and prints whether they exist on the system.
2. Write a script that loops over all `.log` files in `/var/log` and prints their size.
3. Write a function `is_valid_ip()` that returns 0 if the argument looks like an IPv4 address.
4. Write a script that reads `/etc/passwd` line by line and prints only users with a bash shell.
5. Add argument validation and usage() to any script you wrote above.

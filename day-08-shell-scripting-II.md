# Day 8 — Shell Scripting II
> Advanced scripting patterns

---

## 1. Arrays

```bash
# Indexed arrays
fruits=("apple" "banana" "cherry")
fruits[3]="date"

echo "${fruits[0]}"        # apple (zero-indexed)
echo "${fruits[@]}"        # all elements: apple banana cherry date
echo "${#fruits[@]}"       # length: 4
echo "${!fruits[@]}"       # all indices: 0 1 2 3

# Append
fruits+=("elderberry")

# Slice
echo "${fruits[@]:1:2}"    # elements 1 and 2: banana cherry

# Loop over array
for fruit in "${fruits[@]}"; do
    echo "$fruit"
done

# Loop with index
for i in "${!fruits[@]}"; do
    echo "$i: ${fruits[$i]}"
done

# Array from command output
files=($(ls /etc/*.conf))           # WRONG: breaks on filenames with spaces
mapfile -t files < <(find /etc -name "*.conf")  # CORRECT

# Associative arrays (bash 4+, like hash maps)
declare -A config
config["host"]="localhost"
config["port"]="5432"
config["user"]="dbadmin"

echo "${config[host]}"             # localhost
echo "${!config[@]}"               # all keys: host port user
echo "${config[@]}"                # all values

for key in "${!config[@]}"; do
    echo "$key = ${config[$key]}"
done
```

---

## 2. String Manipulation

Bash has powerful built-in string operations — no need to call external tools for simple cases.

```bash
str="Hello, World!"

# Length
echo "${#str}"                  # 13

# Substring: ${var:offset:length}
echo "${str:0:5}"               # Hello
echo "${str:7}"                 # World!
echo "${str: -6}"               # orld! (negative: from end, note the space)

# Remove prefix (shortest match)
path="/home/alice/docs/file.txt"
echo "${path#/home/}"           # alice/docs/file.txt

# Remove prefix (longest match)
echo "${path##*/}"              # file.txt (everything up to last /)

# Remove suffix (shortest match)
echo "${path%.txt}"             # /home/alice/docs/file

# Remove suffix (longest match)
echo "${path%%/*}"              # (empty — longest prefix ending in /)
filename="${path##*/}"
echo "${filename%.*}"           # file (remove extension)
echo "${filename##*.}"          # txt (get extension)

# Replace
echo "${str/World/Linux}"       # Hello, Linux! (first match)
echo "${str//l/L}"              # HeLLo, WorLd! (all matches)
echo "${str/#Hello/Goodbye}"    # Goodbye, World! (prefix match)
echo "${str/%!/,}"              # Hello, World, (suffix match)

# Case conversion (bash 4+)
echo "${str^^}"                 # HELLO, WORLD!
echo "${str,,}"                 # hello, world!
echo "${str^}"                  # Hello, World! (capitalize first)

# Trim whitespace (no built-in, use parameter expansion)
str="  hello  "
trimmed="${str#"${str%%[![:space:]]*}"}"   # trim leading
trimmed="${trimmed%"${trimmed##*[![:space:]]}"}"  # trim trailing
# Easier alternative:
trimmed=$(echo "$str" | xargs)
```

---

## 3. Arithmetic

```bash
# Arithmetic expansion: $(( ))
echo $((2 + 3))           # 5
echo $((10 / 3))          # 3 (integer division)
echo $((10 % 3))          # 1 (modulo)
echo $((2 ** 8))          # 256 (exponentiation)

# Arithmetic in variables
count=5
((count++))               # increment (C-style)
((count += 10))
echo $count               # 15

# Conditional with arithmetic
if (( count > 10 )); then
    echo "Greater than 10"
fi

# Common pattern: loop with counter
for ((i=1; i<=5; i++)); do
    echo "Item $i"
done

# Floating point — bash only does integers, use bc or awk
echo "scale=2; 10/3" | bc           # 3.33
echo "scale=4; sqrt(2)" | bc -l     # 1.4142
awk 'BEGIN { printf "%.2f\n", 10/3 }'  # 3.33
python3 -c "print(f'{10/3:.4f}')"   # 3.3333
```

---

## 4. Input/Output

```bash
# read — get user input
read -p "Enter your name: " name
echo "Hello, $name"

read -sp "Enter password: " password    # -s: silent (no echo)
echo ""  # newline after silent input

read -t 10 -p "Answer within 10 seconds: " answer  # -t: timeout
read -n 1 -p "Press any key: " key   # -n: read exactly N chars

# read multiple variables
read -p "Enter first last: " first last
echo "First: $first, Last: $last"

# read from file
while IFS= read -r line; do
    echo ">> $line"
done < /etc/hostname

# printf — formatted output (more powerful than echo)
printf "Name: %s, Age: %d\n" "Alice" 30
printf "%-20s %5d\n" "Alice" 30    # left-align name, right-align number
printf "%08.2f\n" 3.14159          # 00003.14
printf "Hex: %x\n" 255             # ff

# Heredoc — multi-line string
cat << 'EOF'
This is a multi-line
string. Variables like $HOME
are NOT expanded (single-quoted delimiter).
EOF

# Without quotes on delimiter — variables ARE expanded
name="Alice"
cat << EOF
Hello, $name!
Today is $(date).
EOF

# Herestring — pass string as stdin
grep "root" <<< "root:x:0:0:root:/root:/bin/bash"

# Assign heredoc to variable
config=$(cat << EOF
server {
    listen 80;
    server_name $HOSTNAME;
}
EOF
)
echo "$config"
```

---

## 5. Error Handling — set -e, set -u, set -o pipefail, trap

This is the most important section for production scripting.

```bash
#!/usr/bin/env bash
set -euo pipefail
# set -e  : exit immediately on error (any command returning non-zero)
# set -u  : treat unset variables as errors
# set -o pipefail : pipeline fails if ANY command in it fails

# Without these safety flags:
cp config.txt /nonexistent/path    # fails silently, script continues!
echo "This runs even after the failed cp"

# With set -e: script exits on the failed cp

# set -u example
echo $UNDEFINED_VAR   # without set -u: empty string, no error
                       # with set -u: error: UNDEFINED_VAR: unbound variable

# pipefail example
cat nonexistent.txt | grep "pattern"
# Without pipefail: exit code is grep's (0 if match found, 1 if no match)
#                   the cat failure is ignored!
# With pipefail: exit code is the rightmost non-zero = cat's failure

# trap — run code when signals or events occur
trap 'echo "Script interrupted!"; exit 1' INT TERM
trap 'cleanup' EXIT     # always run cleanup on exit

cleanup() {
    echo "Cleaning up temporary files..."
    rm -f /tmp/myapp_*.tmp
    echo "Done."
}

# Full production template
#!/usr/bin/env bash
set -euo pipefail

# Trap for cleanup
TMPFILE=$(mktemp)
trap 'rm -f "$TMPFILE"' EXIT

# Script logic uses $TMPFILE
# When script exits (normally or on error), tmpfile is always deleted

# Debugging
set -x    # print each command before executing it (trace mode)
set +x    # turn off trace
bash -x script.sh    # run script with trace (don't modify the script)

# BASH_SOURCE, LINENO for error context
trap 'echo "Error on line $LINENO"' ERR
```

---

## 6. Subshells vs Current Shell

```bash
# Subshell ( ) — runs in a child process
# Changes inside do NOT affect the parent
(
    cd /tmp
    export MYVAR="test"
    echo "Inside subshell: $PWD"
)
echo "Back in parent: $PWD"   # unchanged
echo "$MYVAR"                 # empty — subshell export doesn't reach parent

# Use case: run commands in a different directory without cd back
(cd /etc && tar czf /backup/etc.tar.gz .)

# Group command { } — runs in CURRENT shell
# Changes DO affect the current shell
{
    cd /tmp
    MYVAR="test"
    echo "Inside group: $PWD"
}
echo "After group: $PWD"   # /tmp — changed!
echo "$MYVAR"              # test — set in current shell

# Subshell for variable isolation
result=$(
    cd /tmp
    ls -la | wc -l
)
echo "Files in /tmp: $result"
# Current directory unchanged after command substitution
```

---

## 7. Process Substitution

Process substitution allows you to use the output of a command where a file is expected.

```bash
# <(command) — output of command as a file
diff <(ls /etc) <(ls /usr/etc 2>/dev/null)   # compare directory listings
diff <(sort file1.txt) <(sort file2.txt)      # compare sorted contents

# >(command) — pipe output TO a command as if writing to a file
tee >(gzip > output.gz) < input.txt          # write to file AND compress

# Practical: read two files simultaneously
while IFS= read -r line1 <&3 && IFS= read -r line2 <&4; do
    echo "File1: $line1"
    echo "File2: $line2"
done 3< file1.txt 4< file2.txt

# Using diff without temp files
diff <(grep "ERROR" app.log) <(grep "ERROR" app_backup.log)
```

---

## A Complete Production Script

```bash
#!/usr/bin/env bash
# deploy.sh — deploy application from git to server
set -euo pipefail

readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly APP_DIR="/opt/myapp"
readonly BACKUP_DIR="/opt/backups"
readonly TIMESTAMP=$(date +%Y%m%d_%H%M%S)
readonly LOG_FILE="/var/log/deploy_${TIMESTAMP}.log"

# Logging
log() { echo "[$(date '+%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOG_FILE"; }
die() { log "FATAL: $*"; exit 1; }

# Cleanup on exit
cleanup() {
    local exit_code=$?
    if [[ $exit_code -ne 0 ]]; then
        log "Deploy FAILED with exit code $exit_code"
        log "Check log: $LOG_FILE"
    fi
}
trap cleanup EXIT

# Validation
[[ $EUID -eq 0 ]] || die "Must run as root"
[[ -d "$APP_DIR" ]] || die "App directory not found: $APP_DIR"
command -v git &>/dev/null || die "git is not installed"

# Backup
log "Creating backup..."
mkdir -p "$BACKUP_DIR"
tar -czf "${BACKUP_DIR}/app_${TIMESTAMP}.tar.gz" "$APP_DIR"

# Deploy
log "Pulling latest code..."
cd "$APP_DIR"
git pull origin main

log "Installing dependencies..."
npm ci --production

log "Restarting service..."
systemctl restart myapp

log "Deploy complete."
```

---

## Interview Prep

**Q: What does `set -euo pipefail` do and why is it in every production script?**

- `set -e`: Exit immediately when any command returns a non-zero status. Without it, scripts silently continue after failures, executing subsequent commands against a broken state.
- `set -u`: Treat unset variables as errors. Prevents bugs where a typo in a variable name silently becomes an empty string — which can lead to catastrophic commands like `rm -rf "$UNSET_VAR/"`.
- `set -o pipefail`: A pipeline's exit status is the last non-zero exit code. Without it, `false | true` returns 0 — the failure of `false` is invisible.

Together they transform bash scripts from "optimistic by default" to "fail-fast" — the production-safe approach.

---

## Common Gotcha

**Scripts without `set -e` continue silently after errors, cascading into worse failures.**

```bash
# Without set -e
cd /nonexistent_directory        # silently fails, $PWD unchanged
rm -rf ./*                       # deletes everything in ORIGINAL directory!

# With set -e
set -e
cd /nonexistent_directory        # script exits here
rm -rf ./*                       # never reached
```

A related trap: `set -e` does NOT catch errors inside `if` conditions or after `!`:
```bash
set -e
if grep "pattern" file.txt; then   # grep returning 1 (no match) won't exit
    echo "found"
fi
! command_that_fails               # ! negates the exit code — won't trigger -e
```

---

## Practice Exercises

1. Write a script using an associative array to store server name → IP mappings. Print each pair.
2. Write a function that takes a filename and returns the extension (without the dot).
3. Rewrite a script you wrote in Day 7 to use `set -euo pipefail` and proper `trap` cleanup.
4. Write a script that uses process substitution to compare two sorted lists of installed packages.
5. Write a script using `read` with a timeout to ask the user a yes/no question, defaulting to "no".

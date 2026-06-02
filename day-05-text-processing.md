# Day 5 — Text Processing Power
> The Unix way: compose small tools

---

## 1. grep — Search Text

`grep` searches for patterns in text. It's one of the most-used commands in Linux.

```bash
# Basic usage
grep "error" /var/log/syslog        # lines containing "error"
grep -i "error" /var/log/syslog     # case-insensitive
grep -v "error" /var/log/syslog     # invert: lines NOT containing "error"
grep -n "error" /var/log/syslog     # show line numbers
grep -c "error" /var/log/syslog     # count matching lines

# Searching files
grep -r "TODO" /home/alice/code/    # recursive search in directory
grep -l "password" /etc/*.conf      # only show filenames, not lines
grep -L "password" /etc/*.conf      # files that DON'T match

# Context: lines around the match
grep -A 3 "ERROR" app.log           # 3 lines After match
grep -B 3 "ERROR" app.log           # 3 lines Before match
grep -C 3 "ERROR" app.log           # 3 lines around (Context)

# Extended patterns
grep -E "error|warning|critical" app.log   # multiple patterns (extended regex)
grep -w "root" /etc/passwd                 # whole word match only
grep -x "root:.*" /etc/passwd             # whole line match

# Fixed string (faster, no regex interpretation)
grep -F "192.168.1.1" access.log    # treat pattern as literal string

# Real-world examples
grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -rn
# Count failed SSH attempts per IP address

grep -rn "api_key\|password\|secret" /var/www/ --include="*.php"
# Find hardcoded credentials in PHP files
```

---

## 2. sed — Stream Editor

`sed` processes text line by line and applies transformations. Best for find-and-replace, deletion, and insertion.

```bash
# Substitution: sed 's/pattern/replacement/flags'
sed 's/foo/bar/' file.txt           # replace first occurrence per line (stdout only)
sed 's/foo/bar/g' file.txt          # replace ALL occurrences (g = global)
sed 's/foo/bar/I' file.txt          # case-insensitive
sed 's/foo/bar/2' file.txt          # replace only 2nd occurrence

# Edit in place
sed -i 's/foo/bar/g' file.txt       # edit file directly
sed -i.bak 's/foo/bar/g' file.txt   # edit file, keep backup as file.txt.bak

# Deletion
sed '/^#/d' config.conf             # delete comment lines (starting with #)
sed '/^$/d' file.txt                # delete empty lines
sed '5d' file.txt                   # delete line 5
sed '2,5d' file.txt                 # delete lines 2 through 5
sed '/pattern/d' file.txt           # delete lines matching pattern

# Printing specific lines
sed -n '5p' file.txt                # print only line 5
sed -n '2,5p' file.txt              # print lines 2 to 5
sed -n '/start/,/end/p' file.txt    # print lines between patterns

# Insertion and append
sed '3i\new line here' file.txt     # insert before line 3
sed '3a\new line here' file.txt     # append after line 3
sed '/pattern/a\appended line' file.txt  # append after matching line

# Multiple commands
sed -e 's/foo/bar/g' -e '/^#/d' file.txt

# Real-world examples
# Uncomment a line in a config file
sed -i 's/^#ServerName/ServerName/' /etc/apache2/apache2.conf

# Replace a port number in nginx config
sed -i 's/listen 80;/listen 8080;/g' /etc/nginx/sites-available/default

# Strip trailing whitespace from all lines
sed -i 's/[[:space:]]*$//' file.txt
```

> ⚠️ `sed -i` edits in place with no undo. Always test without `-i` first, or use `-i.bak`.

---

## 3. awk — Pattern Processing Language

`awk` is a full programming language for processing structured text. It processes files line by line, splitting each into fields.

```bash
# Basics: awk 'pattern { action }' file
# Default field separator is whitespace
# $1 = first field, $2 = second, ..., $NF = last field, $0 = whole line
# NR = line number, NF = number of fields

# Print specific fields
awk '{print $1}' file.txt           # print first field of each line
awk '{print $1, $3}' file.txt       # print fields 1 and 3
awk '{print NR, $0}' file.txt       # add line numbers

# Custom field separator
awk -F: '{print $1, $3}' /etc/passwd    # : separator, print username and UID
awk -F, '{print $2}' data.csv           # CSV second column

# Patterns (filter before action)
awk '$3 > 1000' /etc/passwd             # lines where field 3 > 1000 (UID > 1000)
awk '/root/' /etc/passwd                # lines matching regex
awk 'NR==5' file.txt                    # only line 5
awk 'NR>=2 && NR<=5' file.txt          # lines 2 through 5

# BEGIN and END blocks
awk 'BEGIN{print "Start"} {print} END{print "End"}' file.txt

# Math and aggregation
awk '{sum += $1} END {print "Total:", sum}' numbers.txt
awk '{sum += $5} END {print "Average:", sum/NR}' data.txt

# Count occurrences
awk '{count[$1]++} END {for (k in count) print k, count[k]}' access.log

# Real-world examples
# Get all usernames with UID >= 1000 (regular users)
awk -F: '$3 >= 1000 {print $1}' /etc/passwd

# Sum disk usage by user
du -s /home/* | awk '{sum[$2] += $1} END {for (k in sum) print k, sum[k]}'

# Parse Apache access log: count requests per IP
awk '{print $1}' /var/log/apache2/access.log | sort | uniq -c | sort -rn | head -10

# Calculate 99th percentile response time from nginx log
awk '{print $NF}' access.log | sort -n | awk 'BEGIN{c=0} {a[c++]=$1} END{print a[int(c*0.99)]}'
```

---

## 4. cut, sort, uniq, wc, tr, paste

These are the "glue" commands — small and composable.

```bash
# cut — extract columns from text
cut -d: -f1 /etc/passwd             # delimiter=:, field 1 (usernames)
cut -d: -f1,3 /etc/passwd           # fields 1 and 3
cut -c1-10 file.txt                 # characters 1 through 10

# sort
sort file.txt                       # alphabetical
sort -n numbers.txt                 # numeric sort
sort -rn numbers.txt                # reverse numeric
sort -k2 file.txt                   # sort by 2nd field
sort -k2 -t: /etc/passwd            # sort by 2nd field, : delimiter
sort -u file.txt                    # sort and remove duplicates

# uniq — remove/count adjacent duplicates (input must be sorted first!)
sort file.txt | uniq                # remove duplicates
sort file.txt | uniq -c             # count occurrences
sort file.txt | uniq -d             # only show duplicates
sort file.txt | uniq -u             # only show unique (non-duplicate) lines

# wc — word/line/char count
wc -l file.txt                      # line count
wc -w file.txt                      # word count
wc -c file.txt                      # byte count
wc -m file.txt                      # character count (multi-byte aware)
wc file.txt                         # all three: lines words bytes

# tr — translate or delete characters
echo "Hello World" | tr 'a-z' 'A-Z'  # uppercase
echo "Hello World" | tr -d ' '        # delete spaces
echo "a:b:c" | tr ':' '\n'            # replace : with newline
echo "aabbcc" | tr -s 'a'             # squeeze: aabbcc -> abbcc
cat file.txt | tr -cd 'a-zA-Z0-9\n'  # remove non-alphanumeric chars

# paste — merge lines of files
paste file1.txt file2.txt            # side by side with tab delimiter
paste -d, file1.txt file2.txt        # comma delimiter
paste -s file.txt                    # all lines on one line, tab-separated
```

---

## 5. Pipelines, Redirection & xargs

### Redirection
```bash
# stdout redirection
command > file.txt          # overwrite file with stdout
command >> file.txt         # append stdout to file
command 2> error.txt        # redirect stderr only
command 2>/dev/null         # discard stderr
command > output.txt 2>&1   # redirect both stdout and stderr to file
command &> output.txt       # shorthand for above (bash only)

# stdin redirection
command < file.txt          # use file as input
command << EOF              # heredoc: multi-line input
line 1
line 2
EOF
command <<< "string"        # herestring: single string as input

# /dev/null — the black hole (discard output)
rm file.txt 2>/dev/null     # suppress "no such file" error
command > /dev/null 2>&1    # suppress all output
```

### Pipelines
```bash
# | sends stdout of left command to stdin of right command
cat /etc/passwd | grep "alice"
# (better style — avoid unnecessary cat)
grep "alice" /etc/passwd

# Complex pipelines
ps aux | grep nginx | grep -v grep | awk '{print $2}'
# List nginx PIDs

cat /var/log/auth.log | grep "Failed" | awk '{print $11}' | sort | uniq -c | sort -rn | head -10
# Top 10 IPs with failed SSH logins
```

### xargs — Turn Output into Arguments
```bash
# xargs reads stdin and builds command arguments from it
find /tmp -name "*.tmp" | xargs rm          # delete all .tmp files
find . -name "*.py" | xargs wc -l           # count lines in all Python files
echo "file1 file2 file3" | xargs ls -la     # ls multiple files

# -I {} : use placeholder for each item
find . -name "*.log" | xargs -I {} cp {} /backup/

# -P : parallel execution
find /data -name "*.csv" | xargs -P 4 -I {} gzip {}  # compress 4 at a time

# Handle filenames with spaces
find . -name "*.txt" -print0 | xargs -0 rm  # null-separated (safe with spaces)
```

---

## 6. Regular Expressions Fundamentals

```
.       any single character (except newline)
*       zero or more of preceding
+       one or more of preceding (ERE only, use -E in grep)
?       zero or one of preceding (ERE)
^       start of line
$       end of line
[abc]   character class: a, b, or c
[^abc]  negated class: anything except a, b, c
[a-z]   range
\b      word boundary (in some contexts)
\d      digit (in Perl-compatible: grep -P)
\w      word character [a-zA-Z0-9_]
\s      whitespace
{n}     exactly n occurrences (ERE)
{n,m}   between n and m occurrences (ERE)
(abc)   grouping (ERE)
a|b     alternation — a or b (ERE)
```

```bash
# BRE (Basic, default grep) vs ERE (Extended, grep -E or egrep)
grep 'colou\?r' file.txt           # BRE: ? must be escaped
grep -E 'colou?r' file.txt         # ERE: ? unescaped

# Real examples
grep -E '^[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}' file.txt  # IP addresses
grep -E '^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$' emails.txt  # email addresses
grep -E '(ERROR|WARN|FATAL)' app.log   # multiple log levels
```

---

## Interview Prep

**Q: Parse /var/log/auth.log to count failed SSH login attempts per IP.**

```bash
grep "Failed password" /var/log/auth.log \
  | awk '{print $11}' \
  | sort \
  | uniq -c \
  | sort -rn \
  | head -10
```

Walk through each step:
1. `grep` filters only failed password lines
2. `awk '{print $11}'` extracts the 11th field (source IP in OpenSSH log format)
3. `sort` groups identical IPs together (required before uniq)
4. `uniq -c` counts consecutive duplicates
5. `sort -rn` sorts by count, descending
6. `head -10` shows top 10 attackers

---

## Common Gotcha

**`sed -i` without a backup destroys files if the pattern is wrong.**

```bash
# You think you're replacing "production" with "staging"
sed -i 's/production/staging/g' config.yaml

# But your regex was wrong, or you ran it on the wrong file
# The original is GONE — no undo, no trash

# Safe habit: always use -i.bak first
sed -i.bak 's/production/staging/g' config.yaml
# Then compare
diff config.yaml config.yaml.bak
# Then delete backup when satisfied
rm config.yaml.bak
```

Also: **`uniq` only removes adjacent duplicates** — always `sort` before `uniq`:
```bash
echo -e "a\nb\na" | uniq     # NOT deduplicated (a appears twice, not adjacent)
echo -e "a\nb\na" | sort | uniq   # correctly deduplicated
```

---

## Practice Exercises

1. Extract all unique IP addresses from an Apache access log (or create a fake one).
2. Use `sed` to uncomment all lines starting with `#Port` in a config file (test on a copy).
3. Use `awk` to calculate the total and average of a column of numbers.
4. Use `cut` and `sort` to list all unique shells in `/etc/passwd`.
5. Build a pipeline to find the top 5 largest files in `/var/log`.
```bash
find /var/log -type f -printf '%s %p\n' 2>/dev/null | sort -rn | head -5
```

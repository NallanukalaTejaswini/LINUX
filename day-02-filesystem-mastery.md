# Day 2 — File System Mastery
> Everything is a file — master it

---

## 1. File Types in Linux

In Linux, **everything is a file** — disks, terminals, processes, network sockets. The `ls -l` command shows the file type as the first character of the permissions string.

| Symbol | Type | Example |
|---|---|---|
| `-` | Regular file | `/etc/passwd`, `script.sh` |
| `d` | Directory | `/home/alice/` |
| `l` | Symbolic link | `/bin -> /usr/bin` |
| `b` | Block device | `/dev/sda` (hard disk) |
| `c` | Character device | `/dev/tty` (terminal) |
| `p` | Named pipe (FIFO) | Used for IPC |
| `s` | Socket | `/run/mysqld/mysqld.sock` |

```bash
ls -la /dev/ | head -20     # see block and character devices
ls -la /tmp/                # often has pipes and sockets from running apps
file /bin/ls                # identify file type by content, not extension
file /etc/passwd            # ASCII text
file /usr/bin/python3       # ELF 64-bit executable
```

---

## 2. Creating, Copying, Moving, Deleting

```bash
# CREATE files and directories
touch file.txt              # create empty file, or update timestamp if exists
touch file1.txt file2.txt   # create multiple files at once
mkdir mydir                 # create directory
mkdir -p a/b/c              # create nested dirs (no error if exists)

# COPY
cp file.txt backup.txt      # copy file
cp -r mydir/ mydir_backup/  # copy directory recursively
cp -p file.txt dest/        # preserve permissions, timestamps, owner
cp -i file.txt dest.txt     # interactive: prompt before overwrite
cp -u src.txt dest.txt      # copy only if src is newer

# MOVE / RENAME
mv file.txt newname.txt     # rename file
mv file.txt /tmp/           # move to another directory
mv -i src dest              # prompt before overwrite
mv -n src dest              # never overwrite existing file

# DELETE
rm file.txt                 # delete file (no trash, no undo!)
rm -i file.txt              # interactive: prompt first
rm -r mydir/                # delete directory recursively
rm -rf mydir/               # force delete, no prompts (DANGEROUS)
rmdir emptydir/             # delete empty directory only
```

> ⚠️ **There is no Recycle Bin.** `rm` is permanent. Always double-check.

```bash
# Safe deletion habit: preview first with ls
ls -la /tmp/oldlogs/
rm -rf /tmp/oldlogs/        # only after confirming what's there
```

---

## 3. Viewing File Contents

```bash
# cat — concatenate and print (best for small files)
cat file.txt
cat -n file.txt             # with line numbers
cat file1.txt file2.txt     # concatenate two files

# less — pager for large files (most useful)
less /var/log/syslog        # navigate with arrows, /search, q to quit
less +F /var/log/syslog     # follow mode (like tail -f)

# more — older, simpler pager (space=next page, q=quit)
more /etc/services

# head — first N lines
head file.txt               # first 10 lines (default)
head -n 20 file.txt         # first 20 lines
head -c 100 file.txt        # first 100 bytes

# tail — last N lines
tail file.txt               # last 10 lines
tail -n 50 file.txt         # last 50 lines
tail -f /var/log/syslog     # follow: live updates as file grows
tail -F /var/log/app.log    # follow by name (handles log rotation)

# tac — reverse of cat (last line first)
tac file.txt

# wc — word/line/char count
wc -l file.txt              # count lines
wc -w file.txt              # count words
wc -c file.txt              # count bytes
```

---

## 4. Wildcards & Globbing

Globbing is **shell expansion** — the shell expands patterns before passing them to commands.

```bash
*           # matches any string (including empty)
?           # matches exactly one character
[abc]       # matches one of: a, b, or c
[a-z]       # matches any lowercase letter
[^abc]      # matches any character NOT a, b, or c
{a,b,c}     # brace expansion — not a glob, but shell expansion
```

```bash
# Examples
ls *.txt                    # all .txt files
ls file?.txt                # file1.txt, fileA.txt, etc.
ls file[0-9].txt            # file0.txt through file9.txt
ls [A-Z]*.log               # files starting with uppercase, ending in .log
rm *.tmp                    # delete all .tmp files in current dir

# Brace expansion (not globbing, but very useful)
mkdir -p project/{src,tests,docs,build}
touch backup_{mon,tue,wed,thu,fri}.tar.gz
echo {1..5}                 # 1 2 3 4 5
echo {a..e}                 # a b c d e

# Combine them
cp /var/log/*.{log,gz} /backup/
```

---

## 5. Hard Links vs Symbolic Links

This is one of the most important and tested concepts in Linux.

### Hard Links
A hard link is **another directory entry pointing to the same inode** (the actual data on disk). Both filenames are equally "real" — neither is the original.

```bash
ln original.txt hardlink.txt

# Verify: same inode number
ls -li original.txt hardlink.txt
# 1234567 -rw-r--r-- 2 user user 42 ... original.txt
# 1234567 -rw-r--r-- 2 user user 42 ... hardlink.txt
#                    ^ link count = 2
```

**Hard link rules:**
- Cannot span filesystems (must be on same partition)
- Cannot link to directories (prevents loops — with one exception: `.` and `..`)
- Data survives until ALL hard links are deleted (link count = 0)

### Symbolic Links (Symlinks / Soft Links)
A symlink is a **special file that contains a path** to another file. Like a Windows shortcut.

```bash
ln -s /etc/nginx/nginx.conf nginx.conf  # create symlink
ls -la nginx.conf
# lrwxrwxrwx 1 user user 22 ... nginx.conf -> /etc/nginx/nginx.conf

ln -s /usr/local/python3.12 /usr/local/bin/python3   # common pattern
```

**Symlink rules:**
- Can span filesystems
- Can point to directories
- Points to a **path** — if the target is deleted, the symlink breaks ("dangling symlink")
- Permissions on symlink itself are irrelevant; what matters is the target's permissions

```bash
# Check if a symlink is broken
file nginx.conf             # shows: broken symbolic link to ...

# Find all broken symlinks in a directory
find /usr/local -xtype l
```

### Comparison Table

| Feature | Hard Link | Soft Link |
|---|---|---|
| Points to | Inode (data) | Path (filename) |
| Cross-filesystem | ❌ No | ✅ Yes |
| Links to dir | ❌ No | ✅ Yes |
| Survives target deletion | ✅ Yes | ❌ No (dangling) |
| Shows with `ls -l` as | `-` (regular file) | `l` |

---

## 6. Finding Files

```bash
# find — the most powerful file search tool
find /etc -name "*.conf"            # find by name in /etc
find / -name "passwd" 2>/dev/null   # suppress permission errors
find /home -type f -name "*.sh"     # only regular files
find /home -type d                  # only directories
find /tmp -mtime +7                 # modified more than 7 days ago
find /tmp -mtime -1                 # modified in last 24 hours
find /var -size +100M               # files larger than 100MB
find /etc -perm 644                 # files with exact permissions 644
find /home -user alice              # files owned by alice
find /tmp -empty                    # empty files and directories

# Execute a command on each result
find /tmp -name "*.log" -delete     # delete all .log files in /tmp
find . -name "*.py" -exec wc -l {} \;   # count lines in each .py file
find . -name "*.txt" -exec grep -l "error" {} \;  # find .txt files containing "error"

# locate — fast database search (pre-indexed)
locate passwd                       # find files matching "passwd"
sudo updatedb                       # update the locate database

# which — find executable in $PATH
which python3                       # /usr/bin/python3
which -a python                     # show all matches in $PATH

# whereis — find binary, source, and man page
whereis nginx
```

---

## 7. File Metadata

```bash
# stat — detailed file information
stat file.txt
# File: file.txt
# Size: 42        Blocks: 8       IO Block: 4096
# Device: 8,1     Inode: 1234567  Links: 1
# Access: 2024-01-15 10:30:00 (atime)
# Modify: 2024-01-14 09:20:00 (mtime) ← file content changed
# Change: 2024-01-14 09:20:00 (ctime) ← metadata changed (permissions, owner)

# Three timestamps to know:
# atime = last access time
# mtime = last modification (content changed) — most important
# ctime = last change (metadata: permissions, owner changed)

# file — identify file type by content
file /usr/bin/python3       # ELF 64-bit LSB pie executable
file /etc/hosts             # ASCII text
file /dev/sda               # block special

# du — disk usage
du -sh /var/log             # total size of directory, human-readable
du -sh /*                   # size of each top-level directory
du -h --max-depth=1 /home   # one level deep
du -ah /etc | sort -rh | head -20  # largest files in /etc

# df — disk free space (mounted filesystems)
df -h                       # all filesystems, human-readable
df -h /var                  # just the filesystem containing /var
df -i                       # inode usage (a filesystem can fill inodes before disk space!)
```

---

## Interview Prep

**Q: What's the difference between a hard link and a soft link?**

A hard link is another name for the same inode — both names point directly to the file data. Deleting the "original" doesn't affect the hard link because they're equal. Soft links contain a path reference; delete the target and the link breaks. Hard links can't cross filesystems or link directories. In practice, symlinks are used far more often (version-pinning binaries, configuration management), while hard links appear in backup tools like rsync's `--link-dest` for space-efficient incremental backups.

---

## Common Gotcha

**`rm -rf /path/to/dir/` vs `rm -rf /path/to/dir //`**

A stray space before the path, or two slashes, or an unset variable can wipe the wrong thing:

```bash
TARGET="/tmp/myapp"
rm -rf $TARGET/             # fine
rm -rf $TARGET /            # if TARGET is empty = rm -rf / (catastrophic)

# Safe pattern: always quote and check
rm -rf "${TARGET:?variable not set}/"
```

The `:?` operator causes the shell to error out if `TARGET` is unset or empty, preventing disaster.

---

## Practice Exercises

1. Create a directory structure: `project/{src,tests,docs}` with one file in each.
2. Create a hard link and a symlink to the same file. Delete the original and test which survives.
3. Find all files in `/etc` modified in the last 7 days.
4. Use `du -sh` to find the 5 largest directories under `/var`.
5. Use `stat` on a file and identify all three timestamps.

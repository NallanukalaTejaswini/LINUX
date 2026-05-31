# Day 3 — Permissions & Ownership
> Who can do what to what

---

## 1. The User/Group/Other Model (UGO)

Every file and directory in Linux has:
- An **owner** (a user)
- An **owning group** (a group)
- Permissions for three classes: **User (owner)**, **Group**, **Other (everyone else)**

```bash
ls -l /etc/hosts
# -rw-r--r-- 1 root root 224 Jan 10 09:00 /etc/hosts
#  ^^^^^^^^^   ^^^^ ^^^^
#  permissions  user group
#
# Breakdown:
# -          = file type (- = regular file)
# rw-        = owner (root) can read and write
# r--        = group (root) can read only
# r--        = others can read only
```

---

## 2. Read, Write, Execute Explained

The three permissions mean different things for **files** vs **directories**:

| Permission | On a File | On a Directory |
|---|---|---|
| `r` (read) | Read file contents | List directory contents (`ls`) |
| `w` (write) | Modify file contents | Create, delete, rename files inside |
| `x` (execute) | Run as a program | Enter the directory (`cd`) and access contents |

```bash
# The x bit on directories is crucial and often misunderstood
# If a directory has r but not x: you can LIST it, but not CD into it or access files
# If a directory has x but not r: you can access files directly if you know their names, but can't ls

chmod 444 /tmp/testdir      # r--r--r-- : can ls but can't cd in
chmod 111 /tmp/testdir      # --x--x--x : can cd in but can't ls
chmod 644 /tmp/testdir      # rw-r--r-- : can ls and read, but no cd for group/other!
```

---

## 3. chmod — Changing Permissions

Two modes: **symbolic** (human-readable) and **octal** (numeric).

### Symbolic Mode
```bash
chmod u+x script.sh         # add execute for owner
chmod g-w file.txt          # remove write for group
chmod o=r file.txt          # set other to read-only (remove w and x)
chmod a+x script.sh         # add execute for all (a = ugo)
chmod u+x,g-w file.txt      # multiple changes at once
chmod -R 755 mydir/         # recursive (apply to dir and all contents)
```

Symbols: `u`=user/owner, `g`=group, `o`=other, `a`=all  
Operators: `+`=add, `-`=remove, `=`=set exactly

### Octal Mode
Each permission is a bit: r=4, w=2, x=1. Sum them per class.

```
rwx = 4+2+1 = 7
rw- = 4+2+0 = 6
r-x = 4+0+1 = 5
r-- = 4+0+0 = 4
--- = 0+0+0 = 0
```

```bash
chmod 755 script.sh         # rwxr-xr-x : owner full, group/other read+execute
chmod 644 config.txt        # rw-r--r-- : owner read/write, group/other read
chmod 600 private.key       # rw------- : owner only (SSH keys must be 600)
chmod 700 ~/scripts/        # rwx------ : only owner can enter
chmod 777 file.txt          # rwxrwxrwx : everyone can do everything (AVOID)

# Common permission patterns:
# 755 = executable scripts, directories
# 644 = regular files, configs
# 600 = private keys, sensitive files
# 700 = private directories
# 400 = read-only (config files that should never change)
```

---

## 4. chown & chgrp

```bash
# chown — change owner (and optionally group)
chown alice file.txt            # change owner to alice
chown alice:developers file.txt # change owner AND group
chown :developers file.txt      # change group only (same as chgrp)
chown -R alice:alice mydir/     # recursive

# chgrp — change group
chgrp developers project/
chgrp -R webteam /var/www/html/

# You must be root (or sudo) to change ownership
sudo chown www-data:www-data /var/www/html/index.php

# Check ownership
ls -la file.txt
stat file.txt
```

---

## 5. Special Permission Bits: SUID, SGID, Sticky Bit

These are the three most important advanced permission concepts and frequent interview topics.

### SUID (Set User ID) — Octal: 4xxx
When set on an **executable**, it runs with the **file owner's privileges**, not the caller's.

```bash
ls -l /usr/bin/passwd
# -rwsr-xr-x 1 root root ... /usr/bin/passwd
#     ^ s here = SUID set

# Why passwd has SUID:
# Regular users need to change their own password
# /etc/shadow is owned by root and not readable by others
# SUID lets passwd run as root so it can write to /etc/shadow
```

```bash
chmod u+s program           # set SUID
chmod 4755 program          # set SUID with octal (4 = SUID bit)
# The 's' appears in place of 'x' for the owner
# If 'S' (capital) appears: SUID set but no execute bit — this is a misconfiguration!
```

### SGID (Set Group ID) — Octal: 2xxx
On an **executable**: runs with the file's group privileges.
On a **directory**: new files created inside inherit the directory's group (very useful for shared dirs).

```bash
chmod g+s /shared/project/  # set SGID on directory
chmod 2755 /shared/project/

ls -l /shared/project/
# drwxr-sr-x ... root developers /shared/project/
#        ^ s = SGID set on directory
# All new files created here will belong to 'developers' group
```

### Sticky Bit — Octal: 1xxx
On a **directory**: users can only delete files they **own**, even if they have write permission on the directory.

```bash
ls -ld /tmp
# drwxrwxrwt 1 root root ... /tmp
#           ^ t = sticky bit set

chmod +t /shared/uploads/   # set sticky bit
chmod 1777 /shared/uploads/ # sticky + everyone can write

# Without sticky: anyone with write permission on the dir can delete anyone's files
# With sticky: you can only delete YOUR OWN files
```

### Summary Table

| Bit | On File | On Directory | Octal |
|---|---|---|---|
| SUID | Runs as file's owner | (no effect) | 4 |
| SGID | Runs as file's group | New files inherit group | 2 |
| Sticky | (no effect) | Users can only delete own files | 1 |

---

## 6. umask — Default Permission Mask

`umask` defines which permissions are **removed** by default when new files/directories are created.

```bash
umask               # show current umask (e.g., 0022)
umask 0027          # set new umask (this session only)
```

**How umask works:**

```
Default for new file:       666 (rw-rw-rw-)
Default for new directory:  777 (rwxrwxrwx)

umask 022:
  File:       666 - 022 = 644 (rw-r--r--)
  Directory:  777 - 022 = 755 (rwxr-xr-x)

umask 027:
  File:       666 - 027 = 640 (rw-r-----)
  Directory:  777 - 027 = 750 (rwxr-x---)
```

(It's a bitwise AND with the complement, but subtraction works for common values.)

```bash
# Common umask values:
# 022 = default for most systems (group and other can read, not write)
# 027 = more restrictive (group can read, others get nothing)
# 077 = most restrictive (owner only — good for servers handling secrets)

# Make umask permanent: add to ~/.bashrc or /etc/profile
echo 'umask 027' >> ~/.bashrc
```

---

## 7. ACLs — Extended Access Control Lists

Standard UGO only allows one owner and one group. ACLs allow **per-user and per-group permissions** on any file.

```bash
# Install if needed
sudo apt install acl

# View ACL on a file
getfacl file.txt
# # file: file.txt
# # owner: alice
# # group: staff
# user::rw-
# group::r--
# other::r--

# Grant bob read+write, even though he's not the owner or in the group
setfacl -m u:bob:rw file.txt

# Grant the 'interns' group read-only
setfacl -m g:interns:r file.txt

# Remove ACL for bob
setfacl -x u:bob file.txt

# Remove all ACLs
setfacl -b file.txt

# Recursive ACL on directory
setfacl -R -m u:bob:rw /var/www/html/

# Default ACL (applied to new files created inside a directory)
setfacl -d -m u:bob:rw /shared/
```

When a file has ACLs, `ls -l` shows a `+` at the end of the permissions:
```
-rw-rw-r--+ 1 alice staff 42 ... file.txt
           ^
```

---

## Interview Prep

**Q: What does `chmod 4755` mean? When would you use SUID?**

`4755` breaks down as:
- `4` = SUID bit
- `7` = owner: rwx (full)
- `5` = group: r-x (read + execute)
- `5` = other: r-x (read + execute)

So it's `rwsr-xr-x`. Anyone can execute this program, and it will run with the **file owner's** effective UID.

Use SUID when an unprivileged user needs to perform a privileged operation on their own behalf — the classic example being `/usr/bin/passwd` which must modify `/etc/shadow` (root-owned) on behalf of a regular user.

Security implication: SUID programs are a security risk if they have vulnerabilities. An attacker exploiting a SUID binary gains the owner's (often root's) privileges. Always audit SUID files: `find / -perm -4000 -type f 2>/dev/null`

---

## Common Gotcha

**`chmod 777` is almost never the right answer.** It's the lazy fix that introduces serious security vulnerabilities.

- On a web server: 777 on files allows the web process to overwrite them — one code injection = full compromise.
- On shared systems: 777 lets any user read, modify, or delete the file.

The correct approach is to identify exactly what permission is needed and grant only that. Common real fixes:
```bash
# Problem: web server can't write to uploads folder
# Wrong fix:
chmod 777 /var/www/html/uploads/

# Right fix: give the web server user (www-data) ownership
sudo chown www-data:www-data /var/www/html/uploads/
sudo chmod 755 /var/www/html/uploads/
```

---

## Practice Exercises

1. Create a file, set it to `chmod 640`. Write out what each class can do.
2. Find all SUID files on your system: `find / -perm -4000 -type f 2>/dev/null`
3. Create a shared directory, set the SGID bit, create two files inside as different users. Check their group ownership.
4. Change your umask to `077`, create a file, check its permissions. Then set it back to `022`.
5. Use `setfacl` to grant a specific user access to a file without changing its owner or group.

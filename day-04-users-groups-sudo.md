# Day 4 — Users, Groups & Sudo
> Identity and privilege in Linux

---

## 1. User Management

Every user has a UID (User ID). The root user always has UID 0. System users (daemons) typically have UIDs 1–999. Regular users start at UID 1000+.

```bash
# Create a user
sudo useradd alice                          # basic creation (no home dir on some distros)
sudo useradd -m alice                       # create with home directory
sudo useradd -m -s /bin/bash alice          # with home dir and specific shell
sudo useradd -m -s /bin/bash -G sudo,docker alice  # with groups

# Higher-level alternative (Debian/Ubuntu — interactive)
sudo adduser alice                          # friendlier, prompts for password etc.

# Set/change password
sudo passwd alice                           # set password for alice
passwd                                      # change YOUR OWN password

# Modify a user
sudo usermod -s /bin/zsh alice             # change shell
sudo usermod -aG docker alice              # add to docker group (-a = append, REQUIRED)
sudo usermod -l newname alice              # rename user
sudo usermod -d /new/home -m alice         # move home directory
sudo usermod -L alice                      # lock account (disable login)
sudo usermod -U alice                      # unlock account

# Delete a user
sudo userdel alice                         # delete user only (keeps home dir)
sudo userdel -r alice                      # delete user AND home directory

# Check who you are
whoami                                     # current username
id                                         # uid, gid, and all groups
id alice                                   # info about another user
```

> ⚠️ **`usermod -aG` — always use `-a` (append).** Without `-a`, the user is REMOVED from all current groups and added only to the specified group.

---

## 2. Group Management

```bash
# Create a group
sudo groupadd developers
sudo groupadd -g 1500 devops              # with specific GID

# Add/remove users from group
sudo usermod -aG developers alice         # add alice to developers
sudo gpasswd -d alice developers          # remove alice from developers
sudo gpasswd -a bob developers            # another way to add

# Modify a group
sudo groupmod -n engineering developers  # rename group
sudo groupmod -g 1600 developers         # change GID

# Delete a group
sudo groupdel developers

# List groups
groups                                    # groups current user belongs to
groups alice                              # groups alice belongs to
getent group developers                   # info about the developers group
getent group | sort                       # all groups on system

# Switch to a group (use group permissions without logging out)
newgrp docker                            # start shell with docker as primary group
```

---

## 3. /etc/passwd, /etc/shadow, /etc/group

These are the three most important identity files in Linux. Know them cold.

### /etc/passwd — User Account Information

World-readable. Contains one line per user, 7 colon-separated fields:

```
username:x:UID:GID:comment:home:shell

root:x:0:0:root:/root:/bin/bash
alice:x:1001:1001:Alice Smith:/home/alice:/bin/bash
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
```

- Field 2 is `x` — means password is in `/etc/shadow`
- `nologin` shell: system accounts that should never get an interactive login
- `false` shell: similar to nologin, explicitly prevents login

```bash
cat /etc/passwd
getent passwd alice         # query user database (works with LDAP too)
```

### /etc/shadow — Password Hashes

Readable only by root. Contains password hashes and aging info:

```
username:$hash:lastchange:min:max:warn:inactive:expire:reserved

alice:$6$rounds=5000$salt$hashhash...:19358:0:99999:7:::
```

- `$6$` = SHA-512 hash algorithm (`$1$`=MD5, `$5$`=SHA-256, `$y$`=yescrypt)
- Fields: min days between changes, max days before required change, warning days, etc.
- `!` or `*` at start of hash = account locked

```bash
sudo cat /etc/shadow
sudo chage -l alice         # view password aging info
sudo chage -M 90 alice      # set max password age to 90 days
sudo chage -E 2025-12-31 alice  # set account expiry
```

### /etc/group — Group Information

```
groupname:x:GID:members

sudo:x:27:alice,bob
docker:x:999:alice
developers:x:1500:alice,carol,dave
```

```bash
cat /etc/group
getent group sudo
```

---

## 4. sudo vs su

### su — Switch User
```bash
su alice                    # switch to alice (requires alice's password)
su -                        # switch to root (full login shell, loads root's env)
su - alice                  # switch to alice with her full environment
su -c "command" alice       # run one command as alice
```

The `-` (or `-l` / `--login`) is important: without it you switch user but keep the current environment variables, PATH, and working directory — this causes subtle breakage.

### sudo — Execute as Another User
```bash
sudo command                # run command as root
sudo -u alice command       # run command as alice
sudo -i                     # interactive root shell (full login)
sudo -s                     # root shell (keeps current env)
sudo !!                     # re-run previous command as root
sudo -l                     # list what sudo commands you're allowed to run

# sudo caches credentials for 15 minutes (configurable)
sudo -k                     # invalidate cached credentials immediately
```

### Key Differences

| | `su` | `sudo` |
|---|---|---|
| Requires | Target user's password | YOUR password |
| Logs commands | No | Yes (to /var/log/auth.log) |
| Granular control | No (all or nothing) | Yes (per-command rules) |
| Best for | Quick local admin work | Production systems, audit trail |

---

## 5. sudoers File & visudo

The `/etc/sudoers` file controls who can run what with sudo.

```bash
# ALWAYS edit with visudo — it validates syntax before saving
sudo visudo

# Syntax errors in sudoers can lock you out of root access!
# visudo prevents this by checking the file before writing it
```

### sudoers Syntax

```
# Format: WHO  WHERE=(AS_WHOM)  WHAT
# User privilege specification
root    ALL=(ALL:ALL) ALL

# Allow alice to run all commands as root from any host
alice   ALL=(ALL) ALL

# Allow alice to run all commands with no password
alice   ALL=(ALL) NOPASSWD: ALL

# Allow bob to only restart nginx
bob     ALL=(root) /usr/bin/systemctl restart nginx

# Allow the 'developers' group to run specific commands
%developers ALL=(root) /usr/bin/systemctl, /usr/bin/apt

# Allow carol to run apt with no password, but everything else needs password
carol   ALL=(ALL) NOPASSWD: /usr/bin/apt, PASSWD: ALL
```

```bash
# Best practice: use drop-in files instead of editing /etc/sudoers directly
sudo visudo -f /etc/sudoers.d/alice

# Contents of /etc/sudoers.d/alice:
alice ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart myapp
```

### Common sudo Patterns

```bash
# Check your sudo privileges
sudo -l

# Typical output:
# User alice may run the following commands on server1:
#     (ALL : ALL) ALL
#     (root) NOPASSWD: /usr/bin/systemctl restart nginx
```

---

## 6. Principle of Least Privilege

Every process and user should have only the minimum permissions needed to perform their function.

```bash
# Bad: giving a deploy script full root
%deploy ALL=(ALL) NOPASSWD: ALL

# Good: only what the deployment actually needs
%deploy ALL=(root) NOPASSWD: /usr/bin/systemctl restart myapp, \
                             /usr/bin/cp /opt/releases/* /opt/myapp/
```

Practical steps:
1. Prefer dedicated service users over running apps as root
2. Use sudo rules scoped to specific commands
3. Review sudo rules periodically: `sudo -l -U alice`
4. Use groups to manage access, not individual users

---

## 7. PAM — Pluggable Authentication Modules

PAM sits between authentication requests (login, sudo, SSH) and the actual credential check. It lets you plug in different auth mechanisms without changing applications.

```bash
ls /etc/pam.d/              # per-service PAM config
cat /etc/pam.d/sudo         # PAM config for sudo
cat /etc/pam.d/sshd         # PAM config for SSH
```

PAM config format — four rule types:
- `auth` — verify identity (password, MFA, smart card)
- `account` — account restrictions (expiry, time-of-day)
- `session` — setup/teardown (mount home, set limits)
- `password` — password update rules

Control flags: `required` (must pass, continues), `requisite` (must pass, stops on fail), `sufficient` (if passes, no need to continue), `optional`

PAM practical: enforce strong passwords via `pam_pwquality`:
```bash
# /etc/security/pwquality.conf
minlen = 12
dcredit = -1    # at least 1 digit
ucredit = -1    # at least 1 uppercase
lcredit = -1    # at least 1 lowercase
ocredit = -1    # at least 1 special character
```

---

## Interview Prep

**Q: How would you give a user sudo access for a single command only?**

```bash
sudo visudo -f /etc/sudoers.d/bob-restart-nginx
```

Add the line:
```
bob ALL=(root) NOPASSWD: /usr/bin/systemctl restart nginx
```

Key points to mention:
- Use `visudo` (never edit sudoers directly)
- Use drop-in files in `/etc/sudoers.d/` rather than editing the main file
- Specify the full path to the command
- Consider whether `NOPASSWD` is appropriate (it is for automated scripts, less so for humans)
- The command path must be exact — `systemctl` only works if that exact binary path is listed

---

## Common Gotcha

**Never edit `/etc/sudoers` directly with a text editor.** A syntax error silently saves a broken file, and the next time you try `sudo`, it fails with `>>> /etc/sudoers: syntax error near line X <<<`. If you're already root, you can fix it. If you're not, you may be locked out.

`visudo` checks syntax before saving. If you have a typo, it tells you:
```
>>> /etc/sudoers: syntax error near line 28 <<<
What now?
Options are:
  (e)dit sudoers file again
  e(x)it without saving changes to sudoers file
  (Q)uit and save anyway (DANGER!)
```

Always choose `e` to fix, never `Q`.

---

## Practice Exercises

1. Create a user `testuser` with a home directory, bash shell, and set a password.
2. Create a group `testgroup`, add `testuser` to it.
3. Inspect `/etc/passwd` and `/etc/shadow` — identify the hash algorithm used.
4. Use `visudo` to give `testuser` sudo access to only `systemctl status`.
5. Run `sudo -l` and interpret the output.
6. Lock and then unlock `testuser`'s account.

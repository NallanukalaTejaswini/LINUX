# 🐧 Linux Mastery — 15-Day Curriculum

A complete, production-grade Linux learning path from zero to sysadmin.  
Every day includes **concepts**, **working examples**, an **interview question**, and a **production gotcha**.

---

## 📚 How to Use This Curriculum

1. Read one file per day — don't rush
2. **Type every command yourself** — reading is not the same as doing
3. Attempt the practice exercises at the end of each day
4. When stuck, come back and re-read the relevant section
5. By Day 15, build the capstone project

> 💡 Recommended: a Linux VM (Ubuntu 22.04 LTS) or a cheap VPS (~$5/mo on Hetzner, DigitalOcean, or Linode)

---

## 🗺️ Learning Path Overview

```
Phase 1 — Foundation (Days 1–4)
  Learn the layout of Linux, how files work, who owns what, and who can do what.

Phase 2 — Power Tools (Days 5–8)
  Master text processing, process control, and shell scripting.

Phase 3 — System Administration (Days 9–12)
  Manage software, networks, remote access, and services.

Phase 4 — Professional Skills (Days 13–15)
  Storage, security hardening, and production operations.
```

---

## 📂 File Index

### Phase 1 — Foundation

| File | Day | Topics |
|------|-----|--------|
| [day-01-the-linux-universe.md](day-01-the-linux-universe.md) | Day 1 | Linux history, Unix philosophy, distros, kernel vs shell vs terminal, Filesystem Hierarchy Standard (FHS), navigation commands, man pages, terminal shortcuts |
| [day-02-filesystem-mastery.md](day-02-filesystem-mastery.md) | Day 2 | File types, create/copy/move/delete, viewing files, wildcards & globbing, hard links vs symbolic links, find/locate, file metadata |
| [day-03-permissions-and-ownership.md](day-03-permissions-and-ownership.md) | Day 3 | UGO permission model, read/write/execute on files vs dirs, chmod (symbolic & octal), chown & chgrp, SUID/SGID/Sticky bit, umask, ACLs |
| [day-04-users-groups-sudo.md](day-04-users-groups-sudo.md) | Day 4 | useradd/usermod/userdel, group management, /etc/passwd & /etc/shadow anatomy, sudo vs su, visudo & sudoers syntax, principle of least privilege, PAM overview |

### Phase 2 — Power Tools

| File | Day | Topics |
|------|-----|--------|
| [day-05-text-processing.md](day-05-text-processing.md) | Day 5 | grep (patterns, context, flags), sed (substitution, deletion, in-place), awk (fields, patterns, BEGIN/END), cut/sort/uniq/wc/tr/paste, pipelines & redirection, xargs, regular expressions |
| [day-06-processes-and-jobs.md](day-06-processes-and-jobs.md) | Day 6 | Process lifecycle (fork/exec/wait/exit), ps/top/htop, kill/killall/pkill/pgrep, signals (SIGTERM vs SIGKILL), foreground/background/nohup/disown, nice & renice, /proc filesystem |
| [day-07-shell-scripting-I.md](day-07-shell-scripting-I.md) | Day 7 | Shebang & execution, variables & quoting rules, environment variables, conditionals ([[ ]]), loops (for/while/until), positional parameters, functions, complete script example |
| [day-08-shell-scripting-II.md](day-08-shell-scripting-II.md) | Day 8 | Indexed & associative arrays, string manipulation, arithmetic, read/printf/heredoc, set -euo pipefail, trap & cleanup, subshells vs group commands, process substitution, production script template |

### Phase 3 — System Administration

| File | Day | Topics |
|------|-----|--------|
| [day-09-package-management.md](day-09-package-management.md) | Day 9 | apt/apt-get, dpkg, yum/dnf, rpm, snap & flatpak, compiling from source, managing PPAs & custom repos, package pinning |
| [day-10-networking.md](day-10-networking.md) | Day 10 | ip addr/link/route, traceroute, dig/nslookup/host, ping/curl/wget/nc, ss/netstat/lsof, iptables basics, ufw, /etc/hosts & /etc/resolv.conf |
| [day-11-ssh-remote-admin.md](day-11-ssh-remote-admin.md) | Day 11 | ssh-keygen (key types), authorized_keys, known_hosts, ~/.ssh/config, local/remote/dynamic port forwarding, scp & rsync, sshd hardening, tmux |
| [day-12-systemd-services.md](day-12-systemd-services.md) | Day 12 | systemd architecture & unit types, systemctl commands, writing service unit files, drop-in overrides, journalctl filtering, Linux boot process, targets (runlevels), systemd timers |

### Phase 4 — Professional Skills

| File | Day | Topics |
|------|-----|--------|
| [day-13-disk-storage.md](day-13-disk-storage.md) | Day 13 | lsblk/blkid/fdisk/parted, MBR vs GPT, filesystem types (ext4/xfs/btrfs), mkfs/mount/umount, /etc/fstab, LVM (PV/VG/LV), RAID levels, iostat/iotop |
| [day-14-security-hardening.md](day-14-security-hardening.md) | Day 14 | SELinux & AppArmor (MAC), fail2ban, auditd & ausearch, AIDE file integrity, Linux capabilities, namespaces & cgroups, CIS benchmark hardening checklist |
| [day-15-production-skills.md](day-15-production-skills.md) | Day 15 | cron & crontab, syslog & logrotate, vmstat/sar/strace/perf, .env management, Linux as Docker's foundation, Ansible basics, interview scenario walkthroughs, final toolkit summary |

---

## 🎯 What Each Day Includes

Every file is structured identically so you always know what to expect:

| Section | Description |
|---------|-------------|
| **Concepts** | The theory behind the tool or feature |
| **Commands & Examples** | Working, copy-pasteable code with comments |
| **Real-world Usage** | How it's actually used in production |
| **Interview Prep** | A high-frequency interview question with a full answer |
| **Common Gotcha** | The anti-pattern or mistake that trips people up |
| **Practice Exercises** | Hands-on tasks to cement the day's learning |

---

## ⚡ Quick Reference — Commands by Day

```
Day 1   pwd, ls, cd, man, tree, history
Day 2   touch, cp, mv, rm, mkdir, cat, less, head, tail, find, stat, du, df
Day 3   chmod, chown, chgrp, umask, setfacl, getfacl
Day 4   useradd, usermod, userdel, groupadd, passwd, sudo, visudo
Day 5   grep, sed, awk, cut, sort, uniq, wc, tr, xargs
Day 6   ps, top, htop, kill, killall, pkill, pgrep, jobs, fg, bg, nice, renice
Day 7   bash scripting: variables, if/else, for/while, functions
Day 8   bash scripting: arrays, strings, set -euo pipefail, trap
Day 9   apt, dpkg, yum, dnf, rpm, snap, make
Day 10  ip, ss, netstat, dig, curl, wget, nc, ping, traceroute, iptables, ufw
Day 11  ssh, ssh-keygen, ssh-copy-id, scp, rsync, tmux
Day 12  systemctl, journalctl, systemd-analyze
Day 13  lsblk, fdisk, parted, mkfs, mount, lvm (pvcreate/vgcreate/lvcreate)
Day 14  auditctl, ausearch, fail2ban-client, getcap, setcap, aa-status
Day 15  crontab, logrotate, vmstat, iostat, sar, strace, ansible-playbook
```

---

## 🏆 Interview Topics by Day

| Day | Interview Question |
|-----|--------------------|
| 1 | What is the difference between a process and a thread? |
| 2 | What's the difference between a hard link and a soft link? |
| 3 | What does chmod 4755 mean? When would you use SUID? |
| 4 | How would you give a user sudo access for a single command only? |
| 5 | Parse /var/log/auth.log to count failed SSH login attempts per IP |
| 6 | Difference between SIGTERM and SIGKILL? When would you use each? |
| 7 | Write a script to check if a file exists and is non-empty |
| 8 | What does `set -euo pipefail` do and why is it in every production script? |
| 9 | How would you pin a package to prevent it from being upgraded? |
| 10 | A service is not responding. Walk me through your network troubleshooting steps |
| 11 | How do you set up passwordless SSH? What are the security implications? |
| 12 | Write a systemd service that restarts a Python app on failure |
| 13 | How would you extend a filesystem online without unmounting it? |
| 14 | How would you audit which files a suspicious process has opened? |
| 15 | A production server is slow. Systematically diagnose the issue |

---

## ⚠️ Top 5 Gotchas (Read Before You Start)

1. **`rm -rf` has no undo.** Always double-check before running it. Quote your variables: `rm -rf "${DIR:?}/"`.
2. **Never `chmod 777`** anything on a production server. Find the actual permission needed.
3. **Never edit `/etc/sudoers` directly.** Always use `visudo`.
4. **`usermod -G` without `-a` removes all existing groups.** Always use `usermod -aG`.
5. **Test a new SSH session before closing your current one** after changing `sshd_config`. A syntax error = lockout.

---

## 🔧 Recommended Setup

```bash
# Spin up Ubuntu 22.04 LTS (VM, WSL2, or VPS)
# Then install these tools before starting:

sudo apt update
sudo apt install -y \
  tree curl wget vim git \
  net-tools dnsutils \
  htop iotop ncdu \
  tmux screen \
  build-essential \
  auditd fail2ban \
  sysstat
```

---

## 📖 Recommended Books

| Book | Why |
|------|-----|
| *The Linux Command Line* — William Shotts | Best beginner book, free online at linuxcommand.org |
| *How Linux Works* — Brian Ward | Internals explained clearly |
| *Linux Bible* — Christopher Negus | Comprehensive reference |
| *Unix and Linux System Administration Handbook* — Nemeth et al. | The sysadmin bible |

---

## 🚀 After Day 15 — What's Next?

```
Containers     → Docker → Kubernetes
Automation     → Ansible → Terraform
Monitoring     → Prometheus + Grafana
CI/CD          → GitHub Actions, GitLab CI
Cloud          → AWS/GCP/Azure (all run on Linux)
Certifications → RHCSA, LPIC-1, CompTIA Linux+
```

---

*Built as a structured self-paced curriculum. Each file is self-contained — jump to any day as a reference.*

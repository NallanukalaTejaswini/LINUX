# Day 1 — The Linux Universe
> History, philosophy & finding your way around

---

## 1. What is Linux & the Unix Philosophy

Linux is an **open-source, Unix-like operating system kernel** created by Linus Torvalds in 1991. It powers everything from your Android phone to the world's top supercomputers and the majority of web servers.

The **Unix Philosophy** (coined by Ken Thompson and Doug McIlroy) is the design DNA of Linux:

1. Write programs that do **one thing** and do it well.
2. Write programs that **work together** (via text streams).
3. Write programs that handle **text streams**, because that is a universal interface.

This is why `ls`, `grep`, `sort`, `wc` each do one job — and you combine them with pipes.

```bash
# Unix philosophy in action: count unique users logged in
who | awk '{print $1}' | sort | uniq | wc -l
```

---

## 2. Linux Distributions

A **distribution (distro)** = Linux kernel + package manager + default software + configuration.

| Distro Family | Examples | Package Manager | Common Use |
|---|---|---|---|
| Debian | Ubuntu, Kali, Mint | apt / dpkg | Servers, desktops |
| Red Hat | RHEL, CentOS, Fedora, AlmaLinux | yum / dnf / rpm | Enterprise servers |
| Arch | Arch, Manjaro, EndeavourOS | pacman | Power users |
| SUSE | openSUSE, SLES | zypper | Enterprise |

For learning: **Ubuntu** (easiest, most documentation). For production: **RHEL/AlmaLinux** or **Debian**.

---

## 3. Kernel vs Shell vs Terminal vs CLI

These four terms are often confused. They are completely different things:

```
Hardware
  └── Kernel (Linux)         ← talks to hardware, manages resources
        └── Shell (bash/zsh) ← interprets your commands
              └── Terminal   ← the window/app that hosts the shell
                    └── CLI  ← text-based interface paradigm
```

- **Kernel**: The core of the OS. Manages CPU, memory, devices. You never interact with it directly.
- **Shell**: A program that reads your commands and executes them. Common shells: `bash`, `zsh`, `fish`, `sh`.
- **Terminal / Terminal Emulator**: The application you type in (e.g., GNOME Terminal, iTerm2, Alacritty).
- **CLI (Command Line Interface)**: The concept of typing commands, as opposed to a GUI.

```bash
# Find out which shell you're using
echo $SHELL

# Find out the current shell process
ps -p $$
```

---

## 4. Filesystem Hierarchy Standard (FHS)

**Everything in Linux lives under `/` (root).** There are no drive letters like `C:\`. The FHS defines where things go:

```
/
├── bin/      Essential user binaries (ls, cp, mv) — symlink to /usr/bin on modern systems
├── boot/     Bootloader files, kernel images
├── dev/      Device files (disks, terminals, /dev/null, /dev/zero)
├── etc/      System-wide configuration files (text files, human-readable)
├── home/     User home directories (/home/alice, /home/bob)
├── lib/      Shared libraries for /bin and /sbin
├── media/    Mount points for removable media (USB, CD)
├── mnt/      Temporary mount points
├── opt/      Optional/third-party software
├── proc/     Virtual filesystem — kernel and process info (not real files)
├── root/     Home directory for the root user (NOT /home/root)
├── run/      Runtime data (PIDs, sockets) — cleared on reboot
├── sbin/     System binaries (for root: fdisk, iptables)
├── srv/      Data for services (web, FTP)
├── sys/      Virtual filesystem — device and kernel info
├── tmp/      Temporary files (cleared on reboot)
├── usr/      User programs and data (read-only)
│   ├── bin/  Most user commands live here
│   ├── lib/  Libraries
│   └── local/  Locally compiled software
└── var/      Variable data (logs, databases, mail, spools)
    ├── log/
    └── www/
```

Key insight: `/proc` and `/sys` are **virtual filesystems** — they don't live on disk. They expose kernel data as files.

```bash
# See your CPU info via /proc
cat /proc/cpuinfo

# See memory info
cat /proc/meminfo

# See currently running kernel version
cat /proc/version
```

---

## 5. Basic Navigation Commands

```bash
# Where am I?
pwd                         # Print Working Directory

# What's here?
ls                          # list files
ls -l                       # long format (permissions, owner, size, date)
ls -a                       # show hidden files (dotfiles starting with .)
ls -lh                      # human-readable sizes (KB, MB, GB)
ls -lt                      # sort by modification time (newest first)
ls -lR                      # recursive listing

# Move around
cd /etc                     # absolute path (starts from /)
cd documents                # relative path (from current directory)
cd ..                       # go up one level
cd -                        # go back to previous directory
cd ~                        # go to your home directory
cd                          # also goes home (no argument)

# Visual tree view
tree                        # requires: sudo apt install tree
tree -L 2                   # only 2 levels deep
tree -d                     # directories only
```

---

## 6. Getting Help

Linux has a rich built-in help system. Never Google first — check the manual.

```bash
# Man pages (the official manual)
man ls                      # manual for ls
man 5 passwd                # section 5 = file formats (/etc/passwd format)
man -k "copy file"          # search man pages by keyword

# Quick help (built into the command)
ls --help
grep --help

# info pages (GNU tools have more detailed info pages)
info coreutils

# tldr — community-written practical examples (install separately)
# sudo apt install tldr && tldr --update
tldr tar
tldr find

# type — what kind of command is this?
type ls                     # ls is /bin/ls
type cd                     # cd is a shell builtin
type ll                     # ll is aliased to `ls -l`

# which — find the full path of a command
which python3               # /usr/bin/python3
```

Man page **sections**:
1. User commands, 2. System calls, 3. Library functions, 4. Special files, 5. File formats, 6. Games, 7. Miscellaneous, 8. Admin commands

```bash
# Navigate man pages:
# Space / f    = next page
# b            = previous page
# /pattern     = search
# n            = next match
# q            = quit
```

---

## 7. Terminal Shortcuts & History

These shortcuts work in bash and most terminals. They will save hours of your life.

### Cursor Movement
| Shortcut | Action |
|---|---|
| `Ctrl + A` | Move to beginning of line |
| `Ctrl + E` | Move to end of line |
| `Alt + F` | Move forward one word |
| `Alt + B` | Move backward one word |
| `Ctrl + U` | Delete from cursor to beginning of line |
| `Ctrl + K` | Delete from cursor to end of line |
| `Ctrl + W` | Delete word before cursor |
| `Ctrl + L` | Clear screen (like `clear`) |
| `Ctrl + C` | Cancel/interrupt current command |
| `Ctrl + Z` | Suspend current process (send to background) |
| `Ctrl + D` | EOF / logout (closes shell) |

### Command History
```bash
history                     # show command history (last 500 by default)
history 20                  # show last 20 commands
!!                          # re-run last command
!ls                         # re-run last command starting with 'ls'
!42                         # re-run command #42 from history
!$                          # last argument of previous command

# Interactive history search
# Ctrl + R  = reverse search through history (start typing)
# Ctrl + G  = cancel search

# Example workflow:
ls /var/log/nginx/          # run a long path command
cd !$                       # cd /var/log/nginx/ — reuses the last argument
```

### Tab Completion
```bash
# Press Tab once to complete, twice to show options
cd /etc/ssh/         # type 'cd /etc/ss' then press Tab
apt install pyth     # press Tab to see python packages
```

---

## Interview Prep

**Q: What is the difference between a process and a thread?**

A **process** is an independent program in execution with its own memory space, file descriptors, and PID. Processes are isolated — one process cannot directly access another's memory.

A **thread** is a unit of execution *within* a process. Threads share the same memory space and file descriptors of their parent process but have their own stack and registers.

- Multiple threads in one process = shared memory, faster communication, but risk of race conditions.
- Multiple processes = isolated, safer, but higher overhead for inter-process communication (IPC).

In Linux, both processes and threads are implemented as **tasks** and created via `clone()` system call. The difference is which resources are shared.

---

## Common Gotcha

**Don't confuse `/` (root directory) with the `root` user's home directory.**

- `/` = the top of the entire filesystem (every file is under here)
- `/root` = the home directory of the `root` superuser
- `/home/yourname` = YOUR home directory

Running `cd /` takes you to the filesystem root. Running `cd ~` or `cd` takes you to your home directory.

---

## Practice Exercises

1. Open a terminal and run `pwd`. Navigate to `/etc` and list its contents with sizes.
2. Read the manual for `ls`. Find the flag that sorts by file size.
3. Run `cat /proc/cpuinfo | grep "model name"` — what CPU does your system report?
4. Use `Ctrl+R` and search for a command you ran earlier.
5. Run `tree /etc -L 1` to see the top level of `/etc`.

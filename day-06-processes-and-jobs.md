# Day 6 — Processes & Jobs
> Control what's running on your system

---

## 1. Process Lifecycle

Every running program is a **process**. Understanding how processes are created and managed is fundamental to Linux administration.

```
fork() → creates a copy of the parent process
exec() → replaces the process image with a new program
wait() → parent waits for child to finish
exit() → process terminates, returns exit code to parent
```

```bash
# Every process has:
# PID  — Process ID (unique number)
# PPID — Parent Process ID
# UID  — User who owns the process
# GID  — Group of the process

# Process 1 (systemd or init) is the ancestor of ALL processes
# When a parent dies before its child, the child is "adopted" by PID 1

# View process tree
pstree
pstree -p              # with PIDs
pstree -u              # with usernames
pstree alice           # processes owned by alice
```

### Exit Codes
```bash
# Every process exits with a code: 0 = success, non-zero = error
ls /etc > /dev/null
echo $?                # 0 (success)

ls /nonexistent > /dev/null 2>&1
echo $?                # 2 (error)

# Conventional exit codes:
# 0   = success
# 1   = general error
# 2   = misuse of command
# 126 = command found but not executable
# 127 = command not found
# 128+N = killed by signal N (e.g., 137 = killed by SIGKILL=9)
```

---

## 2. Viewing Processes

```bash
# ps — process snapshot
ps                     # processes in current terminal only
ps aux                 # ALL processes, all users, detailed
ps aux | grep nginx    # find specific process
ps -ef                 # full format (PPID visible)
ps -ef --forest        # tree view showing parent-child relationships
ps -u alice            # processes owned by alice
ps -p 1234             # info about specific PID

# Column meanings in ps aux:
# USER   PID  %CPU %MEM    VSZ   RSS TTY STAT START   TIME COMMAND
# root     1   0.0  0.1 168940 11320  ?  Ss  10:00   0:03 /sbin/init

# STAT codes:
# R = Running
# S = Sleeping (interruptible)
# D = Sleeping (uninterruptible — usually I/O wait)
# Z = Zombie (finished but parent hasn't collected exit status)
# T = Stopped (Ctrl+Z)
# s = session leader
# + = foreground process group
# < = high priority
# N = low priority (nice)

# top — live process monitor
top
# Inside top:
# k = kill a process (enter PID)
# r = renice (change priority)
# u = filter by user
# M = sort by memory
# P = sort by CPU
# q = quit
# 1 = show per-CPU stats

# htop — better top (install: sudo apt install htop)
htop
# Mouse-clickable, color-coded, easier to use
# F5 = tree view, F6 = sort, F9 = kill, F10 = quit
```

---

## 3. Signals — kill, killall, pkill, pgrep

Signals are software interrupts sent to processes.

```bash
# kill — send signal to process by PID
kill 1234              # send SIGTERM (default) to PID 1234
kill -9 1234           # send SIGKILL
kill -SIGTERM 1234     # same as kill 1234
kill -15 1234          # same as SIGTERM (15 is SIGTERM's number)
kill -l                # list all signals

# killall — kill by name
killall nginx          # SIGTERM to all processes named nginx
killall -9 firefox     # SIGKILL to all firefox processes
killall -u alice       # kill all processes owned by alice

# pkill — kill by name (regex-capable)
pkill nginx            # similar to killall
pkill -9 -u alice      # kill all alice's processes
pkill -f "python script.py"  # match against full command line (-f flag)

# pgrep — find PIDs by name (don't kill, just find)
pgrep nginx            # print PIDs of nginx processes
pgrep -l nginx         # print PID and name
pgrep -u alice         # PIDs of alice's processes
pgrep -a nginx         # PID and full command line
```

### Key Signals

| Signal | Number | Meaning | Can be caught? |
|---|---|---|---|
| SIGHUP | 1 | Hangup / reload config | Yes |
| SIGINT | 2 | Interrupt (Ctrl+C) | Yes |
| SIGQUIT | 3 | Quit with core dump | Yes |
| SIGKILL | 9 | Kill immediately | **No** |
| SIGTERM | 15 | Graceful termination | Yes |
| SIGSTOP | 19 | Pause process | **No** |
| SIGCONT | 18 | Resume paused process | Yes |
| SIGUSR1 | 10 | User-defined signal 1 | Yes |
| SIGUSR2 | 12 | User-defined signal 2 | Yes |

```bash
# SIGHUP (1) — many daemons reload their config on SIGHUP
sudo kill -HUP $(pgrep nginx)      # reload nginx config without restart
sudo kill -HUP $(pgrep sshd)       # reload SSH daemon

# The correct kill sequence:
kill -SIGTERM $PID      # 1. Ask nicely — process can clean up
sleep 5
kill -SIGKILL $PID      # 2. Force kill if still running (last resort)

# One-liner to check if process is still running after SIGTERM:
kill -TERM $PID && sleep 3 && kill -0 $PID 2>/dev/null && kill -KILL $PID
```

---

## 4. Foreground, Background & Job Control

```bash
# Run a command in the background
long_running_command &         # & sends to background immediately
                               # prints: [1] 12345 (job number and PID)

# View background jobs
jobs                           # list jobs in current shell
jobs -l                        # with PIDs

# Bring job to foreground
fg                             # bring most recent background job to fg
fg %1                          # bring job number 1 to foreground
fg %nginx                      # bring job named nginx to foreground

# Send foreground job to background
# 1. Ctrl+Z  → suspends (pauses) the process
# 2. bg      → resumes it in the background

# Practical example:
vim large_file.txt
# (Ctrl+Z to suspend)
# [1]+  Stopped   vim large_file.txt
grep "something" /var/log/*.log   # do something else
fg                                # return to vim

# nohup — run immune to hangup (survives terminal close)
nohup long_command &             # output goes to nohup.out
nohup ./deploy.sh > deploy.log 2>&1 &

# disown — remove job from shell's job table (can't fg/bg anymore, but keeps running)
long_command &
disown %1                       # remove from job table
disown -h %1                    # mark to ignore SIGHUP (stays running on logout)

# Screen/tmux approach (better for long sessions — see Day 11)
```

---

## 5. Process Priority: nice & renice

Linux schedules CPU time based on **priority**. Nice values range from -20 (highest priority) to +19 (lowest priority). Default is 0.

```bash
# Start a process with a custom nice value
nice -n 10 ./backup.sh          # run at lower priority (nice = 10)
nice -n -5 ./critical.sh        # run at higher priority (requires root for negative)
nice ./script.sh                # default: nice +10

# Change priority of running process
renice 15 -p 1234               # lower priority of PID 1234
renice -5 -p 1234               # raise priority (requires root)
renice 10 -u alice              # lower all of alice's processes
renice 10 -g developers         # lower all processes in group

# View nice values
ps aux                          # NI column shows nice value
top                             # NI column
```

Real-world use: run backup jobs, log processing, or batch work at `nice 19` so they don't impact interactive users or production services.

---

## 6. The /proc Filesystem

`/proc` is a virtual filesystem exposing kernel and process data as files.

```bash
# Per-process info at /proc/PID/
ls /proc/1/                     # PID 1 (init/systemd)
cat /proc/1/status              # process status
cat /proc/1/cmdline             # command line (null-separated)
cat /proc/1/environ             # environment variables
ls -la /proc/1/fd/              # open file descriptors
cat /proc/1/maps                # memory maps

# System-wide info
cat /proc/cpuinfo               # CPU details
cat /proc/meminfo               # memory details
cat /proc/uptime                # system uptime in seconds
cat /proc/loadavg               # load averages (1, 5, 15 min)
cat /proc/version               # kernel version
cat /proc/net/tcp               # TCP connections (raw)
cat /proc/sys/vm/swappiness     # current swappiness value
echo 10 > /proc/sys/vm/swappiness  # change kernel parameter live (root only)
```

---

## Interview Prep

**Q: What is the difference between SIGTERM and SIGKILL? When would you use each?**

**SIGTERM (15)** is a *request* to terminate. The process can:
- Catch the signal and handle it (close files, flush buffers, notify peers)
- Ignore it (bad practice, but possible)
- Use it as a trigger to reload config (some daemons)

**SIGKILL (9)** is an *order* from the kernel to terminate immediately. The process:
- Cannot catch it
- Cannot ignore it
- Gets no chance to clean up — buffers unflushed, temp files left, sockets not closed

**When to use each:**
- Always try SIGTERM first. Give the process a few seconds to clean up.
- Use SIGKILL only when SIGTERM fails and you must free resources immediately.
- SIGKILL is appropriate for a frozen/zombie process, or a misbehaving process that ignores SIGTERM.

Production consequence of SIGKILL: databases can have uncommitted transactions, leaving data corrupted. That's why PostgreSQL, MySQL, etc. have crash recovery — they assume they might be SIGKILL'd.

---

## Common Gotcha

**SIGKILL cannot be caught or ignored — it's always a last resort.**

The reason: if SIGKILL could be ignored, a runaway process could make itself unkillable. The kernel reserves the right to always kill any process.

**Zombie processes** are NOT killed by SIGKILL because they're already dead — they're just waiting for the parent to call `wait()` to collect the exit status. The fix for zombies is to fix or restart the parent:

```bash
# Find zombies
ps aux | awk '$8=="Z"'

# Zombies are harmless (no resources) but indicate a bug in the parent
# Fix: kill the parent, which causes init to adopt and reap the zombie
```

---

## Practice Exercises

1. Run `sleep 300 &` three times. Use `jobs`, `fg`, and `kill` to manage them.
2. Start a process with `nice -n 19 sha256sum /dev/urandom &`. Check its nice value with `ps`.
3. Use `pgrep -a sshd` to find the SSH daemon's PID and inspect its `/proc/PID/` directory.
4. Simulate the graceful kill sequence: start a process, SIGTERM it, check if it died, then SIGKILL.
5. Find any zombie processes on your system: `ps aux | awk '$8=="Z"'`

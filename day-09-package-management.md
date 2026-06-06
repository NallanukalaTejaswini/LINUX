# Day 9 — Package Management & Software
> Installing, updating, removing software the right way

---

## 1. apt / apt-get — Debian & Ubuntu

`apt` is the high-level package manager for Debian-based systems. `apt-get` is the lower-level, more scriptable version.

```bash
# Update package index (always do this first)
sudo apt update                        # refresh package lists from repos
sudo apt upgrade                       # upgrade installed packages
sudo apt full-upgrade                  # upgrade + remove obsolete packages

# Install packages
sudo apt install nginx                 # install single package
sudo apt install nginx curl git vim    # install multiple
sudo apt install -y nginx              # non-interactive (auto yes)
sudo apt install --no-install-recommends nginx  # minimal, no suggestions

# Remove packages
sudo apt remove nginx                  # remove but keep config files
sudo apt purge nginx                   # remove and delete config files
sudo apt autoremove                    # remove unused dependency packages
sudo apt autoclean                     # remove cached package files

# Search and information
apt search nginx                       # search by name/description
apt show nginx                         # detailed info about a package
apt list --installed                   # all installed packages
apt list --installed | grep nginx      # check if installed
apt-cache depends nginx                # list package dependencies
apt-cache rdepends nginx               # what depends ON nginx

# Hold a package (prevent upgrade)
sudo apt-mark hold nginx               # pin nginx to current version
sudo apt-mark unhold nginx             # release hold
apt-mark showhold                      # list held packages

# Fix broken packages
sudo apt install -f                    # fix broken dependencies
sudo dpkg --configure -a               # configure any unconfigured packages
```

### apt vs apt-get

| Use Case | Command |
|---|---|
| Interactive use | `apt` (cleaner output, progress bar) |
| Scripts | `apt-get` (stable output format, won't change) |

---

## 2. dpkg — Low-Level Debian Package Tool

`apt` calls `dpkg` under the hood. Use `dpkg` when working with `.deb` files directly.

```bash
# Install a .deb file
sudo dpkg -i package.deb               # install
sudo apt install -f                    # fix dependencies after dpkg install

# Remove
sudo dpkg -r package-name              # remove (keep config)
sudo dpkg -P package-name              # purge (remove config too)

# Query
dpkg -l                                # list all packages
dpkg -l | grep nginx                   # find specific package
dpkg -s nginx                          # package status/info
dpkg -L nginx                          # list files installed by package
dpkg -S /usr/sbin/nginx                # which package owns this file
dpkg --get-selections                  # export all package selections

# Extract without installing
dpkg-deb -x package.deb ./extract/    # extract package contents
dpkg-deb --info package.deb           # package metadata
```

---

## 3. yum / dnf — RHEL, CentOS, Fedora

`dnf` is the modern replacement for `yum` (same syntax, faster).

```bash
# Update
sudo dnf update                        # update all packages
sudo dnf update nginx                  # update specific package
sudo dnf check-update                  # list available updates

# Install / Remove
sudo dnf install nginx                 # install
sudo dnf remove nginx                  # remove
sudo dnf autoremove                    # remove unused dependencies

# Search and info
dnf search nginx
dnf info nginx
dnf list installed
dnf list installed | grep nginx

# History (powerful — can undo transactions)
dnf history                            # list transaction history
dnf history info 5                     # details of transaction 5
sudo dnf history undo 5                # undo that transaction!

# Hold package
sudo dnf versionlock add nginx         # requires dnf-versionlock plugin
sudo dnf versionlock delete nginx

# Groups (install multiple related packages)
dnf group list
sudo dnf group install "Development Tools"
```

### rpm — Low-Level RPM Tool
```bash
sudo rpm -ivh package.rpm              # install
sudo rpm -e package-name               # erase/remove
rpm -qa                                # query all installed packages
rpm -qi nginx                          # info about installed package
rpm -ql nginx                          # list files
rpm -qf /usr/sbin/nginx               # which package owns this file
rpm -qpR package.rpm                  # dependencies of an .rpm file
```

---

## 4. snap & flatpak — Universal Packages

```bash
# snap — Ubuntu's universal package format
sudo snap install code --classic       # install VS Code
sudo snap list                         # list installed snaps
sudo snap refresh                      # update all snaps
sudo snap remove code                  # remove

# Snaps run in sandboxes with confined permissions
# --classic flag = less sandboxing (for tools needing filesystem access)
snap connections code                  # view snap permissions/connections

# flatpak — cross-distro universal packages
sudo apt install flatpak              # install flatpak
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
flatpak install flathub org.gimp.GIMP
flatpak list
flatpak update
flatpak uninstall org.gimp.GIMP
```

**When to use which:**
- `apt`/`dnf`: system-level software, servers, CLI tools
- `snap`/`flatpak`: desktop apps, isolated environments, newest versions
- Never mix snap and apt versions of the same tool (conflicts)

---

## 5. Compiling from Source

Sometimes you need a version not in repos, or custom compile options.

```bash
# General pattern: download → configure → compile → install

# Example: install a custom version of nginx
cd /usr/local/src

# 1. Download source
wget https://nginx.org/download/nginx-1.24.0.tar.gz
tar -xzf nginx-1.24.0.tar.gz
cd nginx-1.24.0

# 2. Install build dependencies
sudo apt install build-essential libpcre3-dev zlib1g-dev libssl-dev

# 3. Configure (check options, set install paths)
./configure --prefix=/usr/local/nginx \
            --with-http_ssl_module \
            --with-http_v2_module

# 4. Compile (use multiple cores)
make -j$(nproc)

# 5. Install
sudo make install

# The configure script checks for dependencies and generates Makefile
# Common configure options:
./configure --help                     # see all options
./configure --prefix=/usr/local        # installation root
./configure --sysconfdir=/etc          # config directory

# Uninstall from source (if Makefile supports it)
sudo make uninstall
```

---

## 6. Managing PPAs and Custom Repos

### Ubuntu PPAs
```bash
# Add a PPA (Personal Package Archive)
sudo add-apt-repository ppa:ondrej/php
sudo apt update
sudo apt install php8.2

# Remove a PPA
sudo add-apt-repository --remove ppa:ondrej/php

# List all repos
ls /etc/apt/sources.list.d/
cat /etc/apt/sources.list
```

### Custom Repos (Debian/Ubuntu)
```bash
# Modern method: signed repos in /etc/apt/sources.list.d/
# Example: add Docker repo
curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list

sudo apt update
sudo apt install docker-ce
```

### Custom Repos (RHEL/CentOS)
```bash
# Add a .repo file
sudo tee /etc/yum.repos.d/nginx.repo << 'EOF'
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
EOF

sudo dnf install nginx
```

---

## Interview Prep

**Q: How would you pin a package to prevent it from being upgraded?**

**Debian/Ubuntu:**
```bash
sudo apt-mark hold nginx
# Verify
apt-mark showhold
# Release
sudo apt-mark unhold nginx
```

Or with dpkg pinning — create `/etc/apt/preferences.d/nginx`:
```
Package: nginx
Pin: version 1.18.*
Pin-Priority: 1001
```

**RHEL/CentOS:**
```bash
sudo dnf install 'dnf-command(versionlock)'
sudo dnf versionlock add nginx
```

**Why you'd pin:** Production systems where an upstream upgrade could break the application. Example: pinning a database client to match the server version, or pinning nginx while a module compatibility issue is being resolved.

---

## Common Gotcha

**Never run `apt upgrade` blindly on a production server.**

```bash
# DANGEROUS on production
sudo apt upgrade   # upgrades everything — might restart services, change configs

# Safer approach:
sudo apt update
apt list --upgradable          # REVIEW what will change
apt-cache show nginx | grep Version   # check what version you'd get

# Upgrade only specific package
sudo apt install --only-upgrade nginx

# On critical systems: use unattended-upgrades for security patches only
sudo apt install unattended-upgrades
sudo dpkg-reconfigure unattended-upgrades
```

Also: after installing packages, check what services started automatically:
```bash
sudo systemctl status nginx   # might be running and exposed to the internet
sudo systemctl disable nginx  # disable autostart if not ready
```

---

## Practice Exercises

1. Install `htop`, `tree`, and `ncdu` using your distro's package manager.
2. Use `dpkg -L` or `rpm -ql` to list all files installed by a package you know.
3. Use `dpkg -S` or `rpm -qf` to find which package owns `/usr/bin/python3`.
4. Hold `vim` at its current version, verify the hold, then unhold it.
5. Download a `.deb` package with `apt-get download nginx` and inspect it with `dpkg-deb --info`.

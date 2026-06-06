# Day 10 — Networking Fundamentals
> Connect, inspect, troubleshoot

---

## 1. Network Interfaces

```bash
# ip — the modern, canonical networking command
ip addr                          # show all interfaces and IP addresses
ip addr show eth0                # specific interface
ip link                          # show interfaces (link layer)
ip link show eth0
ip -s link                       # with statistics (bytes/packets)

# Interface management
sudo ip link set eth0 up         # bring interface up
sudo ip link set eth0 down       # bring interface down
sudo ip addr add 192.168.1.10/24 dev eth0   # add IP address
sudo ip addr del 192.168.1.10/24 dev eth0   # remove IP

# ifconfig — legacy (still common, install net-tools if needed)
ifconfig                         # all interfaces
ifconfig eth0                    # specific interface
sudo ifconfig eth0 192.168.1.10 netmask 255.255.255.0  # set IP

# Understanding output
ip addr show eth0
# 2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 ...
#     link/ether 00:11:22:33:44:55 brd ff:ff:ff:ff:ff:ff   ← MAC address
#     inet 192.168.1.100/24 brd 192.168.1.255               ← IPv4
#     inet6 fe80::211:22ff:fe33:4455/64                      ← IPv6 link-local

# Interface naming:
# eth0, eth1   = traditional Ethernet
# ens3, ens18  = systemd predictable names (PCI slot based)
# enp2s0       = systemd (bus/slot based)
# wlan0, wlp2s0 = Wi-Fi
# lo           = loopback (127.0.0.1 — always up)
```

---

## 2. Routing

```bash
# Show routing table
ip route                         # show routes
ip route show table all          # all routing tables

# Output example:
# default via 192.168.1.1 dev eth0 proto dhcp    ← default gateway
# 192.168.1.0/24 dev eth0 proto kernel           ← directly connected network

# Add/delete routes
sudo ip route add 10.0.0.0/8 via 192.168.1.1      # add static route
sudo ip route del 10.0.0.0/8                        # remove route
sudo ip route add default via 192.168.1.1           # set default gateway

# Traceroute — see every hop to destination
traceroute google.com            # UDP by default
traceroute -T google.com         # TCP (bypass firewalls that block UDP)
traceroute -I google.com         # ICMP
tracepath google.com             # similar, no root needed

# MTU path discovery
tracepath -n google.com          # discover MTU along path
```

---

## 3. DNS Tools

```bash
# dig — the definitive DNS query tool
dig google.com                   # A record (IPv4 address)
dig google.com A                 # explicit type
dig google.com AAAA              # IPv6 address
dig google.com MX                # mail exchange records
dig google.com NS                # nameservers
dig google.com TXT               # text records (SPF, DKIM, etc.)
dig google.com ANY               # all records (often limited now)

# Reverse DNS lookup
dig -x 8.8.8.8                  # what hostname is 8.8.8.8?

# Specific DNS server
dig @8.8.8.8 google.com         # query Google's DNS
dig @1.1.1.1 google.com         # query Cloudflare's DNS

# Short output
dig +short google.com            # just the IP

# nslookup — simpler (interactive or one-shot)
nslookup google.com
nslookup google.com 8.8.8.8     # use specific DNS server

# host — simple DNS lookup
host google.com
host 8.8.8.8                    # reverse lookup

# DNS configuration
cat /etc/resolv.conf             # DNS servers in use
cat /etc/hosts                   # local hostname overrides
cat /etc/nsswitch.conf           # name resolution order (files, dns, mdns)

# Flush DNS cache (systemd)
sudo systemd-resolve --flush-caches
sudo resolvectl flush-caches

# Check DNS resolution details
resolvectl status
resolvectl query google.com
```

---

## 4. Connectivity Testing

```bash
# ping — ICMP echo request
ping google.com                  # ping continuously
ping -c 4 google.com             # send 4 packets then stop
ping -i 0.5 google.com           # 0.5 second interval
ping -s 1400 google.com          # custom packet size (MTU testing)

# curl — transfer data, test HTTP
curl https://google.com          # get page content
curl -I https://google.com       # headers only (HEAD request)
curl -L https://google.com       # follow redirects
curl -o output.html https://google.com   # save to file
curl -v https://google.com       # verbose (show request + response headers)
curl -s https://api.example.com  # silent (no progress bar)

# Test specific HTTP method/data
curl -X POST -H "Content-Type: application/json" \
     -d '{"key":"value"}' https://api.example.com

# Test with auth
curl -u user:pass https://api.example.com
curl -H "Authorization: Bearer TOKEN" https://api.example.com

# wget — download files
wget https://example.com/file.tar.gz         # download file
wget -q https://example.com/file.tar.gz      # quiet
wget -O custom_name.tar.gz https://...       # custom output filename
wget -r -l 2 https://example.com/            # recursive download, 2 levels

# nc (netcat) — the "Swiss army knife" of networking
nc -zv google.com 443            # test if port 443 is open
nc -zv 192.168.1.1 22            # test SSH port
nc -l 1234                       # listen on port 1234
nc 192.168.1.10 1234             # connect to listener
echo "hello" | nc 192.168.1.10 1234   # send data

# Test if a TCP port is reachable (bash built-in, no nc needed)
timeout 3 bash -c "echo >/dev/tcp/google.com/80" 2>/dev/null \
  && echo "Port 80 open" || echo "Port 80 closed"
```

---

## 5. Socket & Connection Monitoring

```bash
# ss — socket statistics (modern replacement for netstat)
ss -tuln                         # listening TCP and UDP ports (numeric)
ss -tulnp                        # with process name
ss -tan                          # all TCP connections
ss -s                            # summary statistics

# Breakdown of flags:
# -t = TCP, -u = UDP, -l = listening, -n = numeric, -p = process

# netstat — legacy (still very common)
netstat -tuln                    # listening ports
netstat -tulnp                   # with process
netstat -an                      # all connections
netstat -rn                      # routing table
netstat -s                       # per-protocol statistics

# lsof — list open files (including sockets)
sudo lsof -i                     # all network connections
sudo lsof -i :80                 # what's using port 80
sudo lsof -i TCP:22              # TCP connections on port 22
sudo lsof -i -P -n               # numeric ports, no hostname resolution
sudo lsof -p 1234                # files opened by PID 1234
sudo lsof -u alice               # files opened by user alice

# Practical: find what's using a port
sudo ss -tulnp | grep :80
sudo lsof -i :80

# Connection states:
# LISTEN       = waiting for connections (server)
# ESTABLISHED  = active connection
# TIME_WAIT    = connection closing, waiting for stray packets
# CLOSE_WAIT   = remote closed, local still open
```

---

## 6. Firewall: iptables & ufw

### ufw — Uncomplicated Firewall (Ubuntu)
```bash
# Enable/disable
sudo ufw enable
sudo ufw disable
sudo ufw status verbose

# Allow/deny rules
sudo ufw allow 22/tcp             # allow SSH
sudo ufw allow 80/tcp             # allow HTTP
sudo ufw allow 443/tcp            # allow HTTPS
sudo ufw allow from 192.168.1.0/24  # allow entire subnet
sudo ufw deny 23/tcp              # deny telnet
sudo ufw delete allow 80/tcp      # remove rule

# App profiles
sudo ufw app list                 # see available app profiles
sudo ufw allow 'Nginx Full'       # allow by profile name

# Rate limiting (anti-brute-force)
sudo ufw limit ssh                # max 6 connections per 30 seconds
```

### iptables — Low-Level Firewall
```bash
# View rules
sudo iptables -L -v -n            # list all rules, verbose, numeric
sudo iptables -L INPUT -v -n      # only INPUT chain

# Basic rules
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT     # allow SSH
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT     # allow HTTP
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A INPUT -i lo -j ACCEPT                  # allow loopback
sudo iptables -A INPUT -j DROP                          # default deny

# Delete a rule (by line number)
sudo iptables -L INPUT --line-numbers
sudo iptables -D INPUT 3          # delete rule 3

# IMPORTANT: iptables rules are NOT persistent across reboots
# Save and restore:
sudo iptables-save > /etc/iptables/rules.v4
sudo iptables-restore < /etc/iptables/rules.v4
# Or install: sudo apt install iptables-persistent
```

---

## 7. Key Config Files

```bash
# /etc/hosts — local DNS overrides (checked before DNS)
cat /etc/hosts
# 127.0.0.1   localhost
# 192.168.1.50  myserver myserver.local

# Add entries:
echo "192.168.1.50  staging.myapp.com" | sudo tee -a /etc/hosts

# /etc/resolv.conf — DNS server configuration
cat /etc/resolv.conf
# nameserver 8.8.8.8
# nameserver 1.1.1.1
# search example.com   ← appended to unqualified names

# /etc/nsswitch.conf — resolution order
grep hosts /etc/nsswitch.conf
# hosts: files mdns4_minimal [NOTFOUND=return] dns
# Order: local /etc/hosts → mDNS → DNS server
```

---

## Interview Prep

**Q: A service is not responding. Walk me through your network troubleshooting steps.**

Systematic approach (OSI bottom-up):

```bash
# 1. Is the interface up and has an IP?
ip addr show

# 2. Can we reach the local gateway?
ip route | grep default
ping $(ip route | grep default | awk '{print $3}')

# 3. Can we resolve DNS?
dig +short google.com

# 4. Can we reach the target host?
ping target-server.com
traceroute target-server.com    # where does it fail?

# 5. Is the service listening on the expected port?
sudo ss -tulnp | grep :8080

# 6. Is the firewall blocking it?
sudo iptables -L -n | grep 8080
sudo ufw status

# 7. Can we connect to the port locally?
nc -zv localhost 8080

# 8. Can we connect remotely?
nc -zv target-server.com 8080

# 9. Check application logs
sudo journalctl -u myservice -f
sudo tail -f /var/log/myapp/error.log
```

---

## Common Gotcha

**`iptables` rules are not persistent by default.** A reboot clears all rules.

```bash
# After setting up iptables rules:
sudo iptables-save | sudo tee /etc/iptables/rules.v4
# Or:
sudo apt install iptables-persistent
sudo netfilter-persistent save

# Also: if you set a default DROP policy and lose connection,
# you're locked out of the server.
# Always set ACCEPT on INPUT before DROP:
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
# THEN set default DROP — not before
sudo iptables -P INPUT DROP
```

---

## Practice Exercises

1. Use `ip addr` to identify your primary interface, IP, subnet mask, and MAC address.
2. Use `dig` to find the MX records for `gmail.com`.
3. Use `ss -tulnp` to list all listening services and their ports.
4. Use `traceroute` to a public IP and identify how many hops it takes.
5. Use `curl -I` to check the HTTP headers and status code of any website.
6. Add a custom entry to `/etc/hosts` and verify it resolves with `ping`.

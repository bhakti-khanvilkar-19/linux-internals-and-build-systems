# Linux Networking — Connectivity, Configuration, & Troubleshooting

## Objective
Master Linux networking: TCP/IP fundamentals, interface configuration, firewall management, DNS, SSH, and troubleshooting tools.

---

## 1) Networking Fundamentals

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                          TCP/IP MODEL vs OSI MODEL                                  │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   OSI Layers                    TCP/IP Model              Protocol Examples         │
│   ───────────────────────────   ──────────────────────    ─────────────────────     │
│   7. Application                                          HTTP, FTP, SSH, DNS       │
│   6. Presentation      ───►     4. Application           SMTP, IMAP, NFS           │
│   5. Session                                                                        │
│   ───────────────────────────   ──────────────────────    ─────────────────────     │
│   4. Transport         ───►     3. Transport              TCP, UDP                  │
│   ───────────────────────────   ──────────────────────    ─────────────────────     │
│   3. Network           ───►     2. Internet               IP, ICMP, ARP             │
│   ───────────────────────────   ──────────────────────    ─────────────────────     │
│   2. Data Link         ───►     1. Network Access         Ethernet, Wi-Fi           │
│   1. Physical                                             Physical cables           │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### IP Addressing

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                           IPv4 ADDRESSING                                           │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   IPv4 Address: 192.168.1.100                                                       │
│   Subnet Mask:  255.255.255.0 (/24)                                                 │
│                                                                                     │
│   ┌──────────────────────┬──────────────────────┐                                   │
│   │   Network Portion    │    Host Portion      │                                   │
│   │   192.168.1          │    .100              │                                   │
│   └──────────────────────┴──────────────────────┘                                   │
│                                                                                     │
│   Private IP Ranges (RFC 1918):                                                     │
│   • 10.0.0.0/8        (10.0.0.0 – 10.255.255.255)      Class A                      │
│   • 172.16.0.0/12     (172.16.0.0 – 172.31.255.255)    Class B                      │
│   • 192.168.0.0/16    (192.168.0.0 – 192.168.255.255)  Class C                      │
│                                                                                     │
│   Special Addresses:                                                                │
│   • 127.0.0.1         Loopback (localhost)                                          │
│   • 0.0.0.0           All interfaces / default route                                │
│   • 255.255.255.255   Broadcast                                                     │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### Common Ports

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                         COMMON PORTS                                                │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   Port    │ Service  │ Protocol │ Description                                       │
│   ────────┼──────────┼──────────┼────────────────────────────────────               │
│   20, 21  │ FTP      │ TCP      │ File Transfer Protocol                            │
│   22      │ SSH      │ TCP      │ Secure Shell                                      │
│   23      │ Telnet   │ TCP      │ Unencrypted remote access (avoid!)                │
│   25      │ SMTP     │ TCP      │ Email sending                                     │
│   53      │ DNS      │ UDP/TCP  │ Domain Name System                                │
│   67, 68  │ DHCP     │ UDP      │ Dynamic Host Configuration                        │
│   80      │ HTTP     │ TCP      │ Web (unencrypted)                                 │
│   443     │ HTTPS    │ TCP      │ Web (encrypted)                                   │
│   110     │ POP3     │ TCP      │ Email retrieval                                   │
│   143     │ IMAP     │ TCP      │ Email retrieval                                   │
│   3306    │ MySQL    │ TCP      │ MySQL database                                    │
│   5432    │ PostgreSQL│ TCP     │ PostgreSQL database                               │
│   6379    │ Redis    │ TCP      │ Redis cache                                       │
│                                                                                     │
│   Port Ranges:                                                                      │
│   • 0-1023       Well-known (require root)                                          │
│   • 1024-49151   Registered                                                         │
│   • 49152-65535  Dynamic/Private                                                    │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 2) Network Configuration Tools

### Modern Tools: ip (iproute2)

```bash
# View interfaces
ip link show                      # All interfaces
ip -br link show                  # Brief format
ip link show eth0                 # Specific interface

# View IP addresses
ip addr show                      # All addresses  (shortcut: ip a)
ip -br addr show                  # Brief format
ip addr show dev eth0             # Specific interface

# View routing table
ip route show                     # All routes (shortcut: ip r)
ip route get 8.8.8.8              # How to reach specific IP

# View neighbors (ARP table)
ip neigh show                     # ARP cache

# Interface up/down
sudo ip link set eth0 up
sudo ip link set eth0 down

# Add/remove IP address
sudo ip addr add 192.168.1.100/24 dev eth0
sudo ip addr del 192.168.1.100/24 dev eth0

# Add/remove route
sudo ip route add 10.0.0.0/8 via 192.168.1.1
sudo ip route del 10.0.0.0/8
sudo ip route add default via 192.168.1.1        # Default gateway
```

### Legacy Tools (Still Useful)

```bash
# ifconfig (net-tools - may need to install)
ifconfig                          # All interfaces
ifconfig eth0                     # Specific interface
sudo ifconfig eth0 192.168.1.100  # Set IP (temporary)
sudo ifconfig eth0 up/down        # Interface up/down

# route
route -n                          # Show routing table
sudo route add default gw 192.168.1.1

# arp
arp -n                            # ARP table
```

---

## 3) Network Configuration Files

### Netplan (Ubuntu 17.10+)

```yaml
# /etc/netplan/00-installer-config.yaml

network:
  version: 2
  renderer: networkd        # or NetworkManager
  
  ethernets:
    eth0:
      dhcp4: true           # DHCP
    
    eth1:
      addresses:
        - 192.168.1.100/24
        - 192.168.1.101/24  # Multiple IPs
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4
        search:
          - mycompany.local
      mtu: 1500

  wifis:
    wlan0:
      dhcp4: true
      access-points:
        "MyWiFi":
          password: "secret123"
```

```bash
# Apply netplan configuration
sudo netplan apply

# Test configuration (dry run)
sudo netplan try                  # Auto-reverts if no confirmation

# Generate backend config
sudo netplan generate

# Debug
sudo netplan --debug apply
```

### /etc/hosts

```bash
cat /etc/hosts

# Format: IP    hostname    aliases
127.0.0.1       localhost
127.0.1.1       mycomputer.localdomain mycomputer
192.168.1.50    webserver.local webserver
192.168.1.51    dbserver.local dbserver
```

### /etc/hostname

```bash
cat /etc/hostname
# mycomputer

# Change hostname
sudo hostnamectl set-hostname newhostname
```

### DNS Configuration

```bash
# Modern Ubuntu uses systemd-resolved
cat /etc/resolv.conf              # Usually a symlink

# Check DNS status
resolvectl status

# Configure DNS in Netplan (recommended)
# Or edit /etc/systemd/resolved.conf for system-wide defaults

# Flush DNS cache
sudo resolvectl flush-caches
```

---

## 4) Network Testing Tools

### ping — Test Connectivity

```bash
ping google.com                    # Continuous ping (Ctrl+C to stop)
ping -c 5 google.com               # 5 pings only
ping -c 5 -i 0.5 google.com        # 0.5 second interval
ping -c 5 -s 1000 google.com       # 1000 byte packets
ping -c 1 -W 2 192.168.1.1         # 2 second timeout
ping6 ::1                          # IPv6 ping
```

### traceroute / tracepath — Trace Route

```bash
traceroute google.com              # Show hops to destination
traceroute -n google.com           # No DNS resolution (faster)
tracepath google.com               # No root required alternative

# mtr - combines ping and traceroute (install: sudo apt install mtr)
mtr google.com                     # Interactive
mtr -r -c 10 google.com            # Report mode, 10 cycles
```

### netstat / ss — Network Statistics

```bash
# ss (modern, faster)
ss -tuln                           # TCP/UDP listening ports (numeric)
ss -tulpn                          # Include process name (needs root)
ss -tan                            # All TCP connections
ss -tan state established          # Only established connections
ss -s                              # Summary statistics

# netstat (legacy but common)
netstat -tuln                      # Listening ports
netstat -tupn                      # With process names
netstat -r                         # Routing table

# Options:
# -t = TCP, -u = UDP, -l = listening
# -n = numeric (no DNS), -p = process name
```

### curl / wget — HTTP Testing

```bash
# curl
curl http://example.com            # GET request
curl -I http://example.com         # Headers only (HEAD)
curl -X POST -d "data" http://...  # POST request
curl -o file.html http://...       # Save to file
curl -O http://example.com/file.zip # Save with original name
curl -v http://example.com         # Verbose (debug)
curl -L http://example.com         # Follow redirects
curl --connect-timeout 5 http://... # Connection timeout

# wget
wget http://example.com/file.zip   # Download file
wget -c http://example.com/file.zip # Resume download
wget -r http://example.com         # Recursive download
wget -q http://example.com         # Quiet mode
```

### dig / nslookup — DNS Queries

```bash
# dig (recommended)
dig google.com                     # A record (default)
dig google.com ANY                 # All records
dig google.com MX                  # Mail servers
dig google.com NS                  # Name servers
dig -x 8.8.8.8                     # Reverse lookup
dig @8.8.8.8 google.com            # Query specific DNS server
dig +short google.com              # Concise output

# nslookup (interactive)
nslookup google.com
nslookup google.com 8.8.8.8        # Use specific DNS

# Check what DNS server is being used
resolvectl status
```

### nc (netcat) — Swiss Army Knife

```bash
# Test TCP port
nc -zv google.com 80               # Check if port open
nc -zv google.com 80-443           # Port range scan

# Simple server
nc -l 8080                         # Listen on port 8080

# Transfer file
# Receiver: nc -l 9999 > received_file
# Sender:   nc receiver_ip 9999 < file_to_send

# Send HTTP request manually
echo "GET / HTTP/1.1\nHost: example.com\n" | nc example.com 80
```

### tcpdump — Packet Capture

```bash
# Basic capture
sudo tcpdump -i eth0               # Capture on interface
sudo tcpdump -i any                # All interfaces
sudo tcpdump -c 10 -i eth0         # Capture 10 packets

# Filter by host/port
sudo tcpdump host 192.168.1.1
sudo tcpdump port 80
sudo tcpdump src 192.168.1.1
sudo tcpdump dst port 443

# Save to file
sudo tcpdump -i eth0 -w capture.pcap

# Read from file
tcpdump -r capture.pcap

# Verbose output
sudo tcpdump -v -i eth0            # Verbose
sudo tcpdump -vv -i eth0           # More verbose
```

---

## 5) Firewall with UFW

### UFW Basics

```bash
# Enable/disable
sudo ufw enable
sudo ufw disable
sudo ufw status                    # Check status
sudo ufw status verbose            # With defaults
sudo ufw status numbered           # With rule numbers

# Default policies
sudo ufw default deny incoming     # Default: block incoming
sudo ufw default allow outgoing    # Default: allow outgoing

# Allow rules
sudo ufw allow 22                  # Allow SSH
sudo ufw allow 22/tcp              # TCP only
sudo ufw allow ssh                 # By service name
sudo ufw allow 80,443/tcp          # Multiple ports

# Allow from specific IP
sudo ufw allow from 192.168.1.100
sudo ufw allow from 192.168.1.0/24 to any port 22

# Deny rules
sudo ufw deny 23                   # Deny telnet
sudo ufw deny from 10.0.0.0/8

# Delete rules
sudo ufw delete allow 80
sudo ufw delete 3                  # By rule number

# Reset
sudo ufw reset                     # Remove all rules
```

### Application Profiles

```bash
# List available profiles
sudo ufw app list

# Allow application
sudo ufw allow 'Nginx Full'
sudo ufw allow 'OpenSSH'

# View profile details
sudo ufw app info 'Nginx Full'
```

---

## 6) SSH — Secure Shell

### Basic SSH

```bash
# Connect
ssh user@hostname
ssh user@192.168.1.100
ssh -p 2222 user@hostname          # Different port

# Run command remotely
ssh user@hostname 'ls -la'
ssh user@hostname 'cat /etc/os-release'

# Copy files with scp
scp file.txt user@hostname:/path/  # Local to remote
scp user@hostname:/path/file.txt . # Remote to local
scp -r folder/ user@hostname:/path/ # Directory (recursive)

# rsync (better for large/multiple files)
rsync -avz folder/ user@hostname:/path/
rsync -avz --progress file.txt user@host:/path/
```

### SSH Key Authentication

```bash
# Generate key pair
ssh-keygen -t ed25519 -C "your_email@example.com"
# Or RSA: ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

# Keys stored in:
# ~/.ssh/id_ed25519      (private - NEVER share)
# ~/.ssh/id_ed25519.pub  (public - share this)

# Copy public key to server
ssh-copy-id user@hostname
# Or manually:
cat ~/.ssh/id_ed25519.pub | ssh user@host 'mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys'

# Now login without password
ssh user@hostname
```

### SSH Config File

```bash
# ~/.ssh/config

Host webserver
    HostName 192.168.1.100
    User admin
    Port 22
    IdentityFile ~/.ssh/id_ed25519

Host dbserver
    HostName 192.168.1.101
    User dbadmin
    Port 2222
    
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 3

# Now connect with:
ssh webserver
ssh dbserver
```

### SSH Server Configuration

```bash
# /etc/ssh/sshd_config

Port 22                            # Change default port (security through obscurity)
PermitRootLogin no                 # Disable root SSH
PasswordAuthentication no          # Key-only authentication
PubkeyAuthentication yes
MaxAuthTries 3
ClientAliveInterval 300

# Restart SSH after changes
sudo systemctl restart sshd
```

---

## 7) Network Troubleshooting Flow

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                    NETWORK TROUBLESHOOTING CHECKLIST                                │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   1. Check Interface Status                                                         │
│      ├── ip link show                    Is interface UP?                           │
│      └── ip addr show                    Is IP configured?                          │
│                                                                                     │
│   2. Check Local Connectivity                                                       │
│      ├── ping 127.0.0.1                  Loopback OK?                               │
│      └── ping <gateway>                  Gateway reachable?                         │
│                                                                                     │
│   3. Check Default Gateway                                                          │
│      └── ip route show                   Default route exists?                      │
│                                                                                     │
│   4. Check DNS                                                                      │
│      ├── ping 8.8.8.8                    Can reach internet by IP?                  │
│      ├── ping google.com                 DNS working?                               │
│      └── dig google.com                  DNS query details                          │
│                                                                                     │
│   5. Check Specific Service                                                         │
│      ├── nc -zv host port                Is port open?                              │
│      ├── ss -tuln                        Is service listening?                      │
│      └── sudo ufw status                 Is firewall blocking?                      │
│                                                                                     │
│   6. Check Routing                                                                  │
│      └── traceroute destination          Where does it fail?                        │
│                                                                                     │
│   7. Packet Capture                                                                 │
│      └── sudo tcpdump -i eth0            Deep packet analysis                       │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 8) Practical Exercises

### Exercise 1: Configure Static IP
```bash
# Edit netplan configuration
sudo nano /etc/netplan/01-static.yaml

# Add configuration (as shown in section 3)
# Apply: sudo netplan apply
# Verify: ip addr show
```

### Exercise 2: Set Up SSH Key Auth
```bash
# Generate keys
ssh-keygen -t ed25519 -C "mykey"

# Copy to remote server
ssh-copy-id user@server

# Test passwordless login
ssh user@server

# Disable password auth on server (optional)
# Edit /etc/ssh/sshd_config: PasswordAuthentication no
# sudo systemctl restart sshd
```

### Exercise 3: Configure UFW
```bash
# Enable UFW with default deny
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH and HTTP
sudo ufw allow ssh
sudo ufw allow 80/tcp

# Enable firewall
sudo ufw enable
sudo ufw status verbose
```

---

## 9) Quick Reference

| Task | Command |
|------|---------|
| Show IP addresses | `ip addr show` |
| Show interfaces | `ip link show` |
| Show routes | `ip route show` |
| Test connectivity | `ping host` |
| Trace route | `traceroute host` |
| Show listening ports | `ss -tuln` |
| DNS lookup | `dig hostname` |
| Test port | `nc -zv host port` |
| Download file | `wget url` or `curl -O url` |
| SSH connect | `ssh user@host` |
| Copy file via SSH | `scp file user@host:/path/` |
| Enable firewall | `sudo ufw enable` |
| Allow port in firewall | `sudo ufw allow 80/tcp` |
| Apply netplan | `sudo netplan apply` |

---

## 10) Interview Questions

**Q1: How do you find what process is using port 80?**
> `sudo ss -tulpn | grep :80` or `sudo lsof -i :80`

**Q2: Difference between TCP and UDP?**
> TCP is connection-oriented with guaranteed delivery and ordering. UDP is connectionless, faster but unreliable. Use TCP for HTTP, SSH; UDP for DNS, streaming.

**Q3: What is the difference between ping and traceroute?**
> `ping` tests if a host is reachable and measures latency. `traceroute` shows the path (all hops) packets take to reach the destination.

**Q4: How do you set up passwordless SSH?**
> Generate keys with `ssh-keygen`, copy public key to server with `ssh-copy-id user@host`, connect normally with `ssh user@host`.

**Q5: How do you persist IP configuration in Ubuntu?**
> Use Netplan. Edit `/etc/netplan/*.yaml`, configure static IP or DHCP, run `sudo netplan apply`.

---

*Next: [12_Storage_Disks.md](12_Storage_Disks.md) — Disk management, partitions, and LVM.*

# Network Basics

Understanding networking fundamentals for system administrators, DevOps engineers, and developers. From basic concepts to practical troubleshooting.

## Why Network Knowledge Matters

In today's connected world, everything depends on networks:
- **Application deployment**: Services need to communicate
- **Security**: Understand attack vectors and protections
- **Troubleshooting**: Diagnose connectivity issues
- **Performance**: Optimize network-dependent applications
- **Cloud computing**: Virtual networks and load balancers

## OSI Model and TCP/IP Stack

### OSI Model (7 Layers)
```
7. Application    - HTTP, HTTPS, FTP, SSH, DNS
6. Presentation   - Encryption, compression, formatting
5. Session        - Session management, authentication
4. Transport      - TCP, UDP (reliability, flow control)
3. Network        - IP routing, addressing
2. Data Link      - Ethernet, WiFi (frame formatting)
1. Physical       - Cables, radio waves, electrical signals
```

### TCP/IP Model (4 Layers)
```
4. Application    - HTTP, DNS, SSH, FTP
3. Transport      - TCP, UDP
2. Internet       - IP, ICMP, ARP
1. Network Access - Ethernet, WiFi
```

## IP Addressing and Subnetting

### IPv4 Addressing
```bash
# IPv4 address format: x.x.x.x (32 bits)
192.168.1.100

# Address classes
Class A: 1.0.0.0    - 126.255.255.255  (/8)
Class B: 128.0.0.0  - 191.255.255.255  (/16)
Class C: 192.0.0.0  - 223.255.255.255  (/24)

# Private address ranges
10.0.0.0     - 10.255.255.255    (Class A)
172.16.0.0   - 172.31.255.255    (Class B)
192.168.0.0  - 192.168.255.255   (Class C)

# Special addresses
127.0.0.1    - Loopback (localhost)
0.0.0.0      - Default route/any address
255.255.255.255 - Broadcast address
```

### Subnet Masks and CIDR
```bash
# Subnet mask notation
255.255.255.0   = /24 (24 bits for network, 8 for hosts)
255.255.240.0   = /20 (20 bits for network, 12 for hosts)
255.255.255.128 = /25 (25 bits for network, 7 for hosts)

# CIDR calculation examples
192.168.1.0/24    # Network: 192.168.1.0, Hosts: 1-254
192.168.1.0/25    # Network: 192.168.1.0, Hosts: 1-126
192.168.1.128/25  # Network: 192.168.1.128, Hosts: 129-254

# Usable hosts calculation
Hosts = 2^(32-prefix) - 2
/24 = 2^8 - 2 = 254 hosts
/25 = 2^7 - 2 = 126 hosts
```

### IPv6 Basics
```bash
# IPv6 address format (128 bits)
2001:0db8:85a3:0000:0000:8a2e:0370:7334

# Compressed notation
2001:db8:85a3::8a2e:370:7334

# Address types
::1/128           # Loopback
2001:db8::/32     # Documentation prefix
fe80::/10         # Link-local
fc00::/7          # Unique local (private)
2000::/3          # Global unicast
```

## Network Protocols

### TCP vs UDP

#### TCP (Transmission Control Protocol)
```bash
# Characteristics
- Connection-oriented
- Reliable delivery
- Ordered delivery
- Flow control
- Error checking
- Higher overhead

# Use cases
- Web browsing (HTTP/HTTPS)
- Email (SMTP, IMAP, POP3)
- File transfer (FTP, SSH)
- Database connections
```

#### UDP (User Datagram Protocol)
```bash
# Characteristics
- Connectionless
- No delivery guarantee
- No ordering guarantee
- No flow control
- Minimal overhead
- Fast

# Use cases
- DNS queries
- Video streaming
- Online gaming
- DHCP
- NTP (time synchronization)
```

### Common Ports
```bash
# Well-known ports (0-1023)
20/21    - FTP (data/control)
22       - SSH
23       - Telnet
25       - SMTP
53       - DNS
67/68    - DHCP
80       - HTTP
110      - POP3
143      - IMAP
443      - HTTPS
993      - IMAPS
995      - POP3S

# Registered ports (1024-49151)
3306     - MySQL
5432     - PostgreSQL
6379     - Redis
8080     - HTTP alternate
9200     - Elasticsearch

# Dynamic/private ports (49152-65535)
# Used for outbound connections
```

## Network Configuration

### Linux Network Configuration
```bash
# Modern approach (ip command)
ip addr show                    # Show IP addresses
ip addr add 192.168.1.100/24 dev eth0  # Add IP address
ip addr del 192.168.1.100/24 dev eth0  # Remove IP address
ip route show                   # Show routing table
ip route add default via 192.168.1.1   # Add default route
ip link show                    # Show network interfaces
ip link set eth0 up             # Bring interface up
ip link set eth0 down           # Bring interface down

# Legacy approach (deprecated but still common)
ifconfig eth0 192.168.1.100 netmask 255.255.255.0
route add default gw 192.168.1.1
```

### Network Interface Configuration Files
```bash
# Ubuntu/Debian (/etc/network/interfaces)
auto eth0
iface eth0 inet static
    address 192.168.1.100
    netmask 255.255.255.0
    gateway 192.168.1.1
    dns-nameservers 8.8.8.8 8.8.4.4

# CentOS/RHEL (/etc/sysconfig/network-scripts/ifcfg-eth0)
DEVICE=eth0
BOOTPROTO=static
IPADDR=192.168.1.100
NETMASK=255.255.255.0
GATEWAY=192.168.1.1
DNS1=8.8.8.8
DNS2=8.8.4.4
ONBOOT=yes

# Modern systemd (netplan on Ubuntu)
# /etc/netplan/01-network-manager-all.yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: false
      addresses: [192.168.1.100/24]
      gateway4: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
```

### DNS Configuration
```bash
# /etc/resolv.conf
nameserver 8.8.8.8
nameserver 8.8.4.4
search company.local
domain company.local

# /etc/hosts (local hostname resolution)
127.0.0.1   localhost
192.168.1.10 server.company.local server
192.168.1.20 database.company.local database

# systemd-resolved (modern systems)
resolvectl status
resolvectl query google.com
```

## Network Services

### DHCP (Dynamic Host Configuration Protocol)
```bash
# DHCP client
dhclient eth0               # Request IP address
dhclient -r eth0           # Release IP address
cat /var/lib/dhcp/dhclient.leases  # View lease information

# DHCP server configuration (/etc/dhcp/dhcpd.conf)
subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.100 192.168.1.200;
    option routers 192.168.1.1;
    option domain-name-servers 8.8.8.8, 8.8.4.4;
    option domain-name "company.local";
    default-lease-time 86400;
    max-lease-time 86400;
}
```

### DNS (Domain Name System)
```bash
# DNS lookup tools
nslookup google.com         # Basic DNS lookup
dig google.com              # Advanced DNS lookup
dig @8.8.8.8 google.com     # Query specific DNS server
dig google.com MX           # Query mail exchange records
dig google.com NS           # Query name server records
dig -x 8.8.8.8              # Reverse DNS lookup

# DNS record types
A       - IPv4 address
AAAA    - IPv6 address
CNAME   - Canonical name (alias)
MX      - Mail exchange
NS      - Name server
PTR     - Pointer (reverse DNS)
SOA     - Start of authority
TXT     - Text record
```

## Network Troubleshooting

### Connectivity Testing
```bash
# Basic connectivity
ping -c 4 google.com        # Test reachability
ping6 -c 4 ipv6.google.com  # IPv6 ping
traceroute google.com       # Trace path to destination
traceroute6 ipv6.google.com # IPv6 traceroute
mtr google.com              # Continuous traceroute

# Port testing
telnet google.com 80        # Test TCP port
nc -zv google.com 80        # Test TCP port with netcat
nc -zvu google.com 53       # Test UDP port with netcat
timeout 5 bash -c "</dev/tcp/google.com/80"  # Bash TCP test
```

### Network Analysis
```bash
# Active connections
netstat -tuln               # Show listening ports
netstat -tun                # Show all connections
ss -tuln                    # Modern alternative to netstat
ss -tulpn                   # Show process names
lsof -i                     # Show all network connections
lsof -i :80                 # Show what's using port 80

# Packet capture
tcpdump -i eth0             # Capture on interface
tcpdump -i eth0 port 80     # Capture HTTP traffic
tcpdump -i eth0 host 192.168.1.100  # Capture specific host
tcpdump -w capture.pcap -i eth0      # Save to file
tcpdump -r capture.pcap     # Read from file

# Network statistics
cat /proc/net/dev           # Interface statistics
iftop                       # Real-time bandwidth monitor
nethogs                     # Network usage by process
vnstat                      # Network traffic statistics
```

### Common Network Issues

#### DNS Resolution Problems
```bash
# Test DNS resolution
nslookup domain.com
dig domain.com

# Check DNS configuration
cat /etc/resolv.conf
systemd-resolve --status

# Flush DNS cache
sudo systemd-resolve --flush-caches  # systemd
sudo service nscd restart           # nscd
```

#### Routing Issues
```bash
# Check routing table
ip route show
route -n

# Test gateway connectivity
ping $(ip route | grep default | awk '{print $3}')

# Trace route to destination
traceroute -n destination

# Check for asymmetric routing
# Ensure return path uses same route
```

#### Firewall Issues
```bash
# Check firewall status
ufw status verbose          # Ubuntu
firewall-cmd --list-all     # CentOS/RHEL
iptables -L -n -v           # iptables

# Temporarily disable firewall for testing
ufw disable                 # Ubuntu
systemctl stop firewalld    # CentOS/RHEL
```

## Network Security Basics

### Port Scanning Detection
```bash
# Monitor for port scans
netstat -an | grep SYN_RECV | wc -l  # Count half-open connections
ss -tan state syn-recv | wc -l       # Modern alternative

# Log connection attempts
iptables -A INPUT -p tcp --dport 22 -j LOG --log-prefix "SSH: "
tail -f /var/log/kern.log | grep "SSH:"
```

### Basic Network Access Control
```bash
# Block specific IP with iptables
iptables -A INPUT -s 192.168.1.100 -j DROP

# Allow only specific networks
iptables -A INPUT -s 192.168.1.0/24 -j ACCEPT
iptables -A INPUT -j DROP

# Rate limiting SSH connections
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --set
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --update --seconds 60 --hitcount 3 -j DROP
```

### Network Monitoring
```bash
# Monitor network connections
watch -n 1 'netstat -tuln | grep LISTEN'

# Monitor bandwidth usage
iftop -i eth0               # Interface bandwidth
nload eth0                  # Interface load

# Monitor for suspicious activity
tail -f /var/log/auth.log | grep "Failed password"
grep "Invalid user" /var/log/auth.log
```

## Virtual Networking

### VLANs (Virtual LANs)
```bash
# Create VLAN interface
ip link add link eth0 name eth0.100 type vlan id 100
ip addr add 192.168.100.1/24 dev eth0.100
ip link set eth0.100 up

# Configure VLAN permanently (Ubuntu)
auto eth0.100
iface eth0.100 inet static
    address 192.168.100.1
    netmask 255.255.255.0
    vlan-raw-device eth0
```

### Network Bridges
```bash
# Create bridge
ip link add br0 type bridge
ip link set br0 up

# Add interfaces to bridge
ip link set eth0 master br0
ip link set eth1 master br0

# Assign IP to bridge
ip addr add 192.168.1.1/24 dev br0
```

### Network Namespaces
```bash
# Create network namespace
ip netns add test_ns

# Run command in namespace
ip netns exec test_ns ip addr show

# Create virtual ethernet pair
ip link add veth0 type veth peer name veth1

# Move interface to namespace
ip link set veth1 netns test_ns

# Configure interfaces
ip addr add 10.0.0.1/24 dev veth0
ip netns exec test_ns ip addr add 10.0.0.2/24 dev veth1
ip link set veth0 up
ip netns exec test_ns ip link set veth1 up
```

---

*Network knowledge is essential for any IT role. Understanding these fundamentals will help you troubleshoot connectivity issues, secure your systems, and design scalable network architectures.*

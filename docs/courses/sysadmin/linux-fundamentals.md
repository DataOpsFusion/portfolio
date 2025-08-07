# Linux Fundamentals

Master the Linux operating system - from basic commands to system administration. Essential knowledge for any technical role.

## Why Linux Matters

Linux powers the modern world:
- **Servers**: 96% of web servers run Linux
- **Cloud computing**: AWS, GCP, Azure all run on Linux
- **Containers**: Docker and Kubernetes are Linux-native
- **Mobile**: Android is based on Linux
- **IoT**: Most embedded systems use Linux
- **Development**: Popular choice for developers

## Getting Started with Linux

### Linux Distributions
Choose the right distribution for your needs:

#### **Ubuntu**
- **Best for**: Beginners, desktop use, development
- **Pros**: User-friendly, great community, extensive documentation
- **Cons**: Can be resource-heavy

#### **CentOS/RHEL**
- **Best for**: Enterprise environments, servers
- **Pros**: Stable, long-term support, enterprise features
- **Cons**: Older packages, learning curve

#### **Debian**
- **Best for**: Servers, stability-focused environments
- **Pros**: Very stable, lightweight, secure
- **Cons**: Conservative package updates

#### **Arch Linux**
- **Best for**: Advanced users, customization
- **Pros**: Cutting-edge packages, minimal base system
- **Cons**: Steep learning curve, manual configuration

### Linux File System Hierarchy
```
/                 # Root directory
├── bin/          # Essential user binaries
├── boot/         # Boot loader files
├── dev/          # Device files
├── etc/          # Configuration files
├── home/         # User home directories
├── lib/          # Shared libraries
├── media/        # Removable media mount points
├── mnt/          # Temporary mount points
├── opt/          # Optional software packages
├── proc/         # Process and kernel information
├── root/         # Root user home directory
├── run/          # Runtime variable data
├── sbin/         # System binaries
├── srv/          # Service data
├── sys/          # System information
├── tmp/          # Temporary files
├── usr/          # User utilities and applications
├── var/          # Variable data (logs, databases)
└── snap/         # Snap packages (Ubuntu)
```

## Essential Commands

### Navigation and File Operations
```bash
# Directory navigation
pwd                    # Print working directory
ls -la                 # List files with details
cd /path/to/directory  # Change directory
cd ~                   # Go to home directory
cd ..                  # Go up one directory
cd -                   # Go to previous directory

# File operations
touch filename         # Create empty file
mkdir -p path/to/dir   # Create directories recursively
cp file1 file2         # Copy file
cp -r dir1/ dir2/      # Copy directory recursively
mv old_name new_name   # Move/rename file
rm file                # Remove file
rm -rf directory       # Remove directory recursively
ln -s target link      # Create symbolic link

# File viewing
cat file               # Display entire file
less file              # View file with pagination
head -n 20 file        # Show first 20 lines
tail -n 20 file        # Show last 20 lines
tail -f file           # Follow file changes (live)
```

### File Permissions
```bash
# Understanding permissions
# drwxrwxrwx
# d = directory, - = file
# rwx = read, write, execute (user, group, others)

# Viewing permissions
ls -l filename

# Changing permissions
chmod 755 filename     # rwxr-xr-x
chmod u+x filename     # Add execute for user
chmod g-w filename     # Remove write for group
chmod o=r filename     # Set others to read only

# Changing ownership
chown user:group filename
chown -R user:group directory/

# Special permissions
chmod +t directory     # Sticky bit
chmod u+s filename     # SUID
chmod g+s filename     # SGID
```

### Process Management
```bash
# Viewing processes
ps aux                 # Show all processes
ps aux | grep nginx    # Find specific process
top                    # Real-time process viewer
htop                   # Enhanced process viewer (if installed)
pstree                 # Process tree view

# Process control
jobs                   # Show active jobs
bg                     # Put job in background
fg                     # Bring job to foreground
nohup command &        # Run command immune to hangups
kill PID               # Kill process by PID
kill -9 PID            # Force kill process
killall process_name   # Kill all processes by name
pkill -f pattern       # Kill processes matching pattern

# System information
uptime                 # System uptime and load
who                    # Who is logged in
w                      # Who is logged in and what they're doing
id                     # Current user and group IDs
uname -a               # System information
```

## System Administration

### Package Management
```bash
# Ubuntu/Debian (APT)
apt update             # Update package list
apt upgrade            # Upgrade installed packages
apt install package    # Install package
apt remove package     # Remove package
apt search keyword     # Search for packages
apt list --installed   # List installed packages

# CentOS/RHEL (YUM/DNF)
yum update             # Update packages
yum install package    # Install package
yum remove package     # Remove package
yum search keyword     # Search packages
dnf install package    # DNF (newer than YUM)

# Arch Linux (Pacman)
pacman -Syu            # Update system
pacman -S package      # Install package
pacman -R package      # Remove package
pacman -Ss keyword     # Search packages

# Universal (Snap)
snap install package   # Install snap package
snap list              # List installed snaps
snap refresh           # Update all snaps
```

### Service Management (systemd)
```bash
# Service control
systemctl start service_name      # Start service
systemctl stop service_name       # Stop service
systemctl restart service_name    # Restart service
systemctl reload service_name     # Reload configuration
systemctl enable service_name     # Enable auto-start
systemctl disable service_name    # Disable auto-start

# Service status
systemctl status service_name     # Show service status
systemctl is-active service_name  # Check if service is active
systemctl is-enabled service_name # Check if service is enabled

# System control
systemctl reboot              # Reboot system
systemctl poweroff            # Shutdown system
systemctl suspend             # Suspend system

# Logs
journalctl -u service_name    # Show service logs
journalctl -f                 # Follow logs
journalctl --since "1 hour ago" # Recent logs
```

### Network Configuration
```bash
# Network information
ip addr show           # Show IP addresses
ip route show          # Show routing table
netstat -tuln          # Show listening ports
ss -tuln               # Modern alternative to netstat
lsof -i :80            # Show what's using port 80

# Network configuration
ip addr add 192.168.1.100/24 dev eth0  # Add IP address
ip route add default via 192.168.1.1   # Add default route

# DNS configuration
cat /etc/resolv.conf   # Show DNS servers
nslookup domain.com    # DNS lookup
dig domain.com         # Advanced DNS lookup

# Network testing
ping -c 4 google.com   # Ping with count
traceroute google.com  # Trace route to destination
wget http://example.com/file  # Download file
curl -I http://example.com    # HTTP headers only
```

### Disk and Storage Management
```bash
# Disk usage
df -h                  # Show disk usage
du -sh /path/to/dir    # Show directory size
du -h --max-depth=1    # Show subdirectory sizes
ncdu                   # Interactive disk usage (if installed)

# Disk partitioning
lsblk                  # List block devices
fdisk -l               # List partitions
fdisk /dev/sda         # Partition disk (interactive)
parted /dev/sda        # Advanced partitioning tool

# File systems
mkfs.ext4 /dev/sda1    # Format as ext4
mkfs.xfs /dev/sda1     # Format as XFS
mount /dev/sda1 /mnt   # Mount filesystem
umount /mnt            # Unmount filesystem
fsck /dev/sda1         # Check filesystem

# Mounting
mount                  # Show mounted filesystems
cat /etc/fstab         # Permanent mount configuration
mount -a               # Mount all filesystems in fstab
```

## System Monitoring and Performance

### Performance Monitoring
```bash
# CPU and memory
top                    # Real-time system monitor
htop                   # Enhanced system monitor
vmstat 1               # Virtual memory statistics
iostat 1               # I/O statistics
sar -u 1               # CPU utilization (sysstat package)

# Memory
free -h                # Memory usage
cat /proc/meminfo      # Detailed memory information
ps aux --sort=-%mem | head  # Processes by memory usage

# Disk I/O
iotop                  # Real-time disk I/O monitor
lsof                   # List open files
fuser -v /path         # Show processes using file/directory

# Network
iftop                  # Network bandwidth monitor
nethogs                # Network usage by process
tcpdump -i eth0        # Packet capture
wireshark              # GUI packet analyzer
```

### Log Management
```bash
# System logs
tail -f /var/log/syslog      # Follow system log
tail -f /var/log/auth.log    # Authentication log
tail -f /var/log/kern.log    # Kernel log
dmesg                        # Kernel ring buffer

# Application logs
tail -f /var/log/nginx/access.log    # Web server access log
tail -f /var/log/mysql/error.log     # Database error log

# Log rotation
logrotate -d /etc/logrotate.conf     # Test log rotation
cat /etc/logrotate.conf              # Log rotation configuration

# Journal logs (systemd)
journalctl                    # All journal logs
journalctl -f                 # Follow journal logs
journalctl -u nginx           # Service-specific logs
journalctl --since yesterday  # Recent logs
```

## Security Basics

### User Management
```bash
# User operations
useradd username            # Add user
usermod -aG group username  # Add user to group
userdel username            # Delete user
passwd username             # Change password
su - username               # Switch user
sudo command                # Run command as root

# Group operations
groupadd groupname          # Add group
groupdel groupname          # Delete group
groups username             # Show user's groups
id username                 # Show user and group IDs
```

### Firewall Management
```bash
# UFW (Ubuntu Firewall)
ufw enable                  # Enable firewall
ufw disable                 # Disable firewall
ufw status                  # Show firewall status
ufw allow 22                # Allow SSH
ufw allow 80/tcp            # Allow HTTP
ufw deny from 192.168.1.100 # Block specific IP

# iptables (traditional)
iptables -L                 # List rules
iptables -A INPUT -p tcp --dport 22 -j ACCEPT  # Allow SSH
iptables-save > /etc/iptables/rules.v4  # Save rules

# firewalld (CentOS/RHEL)
firewall-cmd --state        # Check firewall state
firewall-cmd --list-all     # List all rules
firewall-cmd --add-port=80/tcp --permanent  # Add rule
firewall-cmd --reload       # Reload rules
```

### SSH Configuration
```bash
# SSH client
ssh user@hostname           # Connect to remote host
ssh -i keyfile user@host    # Connect with specific key
scp file user@host:/path    # Copy file to remote host
rsync -av local/ user@host:remote/  # Sync directories

# SSH server configuration
sudo nano /etc/ssh/sshd_config
# Key settings:
# Port 22
# PermitRootLogin no
# PasswordAuthentication no
# PubkeyAuthentication yes

# SSH keys
ssh-keygen -t rsa -b 4096   # Generate key pair
ssh-copy-id user@host       # Copy public key to remote host
```

## Advanced Topics

### Shell Scripting Integration
```bash
# System information script
#!/bin/bash
echo "=== System Information ==="
echo "Hostname: $(hostname)"
echo "Uptime: $(uptime -p)"
echo "Load Average: $(uptime | awk -F'load average:' '{print $2}')"
echo "Memory Usage: $(free -h | awk '/^Mem:/ {print $3 "/" $2}')"
echo "Disk Usage: $(df -h / | awk 'NR==2 {print $3 "/" $2 " (" $5 ")"}')"
```

### Automation with Cron
```bash
# Edit crontab
crontab -e

# Cron examples
0 2 * * *     /path/to/backup.sh              # Daily at 2 AM
*/15 * * * *  /path/to/check_service.sh       # Every 15 minutes
0 0 1 * *     /path/to/monthly_cleanup.sh     # Monthly

# View cron jobs
crontab -l
cat /etc/crontab
```

### Environment Variables
```bash
# System-wide environment
/etc/environment            # System environment variables
/etc/profile               # System-wide shell initialization
/etc/bash.bashrc           # System-wide bash configuration

# User-specific environment
~/.bashrc                  # User bash configuration
~/.profile                 # User shell initialization
~/.bash_profile            # User bash login configuration

# Setting variables
export PATH=$PATH:/usr/local/bin
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk
```

---

*Linux is the foundation of modern computing. Master these fundamentals and you'll have the skills to work with any Unix-like system.*

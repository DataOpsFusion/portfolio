# Bash & Shell Scripting

Master the command line and shell scripting for automation, system administration, and daily productivity. From basic commands to advanced scripting patterns.

## Why Learn Bash?

Shell scripting is the foundation of system automation:
- **Automation**: Reduce repetitive tasks
- **System administration**: Manage servers and services
- **DevOps**: CI/CD pipelines and deployment scripts
- **Data processing**: Text manipulation and file operations
- **Universal**: Available on almost every Unix-like system

## Bash Fundamentals

### Basic Commands
```bash
# File and directory operations
ls -la                  # List files with details
cd /path/to/directory   # Change directory
pwd                     # Print working directory
mkdir -p dir1/dir2     # Create directories recursively
cp -r source dest      # Copy files/directories
mv source dest         # Move/rename files
rm -rf directory       # Remove files/directories
find . -name "*.txt"   # Find files by pattern

# File content operations
cat file.txt           # Display file content
less file.txt          # View file with pagination
head -n 10 file.txt    # Show first 10 lines
tail -f file.txt       # Follow file changes
grep "pattern" file.txt # Search for pattern
sed 's/old/new/g' file # Replace text
awk '{print $1}' file  # Print first column
```

### Command Chaining
```bash
# Pipes - pass output to next command
ls -la | grep "\.txt"              # List only .txt files
ps aux | grep "nginx" | awk '{print $2}'  # Get nginx PIDs

# Command sequencing
command1 && command2               # Run command2 if command1 succeeds
command1 || command2               # Run command2 if command1 fails
command1; command2                 # Run both commands regardless

# Input/Output redirection
command > file.txt                 # Redirect output to file
command >> file.txt                # Append output to file
command < input.txt                # Read input from file
command 2> error.log               # Redirect error output
command &> output.log              # Redirect both stdout and stderr
```

## Variables and Environment

### Variable Assignment
```bash
# Variable assignment (no spaces around =)
name="John Doe"
age=30
path="/usr/local/bin"

# Using variables
echo "Hello, $name"
echo "Age: ${age} years"
echo "Path: $path"

# Command substitution
current_date=$(date)
files_count=$(ls -1 | wc -l)
user_home=$(eval echo ~$USER)
```

### Environment Variables
```bash
# Common environment variables
echo $HOME          # Home directory
echo $PATH          # Executable search path
echo $USER          # Current user
echo $PWD           # Current directory
echo $SHELL         # Current shell

# Setting environment variables
export DATABASE_URL="postgresql://localhost:5432/mydb"
export PATH="$PATH:/usr/local/bin"

# Making variables persistent
echo 'export MY_VAR="value"' >> ~/.bashrc
source ~/.bashrc
```

### Special Variables
```bash
# Script parameters
echo $0             # Script name
echo $1 $2 $3       # First three arguments
echo $#             # Number of arguments
echo $*             # All arguments as single string
echo $@             # All arguments as separate strings
echo $?             # Exit status of last command
echo $$             # Process ID of current script
```

## Control Structures

### Conditional Statements
```bash
# If-then-else
if [ "$age" -gt 18 ]; then
    echo "Adult"
elif [ "$age" -eq 18 ]; then
    echo "Just turned adult"
else
    echo "Minor"
fi

# File/directory tests
if [ -f "file.txt" ]; then
    echo "File exists"
fi

if [ -d "directory" ]; then
    echo "Directory exists"
fi

if [ -r "file.txt" ]; then
    echo "File is readable"
fi

# String comparisons
if [ "$string1" = "$string2" ]; then
    echo "Strings are equal"
fi

if [ -z "$variable" ]; then
    echo "Variable is empty"
fi

# Numeric comparisons
if [ "$num1" -eq "$num2" ]; then echo "Equal"; fi
if [ "$num1" -lt "$num2" ]; then echo "Less than"; fi
if [ "$num1" -gt "$num2" ]; then echo "Greater than"; fi
```

### Loops
```bash
# For loop with list
for item in apple banana orange; do
    echo "Fruit: $item"
done

# For loop with range
for i in {1..10}; do
    echo "Number: $i"
done

# For loop with files
for file in *.txt; do
    echo "Processing: $file"
    # Process file here
done

# While loop
counter=1
while [ $counter -le 10 ]; do
    echo "Count: $counter"
    ((counter++))
done

# Until loop
until [ $counter -gt 20 ]; do
    echo "Count: $counter"
    ((counter++))
done

# Reading file line by line
while IFS= read -r line; do
    echo "Line: $line"
done < file.txt
```

### Case Statements
```bash
case "$1" in
    "start")
        echo "Starting service..."
        ;;
    "stop")
        echo "Stopping service..."
        ;;
    "restart")
        echo "Restarting service..."
        ;;
    *)
        echo "Usage: $0 {start|stop|restart}"
        exit 1
        ;;
esac
```

## Functions

### Function Definition
```bash
# Simple function
greet() {
    echo "Hello, $1!"
}

# Function with local variables
calculate_disk_usage() {
    local directory="$1"
    local usage=$(du -sh "$directory" 2>/dev/null | cut -f1)
    echo "Directory '$directory' uses: $usage"
}

# Function with return value
is_service_running() {
    local service_name="$1"
    if systemctl is-active --quiet "$service_name"; then
        return 0  # Success
    else
        return 1  # Failure
    fi
}

# Using functions
greet "World"
calculate_disk_usage "/var/log"

if is_service_running "nginx"; then
    echo "Nginx is running"
else
    echo "Nginx is not running"
fi
```

## Text Processing

### grep - Pattern Matching
```bash
# Basic pattern matching
grep "error" logfile.txt
grep -i "error" logfile.txt           # Case insensitive
grep -v "debug" logfile.txt           # Invert match (exclude)
grep -n "error" logfile.txt           # Show line numbers
grep -r "TODO" /project/src/          # Recursive search

# Regular expressions
grep "^Error" logfile.txt             # Lines starting with "Error"
grep "Error$" logfile.txt             # Lines ending with "Error"
grep "Error.*critical" logfile.txt    # Lines with "Error" followed by "critical"
grep -E "[0-9]{3}-[0-9]{3}-[0-9]{4}" contacts.txt  # Phone number pattern
```

### sed - Stream Editor
```bash
# Substitution
sed 's/old/new/' file.txt             # Replace first occurrence per line
sed 's/old/new/g' file.txt            # Replace all occurrences
sed 's/old/new/gi' file.txt           # Case insensitive replacement

# Line operations
sed -n '5p' file.txt                  # Print line 5
sed -n '5,10p' file.txt               # Print lines 5-10
sed '5d' file.txt                     # Delete line 5
sed '/pattern/d' file.txt             # Delete lines matching pattern

# In-place editing
sed -i 's/old/new/g' file.txt         # Edit file in place
sed -i.bak 's/old/new/g' file.txt     # Create backup before editing
```

### awk - Text Processing
```bash
# Field processing
awk '{print $1}' file.txt             # Print first field
awk '{print $1, $3}' file.txt         # Print first and third fields
awk -F',' '{print $2}' file.csv       # Use comma as field separator

# Patterns and conditions
awk '/error/ {print $0}' logfile.txt  # Print lines containing "error"
awk '$3 > 100 {print $1, $3}' data.txt # Print if third field > 100
awk 'NR > 1 {print}' file.txt         # Skip header line

# Built-in variables
awk '{print NR, $0}' file.txt         # Print line number and line
awk 'END {print NR}' file.txt         # Print total line count

# Calculations
awk '{sum += $1} END {print sum}' numbers.txt  # Sum first column
awk '{print $1, $2, $1*$2}' data.txt          # Calculate product
```

## Advanced Scripting Patterns

### Error Handling
```bash
#!/bin/bash
set -e  # Exit on any error
set -u  # Exit on undefined variable
set -o pipefail  # Exit on pipe failure

# Error handling function
handle_error() {
    echo "Error occurred in script $0 at line $1"
    exit 1
}
trap 'handle_error $LINENO' ERR

# Check if command exists
if ! command -v git &> /dev/null; then
    echo "Git is required but not installed."
    exit 1
fi

# Check file existence before processing
if [[ ! -f "config.txt" ]]; then
    echo "Config file not found!"
    exit 1
fi
```

### Logging
```bash
# Simple logging function
log() {
    local level="$1"
    shift
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] [$level] $*" | tee -a app.log
}

# Usage
log "INFO" "Starting application"
log "ERROR" "Failed to connect to database"
log "DEBUG" "Processing user ID: $user_id"

# Colored output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

log_error() {
    echo -e "${RED}[ERROR]${NC} $*" >&2
}

log_success() {
    echo -e "${GREEN}[SUCCESS]${NC} $*"
}

log_warning() {
    echo -e "${YELLOW}[WARNING]${NC} $*"
}
```

### Configuration Management
```bash
# Configuration file parsing
load_config() {
    local config_file="$1"
    
    if [[ -f "$config_file" ]]; then
        # Source the config file
        source "$config_file"
    else
        echo "Config file $config_file not found!"
        exit 1
    fi
}

# Default values
DB_HOST="localhost"
DB_PORT="5432"
DB_NAME="myapp"

# Load configuration
load_config "config.sh"

# Validate required configuration
required_vars=("DB_HOST" "DB_USER" "DB_PASSWORD")
for var in "${required_vars[@]}"; do
    if [[ -z "${!var:-}" ]]; then
        log_error "Required variable $var is not set"
        exit 1
    fi
done
```

## System Administration Scripts

### Service Management
```bash
#!/bin/bash
# Service management script

SERVICE_NAME="myapp"
PID_FILE="/var/run/${SERVICE_NAME}.pid"
LOG_FILE="/var/log/${SERVICE_NAME}.log"

start_service() {
    if [[ -f "$PID_FILE" ]]; then
        local pid=$(cat "$PID_FILE")
        if ps -p "$pid" > /dev/null 2>&1; then
            echo "Service is already running (PID: $pid)"
            return 1
        else
            rm -f "$PID_FILE"
        fi
    fi
    
    echo "Starting $SERVICE_NAME..."
    nohup /usr/local/bin/myapp > "$LOG_FILE" 2>&1 & 
    echo $! > "$PID_FILE"
    echo "Service started (PID: $!)"
}

stop_service() {
    if [[ -f "$PID_FILE" ]]; then
        local pid=$(cat "$PID_FILE")
        echo "Stopping $SERVICE_NAME (PID: $pid)..."
        kill "$pid"
        rm -f "$PID_FILE"
        echo "Service stopped"
    else
        echo "Service is not running"
    fi
}

case "$1" in
    start)   start_service ;;
    stop)    stop_service ;;
    restart) stop_service; sleep 2; start_service ;;
    *)       echo "Usage: $0 {start|stop|restart}" ;;
esac
```

### Backup Script
```bash
#!/bin/bash
# Automated backup script

BACKUP_SOURCE="/home/user/important_data"
BACKUP_DEST="/backup"
BACKUP_NAME="backup_$(date +%Y%m%d_%H%M%S)"
RETENTION_DAYS=7

create_backup() {
    local full_backup_path="$BACKUP_DEST/$BACKUP_NAME.tar.gz"
    
    echo "Creating backup: $full_backup_path"
    
    if tar -czf "$full_backup_path" -C "$(dirname "$BACKUP_SOURCE")" "$(basename "$BACKUP_SOURCE")"; then
        echo "Backup created successfully"
        return 0
    else
        echo "Backup failed"
        return 1
    fi
}

cleanup_old_backups() {
    echo "Cleaning up backups older than $RETENTION_DAYS days..."
    find "$BACKUP_DEST" -name "backup_*.tar.gz" -mtime +$RETENTION_DAYS -delete
    echo "Cleanup completed"
}

# Main execution
if [[ ! -d "$BACKUP_SOURCE" ]]; then
    log_error "Source directory does not exist: $BACKUP_SOURCE"
    exit 1
fi

if [[ ! -d "$BACKUP_DEST" ]]; then
    mkdir -p "$BACKUP_DEST"
fi

if create_backup; then
    cleanup_old_backups
    log_success "Backup process completed successfully"
else
    log_error "Backup process failed"
    exit 1
fi
```

### System Monitoring
```bash
#!/bin/bash
# System monitoring script

check_disk_usage() {
    echo "=== Disk Usage ==="
    df -h | awk '$5 > 80 {print "WARNING: " $1 " is " $5 " full"}'
}

check_memory_usage() {
    echo "=== Memory Usage ==="
    local mem_usage=$(free | awk 'NR==2{printf "%.1f", $3*100/$2}')
    if (( $(echo "$mem_usage > 80" | bc -l) )); then
        echo "WARNING: Memory usage is ${mem_usage}%"
    else
        echo "Memory usage: ${mem_usage}%"
    fi
}

check_cpu_load() {
    echo "=== CPU Load ==="
    local load_avg=$(uptime | awk -F'load average:' '{print $2}' | awk '{print $1}' | sed 's/,//')
    local cpu_cores=$(nproc)
    local load_percent=$(echo "scale=2; $load_avg / $cpu_cores * 100" | bc)
    
    if (( $(echo "$load_percent > 80" | bc -l) )); then
        echo "WARNING: CPU load is ${load_percent}%"
    else
        echo "CPU load: ${load_percent}%"
    fi
}

check_services() {
    echo "=== Service Status ==="
    local services=("nginx" "mysql" "redis")
    
    for service in "${services[@]}"; do
        if systemctl is-active --quiet "$service"; then
            echo "$service: Running"
        else
            echo "WARNING: $service is not running"
        fi
    done
}

# Generate report
echo "System Health Report - $(date)"
echo "=================================="
check_disk_usage
echo
check_memory_usage
echo
check_cpu_load
echo
check_services
```

---

*Bash scripting is the Swiss Army knife of system administration. Master these patterns and you'll be able to automate almost anything on Unix-like systems.*

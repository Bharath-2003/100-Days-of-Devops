# Shell Scripting for DevOps

## 🎯 Overview

Shell scripting is essential for DevOps automation. Bash scripts help automate repetitive tasks, manage systems, deploy applications, and orchestrate complex workflows. This guide covers everything from basic syntax to advanced scripting techniques used in production environments.

**Duration:** 1.5 weeks
**Time Investment:** 4-5 hours/day
**Difficulty:** Beginner to Advanced
**Prerequisites:** Basic Linux command line knowledge

---

## 📚 What You'll Learn

### Week 1: Shell Scripting Fundamentals
- Bash basics and syntax
- Variables and data types
- Control structures (if, case, loops)
- Functions and parameters
- Input/output and redirection

### Week 2: Advanced Scripting
- Error handling and debugging
- Text processing (sed, awk, grep)
- Process management
- System administration scripts
- Best practices and security

---

## 🎓 Important Concepts

### 1. Shell Script Basics

```bash
#!/bin/bash
# Shebang - specifies interpreter

# Script Structure
# 1. Shebang
# 2. Comments and documentation
# 3. Variable declarations
# 4. Function definitions
# 5. Main script logic
# 6. Exit with status code

# Comments
# Single line comment

: '
Multi-line comment
Can span multiple lines
'

# Variables
NAME="John"              # String variable
AGE=30                   # Integer variable
readonly PI=3.14159      # Read-only variable
declare -i COUNT=0       # Declare integer

# Variable expansion
echo "Hello, $NAME"      # Simple expansion
echo "Hello, ${NAME}"    # Explicit expansion
echo "Age: $((AGE + 1))" # Arithmetic expansion

# Command substitution
CURRENT_DATE=$(date)     # Modern syntax
CURRENT_DATE=`date`      # Old syntax (avoid)

# Special variables
$0    # Script name
$1-$9 # Positional parameters
$#    # Number of arguments
$@    # All arguments as separate words
$*    # All arguments as single word
$?    # Exit status of last command
$$    # Process ID of script
$!    # Process ID of last background command
```

**Basic Script Template:**
```bash
#!/bin/bash
#
# Script Name: example.sh
# Description: Brief description of what the script does
# Author: Your Name
# Date: 2024-01-15
# Version: 1.0
#
# Usage: ./example.sh [options] arguments
#

set -euo pipefail  # Exit on error, undefined vars, pipe failures
IFS=$'\n\t'        # Set Internal Field Separator

# Global variables
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly SCRIPT_NAME="$(basename "$0")"
readonly LOG_FILE="/var/log/${SCRIPT_NAME%.sh}.log"

# Functions
log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOG_FILE"
}

error() {
    log "ERROR: $*" >&2
    exit 1
}

usage() {
    cat << EOF
Usage: $SCRIPT_NAME [OPTIONS] ARGUMENTS

Description:
    Brief description of the script

Options:
    -h, --help      Show this help message
    -v, --verbose   Enable verbose output
    -d, --debug     Enable debug mode

Examples:
    $SCRIPT_NAME -v argument1
    $SCRIPT_NAME --debug argument1 argument2

EOF
    exit 0
}

# Main function
main() {
    log "Script started"
    
    # Your script logic here
    
    log "Script completed successfully"
}

# Parse command line arguments
while [[ $# -gt 0 ]]; do
    case $1 in
        -h|--help)
            usage
            ;;
        -v|--verbose)
            VERBOSE=true
            shift
            ;;
        -d|--debug)
            set -x  # Enable debug mode
            shift
            ;;
        *)
            break
            ;;
    esac
done

# Run main function
main "$@"
```

---

### 2. Control Structures

```bash
# IF Statement
if [ condition ]; then
    # commands
elif [ condition ]; then
    # commands
else
    # commands
fi

# Test operators
# File tests
-e file    # File exists
-f file    # Regular file
-d file    # Directory
-r file    # Readable
-w file    # Writable
-x file    # Executable
-s file    # File size > 0

# String tests
-z string  # String is empty
-n string  # String is not empty
str1 = str2   # Strings are equal
str1 != str2  # Strings are not equal

# Numeric tests
num1 -eq num2  # Equal
num1 -ne num2  # Not equal
num1 -lt num2  # Less than
num1 -le num2  # Less than or equal
num1 -gt num2  # Greater than
num1 -ge num2  # Greater than or equal

# Examples
if [ -f "/etc/passwd" ]; then
    echo "File exists"
fi

if [ "$USER" = "root" ]; then
    echo "Running as root"
fi

if [ $AGE -ge 18 ]; then
    echo "Adult"
fi

# Modern test syntax [[ ]]
if [[ -f file && -r file ]]; then
    echo "File exists and is readable"
fi

if [[ $NAME =~ ^[A-Z] ]]; then
    echo "Name starts with uppercase"
fi

# CASE Statement
case $variable in
    pattern1)
        commands
        ;;
    pattern2|pattern3)
        commands
        ;;
    *)
        default commands
        ;;
esac

# Example
case $1 in
    start)
        echo "Starting service..."
        ;;
    stop)
        echo "Stopping service..."
        ;;
    restart)
        echo "Restarting service..."
        ;;
    *)
        echo "Usage: $0 {start|stop|restart}"
        exit 1
        ;;
esac

# FOR Loop
for var in list; do
    commands
done

# Examples
for i in 1 2 3 4 5; do
    echo "Number: $i"
done

for file in *.txt; do
    echo "Processing $file"
done

for i in {1..10}; do
    echo "Count: $i"
done

for ((i=0; i<10; i++)); do
    echo "Index: $i"
done

# WHILE Loop
while [ condition ]; do
    commands
done

# Example
count=0
while [ $count -lt 5 ]; do
    echo "Count: $count"
    ((count++))
done

# Read file line by line
while IFS= read -r line; do
    echo "Line: $line"
done < file.txt

# UNTIL Loop
until [ condition ]; do
    commands
done

# Example
count=0
until [ $count -ge 5 ]; do
    echo "Count: $count"
    ((count++))
done
```

---

### 3. Functions

```bash
# Function definition
function_name() {
    # Function body
    local var="value"  # Local variable
    echo "Result"
    return 0           # Return status (0-255)
}

# Call function
function_name

# Function with parameters
greet() {
    local name=$1
    local age=$2
    echo "Hello, $name! You are $age years old."
}

greet "John" 30

# Function with return value
add() {
    local sum=$(($1 + $2))
    echo $sum
}

result=$(add 5 3)
echo "Sum: $result"

# Function with error handling
safe_mkdir() {
    local dir=$1
    
    if [ -z "$dir" ]; then
        echo "Error: Directory name required" >&2
        return 1
    fi
    
    if [ -d "$dir" ]; then
        echo "Directory already exists: $dir" >&2
        return 2
    fi
    
    mkdir -p "$dir" || {
        echo "Error: Failed to create directory: $dir" >&2
        return 3
    }
    
    echo "Directory created: $dir"
    return 0
}

# Advanced function example
backup_file() {
    local file=$1
    local backup_dir=${2:-/backup}
    local timestamp=$(date +%Y%m%d_%H%M%S)
    
    # Validate input
    [[ -z "$file" ]] && { echo "Error: File required" >&2; return 1; }
    [[ ! -f "$file" ]] && { echo "Error: File not found: $file" >&2; return 2; }
    
    # Create backup directory
    mkdir -p "$backup_dir" || return 3
    
    # Create backup
    local backup_file="${backup_dir}/$(basename "$file").${timestamp}.bak"
    cp "$file" "$backup_file" || return 4
    
    echo "Backup created: $backup_file"
    return 0
}
```

---

### 4. Input/Output and Redirection

```bash
# Standard streams
# 0 - stdin  (standard input)
# 1 - stdout (standard output)
# 2 - stderr (standard error)

# Output redirection
echo "text" > file.txt      # Overwrite file
echo "text" >> file.txt     # Append to file
command 2> error.log        # Redirect stderr
command > output.log 2>&1   # Redirect both stdout and stderr
command &> output.log       # Same as above (bash 4+)

# Input redirection
command < input.txt         # Read from file
command << EOF              # Here document
line 1
line 2
EOF

command <<< "string"        # Here string

# Pipes
command1 | command2         # Pipe stdout
command1 |& command2        # Pipe stdout and stderr

# Read user input
read -p "Enter name: " name
read -sp "Enter password: " password  # Silent input
read -t 10 -p "Quick! " input        # Timeout
read -a array -p "Enter items: "     # Read into array

# Reading files
while IFS= read -r line; do
    echo "$line"
done < file.txt

# Process substitution
diff <(command1) <(command2)
command > >(tee file.txt)

# Examples
# Log both to file and console
./script.sh 2>&1 | tee script.log

# Separate stdout and stderr
./script.sh > output.log 2> error.log

# Discard output
command > /dev/null 2>&1
```

---

### 5. Text Processing

```bash
# GREP - Search text
grep pattern file           # Search for pattern
grep -i pattern file        # Case insensitive
grep -r pattern dir/        # Recursive search
grep -v pattern file        # Invert match
grep -n pattern file        # Show line numbers
grep -c pattern file        # Count matches
grep -E 'regex' file        # Extended regex
grep -A 3 pattern file      # Show 3 lines after
grep -B 3 pattern file      # Show 3 lines before
grep -C 3 pattern file      # Show 3 lines around

# SED - Stream editor
sed 's/old/new/' file       # Replace first occurrence
sed 's/old/new/g' file      # Replace all occurrences
sed 's/old/new/gi' file     # Case insensitive replace
sed -i 's/old/new/g' file   # In-place edit
sed '1d' file               # Delete first line
sed '1,5d' file             # Delete lines 1-5
sed -n '10,20p' file        # Print lines 10-20
sed '/pattern/d' file       # Delete lines matching pattern

# AWK - Pattern scanning and processing
awk '{print $1}' file       # Print first column
awk '{print $1,$3}' file    # Print columns 1 and 3
awk -F: '{print $1}' file   # Custom field separator
awk '$3 > 100' file         # Filter rows
awk '{sum+=$1} END {print sum}' file  # Sum column

# Examples
# Extract IP addresses
grep -oE '\b([0-9]{1,3}\.){3}[0-9]{1,3}\b' file

# Replace in multiple files
find . -name "*.txt" -exec sed -i 's/old/new/g' {} +

# Process CSV
awk -F',' '{print $1,$3}' data.csv

# Count unique values
awk '{print $1}' file | sort | uniq -c

# Calculate average
awk '{sum+=$1; count++} END {print sum/count}' file
```

---

### 6. Error Handling and Debugging

```bash
# Exit on error
set -e              # Exit on any error
set -u              # Exit on undefined variable
set -o pipefail     # Exit on pipe failure
set -x              # Print commands (debug)

# Combine options
set -euo pipefail

# Error handling
command || {
    echo "Command failed" >&2
    exit 1
}

# Trap errors
trap 'echo "Error on line $LINENO"' ERR

# Cleanup on exit
cleanup() {
    echo "Cleaning up..."
    rm -f /tmp/tempfile
}
trap cleanup EXIT

# Comprehensive error handling
#!/bin/bash
set -euo pipefail

# Error handler
error_handler() {
    local line=$1
    local command=$2
    echo "Error on line $line: $command" >&2
    exit 1
}

trap 'error_handler $LINENO "$BASH_COMMAND"' ERR

# Cleanup handler
cleanup() {
    local exit_code=$?
    echo "Cleaning up... (exit code: $exit_code)"
    # Cleanup code here
    exit $exit_code
}

trap cleanup EXIT INT TERM

# Debugging techniques
# 1. Enable debug mode
bash -x script.sh

# 2. Debug specific section
set -x
# commands to debug
set +x

# 3. Print variables
echo "DEBUG: var=$var" >&2

# 4. Use logger
logger -t script "Debug message"

# 5. Conditional debugging
DEBUG=${DEBUG:-false}
debug() {
    [[ $DEBUG == true ]] && echo "DEBUG: $*" >&2
}

debug "Variable value: $var"
```

---

### 7. Advanced Scripting Patterns

```bash
# Argument parsing
parse_args() {
    while [[ $# -gt 0 ]]; do
        case $1 in
            -h|--help)
                usage
                exit 0
                ;;
            -v|--verbose)
                VERBOSE=true
                shift
                ;;
            -f|--file)
                FILE="$2"
                shift 2
                ;;
            -o|--output)
                OUTPUT="$2"
                shift 2
                ;;
            --)
                shift
                break
                ;;
            -*)
                echo "Unknown option: $1" >&2
                exit 1
                ;;
            *)
                break
                ;;
        esac
    done
    
    # Remaining arguments
    ARGS=("$@")
}

# Configuration file
load_config() {
    local config_file=${1:-config.conf}
    
    if [[ -f "$config_file" ]]; then
        # Source config file
        source "$config_file"
    else
        echo "Config file not found: $config_file" >&2
        return 1
    fi
}

# Locking mechanism
acquire_lock() {
    local lockfile=${1:-/var/lock/script.lock}
    local timeout=${2:-300}
    local elapsed=0
    
    while [[ -f "$lockfile" ]]; do
        if [[ $elapsed -ge $timeout ]]; then
            echo "Timeout waiting for lock" >&2
            return 1
        fi
        sleep 1
        ((elapsed++))
    done
    
    # Create lock file
    echo $$ > "$lockfile"
    
    # Remove lock on exit
    trap "rm -f $lockfile" EXIT
}

# Parallel execution
parallel_execute() {
    local max_jobs=${1:-4}
    shift
    local commands=("$@")
    
    for cmd in "${commands[@]}"; do
        # Wait if max jobs reached
        while [[ $(jobs -r | wc -l) -ge $max_jobs ]]; do
            sleep 0.1
        done
        
        # Execute in background
        eval "$cmd" &
    done
    
    # Wait for all jobs
    wait
}

# Retry mechanism
retry() {
    local max_attempts=${1:-3}
    local delay=${2:-5}
    shift 2
    local command=("$@")
    local attempt=1
    
    while [[ $attempt -le $max_attempts ]]; do
        if "${command[@]}"; then
            return 0
        fi
        
        echo "Attempt $attempt failed. Retrying in ${delay}s..." >&2
        sleep "$delay"
        ((attempt++))
    done
    
    echo "All $max_attempts attempts failed" >&2
    return 1
}

# Progress bar
progress_bar() {
    local current=$1
    local total=$2
    local width=50
    
    local percent=$((current * 100 / total))
    local filled=$((width * current / total))
    local empty=$((width - filled))
    
    printf "\r["
    printf "%${filled}s" | tr ' ' '='
    printf "%${empty}s" | tr ' ' ' '
    printf "] %3d%%" "$percent"
    
    [[ $current -eq $total ]] && echo
}

# Example usage
for i in {1..100}; do
    progress_bar $i 100
    sleep 0.1
done
```

---

## 🔨 Hands-On Exercises

### Exercise 1: System Monitoring Script (60 minutes)

**Objective:** Create comprehensive system monitoring script

```bash
#!/bin/bash
#
# System Monitor Script
# Monitors CPU, memory, disk, and network
#

set -euo pipefail

# Configuration
readonly THRESHOLD_CPU=80
readonly THRESHOLD_MEM=90
readonly THRESHOLD_DISK=85
readonly LOG_FILE="/var/log/system-monitor.log"
readonly ALERT_EMAIL="admin@example.com"

# Colors
readonly RED='\033[0;31m'
readonly YELLOW='\033[1;33m'
readonly GREEN='\033[0;32m'
readonly NC='\033[0m' # No Color

# Logging
log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOG_FILE"
}

# Check CPU usage
check_cpu() {
    local cpu_usage=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1)
    local cpu_int=${cpu_usage%.*}
    
    echo -e "\n${GREEN}CPU Usage:${NC} ${cpu_usage}%"
    
    if [[ $cpu_int -gt $THRESHOLD_CPU ]]; then
        echo -e "${RED}WARNING: High CPU usage!${NC}"
        log "WARNING: CPU usage is ${cpu_usage}%"
        return 1
    fi
    
    return 0
}

# Check memory usage
check_memory() {
    local mem_info=$(free | grep Mem)
    local total=$(echo $mem_info | awk '{print $2}')
    local used=$(echo $mem_info | awk '{print $3}')
    local percent=$((used * 100 / total))
    
    echo -e "\n${GREEN}Memory Usage:${NC} ${percent}%"
    echo "Total: $(numfmt --to=iec-i --suffix=B $((total * 1024)))"
    echo "Used: $(numfmt --to=iec-i --suffix=B $((used * 1024)))"
    
    if [[ $percent -gt $THRESHOLD_MEM ]]; then
        echo -e "${RED}WARNING: High memory usage!${NC}"
        log "WARNING: Memory usage is ${percent}%"
        return 1
    fi
    
    return 0
}

# Check disk usage
check_disk() {
    echo -e "\n${GREEN}Disk Usage:${NC}"
    
    local alert=0
    while IFS= read -r line; do
        local usage=$(echo "$line" | awk '{print $5}' | sed 's/%//')
        local mount=$(echo "$line" | awk '{print $6}')
        
        if [[ $usage -gt $THRESHOLD_DISK ]]; then
            echo -e "${RED}$line${NC}"
            log "WARNING: Disk usage on $mount is ${usage}%"
            alert=1
        else
            echo "$line"
        fi
    done < <(df -h | grep -v "tmpfs\|loop" | tail -n +2)
    
    return $alert
}

# Check network
check_network() {
    echo -e "\n${GREEN}Network Statistics:${NC}"
    
    # Get network interfaces
    for interface in $(ls /sys/class/net/ | grep -v lo); do
        if [[ -f "/sys/class/net/$interface/statistics/rx_bytes" ]]; then
            local rx_bytes=$(cat /sys/class/net/$interface/statistics/rx_bytes)
            local tx_bytes=$(cat /sys/class/net/$interface/statistics/tx_bytes)
            
            echo "$interface:"
            echo "  RX: $(numfmt --to=iec-i --suffix=B $rx_bytes)"
            echo "  TX: $(numfmt --to=iec-i --suffix=B $tx_bytes)"
        fi
    done
}

# Check running processes
check_processes() {
    echo -e "\n${GREEN}Top 5 CPU Processes:${NC}"
    ps aux --sort=-%cpu | head -6 | tail -5
    
    echo -e "\n${GREEN}Top 5 Memory Processes:${NC}"
    ps aux --sort=-%mem | head -6 | tail -5
}

# Check system load
check_load() {
    local load=$(uptime | awk -F'load average:' '{print $2}')
    local cores=$(nproc)
    
    echo -e "\n${GREEN}System Load:${NC}$load"
    echo "CPU Cores: $cores"
}

# Send alert
send_alert() {
    local message=$1
    
    if command -v mail &> /dev/null; then
        echo "$message" | mail -s "System Alert" "$ALERT_EMAIL"
    fi
}

# Main function
main() {
    echo "======================================"
    echo "System Monitor - $(date)"
    echo "======================================"
    
    local alerts=0
    
    check_cpu || ((alerts++))
    check_memory || ((alerts++))
    check_disk || ((alerts++))
    check_network
    check_processes
    check_load
    
    echo -e "\n======================================"
    
    if [[ $alerts -gt 0 ]]; then
        echo -e "${RED}Total Alerts: $alerts${NC}"
        send_alert "System monitoring detected $alerts issues"
    else
        echo -e "${GREEN}All systems normal${NC}"
    fi
}

# Run if executed directly
if [[ "${BASH_SOURCE[0]}" == "${0}" ]]; then
    main "$@"
fi
```

**Challenge:**
Enhance the script:
1. Add email notifications
2. Create historical graphs
3. Monitor specific services
4. Add custom thresholds per mount point

---

### Exercise 2: Backup Automation Script (60 minutes)

**Objective:** Create automated backup solution

```bash
#!/bin/bash
#
# Backup Automation Script
# Performs incremental backups with rotation
#

set -euo pipefail

# Configuration
readonly BACKUP_SOURCE="/var/www"
readonly BACKUP_DEST="/backup"
readonly RETENTION_DAYS=7
readonly LOG_FILE="/var/log/backup.log"
readonly LOCK_FILE="/var/run/backup.lock"

# Logging
log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOG_FILE"
}

error() {
    log "ERROR: $*" >&2
    exit 1
}

# Acquire lock
acquire_lock() {
    if [[ -f "$LOCK_FILE" ]]; then
        local pid=$(cat "$LOCK_FILE")
        if kill -0 "$pid" 2>/dev/null; then
            error "Backup already running (PID: $pid)"
        else
            log "Removing stale lock file"
            rm -f "$LOCK_FILE"
        fi
    fi
    
    echo $$ > "$LOCK_FILE"
    trap "rm -f $LOCK_FILE" EXIT
}

# Check prerequisites
check_prerequisites() {
    # Check if source exists
    [[ ! -d "$BACKUP_SOURCE" ]] && error "Source directory not found: $BACKUP_SOURCE"
    
    # Check if destination exists
    [[ ! -d "$BACKUP_DEST" ]] && mkdir -p "$BACKUP_DEST"
    
    # Check available space
    local required_space=$(du -sb "$BACKUP_SOURCE" | awk '{print $1}')
    local available_space=$(df -B1 "$BACKUP_DEST" | tail -1 | awk '{print $4}')
    
    if [[ $available_space -lt $required_space ]]; then
        error "Insufficient space. Required: $(numfmt --to=iec-i $required_space), Available: $(numfmt --to=iec-i $available_space)"
    fi
    
    log "Prerequisites check passed"
}

# Create backup
create_backup() {
    local timestamp=$(date +%Y%m%d_%H%M%S)
    local backup_name="backup_${timestamp}"
    local backup_path="${BACKUP_DEST}/${backup_name}"
    
    log "Starting backup: $backup_name"
    
    # Find latest backup for incremental
    local latest_backup=$(find "$BACKUP_DEST" -maxdepth 1 -type d -name "backup_*" | sort -r | head -1)
    
    # Create backup
    if [[ -n "$latest_backup" ]]; then
        log "Creating incremental backup (base: $(basename "$latest_backup"))"
        rsync -av --delete \
            --link-dest="$latest_backup" \
            "$BACKUP_SOURCE/" \
            "$backup_path/" 2>&1 | tee -a "$LOG_FILE"
    else
        log "Creating full backup"
        rsync -av \
            "$BACKUP_SOURCE/" \
            "$backup_path/" 2>&1 | tee -a "$LOG_FILE"
    fi
    
    # Create checksum
    log "Creating checksum"
    find "$backup_path" -type f -exec md5sum {} \; > "${backup_path}.md5"
    
    # Compress backup
    log "Compressing backup"
    tar -czf "${backup_path}.tar.gz" -C "$BACKUP_DEST" "$backup_name"
    
    # Remove uncompressed backup
    rm -rf "$backup_path"
    
    local backup_size=$(du -h "${backup_path}.tar.gz" | awk '{print $1}')
    log "Backup completed: ${backup_name}.tar.gz (Size: $backup_size)"
    
    echo "${backup_path}.tar.gz"
}

# Rotate old backups
rotate_backups() {
    log "Rotating old backups (retention: $RETENTION_DAYS days)"
    
    local count=0
    while IFS= read -r backup; do
        log "Removing old backup: $(basename "$backup")"
        rm -f "$backup" "${backup%.tar.gz}.md5"
        ((count++))
    done < <(find "$BACKUP_DEST" -name "backup_*.tar.gz" -mtime +$RETENTION_DAYS)
    
    log "Removed $count old backup(s)"
}

# Verify backup
verify_backup() {
    local backup_file=$1
    
    log "Verifying backup: $(basename "$backup_file")"
    
    # Check if file exists
    [[ ! -f "$backup_file" ]] && error "Backup file not found: $backup_file"
    
    # Test archive integrity
    if tar -tzf "$backup_file" > /dev/null 2>&1; then
        log "Backup verification successful"
        return 0
    else
        error "Backup verification failed"
    fi
}

# Send notification
send_notification() {
    local status=$1
    local message=$2
    
    # Email notification
    if command -v mail &> /dev/null; then
        echo "$message" | mail -s "Backup $status" "admin@example.com"
    fi
    
    # Slack notification (if webhook configured)
    if [[ -n "${SLACK_WEBHOOK:-}" ]]; then
        curl -X POST "$SLACK_WEBHOOK" \
            -H 'Content-Type: application/json' \
            -d "{\"text\": \"Backup $status: $message\"}"
    fi
}

# Main function
main() {
    log "=== Backup Started ==="
    
    # Acquire lock
    acquire_lock
    
    # Check prerequisites
    check_prerequisites
    
    # Create backup
    local backup_file
    if backup_file=$(create_backup); then
        # Verify backup
        verify_backup "$backup_file"
        
        # Rotate old backups
        rotate_backups
        
        log "=== Backup Completed Successfully ==="
        send_notification "Success" "Backup completed: $(basename "$backup_file")"
    else
        error "Backup failed"
    fi
}

# Run main function
main "$@"
```

**Challenge:**
Add features:
1. Remote backup to S3/Azure
2. Database dumps
3. Encryption
4. Parallel backups

---

### Exercise 3: Deployment Automation Script (90 minutes)

**Objective:** Create zero-downtime deployment script

```bash
#!/bin/bash
#
# Deployment Automation Script
# Blue-Green deployment with rollback
#

set -euo pipefail

# Configuration
readonly APP_NAME="myapp"
readonly APP_DIR="/var/www/${APP_NAME}"
readonly BLUE_DIR="${APP_DIR}/blue"
readonly GREEN_DIR="${APP_DIR}/green"
readonly CURRENT_LINK="${APP_DIR}/current"
readonly BACKUP_DIR="/backup/deployments"
readonly LOG_FILE="/var/log/deploy.log"

# Colors
readonly RED='\033[0;31m'
readonly GREEN='\033[0;32m'
readonly YELLOW='\033[1;33m'
readonly NC='\033[0m'

# Logging
log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOG_FILE"
}

error() {
    echo -e "${RED}ERROR: $*${NC}" >&2
    log "ERROR: $*"
    exit 1
}

success() {
    echo -e "${GREEN}$*${NC}"
    log "$*"
}

warning() {
    echo -e "${YELLOW}WARNING: $*${NC}"
    log "WARNING: $*"
}

# Get current environment
get_current_env() {
    if [[ -L "$CURRENT_LINK" ]]; then
        local target=$(readlink "$CURRENT_LINK")
        basename "$target"
    else
        echo "none"
    fi
}

# Get target environment
get_target_env() {
    local current=$(get_current_env)
    
    if [[ "$current" == "blue" ]]; then
        echo "green"
    else
        echo "blue"
    fi
}

# Pre-deployment checks
pre_deploy_checks() {
    log "Running pre-deployment checks..."
    
    # Check if running as correct user
    if [[ $EUID -ne 0 ]]; then
        error "This script must be run as root"
    fi
    
    # Check required directories
    mkdir -p "$BLUE_DIR" "$GREEN_DIR" "$BACKUP_DIR"
    
    # Check disk space
    local available=$(df -BG "$APP_DIR" | tail -1 | awk '{print $4}' | sed 's/G//')
    if [[ $available -lt 5 ]]; then
        error "Insufficient disk space (${available}GB available)"
    fi
    
    # Check if application is running
    if ! systemctl is-active --quiet "${APP_NAME}"; then
        warning "Application is not running"
    fi
    
    success "Pre-deployment checks passed"
}

# Deploy application
deploy() {
    local version=$1
    local target_env=$(get_target_env)
    local target_dir="${APP_DIR}/${target_env}"
    
    log "Deploying version $version to $target_env environment"
    
    # Backup current deployment
    backup_deployment
    
    # Download/copy new version
    log "Downloading application version $version"
    # Simulate download
    sleep 2
    
    # Extract to target directory
    log "Extracting to $target_dir"
    # Simulate extraction
    sleep 1
    
    # Install dependencies
    log "Installing dependencies"
    cd "$target_dir"
    # npm install or pip install, etc.
    sleep 2
    
    # Run database migrations
    log "Running database migrations"
    # ./manage.py migrate or similar
    sleep 1
    
    # Build assets
    log "Building assets"
    # npm run build or similar
    sleep 2
    
    success "Deployment to $target_env completed"
}

# Health check
health_check() {
    local env=$1
    local url="http://localhost:8080"
    local max_attempts=30
    local attempt=1
    
    log "Running health check for $env environment"
    
    while [[ $attempt -le $max_attempts ]]; do
        if curl -sf "$url/health" > /dev/null 2>&1; then
            success "Health check passed"
            return 0
        fi
        
        log "Health check attempt $attempt/$max_attempts failed"
        sleep 2
        ((attempt++))
    done
    
    error "Health check failed after $max_attempts attempts"
}

# Switch traffic
switch_traffic() {
    local target_env=$1
    local target_dir="${APP_DIR}/${target_env}"
    
    log "Switching traffic to $target_env"
    
    # Update symlink
    ln -sfn "$target_dir" "${CURRENT_LINK}.tmp"
    mv -f "${CURRENT_LINK}.tmp" "$CURRENT_LINK"
    
    # Reload web server
    systemctl reload nginx
    
    success "Traffic switched to $target_env"
}

# Backup deployment
backup_deployment() {
    local current_env=$(get_current_env)
    
    if [[ "$current_env" == "none" ]]; then
        log "No current deployment to backup"
        return 0
    fi
    
    local timestamp=$(date +%Y%m%d_%H%M%S)
    local backup_file="${BACKUP_DIR}/${current_env}_${timestamp}.tar.gz"
    
    log "Backing up current deployment"
    tar -czf "$backup_file" -C "$APP_DIR" "$current_env"
    
    success "Backup created: $backup_file"
}

# Rollback
rollback() {
    local current_env=$(get_current_env)
    local previous_env
    
    if [[ "$current_env" == "blue" ]]; then
        previous_env="green"
    else
        previous_env="blue"
    fi
    
    warning "Rolling back to $previous_env"
    
    # Switch traffic back
    switch_traffic "$previous_env"
    
    # Verify rollback
    if health_check "$previous_env"; then
        success "Rollback completed successfully"
    else
        error "Rollback failed"
    fi
}

# Cleanup old deployments
cleanup() {
    log "Cleaning up old backups"
    
    find "$BACKUP_DIR" -name "*.tar.gz" -mtime +7 -delete
    
    success "Cleanup completed"
}

# Main deployment flow
main() {
    local version=${1:-latest}
    
    log "=== Starting Deployment ==="
    log "Version: $version"
    
    # Pre-deployment checks
    pre_deploy_checks
    
    # Get environments
    local current_env=$(get_current_env)
    local target_env=$(get_target_env)
    
    log "Current environment: $current_env"
    log "Target environment: $target_env"
    
    # Deploy to target environment
    if deploy "$version"; then
        # Health check
        if health_check "$target_env"; then
            # Switch traffic
            switch_traffic "$target_env"
            
            # Final health check
            sleep 5
            if health_check "$target_env"; then
                success "=== Deployment Completed Successfully ==="
                cleanup
            else
                error "Post-switch health check failed"
                rollback
            fi
        else
            error "Health check failed"
            rollback
        fi
    else
        error "Deployment failed"
    fi
}

# Handle script arguments
case "${1:-deploy}" in
    deploy)
        main "${2:-latest}"
        ;;
    rollback)
        rollback
        ;;
    status)
        echo "Current environment: $(get_current_env)"
        ;;
    *)
        echo "Usage: $0 {deploy|rollback|status} [version]"
        exit 1
        ;;
esac
```

**Challenge:**
Enhance deployment:
1. Add canary deployment
2. Implement A/B testing
3. Add smoke tests
4. Integrate with CI/CD

---

## 🎯 Practice Projects

### Project 1: Complete DevOps Toolkit
Build comprehensive script collection:
- System monitoring and alerting
- Automated backups
- Log rotation and analysis
- Service management
- Security hardening

### Project 2: CI/CD Pipeline Scripts
Create deployment automation:
- Build automation
- Testing framework
- Deployment strategies
- Rollback mechanisms
- Environment management

### Project 3: Infrastructure Automation
Automate infrastructure tasks:
- Server provisioning
- Configuration management
- Network setup
- Security policies
- Disaster recovery

### Project 4: Monitoring and Alerting
Build monitoring solution:
- Metrics collection
- Log aggregation
- Alert management
- Dashboard generation
- Report automation

---

## 📝 Best Practices

### 1. Script Structure
- Use shebang (#!/bin/bash)
- Add documentation
- Set error handling (set -euo pipefail)
- Use functions
- Validate inputs
- Provide usage information

### 2. Error Handling
- Check command exit codes
- Use trap for cleanup
- Log errors appropriately
- Provide meaningful error messages
- Implement retry logic
- Handle edge cases

### 3. Security
- Avoid hardcoded credentials
- Use proper file permissions
- Validate user input
- Sanitize variables
- Use quotes around variables
- Avoid eval when possible

### 4. Performance
- Avoid unnecessary subshells
- Use built-in commands
- Minimize external commands
- Implement caching
- Use parallel execution
- Profile scripts

---

## ✅ Completion Checklist

### Week 1
- [ ] Master bash syntax
- [ ] Understand control structures
- [ ] Write functions
- [ ] Handle input/output
- [ ] Process text with sed/awk

### Week 2
- [ ] Implement error handling
- [ ] Debug scripts effectively
- [ ] Apply best practices
- [ ] Complete automation projects
- [ ] Document scripts

---

**Next:** Continue with specialized topics or return to [Main README](../../README.md)
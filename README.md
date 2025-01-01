
#  System Monitoring Script for macOS

## Overview
This script monitors the system's CPU, memory, disk, and network usage on macOS. It logs the data into a file, generates alerts when thresholds are exceeded, and displays the latest logs for review.

## Features
- **Dynamic Network Detection**: Automatically detects the primary network interface.
- **Custom Thresholds**: Set specific thresholds for CPU, memory, and disk usage.
- **Friendly Logging**: Logs are stored in an organized format with separators and alerts.
- **Tailored Alerts**: Alerts are triggered when usage exceeds the defined thresholds.

## Usage
1. Clone or download this script.
2. Make the script executable:
   ```bash
   chmod +x system_monitor.sh
   ```
3. Run the script:
   ```bash
   ./system_monitor.sh
   ```
4. Check the log file for details:
   ```bash
   cat unique_system_monitor.log
   ```

## Script Content
```bash
#!/bin/bash

# System Monitoring Script for macOS
# -----------------------------------------
# Logs system metrics, alerts based on thresholds, and displays results in a user-friendly way.

# Configuration
LOG_FILE="system_monitor.log"
CPU_THRESHOLD=75  # CPU usage alert threshold
MEM_THRESHOLD=70  # Memory usage alert threshold
DISK_THRESHOLD=85 # Disk usage alert threshold

# Detect the primary network interface dynamically
NET_INTERFACE=$(route get default | grep 'interface:' | awk '{print $2}')

# Function: Log Message
log_message() {
    echo "$1" >> "$LOG_FILE"
}

# Function: Separator
log_separator() {
    log_message "-------------------------------------"
}

# Gather Metrics

# CPU Usage
CPU_USAGE=$(top -l 1 | awk '/CPU usage/ {print $3}' | sed 's/%//')

# Memory Usage
MEM_USED=$(vm_stat | awk '/Pages active/ {print $3}' | sed 's/\.//')
MEM_FREE=$(vm_stat | awk '/Pages free/ {print $3}' | sed 's/\.//')
MEM_TOTAL=$((MEM_USED + MEM_FREE))
MEMORY_USAGE=$(echo "scale=2; $MEM_USED / $MEM_TOTAL * 100" | bc)

# Disk Usage
DISK_USAGE=$(df / | tail -1 | awk '{print $5}' | sed 's/%//')

# Network Statistics
NETWORK_RX=$(netstat -ib | grep "$NET_INTERFACE" | awk '{print $7}' | tail -1)
NETWORK_TX=$(netstat -ib | grep "$NET_INTERFACE" | awk '{print $10}' | tail -1)

# Log Data
log_separator
log_message "Unique System Monitoring Report: $(date)"
log_message "CPU Usage: $CPU_USAGE%"
log_message "Memory Usage: $MEMORY_USAGE%"
log_message "Disk Usage: $DISK_USAGE%"
log_message "Network Received: $NETWORK_RX bytes"
log_message "Network Transmitted: $NETWORK_TX bytes"

# Alerts
if (( $(echo "$CPU_USAGE > $CPU_THRESHOLD" | bc -l) )); then
    log_message "ALERT: CPU usage has exceeded $CPU_THRESHOLD%!"
fi

if (( $(echo "$MEMORY_USAGE > $MEM_THRESHOLD" | bc -l) )); then
    log_message "ALERT: Memory usage has exceeded $MEM_THRESHOLD%!"
fi

if (( DISK_USAGE > DISK_THRESHOLD )); then
    log_message "ALERT: Disk usage has exceeded $DISK_THRESHOLD%!"
fi

# Display Summary
log_message "Log captured successfully!"
log_separator
tail -n 10 "$LOG_FILE"

# End of Script
```

## Requirements
- macOS
- Basic shell utilities (`top`, `vm_stat`, `df`, `netstat`, `awk`, `sed`, `bc`)

## Notes
- Ensure the script has execute permissions before running.
- Adjust thresholds as needed to match your system's performance expectations.


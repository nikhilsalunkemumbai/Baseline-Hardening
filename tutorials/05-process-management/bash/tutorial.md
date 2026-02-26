# Bash Tutorial: Process Management and Automation

## Introduction

Bash, combined with standard Unix-like utilities, provides powerful and direct control over system processes. This tutorial will guide you through the essential Bash commands for listing, monitoring, and managing processes, enabling effective system administration and automation, while adhering to our principles of minimal dependencies and cross-platform compatibility across Linux and macOS.

## Framework Alignment

This tutorial on "**Process Management and Automation**" using **Bash** demonstrates the foundational skills necessary to implement components of a "Cross-Platform Baseline Hardening & Auditing Framework." The techniques learned here for managing and auditing running processes are directly applicable to auditing systems against defined security baselines and ensuring compliance.


## Core Bash Utilities for Process Management

*   **`ps`**: Reports a snapshot of the current processes. Highly configurable.
    *   `aux`: Display all processes, owned by any user, with full format (BSD syntax, common).
    *   `ef`: Full format listing (SysV syntax, common).
    *   `-e`: Select all processes.
    *   `-o <format>`: User-defined format (e.g., `pid,ppid,user,cmd,%cpu,%mem`).
*   **`top`** / **`htop`**: Display Linux processes (or tasks) dynamically. (`htop` is an enhanced interactive version).
*   **`pgrep`**: Look up processes based on name or other attributes. Returns PIDs.
*   **`pkill`**: Send a signal to processes based on name or other attributes.
*   **`kill`**: Send a signal to processes by PID.
    *   `-9`: SIGKILL (forceful termination).
    *   `-15`: SIGTERM (graceful termination, default).
*   **`killall`**: Kill processes by name.
*   **`nohup`**: Run a command immune to hangups, with output to a non-tty.
*   **`&`**: Run a command in the background.
*   **`jobs`**: List background jobs in the current shell.
*   **`fg`**: Bring a background job to the foreground.
*   **`bg`**: Resume a suspended job in the background.

## Implementing Core Functionality with Bash

### 1. Process Listing

#### a. List All Running Processes

```bash
# Common Linux/macOS syntax for a comprehensive list
ps aux

# Alternative SysV syntax (common on Linux)
ps -ef
```

#### b. List Processes with Custom Output Format

```bash
# Display PID, PPID, User, %CPU, %MEM, Command
ps -eo pid,ppid,user,%cpu,%mem,cmd
```

#### c. Filter Processes by Name/Command

```bash
# Find processes named 'apache2' (case-insensitive)
ps aux | grep -i "apache2" | grep -v "grep" # Exclude the grep process itself

# Using pgrep for more precise filtering by name
pgrep -l "sshd" # -l shows PID and name
```

#### d. Filter Processes by User

```bash
# List all processes owned by user 'www-data'
ps -u www-data

# Using pgrep for processes owned by root
pgrep -u root -l
```

#### e. Filter Processes by PID

```bash
# Get details for a specific PID (e.g., PID 1234)
ps -p 1234 -eo pid,user,cmd,%cpu,%mem
```

### 2. Process Details

Getting detailed information typically involves combining `ps` output or looking into `/proc/<PID>` on Linux.

```bash
#!/bin/bash

# Function to get detailed info for a PID
get_process_details() {
    local pid="$1"
    echo "--- Details for PID $pid ---"
    # Basic info
    ps -p "$pid" -eo pid,ppid,user,group,%cpu,%mem,stime,etime,cmd --forest

    # On Linux, /proc/<PID> contains more info
    if [ -d "/proc/$pid" ]; then
        echo "--- /proc/$pid Contents (Linux) ---"
        echo "Command Line: $(cat /proc/$pid/cmdline | tr '\0' ' ')"
        echo "Status: $(cat /proc/$pid/status | grep "State")"
        echo "Memory Info: $(grep -E "VmSize|VmRSS" /proc/$pid/status)"
        # You can explore /proc/$pid/fd/ for open file descriptors, etc.
    fi
    echo ""
}

# Example Usage:
# get_process_details $$ # Get details for current shell
```

### 3. Process Control

#### a. Terminate a Process Gracefully (`kill` or `pkill`)

```bash
# Terminate by PID (e.g., PID 12345)
kill 12345 # Sends SIGTERM (signal 15) by default

# Terminate by name (e.g., 'nginx')
pkill nginx # Sends SIGTERM by default
```

#### b. Terminate a Process Forcefully (`kill -9` or `pkill -9` / `killall`)

```bash
# Forcefully terminate by PID (e.g., PID 12345)
kill -9 12345 # Sends SIGKILL (signal 9)

# Forcefully terminate by name (e.g., 'nginx')
pkill -9 nginx
killall -9 nginx # killall also works by name, often more aggressive
```

#### c. Start a Process in the Background

```bash
# Run a long-running command in the background, output redirected to nohup.out
nohup sleep 600 &

# Run a simple command in the background, output to current terminal
your_command_here &
echo "Process started with PID $!" # $! contains PID of last background process
```

### 4. Process Monitoring (Basic)

#### a. Identify Top CPU/Memory Consuming Processes

```bash
# Top 5 CPU consuming processes
ps aux --sort=-%cpu | head -n 6 # 1 header line + 5 processes

# Top 5 Memory consuming processes
ps aux --sort=-%mem | head -n 6
```

## Automating Tasks with Bash

Bash scripts can integrate these commands for powerful automation.

```bash
#!/bin/bash

# Script to check for rogue processes and restart a service

SERVICE_NAME="my_app_service"
ROGUE_PROCESS_NAME="malicious_script.py"

# Check if rogue process is running
if pgrep -x "$ROGUE_PROCESS_NAME" > /dev/null; then
    echo "WARNING: Rogue process '$ROGUE_PROCESS_NAME' detected!"
    echo "Terminating PID(s): $(pgrep -x "$ROGUE_PROCESS_NAME")"
    pkill -9 "$ROGUE_PROCESS_NAME"
    logger -t process_monitor "Rogue process '$ROGUE_PROCESS_NAME' terminated."
fi

# Check if critical service is running
if ! pgrep -x "$SERVICE_NAME" > /dev/null; then
    echo "ERROR: Service '$SERVICE_NAME' is not running. Attempting to restart."
    /etc/init.d/"$SERVICE_NAME" start # Or systemctl start "$SERVICE_NAME"
    logger -t process_monitor "Service '$SERVICE_NAME' restarted."
fi
```

## Guiding Principles in Bash

*   **Portability:** Commands like `ps`, `kill`, `pgrep`, `pkill`, `nohup` are standard on virtually all Unix-like systems (Linux, macOS). Syntax might vary slightly (`ps aux` vs `ps -ef`).
*   **Efficiency:** Direct execution of compiled system utilities is very fast. Pipelines allow for efficient processing of output.
*   **Minimal Dependencies:** Relies entirely on core system utilities, requiring no external language runtimes or libraries beyond Bash itself.
*   **CLI-centric:** All operations are command-line based, ideal for direct execution, scripting, and integration with task schedulers like `cron`.

## Conclusion

Bash provides a direct and efficient way to interact with and manage system processes. By mastering the `ps` family of commands, `kill`, `pgrep`, and `pkill`, you can build powerful scripts for monitoring system health, troubleshooting, and automating process-related tasks. While rich in features for Unix-like environments, direct process control on Windows systems would require different approaches (e.g., via `tasklist`/`taskkill` or PowerShell). The next step is to apply this knowledge in practical exercises.
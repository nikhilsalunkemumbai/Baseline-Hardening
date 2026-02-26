# Bash Tutorial: System Information Snapshot and Reporting

## Introduction

Bash is an excellent environment for quickly gathering system information due to its direct access to underlying operating system commands. This tutorial focuses on using standard command-line utilities available on most Unix-like systems (Linux, macOS) to collect a snapshot of system configuration and state, aligning with our design principles of portability, efficiency, and minimal dependencies.

## Framework Alignment

This tutorial on "**System Information Snapshot and Reporting**" using **Bash** demonstrates the foundational skills necessary to implement components of a "Cross-Platform Baseline Hardening & Auditing Framework." The techniques learned here for collecting and reporting system configuration data are directly applicable to auditing systems against defined security baselines and ensuring compliance.


## Core Bash Utilities for System Information

The following standard utilities are commonly used for system information gathering:

*   **`uname`**: Prints system information (kernel name, hostname, kernel release, kernel version, machine hardware name, operating system).
    *   `-a`: All information.
    *   `-s`: Kernel name.
    *   `-r`: Kernel release.
    *   `-m`: Machine hardware name.
*   **`cat /etc/os-release` or `lsb_release -a`**: Provides OS distribution information (Linux specific).
*   **`lscpu`**: Displays information about the CPU architecture (Linux specific, often part of `util-linux` or `procps-ng`).
*   **`cat /proc/cpuinfo`**: Provides detailed CPU information (Linux specific).
*   **`free`**: Displays amount of free and used memory in the system (Linux specific).
    *   `-h`: Human-readable format.
*   **`cat /proc/meminfo`**: Provides detailed memory information (Linux specific).
*   **`df`**: Reports file system disk space usage.
    *   `-h`: Human-readable format.
    *   `-T`: Print file system type.
*   **`mount`**: Shows mounted file systems.
*   **`ip`**: Show / manipulate routing, network devices, interfaces (modern Linux).
    *   `ip a`: Show address information.
*   **`ifconfig`**: Configure a network interface (legacy, common on macOS, older Linux).
*   **`ps`**: Report a snapshot of the current processes.
    *   `aux`: Display all processes, owned by any user, with full format.
    *   `ps -e -o pid,comm,user,%cpu,%mem`: Custom format.
*   **`uptime`**: Tell how long the system has been running.
*   **`who`** / **`w`**: Show who is logged on.

## Implementing Core Functionality with Bash

### 1. Operating System Information

```bash
echo "--- OS Information ---"
echo "Kernel Name: $(uname -s)"
echo "Kernel Release: $(uname -r)"
echo "Architecture: $(uname -m)"
echo "OS Type: $(uname -o)" # GNU/Linux
echo "Hostname: $(uname -n)"

# Linux Distribution Specifics (using /etc/os-release)
if [ -f "/etc/os-release" ]; then
    echo "--- Linux Distribution Details ---"
    cat /etc/os-release | grep "PRETTY_NAME\|VERSION\|ID"
elif command -v lsb_release >/dev/null 2>&1; then
    echo "--- Linux Distribution Details (lsb_release) ---"
    lsb_release -a
fi
```

### 2. CPU Information

```bash
echo "--- CPU Information ---"
if command -v lscpu >/dev/null 2>&1; then
    lscpu | grep "Model name:\|CPU(s):\|Core(s) per socket:\|Architecture:"
elif [ -f "/proc/cpuinfo" ]; then
    echo "Processor: $(grep -m 1 'model name' /proc/cpuinfo | cut -d: -f2 | sed 's/^\s*//')"
    echo "Cores: $(grep -c 'processor' /proc/cpuinfo)"
    echo "Architecture: $(uname -m)"
else
    echo "CPU information tools (lscpu, /proc/cpuinfo) not found."
fi
```

### 3. Memory Information

```bash
echo "--- Memory Information ---"
if command -v free >/dev/null 2>&1; then
    free -h | grep "Mem:\|Swap:"
elif [ -f "/proc/meminfo" ]; then
    grep "MemTotal\|MemFree\|SwapTotal\|SwapFree" /proc/meminfo
else
    echo "Memory information tools (free, /proc/meminfo) not found."
fi
```

### 4. Disk Information

```bash
echo "--- Disk Information ---"
df -hT | awk '{if(NR==1) print $0; else if($1!~/^tmpfs/ && $1!~/^udev/ && $1!~/^devtmpfs/) print $0}'
```

### 5. Network Information

```bash
echo "--- Network Information ---"
if command -v ip >/dev/null 2>&1; then
    ip -o a | awk '{print "Interface:", $2, "IP:", $4}' # Basic IP info
    echo "--- All Network Interfaces ---"
    ip a
elif command -v ifconfig >/dev/null 2>&1; then
    echo "--- Network Interfaces (ifconfig) ---"
    ifconfig
else
    echo "Network information tools (ip, ifconfig) not found."
fi
```

### 6. Running Processes (Basic)

```bash
echo "--- Running Processes (Top 10 by CPU) ---"
ps aux --sort=-%cpu | head -n 11 # 1 header + 10 processes
```

### 7. System Uptime

```bash
echo "--- System Uptime ---"
uptime -p
```

### 8. Logged-in Users

```bash
echo "--- Logged-in Users ---"
who -a
```

## Guiding Principles in Bash

*   **Portability:** Commands like `uname`, `df`, `ps`, `uptime`, `who` are highly portable across Unix-like systems. Where commands differ (e.g., `lscpu` vs. `/proc/cpuinfo`, `ip` vs. `ifconfig`), conditional checks (`if command -v ...`) are used to ensure broader compatibility.
*   **Efficiency:** Direct execution of compiled system utilities is extremely efficient. Piping data between commands processes it in streams, minimizing memory usage.
*   **Minimal Dependencies:** All tools used are standard utilities found on virtually any modern Linux or macOS system, requiring no additional installations.
*   **CLI-centric:** The entire approach is built on command-line execution, making it perfect for inclusion in scripts and automation.
*   **Actionable Output:** The output is typically text-based, which can be easily parsed further by other Bash tools (e.g., `grep`, `awk`) or redirected to files.

## Creating a Full System Snapshot Script

You can combine these snippets into a single Bash script to generate a comprehensive system information report.

```bash
#!/bin/bash

# sys_snapshot.sh

echo "### System Information Snapshot ###"
echo "Report generated on: $(date)"
echo ""

# OS Information
echo "--- OS Information ---"
echo "Kernel Name: $(uname -s)"
echo "Kernel Release: $(uname -r)"
echo "Architecture: $(uname -m)"
echo "OS Type: $(uname -o)"
echo "Hostname: $(uname -n)"

if [ -f "/etc/os-release" ]; then
    echo "--- Linux Distribution Details ---"
    cat /etc/os-release | grep "PRETTY_NAME\|VERSION\|ID"
elif command -v lsb_release >/dev/null 2>&1; then
    echo "--- Linux Distribution Details (lsb_release) ---"
    lsb_release -a
fi
echo ""

# CPU Information
echo "--- CPU Information ---"
if command -v lscpu >/dev/null 2>&1; then
    lscpu | grep "Model name:\|CPU(s):\|Core(s) per socket:\|Architecture:"
elif [ -f "/proc/cpuinfo" ]; then
    echo "Processor: $(grep -m 1 'model name' /proc/cpuinfo | cut -d: -f2 | sed 's/^\s*//')"
    echo "Cores: $(grep -c 'processor' /proc/cpuinfo)"
    echo "Architecture: $(uname -m)"
else
    echo "CPU information tools (lscpu, /proc/cpuinfo) not found."
fi
echo ""

# Memory Information
echo "--- Memory Information ---"
if command -v free >/dev/null 2>&1; then
    free -h | grep "Mem:\|Swap:"
elif [ -f "/proc/meminfo" ]; then
    grep "MemTotal\|MemFree\|SwapTotal\|SwapFree" /proc/meminfo
else
    echo "Memory information tools (free, /proc/meminfo) not found."
fi
echo ""

# Disk Information
echo "--- Disk Information ---"
df -hT | awk '{if(NR==1) print $0; else if($1!~/^tmpfs/ && $1!~/^udev/ && $1!~/^devtmpfs/) print $0}'
echo ""

# Network Information
echo "--- Network Information ---"
if command -v ip >/dev/null 2>&1; then
    ip -o a | awk '{print "Interface:", $2, "IP:", $4}'
    echo "--- All Network Interfaces ---"
    ip a
elif command -v ifconfig >/dev/null 2>&1; then
    echo "--- Network Interfaces (ifconfig) ---"
    ifconfig
else
    echo "Network information tools (ip, ifconfig) not found."
fi
echo ""

# Running Processes (Top 10 by CPU)
echo "--- Running Processes (Top 10 by CPU) ---"
ps aux --sort=-%cpu | head -n 11
echo ""

# System Uptime
echo "--- System Uptime ---"
uptime -p
echo ""

# Logged-in Users
echo "--- Logged-in Users ---"
who -a
echo ""
```
To make this script executable: `chmod +x sys_snapshot.sh`
Then run: `./sys_snapshot.sh > system_report.txt`

## Conclusion

Bash provides a direct and efficient way to gather comprehensive system information using a collection of standard command-line utilities. Its strength lies in its ability to execute these utilities and combine their output through piping, creating powerful and lightweight reporting scripts with minimal dependencies. The ability to adapt to different tools based on OS availability (e.g., `ip` vs `ifconfig`) ensures a good level of portability. The next step is to apply this knowledge in practical exercises.
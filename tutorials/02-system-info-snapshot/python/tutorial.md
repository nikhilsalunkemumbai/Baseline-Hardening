# Python Tutorial: System Information Snapshot and Reporting

## Introduction

Python's standard library, combined with its ability to execute external commands, makes it a powerful and highly portable language for gathering system information. This tutorial will demonstrate how to build a system information snapshot utility in Python, adhering to our principles of minimal dependencies, cross-platform compatibility, and actionable output. We will primarily leverage the `platform`, `os`, and `subprocess` modules.

## Framework Alignment

This tutorial on "**System Information Snapshot and Reporting**" using **Python** demonstrates the foundational skills necessary to implement components of a "Cross-Platform Baseline Hardening & Auditing Framework." The techniques learned here for collecting and reporting system configuration data are directly applicable to auditing systems against defined security baselines and ensuring compliance.


## Core Python Modules for System Information

*   **`platform`**: Provides access to underlying platform’s identifying data, such as its hardware, operating system, and Python version information.
*   **`os`**: Provides a way of using operating system dependent functionality. Useful for path manipulation and some system calls.
*   **`sys`**: Provides access to system-specific parameters and functions.
*   **`subprocess`**: Allows you to spawn new processes, connect to their input/output/error pipes, and obtain their return codes. Essential for running external system commands.
*   **`json`**: For serializing collected data into JSON format.
*   **`re`**: For robust parsing of text output from external commands.

## Implementing Core Functionality with Python

### Utility Function for Running Commands

A helper function to run shell commands and capture their output:

```python
import subprocess
import json
import re
import platform
import os
import sys

def run_command(command, shell=False):
    """Runs a shell command and returns its stdout. Handles errors."""
    try:
        # Use shell=True only if the command string needs shell features (e.g., pipes, wildcards)
        # Otherwise, pass command as a list of strings for better security and clarity.
        result = subprocess.run(
            command,
            shell=shell,
            capture_output=True,
            text=True,
            check=True # Raise an exception for non-zero exit codes
        )
        return result.stdout.strip()
    except subprocess.CalledProcessError as e:
        # Command failed to execute successfully
        return f"ERROR: Command '{' '.join(command) if isinstance(command, list) else command}' failed: {e.stderr.strip()}"
    except FileNotFoundError:
        # Command not found
        return f"ERROR: Command '{command[0] if isinstance(command, list) else command.split()[0]}' not found."
    except Exception as e:
        return f"ERROR: An unexpected error occurred: {e}"

```

### 1. Operating System Information

```python
def get_os_info():
    os_info = {
        "system": platform.system(),
        "node_name": platform.node(),
        "release": platform.release(),
        "version": platform.version(),
        "machine": platform.machine(),
        "processor": platform.processor(),
        "os_name": platform.platform(),
        "architecture": platform.architecture()[0],
    }

    if os_info["system"] == "Linux":
        # Attempt to get distribution details from /etc/os-release
        os_release_content = run_command(["cat", "/etc/os-release"])
        if not os_release_content.startswith("ERROR"):
            for line in os_release_content.splitlines():
                if "=" in line:
                    key, value = line.split("=", 1)
                    if key in ["NAME", "PRETTY_NAME", "VERSION", "ID"]:
                        os_info[key.lower()] = value.strip('"')
    
    return os_info

# Example Usage:
# print("--- OS Information ---")
# print(json.dumps(get_os_info(), indent=2))
```

### 2. CPU Information

```python
def get_cpu_info():
    cpu_info = {}
    system = platform.system()

    if system == "Linux":
        lscpu_output = run_command(["lscpu"])
        if not lscpu_output.startswith("ERROR"):
            for line in lscpu_output.splitlines():
                if "Model name:" in line:
                    cpu_info["model_name"] = line.split(":")[1].strip()
                elif "CPU(s):" in line:
                    cpu_info["cpus_total"] = int(line.split(":")[1].strip())
                elif "Core(s) per socket:" in line:
                    cpu_info["cores_per_socket"] = int(line.split(":")[1].strip())
                elif "Architecture:" in line:
                    cpu_info["architecture"] = line.split(":")[1].strip()
            if "cpus_total" in cpu_info and "cores_per_socket" in cpu_info:
                cpu_info["sockets"] = cpu_info["cpus_total"] // cpu_info["cores_per_socket"]
        else: # Fallback to /proc/cpuinfo
            cpuinfo_content = run_command(["cat", "/proc/cpuinfo"])
            if not cpuinfo_content.startswith("ERROR"):
                processors = []
                current_processor = {}
                for line in cpuinfo_content.splitlines():
                    if line.strip() == "":
                        if current_processor:
                            processors.append(current_processor)
                        current_processor = {}
                    elif ":" in line:
                        key, value = line.split(":", 1)
                        current_processor[key.strip()] = value.strip()
                if current_processor:
                    processors.append(current_processor)
                
                if processors:
                    cpu_info["model_name"] = processors[0].get("model name")
                    cpu_info["cpus_total"] = len(processors)
                    # Estimating cores/sockets is harder from just /proc/cpuinfo without grouping
                    cpu_info["architecture"] = platform.machine()
            else:
                cpu_info["error"] = lscpu_output # or cpuinfo_content
    elif system == "Windows":
        # Use WMIC for more detail, or just platform.processor()
        cpu_info["model_name"] = platform.processor()
        # Number of logical processors
        cpu_count = run_command(["wmic", "cpu", "get", "NumberOfLogicalProcessors", "/value"])
        match = re.search(r"NumberOfLogicalProcessors=(\d+)", cpu_count)
        if match:
            cpu_info["cpus_total"] = int(match.group(1))
    elif system == "Darwin": # macOS
        cpu_info["model_name"] = run_command(["sysctl", "-n", "machdep.cpu.brand_string"])
        cpu_info["cpus_total"] = int(run_command(["sysctl", "-n", "hw.ncpu"]))
    
    return cpu_info

# Example Usage:
# print("--- CPU Information ---")
# print(json.dumps(get_cpu_info(), indent=2))
```

### 3. Memory Information

```python
def get_memory_info():
    mem_info = {}
    system = platform.system()

    if system == "Linux":
        free_output = run_command(["free", "-h"])
        if not free_output.startswith("ERROR"):
            lines = free_output.splitlines()
            if len(lines) > 1:
                mem_line = lines[1].split()
                mem_info["total_memory"] = mem_line[1]
                mem_info["used_memory"] = mem_line[2]
                mem_info["free_memory"] = mem_line[3]
        else: # Fallback to /proc/meminfo
            meminfo_content = run_command(["cat", "/proc/meminfo"])
            if not meminfo_content.startswith("ERROR"):
                for line in meminfo_content.splitlines():
                    if "MemTotal:" in line:
                        mem_info["total_memory_kb"] = int(line.split(":")[1].strip().split()[0])
                    elif "MemFree:" in line:
                        mem_info["free_memory_kb"] = int(line.split(":")[1].strip().split()[0])
            if "total_memory_kb" in mem_info and "free_memory_kb" in mem_info:
                mem_info["total_memory_gb"] = f"{mem_info['total_memory_kb'] / (1024**2):.2f} GB"
                mem_info["free_memory_gb"] = f"{mem_info['free_memory_kb'] / (1024**2):.2f} GB"
                mem_info["used_memory_gb"] = f"{(mem_info['total_memory_kb'] - mem_info['free_memory_kb']) / (1024**2):.2f} GB"
                del mem_info["total_memory_kb"]
                del mem_info["free_memory_kb"]
            else:
                mem_info["error"] = free_output # or meminfo_content
    elif system == "Windows":
        wmic_mem = run_command(["wmic", "ComputerSystem", "get", "TotalPhysicalMemory", "/value"])
        total_mem_match = re.search(r"TotalPhysicalMemory=(\d+)", wmic_mem)
        if total_mem_match:
            total_bytes = int(total_mem_match.group(1))
            mem_info["total_memory_gb"] = f"{total_bytes / (1024**3):.2f} GB"
            # Getting free memory is more complex, might need 'systeminfo' and parse
    elif system == "Darwin": # macOS
        total_bytes = int(run_command(["sysctl", "-n", "hw.memsize"]))
        mem_info["total_memory_gb"] = f"{total_bytes / (1024**3):.2f} GB"
        # Free memory needs parsing from 'vm_stat' or 'top'
    
    return mem_info

# Example Usage:
# print("--- Memory Information ---")
# print(json.dumps(get_memory_info(), indent=2))
```

### 4. Disk Information

```python
def get_disk_info():
    disks = []
    system = platform.system()

    if system == "Linux" or system == "Darwin": # Linux/macOS
        df_output = run_command(["df", "-hT"])
        if not df_output.startswith("ERROR"):
            lines = df_output.splitlines()
            for line in lines[1:]: # Skip header
                parts = line.split()
                if len(parts) >= 7 and not parts[0].startswith("tmpfs") and not parts[0].startswith("udev"):
                    disks.append({
                        "filesystem": parts[0],
                        "type": parts[1],
                        "size": parts[2],
                        "used": parts[3],
                        "available": parts[4],
                        "mount_point": parts[6]
                    })
        else:
            disks.append({"error": df_output})
    elif system == "Windows":
        wmic_logicaldisk = run_command(["wmic", "logicaldisk", "get", "Caption,FileSystem,FreeSpace,Size", "/value"])
        if not wmic_logicaldisk.startswith("ERROR"):
            current_disk = {}
            for line in wmic_logicaldisk.splitlines():
                if "=" in line:
                    key, value = line.split("=", 1)
                    current_disk[key.strip()] = value.strip()
                elif line.strip() == "" and current_disk:
                    # Convert bytes to GB
                    if "Size" in current_disk:
                        current_disk["SizeGB"] = f"{int(current_disk['Size']) / (1024**3):.2f} GB"
                        del current_disk["Size"]
                    if "FreeSpace" in current_disk:
                        current_disk["FreeSpaceGB"] = f"{int(current_disk['FreeSpace']) / (1024**3):.2f} GB"
                        del current_disk["FreeSpace"]
                    disks.append(current_disk)
                    current_disk = {}
            if current_disk: # Add the last one if not empty
                if "Size" in current_disk:
                    current_disk["SizeGB"] = f"{int(current_disk['Size']) / (1024**3):.2f} GB"
                    del current_disk["Size"]
                if "FreeSpace" in current_disk:
                    current_disk["FreeSpaceGB"] = f"{int(current_disk['FreeSpace']) / (1024**3):.2f} GB"
                    del current_disk["FreeSpace"]
                disks.append(current_disk)
        else:
            disks.append({"error": wmic_logicaldisk})
            
    return disks

# Example Usage:
# print("--- Disk Information ---")
# print(json.dumps(get_disk_info(), indent=2))
```

### 5. Network Information

```python
def get_network_info():
    interfaces = []
    system = platform.system()

    if system == "Linux":
        ip_a_output = run_command(["ip", "-json", "a"]) # Use -json for easier parsing
        if not ip_a_output.startswith("ERROR"):
            try:
                ip_data = json.loads(ip_a_output)
                for entry in ip_data:
                    current_if = {
                        "name": entry.get("ifname"),
                        "mac_address": entry.get("address"),
                        "state": entry.get("operstate"),
                        "ipv4_addresses": [],
                        "ipv6_addresses": []
                    }
                    for addr_info in entry.get("addr_info", []):
                        if addr_info.get("family") == "inet":
                            current_if["ipv4_addresses"].append(addr_info.get("local"))
                        elif addr_info.get("family") == "inet6":
                            current_if["ipv6_addresses"].append(addr_info.get("local"))
                    interfaces.append(current_if)
            except json.JSONDecodeError:
                # Fallback to text parsing if -json is not supported or fails
                text_output = run_command(["ip", "a"])
                if not text_output.startswith("ERROR"):
                    # Basic text parsing (can be complex for full detail)
                    pass # Simplified for brevity, requires robust regex for production
                interfaces.append({"error": "Failed to parse 'ip -json a' or 'ip a' text output."})
        else:
            interfaces.append({"error": ip_a_output})
    elif system == "Windows":
        # Using Get-NetAdapter and Get-NetIPAddress via PowerShell, then parse
        ps_command = (
            "Get-NetAdapter | Select-Object Name, MacAddress, Status, LinkSpeed | ConvertTo-Json;"
            "Get-NetIPAddress | Where-Object {$_.AddressFamily -eq 'IPv4' -and $_.InterfaceAlias -notlike 'Loopback*'} | Select-Object InterfaceAlias, IPAddress | ConvertTo-Json"
        )
        # Note: Executing PowerShell from Python needs careful handling of execution policy and quoting
        # This is a complex example and might be better handled by directly interacting with WMI via pywin32 or similar
        # For minimal dependency, might rely on 'ipconfig' and regex.
        ipconfig_output = run_command(["ipconfig", "/all"]) # Fallback for minimal dependency
        if not ipconfig_output.startswith("ERROR"):
            # Basic parsing example for ipconfig
            current_adapter = {}
            for line in ipconfig_output.splitlines():
                if "Ethernet adapter" in line or "Wireless LAN adapter" in line:
                    if current_adapter: interfaces.append(current_adapter)
                    current_adapter = {"name": line.split(":",1)[0].strip(), "ipv4_addresses": [], "mac_address": ""}
                elif "IPv4 Address" in line and current_adapter:
                    match = re.search(r"IPv4 Address[.: ]+ (\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3})", line)
                    if match: current_adapter["ipv4_addresses"].append(match.group(1))
                elif "Physical Address" in line and current_adapter:
                    match = re.search(r"Physical Address[.: ]+ ([\dA-Fa-f-]{17})", line)
                    if match: current_adapter["mac_address"] = match.group(1)
            if current_adapter: interfaces.append(current_adapter)
        else:
            interfaces.append({"error": ipconfig_output})
    elif system == "Darwin": # macOS
        ifconfig_output = run_command(["ifconfig"])
        if not ifconfig_output.startswith("ERROR"):
            # Basic parsing, similar to ipconfig
            pass # Simplified for brevity
        interfaces.append({"error": ifconfig_output if ifconfig_output.startswith("ERROR") else "Parsing 'ifconfig' on macOS not implemented."})
    
    return interfaces

# Example Usage:
# print("--- Network Information ---")
# print(json.dumps(get_network_info(), indent=2))
```

### 6. Running Processes (Basic)

```python
def get_processes():
    processes = []
    system = platform.system()

    if system == "Linux" or system == "Darwin": # Linux/macOS
        ps_output = run_command(["ps", "aux"])
        if not ps_output.startswith("ERROR"):
            lines = ps_output.splitlines()
            for line in lines[1:]: # Skip header
                parts = line.split(None, 10) # Split by any whitespace, max 10 times
                if len(parts) >= 11:
                    processes.append({
                        "user": parts[0],
                        "pid": parts[1],
                        "cpu_percent": parts[2],
                        "mem_percent": parts[3],
                        "command": parts[10]
                    })
        else:
            processes.append({"error": ps_output})
    elif system == "Windows":
        wmic_process = run_command(["wmic", "process", "get", "ProcessId,Name,WorkingSetSize", "/value"])
        if not wmic_process.startswith("ERROR"):
            current_process = {}
            for line in wmic_process.splitlines():
                if "=" in line:
                    key, value = line.split("=", 1)
                    current_process[key.strip()] = value.strip()
                elif line.strip() == "" and current_process:
                    processes.append(current_process)
                    current_process = {}
            if current_process: processes.append(current_process)
        else:
            processes.append({"error": wmic_process})
            
    return processes

# Example Usage:
# print("--- Running Processes ---")
# # Display top 10 by CPU (requires sorting, omitted for brevity in snippet)
# print(json.dumps(get_processes()[:10], indent=2))
```

### 7. System Uptime

```python
def get_uptime_info():
    uptime_info = {}
    system = platform.system()

    if system == "Linux" or system == "Darwin":
        uptime_output = run_command(["uptime", "-p"])
        if not uptime_output.startswith("ERROR"):
            uptime_info["uptime"] = uptime_output.replace("up ", "")
        else:
            uptime_info["error"] = uptime_output
    elif system == "Windows":
        # On Windows, 'systeminfo' or WMI can provide this
        # wmic_os = run_command(["wmic", "os", "get", "LastBootUpTime", "/value"])
        # match = re.search(r"LastBootUpTime=(\d{14})", wmic_os)
        # if match:
        #     boot_time_str = match.group(1)
        #     # Convert to datetime object and calculate delta
        uptime_output = run_command(["systeminfo"])
        match = re.search(r"System Boot Time:\s+(.+)", uptime_output)
        if match:
            boot_time_str = match.group(1).strip()
            # This would require datetime parsing, simplified for the pseudo-code tutorial
            uptime_info["boot_time_raw"] = boot_time_str
        else:
            uptime_info["error"] = uptime_output
    return uptime_info

# Example Usage:
# print("--- System Uptime ---")
# print(json.dumps(get_uptime_info(), indent=2))
```

### 8. Logged-in Users

```python
def get_logged_in_users():
    users = []
    system = platform.system()

    if system == "Linux" or system == "Darwin":
        who_output = run_command(["who"])
        if not who_output.startswith("ERROR"):
            for line in who_output.splitlines():
                parts = line.split(None, 4) # user tty date time remote
                if len(parts) >= 4:
                    users.append({
                        "user": parts[0],
                        "tty": parts[1],
                        "login_time": f"{parts[2]} {parts[3]}"
                    })
        else:
            users.append({"error": who_output})
    elif system == "Windows":
        # 'query user' or 'qwinsta' commands, or WMI
        # For minimal dependency:
        query_user_output = run_command(["query", "user"])
        if not query_user_output.startswith("ERROR"):
            # Parsing 'query user' output
            for line in query_user_output.splitlines()[1:]: # Skip header
                parts = line.split()
                if len(parts) >= 3:
                    users.append({"user": parts[0], "session_name": parts[1], "id": parts[2]})
        else:
            users.append({"error": query_user_output})

    return users

# Example Usage:
# print("--- Logged-in Users ---")
# print(json.dumps(get_logged_in_users(), indent=2))
```

## Creating a Full System Snapshot Script (`sys_snapshot.py`)

Here's how you might structure a complete Python script:

```python
#!/usr/bin/env python3

# sys_snapshot.py
# A script to collect system information and report it in a structured format.

import subprocess
import json
import re
import platform
import os
import sys
from datetime import datetime

# --- Helper Function ---
def run_command(command, shell=False):
    """Runs a shell command and returns its stdout. Handles errors."""
    try:
        result = subprocess.run(
            command,
            shell=shell,
            capture_output=True,
            text=True,
            check=True
        )
        return result.stdout.strip()
    except subprocess.CalledProcessError as e:
        return f"ERROR: Command '{' '.join(command) if isinstance(command, list) else command}' failed: {e.stderr.strip()}"
    except FileNotFoundError:
        return f"ERROR: Command '{command[0] if isinstance(command, list) else command.split()[0]}' not found."
    except Exception as e:
        return f"ERROR: An unexpected error occurred: {e}"

# --- Data Collection Functions (as defined above) ---

def get_os_info():
    os_info = {
        "system": platform.system(),
        "node_name": platform.node(),
        "release": platform.release(),
        "version": platform.version(),
        "machine": platform.machine(),
        "processor": platform.processor(),
        "os_name": platform.platform(),
        "architecture": platform.architecture()[0],
    }

    if os_info["system"] == "Linux":
        os_release_content = run_command(["cat", "/etc/os-release"])
        if not os_release_content.startswith("ERROR"):
            for line in os_release_content.splitlines():
                if "=" in line:
                    key, value = line.split("=", 1)
                    if key in ["NAME", "PRETTY_NAME", "VERSION", "ID"]:
                        os_info[key.lower()] = value.strip('"')
    return os_info

def get_cpu_info():
    cpu_info = {}
    system = platform.system()
    if system == "Linux":
        lscpu_output = run_command(["lscpu"])
        if not lscpu_output.startswith("ERROR"):
            for line in lscpu_output.splitlines():
                if "Model name:" in line: cpu_info["model_name"] = line.split(":")[1].strip()
                elif "CPU(s):" in line: cpu_info["cpus_total"] = int(line.split(":")[1].strip())
                elif "Core(s) per socket:" in line: cpu_info["cores_per_socket"] = int(line.split(":")[1].strip())
                elif "Architecture:" in line: cpu_info["architecture"] = line.split(":")[1].strip()
            if "cpus_total" in cpu_info and "cores_per_socket" in cpu_info:
                cpu_info["sockets"] = cpu_info["cpus_total"] // cpu_info["cores_per_socket"]
        else:
            cpuinfo_content = run_command(["cat", "/proc/cpuinfo"])
            if not cpuinfo_content.startswith("ERROR"):
                processors = []
                current_processor = {}
                for line in cpuinfo_content.splitlines():
                    if line.strip() == "":
                        if current_processor: processors.append(current_processor)
                        current_processor = {}
                    elif ":" in line:
                        key, value = line.split(":", 1)
                        current_processor[key.strip()] = value.strip()
                if current_processor: processors.append(current_processor)
                if processors:
                    cpu_info["model_name"] = processors[0].get("model name")
                    cpu_info["cpus_total"] = len(processors)
                    cpu_info["architecture"] = platform.machine()
            else: cpu_info["error"] = lscpu_output
    elif system == "Windows":
        cpu_info["model_name"] = platform.processor()
        cpu_count = run_command(["wmic", "cpu", "get", "NumberOfLogicalProcessors", "/value"])
        match = re.search(r"NumberOfLogicalProcessors=(\d+)", cpu_count)
        if match: cpu_info["cpus_total"] = int(match.group(1))
    elif system == "Darwin":
        cpu_info["model_name"] = run_command(["sysctl", "-n", "machdep.cpu.brand_string"])
        cpu_info["cpus_total"] = int(run_command(["sysctl", "-n", "hw.ncpu"]))
    return cpu_info

def get_memory_info():
    mem_info = {}
    system = platform.system()
    if system == "Linux":
        free_output = run_command(["free", "-h"])
        if not free_output.startswith("ERROR"):
            lines = free_output.splitlines()
            if len(lines) > 1:
                mem_line = lines[1].split()
                mem_info["total_memory"] = mem_line[1]
                mem_info["used_memory"] = mem_line[2]
                mem_info["free_memory"] = mem_line[3]
        else:
            meminfo_content = run_command(["cat", "/proc/meminfo"])
            if not meminfo_content.startswith("ERROR"):
                for line in meminfo_content.splitlines():
                    if "MemTotal:" in line: mem_info["total_memory_kb"] = int(line.split(":")[1].strip().split()[0])
                    elif "MemFree:" in line: mem_info["free_memory_kb"] = int(line.split(":")[1].strip().split()[0])
                if "total_memory_kb" in mem_info and "free_memory_kb" in mem_info:
                    mem_info["total_memory_gb"] = f"{mem_info['total_memory_kb'] / (1024**2):.2f} GB"
                    mem_info["free_memory_gb"] = f"{mem_info['free_memory_kb'] / (1024**2):.2f} GB"
                    mem_info["used_memory_gb"] = f"{(mem_info['total_memory_kb'] - mem_info['free_memory_kb']) / (1024**2):.2f} GB"
                    del mem_info["total_memory_kb"]
                    del mem_info["free_memory_kb"]
            else: mem_info["error"] = free_output
    elif system == "Windows":
        wmic_mem = run_command(["wmic", "ComputerSystem", "get", "TotalPhysicalMemory", "/value"])
        total_mem_match = re.search(r"TotalPhysicalMemory=(\d+)", wmic_mem)
        if total_mem_match:
            total_bytes = int(total_mem_match.group(1))
            mem_info["total_memory_gb"] = f"{total_bytes / (1024**3):.2f} GB"
    elif system == "Darwin":
        total_bytes = int(run_command(["sysctl", "-n", "hw.memsize"]))
        mem_info["total_memory_gb"] = f"{total_bytes / (1024**3):.2f} GB"
    return mem_info

def get_disk_info():
    disks = []
    system = platform.system()
    if system == "Linux" or system == "Darwin":
        df_output = run_command(["df", "-hT"])
        if not df_output.startswith("ERROR"):
            lines = df_output.splitlines()
            for line in lines[1:]:
                parts = line.split()
                if len(parts) >= 7 and not parts[0].startswith("tmpfs") and not parts[0].startswith("udev"):
                    disks.append({
                        "filesystem": parts[0], "type": parts[1], "size": parts[2],
                        "used": parts[3], "available": parts[4], "mount_point": parts[6]
                    })
        else: disks.append({"error": df_output})
    elif system == "Windows":
        wmic_logicaldisk = run_command(["wmic", "logicaldisk", "get", "Caption,FileSystem,FreeSpace,Size", "/value"])
        if not wmic_logicaldisk.startswith("ERROR"):
            current_disk = {}
            for line in wmic_logicaldisk.splitlines():
                if "=" in line:
                    key, value = line.split("=", 1)
                    current_disk[key.strip()] = value.strip()
                elif line.strip() == "" and current_disk:
                    if "Size" in current_disk: current_disk["SizeGB"] = f"{int(current_disk['Size']) / (1024**3):.2f} GB"; del current_disk["Size"]
                    if "FreeSpace" in current_disk: current_disk["FreeSpaceGB"] = f"{int(current_disk['FreeSpace']) / (1024**3):.2f} GB"; del current_disk["FreeSpace"]
                    disks.append(current_disk); current_disk = {}
            if current_disk:
                if "Size" in current_disk: current_disk["SizeGB"] = f"{int(current_disk['Size']) / (1024**3):.2f} GB"; del current_disk["Size"]
                if "FreeSpace" in current_disk: current_disk["FreeSpaceGB"] = f"{int(current_disk['FreeSpace']) / (1024**3):.2f} GB"; del current_disk["FreeSpace"]
                disks.append(current_disk)
        else: disks.append({"error": wmic_logicaldisk})
    return disks

def get_network_info():
    interfaces = []
    system = platform.system()
    if system == "Linux":
        ip_a_output = run_command(["ip", "-json", "a"])
        if not ip_a_output.startswith("ERROR"):
            try:
                ip_data = json.loads(ip_a_output)
                for entry in ip_data:
                    current_if = {
                        "name": entry.get("ifname"), "mac_address": entry.get("address"),
                        "state": entry.get("operstate"), "ipv4_addresses": [], "ipv6_addresses": []
                    }
                    for addr_info in entry.get("addr_info", []):
                        if addr_info.get("family") == "inet": current_if["ipv4_addresses"].append(addr_info.get("local"))
                        elif addr_info.get("family") == "inet6": current_if["ipv6_addresses"].append(addr_info.get("local"))
                    interfaces.append(current_if)
            except json.JSONDecodeError: interfaces.append({"error": "Failed to parse 'ip -json a'."})
        else: interfaces.append({"error": ip_a_output})
    elif system == "Windows":
        ipconfig_output = run_command(["ipconfig", "/all"])
        if not ipconfig_output.startswith("ERROR"):
            current_adapter = {}
            for line in ipconfig_output.splitlines():
                if "Ethernet adapter" in line or "Wireless LAN adapter" in line:
                    if current_adapter: interfaces.append(current_adapter)
                    current_adapter = {"name": line.split(":",1)[0].strip(), "ipv4_addresses": [], "mac_address": ""}
                elif "IPv4 Address" in line and current_adapter:
                    match = re.search(r"IPv4 Address[.: ]+ (\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3})", line)
                    if match: current_adapter["ipv4_addresses"].append(match.group(1))
                elif "Physical Address" in line and current_adapter:
                    match = re.search(r"Physical Address[.: ]+ ([\dA-Fa-f-]{17})", line)
                    if match: current_adapter["mac_address"] = match.group(1)
            if current_adapter: interfaces.append(current_adapter)
        else: interfaces.append({"error": ipconfig_output})
    elif system == "Darwin":
        interfaces.append({"error": "Parsing 'ifconfig' on macOS not implemented in this tutorial."})
    return interfaces

def get_processes():
    processes = []
    system = platform.system()
    if system == "Linux" or system == "Darwin":
        ps_output = run_command(["ps", "aux"])
        if not ps_output.startswith("ERROR"):
            lines = ps_output.splitlines()
            for line in lines[1:]:
                parts = line.split(None, 10)
                if len(parts) >= 11:
                    processes.append({"user": parts[0], "pid": parts[1], "cpu_percent": parts[2], "mem_percent": parts[3], "command": parts[10]})
        else: processes.append({"error": ps_output})
    elif system == "Windows":
        wmic_process = run_command(["wmic", "process", "get", "ProcessId,Name,WorkingSetSize", "/value"])
        if not wmic_process.startswith("ERROR"):
            current_process = {}
            for line in wmic_process.splitlines():
                if "=" in line:
                    key, value = line.split("=", 1)
                    current_process[key.strip()] = value.strip()
                elif line.strip() == "" and current_process:
                    processes.append(current_process); current_process = {}
            if current_process: processes.append(current_process)
        else: processes.append({"error": wmic_process})
    return processes

def get_uptime_info():
    uptime_info = {}
    system = platform.system()
    if system == "Linux" or system == "Darwin":
        uptime_output = run_command(["uptime", "-p"])
        if not uptime_output.startswith("ERROR"): uptime_info["uptime"] = uptime_output.replace("up ", "")
        else: uptime_info["error"] = uptime_output
    elif system == "Windows":
        systeminfo_output = run_command(["systeminfo"])
        match = re.search(r"System Boot Time:\s+(.+)", systeminfo_output)
        if match: uptime_info["boot_time_raw"] = match.group(1).strip()
        else: uptime_info["error"] = systeminfo_output
    return uptime_info

def get_logged_in_users():
    users = []
    system = platform.system()
    if system == "Linux" or system == "Darwin":
        who_output = run_command(["who"])
        if not who_output.startswith("ERROR"):
            for line in who_output.splitlines():
                parts = line.split(None, 4)
                if len(parts) >= 4: users.append({"user": parts[0], "tty": parts[1], "login_time": f"{parts[2]} {parts[3]}"})
        else: users.append({"error": who_output})
    elif system == "Windows":
        query_user_output = run_command(["query", "user"])
        if not query_user_output.startswith("ERROR"):
            for line in query_user_output.splitlines()[1:]:
                parts = line.split()
                if len(parts) >= 3: users.append({"user": parts[0], "session_name": parts[1], "id": parts[2]})
        else: users.append({"error": query_user_output})
    return users

# --- Main Logic ---
def main():
    report = {
        "timestamp": datetime.now().isoformat(),
        "system_info": {
            "os": get_os_info(),
            "cpu": get_cpu_info(),
            "memory": get_memory_info(),
            "disks": get_disk_info(),
            "network": get_network_info(),
            "processes": get_processes(),
            "uptime": get_uptime_info(),
            "logged_in_users": get_logged_in_users(),
        }
    }

    # Output to stdout as JSON
    print(json.dumps(report, indent=2))

if __name__ == "__main__":
    main()

```

## Guiding Principles in Python

*   **Portability:** Python's `platform` module helps identify the OS, allowing for conditional execution of OS-specific commands. The `subprocess` module is cross-platform, making it possible to call native system commands on any OS.
*   **Efficiency:** For most system information gathering, executing native commands via `subprocess` is efficient. Python's overhead for parsing is generally minimal.
*   **Minimal Dependencies:** This tutorial strictly uses Python's standard library. While libraries like `psutil` can offer a more unified, cross-platform API for system metrics, `subprocess` calls provide a zero-dependency alternative.
*   **CLI-centric:** The script is designed to be executed from the command line, taking no arguments (or optional arguments for output format if extended), and printing structured JSON output to `stdout`.
*   **Actionable Output:** By converting the collected data into a structured format (JSON), it becomes easily consumable by other scripts, monitoring tools, or for storage and analysis.

## Conclusion

Python provides a flexible and powerful environment for collecting system information. By combining its standard library modules with the ability to execute and parse output from native system commands, you can build highly effective, portable, and minimally dependent system snapshot tools. The structured nature of Python makes processing and reporting this information much more manageable than raw text output from Bash. The next step is to apply this knowledge in practical exercises.
# Python Tutorial: Process Management and Automation

## Introduction

Python's standard library provides robust capabilities for interacting with the operating system, making it an excellent language for process management and automation. This tutorial will demonstrate how to list, inspect, and control processes using Python's `os` and `subprocess` modules, adhering to our principles of minimal dependencies, cross-platform compatibility, and structured output. While external libraries like `psutil` offer a more unified cross-platform API for process details, we will primarily focus on standard library features and parsing output from native system commands for maximum self-containment.

## Framework Alignment

This tutorial on "**Process Management and Automation**" using **Python** demonstrates the foundational skills necessary to implement components of a "Cross-Platform Baseline Hardening & Auditing Framework." The techniques learned here for managing and auditing running processes are directly applicable to auditing systems against defined security baselines and ensuring compliance.


## Core Python Modules for Process Management

*   **`os`**: Provides functions for interacting with the operating system, including process-related functions like `os.getpid()`, `os.kill()`.
*   **`subprocess`**: Allows you to spawn new processes, connect to their input/output/error pipes, and obtain their return codes. Essential for running external commands like `ps`, `top`, `tasklist`, `taskkill`.
*   **`signal`**: (Unix-like systems only) Provides mechanisms to handle asynchronous events (signals), such as sending `SIGTERM` or `SIGKILL`.
*   **`json`**: For serializing collected process data into JSON format.
*   **`re`**: For robust parsing of text output from external commands.

## Implementing Core Functionality with Python

### Utility Function for Running Commands

```python
import os
import subprocess
import json
import re
import signal # For os.kill
import platform
import sys
from datetime import datetime

def run_command(command, shell=False, timeout_seconds=None):
    """Runs a shell command and returns its stdout. Handles errors."""
    try:
        result = subprocess.run(
            command,
            shell=shell,
            capture_output=True,
            text=True,
            check=True, # Raise an exception for non-zero exit codes
            timeout=timeout_seconds
        )
        return result.stdout.strip()
    except subprocess.CalledProcessError as e:
        return f"ERROR: Command '{' '.join(command) if isinstance(command, list) else command}' failed: {e.stderr.strip()}"
    except subprocess.TimeoutExpired:
        return f"ERROR: Command '{' '.join(command) if isinstance(command, list) else command}' timed out."
    except FileNotFoundError:
        return f"ERROR: Command '{command[0] if isinstance(command, list) else command.split()[0]}' not found."
    except Exception as e:
        return f"ERROR: An unexpected error occurred: {e}"
```

### 1. Process Listing

#### a. List All Running Processes

This typically involves parsing the output of native system commands.

```python
def get_processes_list():
    processes = []
    system = platform.system()

    if system == "Linux" or system == "Darwin": # Unix-like
        # ps aux on Linux/macOS
        ps_output = run_command(["ps", "aux"])
        if not ps_output.startswith("ERROR"):
            lines = ps_output.splitlines()
            if len(lines) > 0:
                header = lines[0].lower().split() # Extract header for mapping
                for line in lines[1:]:
                    parts = line.split(None, len(header) - 1) # Split based on header count
                    if len(parts) >= 11: # Basic check for expected fields
                        # Example parsing, adjust based on exact ps output
                        try:
                            processes.append({
                                "user": parts[0],
                                "pid": int(parts[1]),
                                "cpu_percent": float(parts[2]),
                                "mem_percent": float(parts[3]),
                                "vsz": parts[4], # Virtual memory size
                                "rss": parts[5], # Resident set size
                                "tty": parts[6],
                                "stat": parts[7],
                                "start": parts[8],
                                "time": parts[9],
                                "command": " ".join(parts[10:]).strip() # Rest is command
                            })
                        except ValueError:
                            # Handle parsing errors for malformed lines
                            pass
        else:
            processes.append({"error": ps_output})
    elif system == "Windows":
        # tasklist command on Windows
        tasklist_output = run_command(["tasklist", "/FO", "CSV", "/NH"]) # CSV output, No Header
        if not tasklist_output.startswith("ERROR"):
            for line in tasklist_output.splitlines():
                try:
                    # Parse CSV-like output
                    parts = line.strip().strip('"').split('","')
                    if len(parts) >= 6: # ImageName, PID, SessionName, Session#, MemUsage, Status
                        processes.append({
                            "name": parts[0],
                            "pid": int(parts[1]),
                            "session_name": parts[2],
                            "session_id": int(parts[3]),
                            "mem_usage_raw": parts[4], # e.g., "1,234 K"
                            "status": parts[5],
                            # More detailed info might need WMIC/PowerShell and specific parsing
                        })
                except (ValueError, IndexError):
                    pass
        else:
            processes.append({"error": tasklist_output})
    
    return processes

# Example Usage:
# print("--- All Processes (first 5) ---")
# print(json.dumps(get_processes_list()[:5], indent=2))
```

#### b. Filter Processes

```python
def filter_processes(process_list, name_contains=None, user=None, pid=None):
    filtered = []
    for proc in process_list:
        match = True
        if name_contains and name_contains.lower() not in proc.get("name", "").lower() and 
           name_contains.lower() not in proc.get("command", "").lower():
            match = False
        if user and proc.get("user") != user:
            match = False
        if pid and proc.get("pid") != pid:
            match = False
        
        if match:
            filtered.append(proc)
    return filtered

# Example Usage:
# all_procs = get_processes_list()
# print("--- Processes with 'bash' in name ---")
# bash_procs = filter_processes(all_procs, name_contains="bash")
# print(json.dumps(bash_procs[:3], indent=2))
```

### 2. Process Control

#### a. Terminate Process by PID (`os.kill`)

```python
def terminate_process_by_pid(pid, force=False):
    """Terminates a process by PID."""
    system = platform.system()
    try:
        if system == "Windows":
            # taskkill /F /PID <PID>
            if force:
                command = ["taskkill", "/F", "/PID", str(pid)]
            else:
                command = ["taskkill", "/PID", str(pid)]
            result = run_command(command)
            if "SUCCESS" in result:
                return f"Process {pid} terminated."
            else:
                return f"Error terminating process {pid}: {result}"
        else: # Unix-like
            signal_type = signal.SIGKILL if force else signal.SIGTERM
            os.kill(pid, signal_type)
            return f"Process {pid} terminated."
    except ProcessLookupError:
        return f"Error: Process {pid} not found."
    except PermissionError:
        return f"Error: Permission denied to terminate process {pid}."
    except Exception as e:
        return f"Error terminating process {pid}: {e}"

# Example Usage:
# # Find a process PID to kill (e.g., a 'sleep' command)
# p = subprocess.Popen(["sleep", "60"]) # Start a dummy process
# print(f"Started sleep with PID: {p.pid}")
# print(terminate_process_by_pid(p.pid))
```

#### b. Start New Process (`subprocess.Popen`)

```python
def start_new_process(command_args, run_in_background=False):
    """Starts a new process."""
    try:
        if run_in_background:
            # Popen is good for background; detach from parent
            process = subprocess.Popen(command_args, stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL, preexec_fn=os.setsid if platform.system() != "Windows" else None)
            return f"Process started in background with PID: {process.pid}"
        else:
            # run() waits for process to complete
            result = subprocess.run(command_args, capture_output=True, text=True, check=True)
            return f"Process completed. Output:
{result.stdout}"
    except FileNotFoundError:
        return f"Error: Command '{command_args[0]}' not found."
    except Exception as e:
        return f"Error starting process: {e}"

# Example Usage:
# print(start_new_process(["notepad.exe"], run_in_background=True)) # Windows
# print(start_new_process(["sleep", "5"], run_in_background=False)) # Unix-like
```

### 3. Process Monitoring (Basic - parsing `ps` or `top` output)

This requires parsing the CPU/memory columns from `ps aux` (Unix-like) or `tasklist` (Windows).

```python
def get_top_cpu_processes(count=5):
    """Identifies top N CPU-consuming processes."""
    processes = []
    system = platform.system()

    if system == "Linux" or system == "Darwin":
        # ps aux output is already sorted by CPU with default --sort=-%cpu on some systems
        ps_output = run_command(["ps", "aux", "--sort=-%cpu"])
        if not ps_output.startswith("ERROR"):
            lines = ps_output.splitlines()
            if len(lines) > 0:
                for line in lines[1:count+1]: # Skip header, get top 'count'
                    parts = line.split(None, 10)
                    if len(parts) >= 11:
                        processes.append({
                            "user": parts[0],
                            "pid": int(parts[1]),
                            "cpu_percent": float(parts[2]),
                            "mem_percent": float(parts[3]),
                            "command": " ".join(parts[10:]).strip()
                        })
    elif system == "Windows":
        # tasklist /FO CSV /V /NH for more detail including CPU, but parsing is harder
        # For simplicity, we'd need to run a PowerShell command via subprocess
        # or parse `tasklist /svc` for CPU
        # For minimal dependencies, this is challenging without psutil
        processes.append({"error": "Top CPU processes parsing for Windows requires more complex logic or psutil."})
    return processes

# Example Usage:
# print("--- Top 5 CPU Processes ---")
# print(json.dumps(get_top_cpu_processes(5), indent=2))
```

### 4. Full Script Structure (`process_manager.py`)

```python
#!/usr/bin/env python3

import os
import subprocess
import json
import re
import signal
import platform
import sys
import argparse
from datetime import datetime

# --- Helper Function (run_command) ---
def run_command(command, shell=False, timeout_seconds=None):
    # ... (same as above) ...
    try:
        result = subprocess.run(
            command, shell=shell, capture_output=True, text=True, check=True, timeout=timeout_seconds
        )
        return result.stdout.strip()
    except subprocess.CalledProcessError as e: return f"ERROR: Command failed: {e.stderr.strip()}"
    except subprocess.TimeoutExpired: return f"ERROR: Command timed out."
    except FileNotFoundError: return f"ERROR: Command not found."
    except Exception as e: return f"ERROR: An unexpected error occurred: {e}"

# --- Process Listing Functions ---
def get_processes_list():
    # ... (same as above) ...
    processes = []
    system = platform.system()
    if system == "Linux" or system == "Darwin":
        ps_output = run_command(["ps", "aux"])
        if not ps_output.startswith("ERROR"):
            lines = ps_output.splitlines()
            if len(lines) > 0:
                for line in lines[1:]:
                    parts = line.split(None, 10)
                    if len(parts) >= 11:
                        try:
                            processes.append({
                                "user": parts[0], "pid": int(parts[1]), "cpu_percent": float(parts[2]),
                                "mem_percent": float(parts[3]), "vsz": parts[4], "rss": parts[5],
                                "tty": parts[6], "stat": parts[7], "start": parts[8], "time": parts[9],
                                "command": " ".join(parts[10:]).strip()
                            })
                        except ValueError: pass
        else: processes.append({"error": ps_output})
    elif system == "Windows":
        tasklist_output = run_command(["tasklist", "/FO", "CSV", "/NH"])
        if not tasklist_output.startswith("ERROR"):
            for line in tasklist_output.splitlines():
                try:
                    parts = line.strip().strip('"').split('","')
                    if len(parts) >= 6:
                        processes.append({
                            "name": parts[0], "pid": int(parts[1]), "session_name": parts[2],
                            "session_id": int(parts[3]), "mem_usage_raw": parts[4], "status": parts[5],
                        })
                except (ValueError, IndexError): pass
        else: processes.append({"error": tasklist_output})
    return processes

def filter_processes(process_list, name_contains=None, user=None, pid=None):
    # ... (same as above) ...
    filtered = []
    for proc in process_list:
        match = True
        if name_contains and name_contains.lower() not in proc.get("name", "").lower() and 
           name_contains.lower() not in proc.get("command", "").lower(): match = False
        if user and proc.get("user") != user: match = False
        if pid and proc.get("pid") != pid: match = False
        if match: filtered.append(proc)
    return filtered

def get_top_cpu_processes(count=5):
    # ... (same as above) ...
    processes = []
    system = platform.system()
    if system == "Linux" or system == "Darwin":
        ps_output = run_command(["ps", "aux", "--sort=-%cpu"])
        if not ps_output.startswith("ERROR"):
            lines = ps_output.splitlines()
            if len(lines) > 0:
                for line in lines[1:count+1]:
                    parts = line.split(None, 10)
                    if len(parts) >= 11:
                        processes.append({
                            "user": parts[0], "pid": int(parts[1]), "cpu_percent": float(parts[2]),
                            "mem_percent": float(parts[3]), "command": " ".join(parts[10:]).strip()
                        })
    elif system == "Windows": processes.append({"error": "Top CPU processes parsing for Windows requires more complex logic or psutil."})
    return processes

# --- Process Control Functions ---
def terminate_process_by_pid(pid, force=False):
    # ... (same as above) ...
    system = platform.system()
    try:
        if system == "Windows":
            command = ["taskkill", "/PID", str(pid)]
            if force: command.insert(1, "/F")
            result = run_command(command)
            if "SUCCESS" in result: return f"Process {pid} terminated."
            else: return f"Error terminating process {pid}: {result}"
        else:
            signal_type = signal.SIGKILL if force else signal.SIGTERM
            os.kill(pid, signal_type)
            return f"Process {pid} terminated."
    except ProcessLookupError: return f"Error: Process {pid} not found."
    except PermissionError: return f"Error: Permission denied to terminate process {pid}."
    except Exception as e: return f"Error terminating process {pid}: {e}"

def start_new_process(command_args, run_in_background=False):
    # ... (same as above) ...
    try:
        if run_in_background:
            process = subprocess.Popen(command_args, stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL, preexec_fn=os.setsid if platform.system() != "Windows" else None)
            return f"Process started in background with PID: {process.pid}"
        else:
            result = subprocess.run(command_args, capture_output=True, text=True, check=True)
            return f"Process completed. Output:
{result.stdout}"
    except FileNotFoundError: return f"Error: Command '{command_args[0]}' not found."
    except Exception as e: return f"Error starting process: {e}"

# --- Main Logic ---
def main():
    parser = argparse.ArgumentParser(description="Cross-Platform Process Management Utility.")
    subparsers = parser.add_subparsers(dest='command', help='Available commands')

    # List command
    list_parser = subparsers.add_parser('list', help='List running processes.')
    list_parser.add_argument('--name', type=str, help='Filter by process name (substring).')
    list_parser.add_argument('--user', type=str, help='Filter by user.')
    list_parser.add_argument('--pid', type=int, help='Filter by PID.')
    list_parser.add_argument('--top-cpu', type=int, help='List top N CPU consuming processes.')
    list_parser.add_argument('--top-mem', type=int, help='List top N memory consuming processes (not fully implemented in stdlib).')

    # Kill command
    kill_parser = subparsers.add_parser('kill', help='Terminate a process.')
    kill_parser.add_argument('pid', type=int, help='PID of the process to terminate.')
    kill_parser.add_argument('--force', action='store_true', help='Force termination (SIGKILL/taskkill /F).')

    # Start command
    start_parser = subparsers.add_parser('start', help='Start a new process.')
    start_parser.add_argument('command_args', nargs=argparse.REMAINDER, help='Command and its arguments to run.')
    start_parser.add_argument('--background', action='store_true', help='Run process in the background.')

    args = parser.parse_args()

    if args.command == 'list':
        if args.top_cpu:
            results = get_top_cpu_processes(args.top_cpu)
            print(json.dumps(results, indent=2))
        elif args.top_mem:
            print("Top memory processes not fully supported by standard library parsing on all OS. Consider `psutil` or platform-specific parsing.")
            # Similar logic as top_cpu, but sorting by mem_percent or RSS
        else:
            all_procs = get_processes_list()
            filtered_procs = filter_processes(all_procs, args.name, args.user, args.pid)
            print(json.dumps(filtered_procs, indent=2))
    elif args.command == 'kill':
        if args.pid:
            print(terminate_process_by_pid(args.pid, args.force))
        else:
            parser.error("PID is required for 'kill' command.")
    elif args.command == 'start':
        if args.command_args:
            print(start_new_process(args.command_args, args.background))
        else:
            parser.error("Command and arguments are required for 'start' command.")

if __name__ == "__main__":
    main()
```

## Guiding Principles in Python

*   **Portability:** Python's `platform` module helps identify the OS, allowing for conditional execution of OS-specific commands (`ps` vs `tasklist`, `os.kill` vs `taskkill`). The `subprocess` module is cross-platform.
*   **Efficiency:** Executing native commands via `subprocess` is efficient. Python's overhead for parsing is generally minimal.
*   **Minimal Dependencies:** This tutorial primarily uses Python's standard library. The `os`, `subprocess`, `json`, `re`, `signal`, `platform`, `sys`, and `argparse` modules are all standard.
*   **CLI-centric:** The script uses `argparse` for robust command-line argument handling, making it a flexible and user-friendly CLI tool.
*   **Structured Data Handling:** Results are collected into Python dictionaries and lists, which are easily serialized to JSON, providing a clean, machine-readable output for further processing.

## Conclusion

Python provides a powerful and flexible environment for process management and automation. By leveraging its standard library modules for interacting with the operating system and parsing command-line tool output, you can create highly effective, portable, and minimally dependent process management tools. The structured nature of Python's data handling and output makes it ideal for integrating process monitoring into automated workflows. The next step is to apply this knowledge in practical exercises.
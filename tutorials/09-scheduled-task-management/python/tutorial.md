# Python Tutorial: Scheduled Task/Job Management

## Introduction

Python, through its powerful `subprocess` module, provides a cross-platform way to interact with the underlying operating system's scheduled task mechanisms. This allows for programmatically listing, creating, modifying, and deleting scheduled jobs (e.g., cron jobs on Linux, Task Scheduler entries on Windows). This tutorial will focus on using `subprocess` to manage these tasks, adhering to our principles of minimal dependencies and CLI-centric operation, acknowledging that the commands executed will be OS-specific.

## Framework Alignment

This tutorial on "**Scheduled Task/Job Management**" using **Python** demonstrates the foundational skills necessary to implement components of a "Cross-Platform Baseline Hardening & Auditing Framework." The techniques learned here for managing scheduled tasks are directly applicable to auditing systems against defined security baselines and ensuring compliance.


## Core Python Modules for Scheduled Task Management

*   **`subprocess`**: The primary module for running external commands and interacting with their input/output.
*   **`os`**: Provides a way of using operating system dependent functionality, useful for path manipulation and environmental checks.
*   **`sys`**: Provides access to system-specific parameters and functions, like `sys.platform` for OS detection.
*   **`re`**: Regular expression operations, useful for parsing command output.

## OS-Specific Tools Covered

*   **Linux/macOS**: `crontab` (for user cron jobs), potentially `systemctl` for systemd timers.
*   **Windows**: `schtasks` (for Task Scheduler).

## Implementing Core Functionality with Python

### 1. Helper Function for Running Commands

```python
import subprocess
import os
import sys
import re
import json
from datetime import datetime

def run_command(command, shell=False, check_output=True, capture_output=True):
    """
    Helper function to run shell commands safely.
    Returns (stdout, stderr, returncode)
    """
    try:
        if capture_output:
            result = subprocess.run(command, shell=shell, check=check_output,
                                    capture_output=True, text=True, errors='ignore')
            return result.stdout, result.stderr, result.returncode
        else:
            result = subprocess.run(command, shell=shell, check=check_output)
            return None, None, result.returncode
    except subprocess.CalledProcessError as e:
        print(f"Command failed with exit code {e.returncode}: {e.cmd}", file=sys.stderr)
        if capture_output:
            print(f"STDOUT: {e.stdout}", file=sys.stderr)
            print(f"STDERR: {e.stderr}", file=sys.stderr)
        return e.stdout, e.stderr, e.returncode
    except FileNotFoundError:
        print(f"Error: Command '{command[0]}' not found. Is it in your PATH?", file=sys.stderr)
        return None, "Command not found", 127
    except Exception as e:
        print(f"An unexpected error occurred: {e}", file=sys.stderr)
        return None, str(e), 1
```

### 2. Linux (Cron Job Management)

#### a. List all cron jobs for the current user

```python
def list_cron_jobs():
    """Lists cron jobs for the current user."""
    stdout, stderr, rc = run_command(["crontab", "-l"])
    if rc == 0:
        return stdout.strip().split('
')
    elif "no crontab for" in stderr.lower():
        return []
    else:
        print(f"Error listing cron jobs: {stderr}", file=sys.stderr)
        return None

# Example Usage:
# print("
--- Current Cron Jobs (Linux) ---")
# cron_jobs = list_cron_jobs()
# if cron_jobs is not None:
#     for job in cron_jobs:
#         print(job)
```

#### b. Add/Modify a cron job

Programmatically adding/modifying cron jobs usually involves retrieving the current crontab, adding/modifying a line, and then pushing it back.

```python
def add_cron_job(cron_entry, check_exists=True):
    """Adds a cron job for the current user."""
    current_crontab_lines = list_cron_jobs()
    if current_crontab_lines is None:
        return False

    if check_exists and cron_entry in current_crontab_lines:
        print(f"Cron job already exists: {cron_entry}", file=sys.stderr)
        return True

    # Create a new crontab content
    new_crontab_content = "
".join(current_crontab_lines + [cron_entry])
    
    # Use subprocess to pipe the new content to crontab
    stdout, stderr, rc = run_command(["crontab", "-"], input=new_crontab_content)
    if rc == 0:
        print(f"Cron job added: {cron_entry}")
        return True
    else:
        print(f"Error adding cron job: {stderr}", file=sys.stderr)
        return False

# Example Usage:
# if sys.platform.startswith('linux') or sys.platform == 'darwin':
#     test_cron_job = "* * * * * echo 'Hello from Python cron' >> /tmp/python_cron_test.log"
#     add_cron_job(test_cron_job)
#     print("
Verify with 'crontab -l'")
```

#### c. Delete a cron job

```python
def delete_cron_job(cron_entry):
    """Deletes a cron job for the current user."""
    current_crontab_lines = list_cron_jobs()
    if current_crontab_lines is None:
        return False
    
    if cron_entry not in current_crontab_lines:
        print(f"Cron job not found: {cron_entry}", file=sys.stderr)
        return False
    
    # Filter out the job to delete
    filtered_crontab_lines = [line for line in current_crontab_lines if line != cron_entry]
    new_crontab_content = "
".join(filtered_crontab_lines)

    # If all jobs are removed, use crontab -r
    if not new_crontab_content.strip():
        stdout, stderr, rc = run_command(["crontab", "-r"], check_output=False)
        if rc == 0:
            print(f"All cron jobs deleted for user. Removed: {cron_entry}")
            return True
        else:
            print(f"Error deleting all cron jobs: {stderr}", file=sys.stderr)
            return False
    else:
        stdout, stderr, rc = run_command(["crontab", "-"], input=new_crontab_content)
        if rc == 0:
            print(f"Cron job deleted: {cron_entry}")
            return True
        else:
            print(f"Error deleting cron job: {stderr}", file=sys.stderr)
            return False

# Example Usage:
# if sys.platform.startswith('linux') or sys.platform == 'darwin':
#     delete_cron_job(test_cron_job)
#     print("
Verify with 'crontab -l'")
```

### 3. Windows (Task Scheduler Management using `schtasks`)

#### a. List all scheduled tasks

```python
def list_windows_scheduled_tasks():
    """Lists all scheduled tasks on Windows."""
    stdout, stderr, rc = run_command(["schtasks", "/query", "/fo", "CSV", "/nh"])
    tasks = []
    if rc == 0 and stdout:
        lines = stdout.strip().split('
')
        for line in lines:
            parts = line.split('","')
            if len(parts) >= 2:
                folder = parts[0].strip('"')
                name = parts[1].strip('"')
                full_path = os.path.join(folder, name) if folder != "" else name
                tasks.append(full_path)
    elif stderr and "No tasks are currently running" not in stderr:
        print(f"Error listing tasks: {stderr}", file=sys.stderr)
    return tasks

# Example Usage:
# if sys.platform == 'win32':
#     print("
--- Current Windows Scheduled Tasks ---")
#     win_tasks = list_windows_scheduled_tasks()
#     for task in win_tasks:
#         print(task)
```

#### b. Create a scheduled task

```python
def create_windows_scheduled_task(task_name, command, schedule="DAILY", start_time="02:00", description=""):
    """Creates a daily scheduled task on Windows."""
    # Ensure command and start_time are quoted if they contain spaces
    cmd_args = [
        "schtasks",
        "/create",
        "/tn", task_name,
        "/tr", command, # Task Run
        "/sc", schedule, # Schedule Type (DAILY, WEEKLY, MONTHLY, ONCE, etc.)
        "/st", start_time, # Start Time (HH:MM)
        "/ru", "SYSTEM", # Run as SYSTEM user
        "/rp", "", # No password for SYSTEM
        "/f" # Force creation if task exists
    ]
    if description:
        cmd_args.extend(["/d", description])

    stdout, stderr, rc = run_command(cmd_args)
    if rc == 0:
        print(f"Scheduled task '{task_name}' created.")
        return True
    else:
        print(f"Error creating task '{task_name}': {stderr}", file=sys.stderr)
        return False

# Example Usage:
# if sys.platform == 'win32':
#     test_task_name = "MyPythonTestTask"
#     test_command = "powershell.exe -Command "Write-Host 'Hello from Python Scheduled Task!' | Out-File C:	emp\python_task_log.txt -Append""
#     create_windows_scheduled_task(test_task_name, test_command, description="A test task created by Python.")
#     print("
Verify with 'schtasks /query /tn MyPythonTestTask'")
```

#### c. Delete a scheduled task

```python
def delete_windows_scheduled_task(task_name):
    """Deletes a scheduled task on Windows."""
    stdout, stderr, rc = run_command(["schtasks", "/delete", "/tn", task_name, "/f"])
    if rc == 0:
        print(f"Scheduled task '{task_name}' deleted.")
        return True
    else:
        # rc will be 1 if task not found, schtasks returns an error message to stdout sometimes
        if "ERROR: The system cannot find the file specified." in stderr or "ERROR: The specified task not found." in stdout:
            print(f"Scheduled task '{task_name}' not found.", file=sys.stderr)
            return False
        else:
            print(f"Error deleting task '{task_name}': {stderr}", file=sys.stderr)
            return False

# Example Usage:
# if sys.platform == 'win32':
#     delete_windows_scheduled_task(test_task_name)
#     print("
Verify with 'schtasks /query | findstr MyPythonTestTask' (should not show up)")
```

### 4. Consolidated Python Script (Cross-Platform Task Manager)

```python
#!/usr/bin/env python3

import subprocess
import os
import sys
import re
import json
import argparse
from datetime import datetime

# --- run_command helper function (as defined above) ---
def run_command(command, shell=False, check_output=True, capture_output=True, input=None):
    """
    Helper function to run shell commands safely.
    Returns (stdout, stderr, returncode)
    """
    try:
        # input is only for text=True
        if input is not None and not capture_output:
            raise ValueError("Cannot provide input if capture_output is False.")
            
        result = subprocess.run(command, shell=shell, check=check_output,
                                capture_output=capture_output, text=True, errors='ignore',
                                input=input)
        return result.stdout, result.stderr, result.returncode
    except subprocess.CalledProcessError as e:
        print(f"Command failed with exit code {e.returncode}: {e.cmd}", file=sys.stderr)
        if capture_output:
            print(f"STDOUT: {e.stdout}", file=sys.stderr)
            print(f"STDERR: {e.stderr}", file=sys.stderr)
        return e.stdout, e.stderr, e.returncode
    except FileNotFoundError:
        print(f"Error: Command '{command[0]}' not found. Is it in your PATH?", file=sys.stderr)
        return None, "Command not found", 127
    except Exception as e:
        print(f"An unexpected error occurred: {e}", file=sys.stderr)
        return None, str(e), 1

# --- Linux Cron Job Management (as defined above) ---
def list_cron_jobs():
    """Lists cron jobs for the current user."""
    stdout, stderr, rc = run_command(["crontab", "-l"])
    if rc == 0: return stdout.strip().split('
')
    elif "no crontab for" in stderr.lower(): return []
    else: print(f"Error listing cron jobs: {stderr}", file=sys.stderr); return None

def add_cron_job(cron_entry, check_exists=True):
    """Adds a cron job for the current user."""
    current_crontab_lines = list_cron_jobs()
    if current_crontab_lines is None: return False
    if check_exists and cron_entry in current_crontab_lines:
        print(f"Cron job already exists: {cron_entry}", file=sys.stderr); return True
    new_crontab_content = "
".join(current_crontab_lines + [cron_entry])
    stdout, stderr, rc = run_command(["crontab", "-"], input=new_crontab_content)
    if rc == 0: print(f"Cron job added: {cron_entry}"); return True
    else: print(f"Error adding cron job: {stderr}", file=sys.stderr); return False

def delete_cron_job(cron_entry):
    """Deletes a cron job for the current user."""
    current_crontab_lines = list_cron_jobs()
    if current_crontab_lines is None: return False
    if cron_entry not in current_crontab_lines:
        print(f"Cron job not found: {cron_entry}", file=sys.stderr); return False
    filtered_crontab_lines = [line for line in current_crontab_lines if line != cron_entry]
    new_crontab_content = "
".join(filtered_crontab_lines)
    if not new_crontab_content.strip():
        stdout, stderr, rc = run_command(["crontab", "-r"], check_output=False)
        if rc == 0: print(f"All cron jobs deleted for user. Removed: {cron_entry}"); return True
        else: print(f"Error deleting all cron jobs: {stderr}", file=sys.stderr); return False
    else:
        stdout, stderr, rc = run_command(["crontab", "-"], input=new_crontab_content)
        if rc == 0: print(f"Cron job deleted: {cron_entry}"); return True
        else: print(f"Error deleting cron job: {stderr}", file=sys.stderr); return False

# --- Windows Task Scheduler Management (as defined above) ---
def list_windows_scheduled_tasks():
    """Lists all scheduled tasks on Windows."""
    stdout, stderr, rc = run_command(["schtasks", "/query", "/fo", "CSV", "/nh"])
    tasks = []
    if rc == 0 and stdout:
        lines = stdout.strip().split('
')
        for line in lines:
            parts = line.split('","')
            if len(parts) >= 2:
                folder = parts[0].strip('"')
                name = parts[1].strip('"')
                full_path = os.path.join(folder, name) if folder != "" else name
                tasks.append(full_path)
    elif stderr and "No tasks are currently running" not in stderr:
        print(f"Error listing tasks: {stderr}", file=sys.stderr)
    return tasks

def create_windows_scheduled_task(task_name, command, schedule="DAILY", start_time="02:00", description=""):
    """Creates a daily scheduled task on Windows."""
    cmd_args = [
        "schtasks", "/create", "/tn", task_name, "/tr", command, "/sc", schedule, "/st", start_time,
        "/ru", "SYSTEM", "/rp", "", "/f"
    ]
    if description: cmd_args.extend(["/d", description])
    stdout, stderr, rc = run_command(cmd_args)
    if rc == 0: print(f"Scheduled task '{task_name}' created."); return True
    else: print(f"Error creating task '{task_name}': {stderr}", file=sys.stderr); return False

def delete_windows_scheduled_task(task_name):
    """Deletes a scheduled task on Windows."""
    stdout, stderr, rc = run_command(["schtasks", "/delete", "/tn", task_name, "/f"])
    if rc == 0: print(f"Scheduled task '{task_name}' deleted."); return True
    else:
        if "ERROR: The system cannot find the file specified." in stderr or "ERROR: The specified task not found." in stdout:
            print(f"Scheduled task '{task_name}' not found.", file=sys.stderr); return False
        else: print(f"Error deleting task '{task_name}': {stderr}", file=sys.stderr); return False

def enable_windows_scheduled_task(task_name):
    stdout, stderr, rc = run_command(["schtasks", "/change", "/tn", task_name, "/enable"])
    if rc == 0: print(f"Task '{task_name}' enabled."); return True
    else: print(f"Error enabling task '{task_name}': {stderr}", file=sys.stderr); return False

def disable_windows_scheduled_task(task_name):
    stdout, stderr, rc = run_command(["schtasks", "/change", "/tn", task_name, "/disable"])
    if rc == 0: print(f"Task '{task_name}' disabled."); return True
    else: print(f"Error disabling task '{task_name}': {stderr}", file=sys.stderr); return False

def run_windows_scheduled_task(task_name):
    stdout, stderr, rc = run_command(["schtasks", "/run", "/tn", task_name])
    if rc == 0: print(f"Task '{task_name}' initiated."); return True
    else: print(f"Error running task '{task_name}': {stderr}", file=sys.stderr); return False


def main():
    parser = argparse.ArgumentParser(description="Cross-platform scheduled task manager.")
    parser.add_argument("action", choices=['list', 'add', 'delete', 'enable', 'disable', 'run'], help="Action to perform.")
    parser.add_argument("--name", help="Name of the scheduled task/cron job.")
    parser.add_argument("--command", help="Command or script to execute (for 'add' action).")
    parser.add_argument("--schedule", default="DAILY", help="Schedule type (e.g., DAILY, WEEKLY, or cron string for Linux).")
    parser.add_argument("--start-time", default="02:00", help="Start time for tasks (HH:MM).")
    parser.add_argument("--description", help="Description for the task.")
    
    args = parser.parse_args()

    platform = sys.platform

    if args.action == 'list':
        if platform.startswith('linux') or platform == 'darwin':
            print("--- Linux/macOS Cron Jobs ---")
            jobs = list_cron_jobs()
            for job in jobs: print(job)
        elif platform == 'win32':
            print("--- Windows Scheduled Tasks ---")
            tasks = list_windows_scheduled_tasks()
            for task in tasks: print(task)
        else:
            print(f"Unsupported platform: {platform}")
            sys.exit(1)

    elif args.action == 'add':
        if not args.name or not args.command:
            parser.error("--name and --command are required for 'add' action.")
        if platform.startswith('linux') or platform == 'darwin':
            add_cron_job(f"{args.schedule} {args.command}") # Schedule is a raw cron string
        elif platform == 'win32':
            create_windows_scheduled_task(args.name, args.command, args.schedule, args.start_time, args.description)
        else:
            print(f"Unsupported platform: {platform}")
            sys.exit(1)

    elif args.action == 'delete':
        if not args.name:
            parser.error("--name is required for 'delete' action.")
        if platform.startswith('linux') or platform == 'darwin':
            # This would require finding the specific line by name, which is harder for cron.
            # For simplicity, if --name is given, try to find a job with this name in command part.
            # More robust way involves parsing crontab. Here, we'll ask for full entry.
            print("For Linux 'delete', please provide the exact full cron entry as --name.")
            delete_cron_job(args.name)
        elif platform == 'win32':
            delete_windows_scheduled_task(args.name)
        else:
            print(f"Unsupported platform: {platform}")
            sys.exit(1)

    elif args.action == 'enable':
        if not args.name: parser.error("--name is required for 'enable' action.")
        if platform == 'win32': enable_windows_scheduled_task(args.name)
        else: print(f"Enable action only supported for Windows tasks on {platform}", file=sys.stderr)
    elif args.action == 'disable':
        if not args.name: parser.error("--name is required for 'disable' action.")
        if platform == 'win32': disable_windows_scheduled_task(args.name)
        else: print(f"Disable action only supported for Windows tasks on {platform}", file=sys.stderr)
    elif args.action == 'run':
        if not args.name: parser.error("--name is required for 'run' action.")
        if platform == 'win32': run_windows_scheduled_task(args.name)
        else: print(f"Run action only supported for Windows tasks on {platform}", file=sys.stderr)

if __name__ == "__main__":
    main()
```
To run:
*   Linux: `python task_manager.py list`
*   Windows: `python task_manager.py list`
*   Linux: `python task_manager.py add --name "my_cron_job" --command "echo 'Hello' >> /tmp/test.log" --schedule "* * * * *"`
*   Windows: `python task_manager.py add --name "MyPythonTask" --command "powershell.exe -Command "Write-Host 'Hello from Python!'"" --schedule DAILY --start-time "09:00"`

## Guiding Principles in Python

*   **Portability:** Python acts as a wrapper, executing OS-native commands. The `sys.platform` check allows for dynamic command selection, making the Python script itself cross-platform, even if the underlying scheduling mechanisms are not.
*   **Efficiency:** Direct calls to `crontab` or `schtasks` are efficient as they interact directly with the OS schedulers.
*   **Minimal Dependencies:** Relies almost entirely on Python's standard library (`subprocess`, `os`, `sys`, `re`, `json`, `argparse`).
*   **CLI-centric:** The script uses `argparse` to provide a flexible command-line interface, making it suitable for scripting and automation.
*   **Structured Data Handling:** Python can parse the output of these commands (especially when requesting CSV or XML output from `schtasks`) and present it in a structured way (e.g., JSON), which is crucial for integration into larger systems.

## Conclusion

Python, by effectively leveraging its `subprocess` module, serves as a versatile tool for managing scheduled tasks and jobs across both Linux/macOS and Windows environments. While it abstracts the OS differences at the script level, it uses the native, efficient scheduling mechanisms of each platform. This approach enables the creation of powerful, automated administration scripts that can list, create, modify, and delete tasks, thereby enhancing system reliability and operational efficiency. The next step is to apply this knowledge in practical exercises.
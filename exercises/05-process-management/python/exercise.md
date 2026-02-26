# Python Exercise: Process Management and Automation

## Objective

This exercise challenges you to apply your Python scripting skills to manage and automate processes on a cross-platform system. You will use Python's standard library (`os`, `subprocess`, `signal`) to list, filter, monitor, start, and terminate processes, demonstrating proficiency in system administration, troubleshooting, and structured data handling.

## Framework Alignment

This exercise on "**Process Management and Automation**" using **Python** provides practical experience in building and utilizing tools for a "Cross-Platform Baseline Hardening & Auditing Framework." By completing these tasks, you will learn how to audit and manage running processes, ensuring that only authorized services are active and identifying potential security risks or unauthorized activity—essential steps in maintaining a secure and auditable environment.


## Scenario

You are tasked with developing a portable Python script to monitor and control processes on various operating systems (Linux, macOS, Windows). Your script needs to:
1.  List and filter processes based on various criteria.
2.  Identify processes consuming significant resources.
3.  Be able to start and terminate processes.
4.  Output collected process data in a structured JSON format.

## Setup

For tasks involving process control, you will need to start some dummy processes. For example:
*   **Linux/macOS:** `sleep 600 &` or a simple Python script like `python -c "import time; time.sleep(600)" &`
*   **Windows:** `start /B notepad.exe` or `powershell.exe -NoProfile -Command "Start-Sleep -Seconds 600"`

## Tasks

Write a Python script (`process_manager.py` or similar name) that, when executed, can perform the following tasks. Your script should be structured with functions for clarity and output results in a structured JSON format where appropriate.

### Part 1: Process Listing and Monitoring

1.  **List All Running Processes:**
    *   Implement a function `get_all_processes()` that uses `subprocess` to run `ps aux` (Unix-like) or `tasklist` (Windows) and parses its output to return a list of dictionaries, where each dictionary represents a process and contains at least: `pid`, `name` (or `command`), `user`, `cpu_percent`, `mem_percent`.

2.  **Filter Processes by Name:**
    *   Implement a function `filter_processes_by_name(process_list, name_pattern)` that filters the list of processes obtained from `get_all_processes()` based on a name substring (case-insensitive).

3.  **Find Processes by PID:**
    *   Implement a function `get_process_by_pid(process_list, pid)` that returns the dictionary for a specific process ID.

4.  **Identify Top 5 CPU-Consuming Processes:**
    *   Implement a function `get_top_cpu_processes(process_list, count=5)` that sorts the process list by `cpu_percent` (descending) and returns the top `count` processes.

5.  **Identify Top 5 Memory-Consuming Processes:**
    *   Implement a function `get_top_mem_processes(process_list, count=5)` that sorts the process list by `mem_percent` (descending) and returns the top `count` processes.

6.  **Export Process List as JSON:**
    *   Your script's main execution should be able to print the results of any of the above listing/filtering tasks as a JSON array to `stdout`.

### Part 2: Process Control

Implement functions in your script for process control.

1.  **Start a Dummy Background Process:**
    *   Implement `start_background_process(command_args)` that starts a process (e.g., `sleep 600` or `notepad.exe`) in the background and returns its PID.

2.  **Terminate Process by PID (Graceful):**
    *   Implement `terminate_process_by_pid(pid)` that attempts to gracefully terminate a process using its PID.

3.  **Terminate Process by PID (Forceful):**
    *   Implement `force_terminate_process_by_pid(pid)` that forcefully terminates a process using its PID.

4.  **Terminate Processes by Name:**
    *   Implement `terminate_processes_by_name(name_pattern, force=False)` that finds all processes matching `name_pattern` and terminates them.

## Deliverables

Provide the complete Python script file (`process_manager.py`) that implements all the above tasks. Your script should be executable from the command line, possibly using `argparse` to select which function to run and what parameters to use (e.g., `python process_manager.py list --top-cpu 5`).

## Reflection Questions

1.  How did Python's `subprocess` module facilitate running native system commands (`ps`, `tasklist`) and parsing their output? What were the challenges?
2.  Explain the cross-platform considerations you had to make for process listing and termination (e.g., `ps aux` vs `tasklist`, `os.kill` vs `taskkill`).
3.  Describe how returning process information as Python dictionaries/objects and then serializing to JSON simplifies further programmatic analysis compared to parsing raw text.
4.  If you were to use a third-party library like `psutil`, how would it simplify process information retrieval and control, especially for cross-platform compatibility?
5.  What are the advantages and disadvantages of using Python for process management and automation compared to Bash, PowerShell, or a database like SQLite?

---
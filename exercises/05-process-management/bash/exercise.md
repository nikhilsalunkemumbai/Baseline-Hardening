# Bash Exercise: Process Management and Automation

## Objective

This exercise challenges you to apply your Bash scripting skills to manage and automate processes on a Unix-like system. You will use standard command-line utilities to list, filter, monitor, start, and terminate processes, demonstrating proficiency in system administration and troubleshooting.

## Framework Alignment

This exercise on "**Process Management and Automation**" using **Bash** provides practical experience in building and utilizing tools for a "Cross-Platform Baseline Hardening & Auditing Framework." By completing these tasks, you will learn how to audit and manage running processes, ensuring that only authorized services are active and identifying potential security risks or unauthorized activity—essential steps in maintaining a secure and auditable environment.


## Scenario

You are a system administrator monitoring a server. You need to quickly identify running processes, check for resource hogs, and be able to control (start/stop) certain applications. You will simulate some processes to practice these tasks.

## Tasks

Using only standard Bash commands and utilities (e.g., `ps`, `pgrep`, `pkill`, `kill`, `top`, `nohup`, `&`), provide the command-line solution or a simple script for each of the following tasks.

### Part 1: Process Listing and Monitoring

1.  **List All Running Processes:**
    *   Display a comprehensive list of all running processes, showing at least: PID, PPID, User, %CPU, %MEM, and Command.

2.  **Filter Processes by Name:**
    *   Find all processes whose command line contains "bash" (case-insensitive). Exclude the `grep` process itself from the results.

3.  **Find Processes by User:**
    *   List all processes currently running under your own user account.

4.  **Identify Top 5 CPU-Consuming Processes:**
    *   Display the top 5 processes currently consuming the most CPU, showing their PID, User, %CPU, and Command.

5.  **Identify Top 5 Memory-Consuming Processes:**
    *   Display the top 5 processes currently consuming the most memory, showing their PID, User, %MEM, and Command.

### Part 2: Process Control

For these tasks, you will need to start some dummy processes to control.

1.  **Start a Dummy Background Process:**
    *   Start a `sleep 600` command in the background. Note its PID. (Hint: use `nohup` or `&`).

2.  **Verify Background Process:**
    *   Confirm that your `sleep 600` process is running in the background.

3.  **Terminate Process by PID (Graceful):**
    *   Gracefully terminate the `sleep 600` process you started using its PID.

4.  **Start Multiple Dummy Background Processes:**
    *   Start two separate `sleep 300` commands in the background.

5.  **Terminate Processes by Name (Graceful):**
    *   Terminate all currently running `sleep` processes by their name using a single command.

6.  **Simulate a Hanging Process & Force Terminate:**
    *   Start a process that ignores `SIGTERM` (a simple `while true; do echo "running"; sleep 1; done` in a subshell, or `sleep 600` again).
    *   Attempt to gracefully terminate it. Observe that it doesn't stop.
    *   Then, forcefully terminate it.

## Deliverables

For each task, provide the single Bash command-line pipeline or script snippet that produces the required output or performs the action.

## Reflection Questions

1.  Compare `ps aux` and `ps -ef`. When would you choose one over the other for listing processes?
2.  Explain the difference between `kill` and `kill -9` (SIGTERM vs. SIGKILL). Why is it generally recommended to try `SIGTERM` first?
3.  How do `pgrep` and `pkill` simplify finding and terminating processes by name compared to combining `ps` and `grep`?
4.  Describe a scenario where starting a process with `nohup` would be crucial.
5.  What are the challenges of performing process management in Bash on a Windows system? How would your approach change?

---
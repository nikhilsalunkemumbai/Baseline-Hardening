# Python Exercise: Scheduled Task/Job Management

## Objective

This exercise challenges you to apply your Python scripting skills to manage scheduled tasks across different operating systems (Linux/macOS cron and Windows Task Scheduler). You will create a cross-platform Python script that can list, add, enable/disable, run, and delete scheduled jobs, demonstrating proficiency in automating system administration tasks in heterogeneous environments.

## Framework Alignment

This exercise on "**Scheduled Task/Job Management**" using **Python** provides practical experience in building and utilizing tools for a "Cross-Platform Baseline Hardening & Auditing Framework." By completing these tasks, you will learn how to audit and manage scheduled tasks, ensuring compliance with security policies and preventing unauthorized persistence mechanisms—essential steps in maintaining a secure and auditable environment.


## Scenario

You are a DevOps engineer managing a mixed environment of Linux and Windows servers. You need a unified way to automate the setup and verification of recurring maintenance tasks. Your goal is to develop a Python utility that abstracts away some of the OS-specific commands for managing scheduled jobs.

## Setup

1.  **Create a dummy script for your scheduled task:**
    Create a file named `dummy_scheduled_script.py` with the following content:
    ```python
    import datetime
    import sys
    import os

    # Determine a writable log file path based on OS
    if sys.platform == 'win32':
        log_dir = os.environ.get('TEMP', 'C:\Temp') # Use TEMP env var or fallback
    else:
        log_dir = '/tmp'
    
    log_file_path = os.path.join(log_dir, f"scheduled_run_{sys.platform}.log")

    with open(log_file_path, "a") as f:
        f.write(f"Scheduled script ran at {datetime.datetime.now()} on {sys.platform}
")
    ```
    Ensure the directory for the log file is writable by the user that will execute the scheduled task. For Windows, `C:\Temp` or the system TEMP directory is common.

## Tasks

Write a single Python script (`task_manager.py`) that uses `sys.platform` to detect the operating system and then performs the following tasks. Your script should be executable from the command line and output a structured JSON report of all tasks found on the system.

### Part 1: Cross-Platform Listing

1.  **Detect OS and List Tasks:**
    *   Implement logic to detect if the script is running on Linux/macOS or Windows.
    *   If Linux/macOS, list all cron jobs for the current user.
    *   If Windows, list all scheduled tasks.
    *   Store the results in a structured format (e.g., a list of dictionaries).

### Part 2: Linux (Cron Specific Tasks - If OS is Linux/macOS)

1.  **Add Hourly Cron Job:**
    *   Add a new cron job that runs `python /path/to/dummy_scheduled_script.py` every hour.
    *   Set the path to `dummy_scheduled_script.py` as its absolute path.

2.  **Verify Cron Job Existence:**
    *   List cron jobs and confirm the newly added job is present.

3.  **Delete Hourly Cron Job:**
    *   Remove the hourly cron job you just created.

### Part 3: Windows (Task Scheduler Specific Tasks - If OS is Windows)

1.  **Create Daily Scheduled Task:**
    *   Create a new scheduled task named `PythonDailyScript`.
    *   This task should run `python <PATH_TO_SCRIPT>\dummy_scheduled_script.py` daily at `03:00 AM`.
    *   Set the path to `dummy_scheduled_script.py` as its absolute path.
    *   It should run as `SYSTEM`.

2.  **Verify Scheduled Task Existence:**
    *   List scheduled tasks and confirm `PythonDailyScript` is present and enabled.

3.  **Disable Scheduled Task:**
    *   Disable the `PythonDailyScript` task.

4.  **Enable Scheduled Task:**
    *   Re-enable the `PythonDailyDailyScript` task.

5.  **Run Scheduled Task Immediately:**
    *   Execute the `PythonDailyScript` on demand.

6.  **Delete Scheduled Task:**
    *   Remove the `PythonDailyScript` task.

### Part 4: Structured Reporting

1.  **Generate JSON Report:**
    *   After performing the relevant tasks for the detected OS, generate a JSON output that lists all currently existing scheduled tasks/cron jobs on that system, including their name, command, and status.

## Deliverables

Provide the complete Python script file (`task_manager.py`) that implements all the above tasks. Your script should be executable from the command line.

## Reflection Questions

1.  How does Python's `subprocess` module facilitate cross-platform interaction with OS-specific scheduling utilities? What are its limitations?
2.  Describe the differences in managing cron jobs (Linux/macOS) versus scheduled tasks (Windows) from a programmatic perspective (e.g., how you add/delete them).
3.  Explain the importance of specifying absolute paths for scripts run by scheduled tasks, especially in environments where the `PATH` variable might differ.
4.  How does generating a JSON report of scheduled tasks contribute to a more robust and automatable system administration workflow?
5.  What are the security implications of using Python to create or modify scheduled tasks, especially when they run with elevated privileges?

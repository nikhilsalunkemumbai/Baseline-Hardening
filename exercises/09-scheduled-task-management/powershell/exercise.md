# PowerShell Exercise: Scheduled Task/Job Management

## Objective

This exercise challenges you to apply your PowerShell scripting and cmdlet skills to manage scheduled tasks on Windows systems. You will learn to create, list, verify, modify, disable, enable, run, and delete recurring tasks using the `ScheduledTasks` module, demonstrating proficiency in automating system administration and security operations.

## Framework Alignment

This exercise on "**Scheduled Task/Job Management**" using **PowerShell** provides practical experience in building and utilizing tools for a "Cross-Platform Baseline Hardening & Auditing Framework." By completing these tasks, you will learn how to audit and manage scheduled tasks, ensuring compliance with security policies and preventing unauthorized persistence mechanisms—essential steps in maintaining a secure and auditable environment.


## Scenario

You are a system administrator responsible for several Windows servers. You need to automate two key tasks:
1.  A daily system maintenance script.
2.  Ensuring a critical security scan runs weekly.

You will use PowerShell cmdlets to interact with the Windows Task Scheduler.

## Setup

1.  **Create a dummy script for your scheduled task:**
    Create a file named `C:	emp\daily_maintenance.ps1` (or adjust path as needed) with the following content:
    ```powershell
    Add-Content -Path "C:	emp\maintenance_log.txt" -Value "Daily maintenance ran at $(Get-Date)"
    ```
    Ensure the `C:	emp` directory exists, or choose an existing writable directory.

## Tasks

Using only standard PowerShell cmdlets (from the `ScheduledTasks` module), provide the command-line solution or a simple script for each of the following tasks.

### Part 1: Managing Scheduled Tasks

1.  **List All Scheduled Tasks:**
    *   Display a list of all scheduled tasks on your system, showing their `TaskName` and `State`.

2.  **Create Daily Maintenance Task:**
    *   Create a new scheduled task named `DailyMaintenanceTask`.
    *   This task should run `C:	emp\daily_maintenance.ps1` daily at `03:00 AM`.
    *   It should run as `NT AUTHORITY\SYSTEM`.
    *   Provide a description: "Performs daily system maintenance."

3.  **Verify New Scheduled Task:**
    *   List scheduled tasks and use `Where-Object` to confirm that `DailyMaintenanceTask` has been successfully created and is enabled.

4.  **Modify Existing Task (Hypothetical):**
    *   Assume there's an existing task named `SecurityScanTask`. Modify its description to "Runs weekly security vulnerability scan." (If you don't have this task, create a dummy one for this step, e.g., running `notepad.exe` weekly, then modify its description).

5.  **Disable Daily Maintenance Task:**
    *   Disable the `DailyMaintenanceTask` without deleting it.

6.  **Enable Daily Maintenance Task:**
    *   Re-enable the `DailyMaintenanceTask`.

7.  **Run Daily Maintenance Task Immediately:**
    *   Start the `DailyMaintenanceTask` on demand, without waiting for its scheduled trigger.

8.  **Delete Daily Maintenance Task:**
    *   Remove the `DailyMaintenanceTask` completely from the Task Scheduler.

### Part 2: Structured Report of Selected Tasks

Create a single PowerShell script (`generate_task_report.ps1`) that performs the following:

1.  **List Critical Tasks:**
    *   Retrieve details for a few specific tasks (e.g., `DailyMaintenanceTask` if it exists, `Windows Defender Scheduled Scan`, `GoogleUpdateTaskMachineUA`).
2.  **Extract Key Information:**
    *   For each task, extract its `TaskName`, `State`, `LastRunTime`, `NextRunTime`, and the `Executable` and `Arguments` of its primary action.
3.  **Generate JSON Output:**
    *   Collect this information into an array of PowerShell custom objects.
    *   Convert this array into a JSON string and print it to the console.

## Deliverables

For Part 1, provide the exact PowerShell command-line solution for each task. For Part 2, provide the complete PowerShell script file (`generate_task_report.ps1`).

## Reflection Questions

1.  How does PowerShell's object-oriented nature simplify managing scheduled tasks compared to text-based parsing of `schtasks` output?
2.  Explain the purpose of `New-ScheduledTaskAction`, `New-ScheduledTaskTrigger`, and `New-ScheduledTaskPrincipal` in creating a scheduled task.
3.  What are the advantages of using `Register-ScheduledTask` over directly importing an XML task definition file?
4.  Describe how you would use the output from `generate_task_report.ps1` in an automated auditing or monitoring system.
5.  What security considerations are paramount when creating scheduled tasks, especially when specifying the `UserId`?

---
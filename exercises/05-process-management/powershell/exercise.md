# PowerShell Exercise: Process Management and Automation

## Objective

This exercise challenges you to apply your PowerShell scripting skills to manage and automate processes on a Windows (or PowerShell Core-enabled Linux/macOS) system. You will use standard PowerShell cmdlets to list, filter, monitor, start, and terminate processes, demonstrating proficiency in system administration and troubleshooting.

## Framework Alignment

This exercise on "**Process Management and Automation**" using **PowerShell** provides practical experience in building and utilizing tools for a "Cross-Platform Baseline Hardening & Auditing Framework." By completing these tasks, you will learn how to audit and manage running processes, ensuring that only authorized services are active and identifying potential security risks or unauthorized activity—essential steps in maintaining a secure and auditable environment.


## Scenario

You are a system administrator monitoring a server. You need to quickly identify running processes, check for resource hogs, and be able to control (start/stop) certain applications. You will simulate some processes to practice these tasks and aim to produce structured output for reporting.

## Tasks

Using only standard PowerShell cmdlets (e.g., `Get-Process`, `Stop-Process`, `Start-Process`, `Where-Object`, `Sort-Object`), provide the command-line solution or a simple script for each of the following tasks.

### Part 1: Process Listing and Monitoring

1.  **List All Running Processes:**
    *   Display a comprehensive list of all running processes, showing at least: `Id` (PID), `ProcessName`, `CPU` (CPU time), `WS` (Working Set memory), and `StartTime`. Format the output as a table.

2.  **Filter Processes by Name:**
    *   Find all processes whose `ProcessName` contains "chrome" (case-insensitive). Display their `Id`, `ProcessName`, and `CPU`.

3.  **Find Processes by PID:**
    *   Find details for a specific process by its `Id` (PID). Choose any running process's PID from your system. Display all properties of this process as a list.

4.  **Identify Top 5 CPU-Consuming Processes:**
    *   Display the top 5 processes currently consuming the most CPU time, showing their `Id`, `ProcessName`, and `CPU`.

5.  **Identify Top 5 Memory-Consuming Processes:**
    *   Display the top 5 processes currently consuming the most memory (Working Set - `WS`), showing their `Id`, `ProcessName`, and `WS`.

6.  **Export Process List as JSON:**
    *   Get a list of all processes and export their `Id`, `ProcessName`, `CPU`, and `WS` properties as a JSON array to the console.

### Part 2: Process Control

For these tasks, you will need to start some dummy processes to control.

1.  **Start a Dummy Background Process:**
    *   Start a Notepad process (on Windows) or a `sleep 600` process (on Linux/macOS using `Start-Process powershell.exe -ArgumentList 'sleep 600'`) in the background. Note its PID.

2.  **Verify Background Process:**
    *   Confirm that your dummy process is running in the background.

3.  **Terminate Process by PID (Graceful):**
    *   Gracefully terminate the dummy process you started using its PID.

4.  **Start Multiple Dummy Background Processes:**
    *   Start two separate Notepad processes (Windows) or `sleep 300` processes (Linux/macOS) in the background.

5.  **Terminate Processes by Name (Graceful):**
    *   Terminate all currently running instances of your dummy processes (e.g., all "notepad" processes or all "powershell" processes running `sleep`).

6.  **Simulate a Hanging Process & Force Terminate:**
    *   Start a Notepad process.
    *   Attempt to gracefully terminate it using `Stop-Process -Name "notepad"`. It might stop, but for a hanging process, it might not.
    *   Then, forcefully terminate it using `Stop-Process -Name "notepad" -Force`.

## Deliverables

For each task, provide the single PowerShell command-line pipeline or script snippet that produces the required output or performs the action.

## Reflection Questions

1.  How does PowerShell's object pipeline (`Get-Process | Where-Object | Sort-Object`) simplify process management compared to parsing text output from tools like `ps` in Bash?
2.  Explain the difference between gracefully and forcefully terminating a process (`Stop-Process` vs. `Stop-Process -Force`). When would you use each?
3.  What are the advantages of using `Start-Process -PassThru` when launching processes in the background for scripting?
4.  Describe a scenario where exporting a list of processes as JSON (Task 1.6) would be highly beneficial for automation or integration.
5.  What are the advantages and disadvantages of using PowerShell for process management compared to Bash or a Python script using `psutil`?

---
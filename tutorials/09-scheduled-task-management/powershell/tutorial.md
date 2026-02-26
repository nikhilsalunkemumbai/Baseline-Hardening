# PowerShell Tutorial: Scheduled Task/Job Management

## Introduction

PowerShell provides a comprehensive and robust set of cmdlets for managing scheduled tasks on Windows systems, interacting directly with the Task Scheduler. These cmdlets allow for programmatically listing, creating, modifying, deleting, enabling, disabling, and running scheduled tasks. This tutorial will guide you through using these powerful tools, emphasizing their structured output and seamless integration into automation workflows, while adhering to our principles of minimal dependencies and CLI-centric operation.

## Framework Alignment

This tutorial on "**Scheduled Task/Job Management**" using **PowerShell** demonstrates the foundational skills necessary to implement components of a "Cross-Platform Baseline Hardening & Auditing Framework." The techniques learned here for managing scheduled tasks are directly applicable to auditing systems against defined security baselines and ensuring compliance.


## Core PowerShell Cmdlets for Scheduled Task Management

The primary module for scheduled tasks is `ScheduledTasks`.

*   **`Get-ScheduledTask`**: Retrieves scheduled tasks on the local or remote computer.
*   **`Register-ScheduledTask`**: Creates and registers a scheduled task. This is the main cmdlet for creating new tasks.
*   **`Set-ScheduledTask`**: Modifies properties of a scheduled task.
*   **`Unregister-ScheduledTask`**: Deletes a scheduled task.
*   **`Start-ScheduledTask`**: Starts a scheduled task immediately.
*   **`Disable-ScheduledTask`**: Disables a scheduled task.
*   **`Enable-ScheduledTask`**: Enables a scheduled task.
*   **`New-ScheduledTaskAction`**: Creates a scheduled task action object (what the task does).
*   **`New-ScheduledTaskTrigger`**: Creates a scheduled task trigger object (when the task runs).
*   **`New-ScheduledTaskSettingsSet`**: Creates a scheduled task settings object (how the task behaves).
*   **`New-ScheduledTaskPrincipal`**: Creates a scheduled task principal object (who the task runs as).

## Implementing Core Functionality with PowerShell

### 1. Listing Scheduled Tasks

#### a. List all scheduled tasks

```powershell
Get-ScheduledTask | Select-Object TaskName, State, @{N='Actions';E={($_.Actions | ForEach-Object {$_.Executable + " " + $_.Arguments}) -join '; '}}
```

#### b. Filter tasks by name or state

```powershell
# Filter by name (e.g., tasks related to "Update")
Get-ScheduledTask | Where-Object { $_.TaskName -like "*Update*" } | Select-Object TaskName, State

# Filter by state (e.g., disabled tasks)
Get-ScheduledTask | Where-Object { $_.State -eq "Disabled" } | Select-Object TaskName, State
```

### 2. Creating a Scheduled Task

Creating a scheduled task involves defining the Action (what to do), the Trigger (when to do it), and the Settings (how to do it).

Let's create a task that runs a PowerShell script daily at a specific time.

```powershell
# Define variables
$taskName = "MyDailyLogTask"
$taskDescription = "Logs a message to a file daily."
$scriptPath = "C:	emp\log_message.ps1" # Ensure this script exists or create it

# Create a dummy script for the task to run
@"
Add-Content -Path "C:	emp\daily_log.txt" -Value "Task '$taskName' ran at $(Get-Date)"
"@ | Set-Content -Path $scriptPath

# 1. Define the Action: What the task does
$taskAction = New-ScheduledTaskAction -Execute "powershell.exe" -Argument "-NoProfile -File `"$scriptPath`""

# 2. Define the Trigger: When the task runs (daily at 2 AM)
$taskTrigger = New-ScheduledTaskTrigger -Daily -At "2:00 AM"

# 3. Define the Settings: How the task behaves
$taskSettings = New-ScheduledTaskSettingsSet -AllowStartIfOnBatteries -DontStopIfOnBatteries -ExecutionTimeLimit (New-TimeSpan -Hours 1)

# 4. Define the Principal: Who the task runs as (SYSTEM, or specify a user)
# For running as SYSTEM:
$taskPrincipal = New-ScheduledTaskPrincipal -UserId "NT AUTHORITY\SYSTEM" -LogonType ServiceAccount

# For running as current user (interactive or not):
# $taskPrincipal = New-ScheduledTaskPrincipal -UserId $env:USERNAME -LogonType Interactive -RunLevel Highest

# 5. Register the task
Register-ScheduledTask -TaskName $taskName -Description $taskDescription -Action $taskAction -Trigger $taskTrigger -Settings $taskSettings -Principal $taskPrincipal

Write-Host "Scheduled task '$taskName' created."

# Verify its creation
Get-ScheduledTask -TaskName $taskName | Select-Object TaskName, State, @{N='Actions';E={($_.Actions | ForEach-Object {$_.Executable + " " + $_.Arguments}) -join '; '}}, @{N='Triggers';E={($_.Triggers | ForEach-Object {$_.RepetitionInterval}) -join '; '}}
```

### 3. Modifying a Scheduled Task

To modify a task, you retrieve it, modify its properties, and then use `Set-ScheduledTask`.

```powershell
$taskName = "MyDailyLogTask"

# Retrieve the task
$task = Get-ScheduledTask -TaskName $taskName

# Example: Change the task description and add a new trigger
$task.Description = "Updated: Now also logs to a different file."
$task.Actions[0].Arguments = "-NoProfile -Command `"Add-Content -Path 'C:	emp\daily_log_v2.txt' -Value 'Task $taskName ran at $(Get-Date)'`""

# Add a trigger to run every 15 minutes as well
# $taskTriggerInterval = New-ScheduledTaskTrigger -Once -At (Get-Date) -RepetitionInterval (New-TimeSpan -Minutes 15) -RepetitionDuration (New-TimeSpan -Days 1)
# $task.Triggers.Add($taskTriggerInterval)

# Apply the changes
Set-ScheduledTask -Task $task # Using the modified task object directly

Write-Host "Scheduled task '$taskName' modified."
Get-ScheduledTask -TaskName $taskName | Select-Object TaskName, Description, @{N='Actions';E={($_.Actions | ForEach-Object {$_.Executable + " " + $_.Arguments}) -join '; '}}
```

### 4. Deleting a Scheduled Task

```powershell
$taskName = "MyDailyLogTask"

Unregister-ScheduledTask -TaskName $taskName -Confirm:$false
Write-Host "Scheduled task '$taskName' deleted."

# Verify deletion
if (-not (Get-ScheduledTask -TaskName $taskName -ErrorAction SilentlyContinue)) {
    Write-Host "Task '$taskName' no longer exists."
}
```

### 5. Enabling/Disabling a Scheduled Task

```powershell
$taskName = "MyDailyLogTask" # Assuming the task still exists

# Create a temporary task for this example if it was deleted
# ... (task creation code from above) ...

# Disable the task
Disable-ScheduledTask -TaskName $taskName
Write-Host "Scheduled task '$taskName' disabled. State: $( (Get-ScheduledTask -TaskName $taskName).State )"

# Enable the task
Enable-ScheduledTask -TaskName $taskName
Write-Host "Scheduled task '$taskName' enabled. State: $( (Get-ScheduledTask -TaskName $taskName).State )"
```

### 6. Running a Scheduled Task Immediately

```powershell
$taskName = "MyDailyLogTask" # Assuming the task still exists and is enabled

# Create a temporary task for this example if it was deleted
# ... (task creation code from above) ...

Write-Host "Starting scheduled task '$taskName' immediately..."
Start-ScheduledTask -TaskName $taskName
Write-Host "Task '$taskName' initiated."
# You can check the task history in Task Scheduler or the log file (C:	emp\daily_log.txt)
```

### 7. Consolidated Script Example (Creating and Managing)

```powershell
# manage_scheduled_task.ps1

param (
    [string]$TaskName = "MyAutomatedCleanup",
    [string]$ActionPath = "C:	emp\cleanup_script.ps1",
    [string]$TaskDescription = "Automatically cleans temporary files",
    [string]$ScheduleTime = "3:00 AM",
    [switch]$Create,
    [switch]$Delete,
    [switch]$Enable,
    [switch]$Disable,
    [switch]$Run,
    [switch]$List
)

# Create a dummy script for the task to run
@"
Remove-Item -Path "$($env:TEMP)\*.tmp" -Force -ErrorAction SilentlyContinue
Add-Content -Path "C:	emp\cleanup_log.txt" -Value "Cleanup task ran at $(Get-Date)"
"@ | Set-Content -Path $ActionPath


if ($Create) {
    if (Get-ScheduledTask -TaskName $TaskName -ErrorAction SilentlyContinue) {
        Write-Warning "Task '$TaskName' already exists."
    } else {
        $action = New-ScheduledTaskAction -Execute "powershell.exe" -Argument "-NoProfile -File `"$ActionPath`""
        $trigger = New-ScheduledTaskTrigger -Daily -At $ScheduleTime
        $principal = New-ScheduledTaskPrincipal -UserId "NT AUTHORITY\SYSTEM" -LogonType ServiceAccount
        Register-ScheduledTask -TaskName $TaskName -Description $TaskDescription -Action $action -Trigger $trigger -Principal $principal
        Write-Host "Task '$TaskName' created."
    }
}

if ($Delete) {
    if (Get-ScheduledTask -TaskName $TaskName -ErrorAction SilentlyContinue) {
        Unregister-ScheduledTask -TaskName $TaskName -Confirm:$false
        Write-Host "Task '$TaskName' deleted."
    } else {
        Write-Warning "Task '$TaskName' not found."
    }
}

if ($Enable) {
    Enable-ScheduledTask -TaskName $TaskName
    Write-Host "Task '$TaskName' enabled."
}

if ($Disable) {
    Disable-ScheduledTask -TaskName $TaskName
    Write-Host "Task '$TaskName' disabled."
}

if ($Run) {
    Start-ScheduledTask -TaskName $TaskName
    Write-Host "Task '$TaskName' initiated."
}

if ($List) {
    Get-ScheduledTask -TaskName $TaskName -ErrorAction SilentlyContinue | Select-Object TaskName, State, @{N='Actions';E={($_.Actions | ForEach-Object {$_.Executable + " " + $_.Arguments}) -join '; '}}
}

# Example Usage:
# .\manage_scheduled_task.ps1 -Create -List
# .\manage_scheduled_task.ps1 -Disable -List
# .\manage_scheduled_task.ps1 -Run -List
# .\manage_scheduled_task.ps1 -Delete
```

## Guiding Principles in PowerShell

*   **Portability:** While `ScheduledTasks` cmdlets are Windows-specific, PowerShell Core itself is cross-platform. For Linux, one would use `cron` or `systemd` interactions, potentially via `Invoke-Command` if managing remotely.
*   **Efficiency:** Direct interaction with the Task Scheduler API via cmdlets is highly efficient and performant.
*   **Minimal Dependencies:** Relies entirely on the PowerShell runtime and its built-in modules.
*   **CLI-centric:** All operations are command-line driven, allowing for powerful scripting and integration into larger automation systems.
*   **Structured Data Handling:** Cmdlets return objects, making it easy to filter, sort, and process task information programmatically.

## Conclusion

PowerShell offers a highly effective and programmatic way to manage scheduled tasks on Windows systems. Its cmdlets provide fine-grained control over task creation, modification, and execution, all within a structured and automatable framework. This capability is crucial for implementing robust system maintenance, security automation, and various other recurring administrative duties. The next step is to apply this knowledge in practical exercises.
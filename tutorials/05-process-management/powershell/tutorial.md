# PowerShell Tutorial: Process Management and Automation

## Introduction

PowerShell offers a native, object-oriented approach to process management, providing a significant advantage over text-based parsing in traditional shells. Its cmdlets (`Get-Process`, `Stop-Process`, `Start-Process`) return rich objects that can be easily filtered, sorted, and manipulated, making it ideal for monitoring, controlling, and automating tasks related to running applications and services. This tutorial will guide you through using PowerShell for process management, emphasizing its structured approach and cross-platform capabilities.

## Framework Alignment

This tutorial on "**Process Management and Automation**" using **PowerShell** demonstrates the foundational skills necessary to implement components of a "Cross-Platform Baseline Hardening & Auditing Framework." The techniques learned here for managing and auditing running processes are directly applicable to auditing systems against defined security baselines and ensuring compliance.


## Core PowerShell Cmdlets for Process Management

*   **`Get-Process`**: Retrieves processes running on the local computer. This is the primary cmdlet for listing and inspecting processes.
    *   `-Name`: Filter by process name.
    *   `-Id`: Filter by process ID.
    *   `-ComputerName`: (Windows) Retrieve processes from remote computers.
    *   `-IncludeUserName`: (Windows) Include the user name that owns the process.
*   **`Where-Object` (alias `where`)**: Filters objects based on property values. Essential for refining process lists.
*   **`Select-Object` (alias `select`)**: Selects specified properties of an object or set of objects. Used for extracting and reformatting process data.
*   **`Sort-Object` (alias `sort`)**: Sorts objects by property value. Useful for finding top CPU/memory consumers.
*   **`Stop-Process`**: Stops one or more running processes.
    *   `-Id`: Stop by process ID.
    *   `-Name`: Stop by process name.
    *   `-Force`: Force the termination (equivalent to `kill -9`).
*   **`Start-Process`**: Starts one or more processes on the local computer.
    *   `-FilePath`: Path to the executable.
    *   `-ArgumentList`: Arguments to pass to the executable.
    *   `-NoNewWindow`: Run the process in the current window.
    *   `-PassThru`: Return an object for the process.
*   **`Wait-Process`**: Waits for the processes to be terminated before returning to the prompt.

## Implementing Core Functionality with PowerShell

### 1. Process Listing

#### a. List All Running Processes with Key Metadata

```powershell
Write-Host "--- All Running Processes ---"
Get-Process | Select-Object Id, ProcessName, Handles, CPU, WS, PM, StartTime, Path, Responding | Format-Table -AutoSize
# Id: PID, WS: Working Set (Memory), PM: Private Memory
```

#### b. Filter Processes by Name/Command

```powershell
# Find processes named 'chrome'
Write-Host "--- Chrome Processes ---"
Get-Process -Name "chrome" | Select-Object Id, ProcessName, CPU, WS | Format-Table -AutoSize

# Find processes where the name contains 'code' (e.g., VS Code)
Write-Host "--- Processes containing 'code' ---"
Get-Process | Where-Object {$_.ProcessName -like "*code*"} | Select-Object Id, ProcessName, CPU, WS | Format-Table -AutoSize
```

#### c. Filter Processes by User (Windows-specific, or PowerShell Core with `psutil` or similar)

```powershell
Write-Host "--- Processes by User (current user example) ---"
# On Windows, Get-Process -IncludeUserName (if available) or Get-WmiObject/Get-CimInstance win32_process
# For simplicity, here's an example using current user context for its own processes
Get-Process | Where-Object { $_.MainWindowTitle } | Select-Object Id, ProcessName, @{Name="User";Expression={(Get-WmiObject Win32_Process -Filter "ProcessId = $($_.Id)").GetOwner().User}} | Format-Table -AutoSize
```
*Note: Getting user for all processes efficiently cross-platform requires `Get-CimInstance Win32_Process` on Windows, and parsing `ps aux` on Linux/macOS, or using external libraries like `psutil` in Python for true cross-platform user mapping.*

#### d. Filter Processes by PID

```powershell
# Get details for a specific PID (e.g., PID 1234)
Write-Host "--- Process Details for PID 1234 ---"
Get-Process -Id 1234 | Select-Object * | Format-List
```

### 2. Process Details

```powershell
# Get comprehensive details for a process by name (e.g., 'notepad')
Write-Host "--- Detailed Notepad Process Information ---"
Get-Process -Name "notepad" | Select-Object * | Format-List
```

### 3. Process Control

#### a. Terminate a Process Gracefully (`Stop-Process`)

```powershell
# Start a notepad process for testing
# Start-Process notepad.exe -PassThru -NoNewWindow
# $notepadProcess = Get-Process -Name "notepad" | Select-Object -First 1

# Terminate by PID
# Stop-Process -Id $notepadProcess.Id

# Terminate by Name
# Stop-Process -Name "notepad"
```

#### b. Terminate a Process Forcefully (`Stop-Process -Force`)

```powershell
# Terminate by PID (e.g., PID 12345) forcefully
# Stop-Process -Id 12345 -Force

# Terminate by Name (e.g., 'badapp') forcefully
# Stop-Process -Name "badapp" -Force
```

#### c. Start a New Process

```powershell
# Start notepad, wait for it to exit
# Start-Process -FilePath "notepad.exe" -Wait

# Start a process in the background, collect its object
# $bgProcess = Start-Process -FilePath "powershell.exe" -ArgumentList "-NoProfile -Command 'Start-Sleep -Seconds 30'" -PassThru -NoNewWindow
# Write-Host "Background process PID: $($bgProcess.Id)"
```

### 4. Process Monitoring (Basic)

#### a. Identify Top CPU/Memory Consuming Processes

```powershell
# Top 5 CPU consuming processes
Write-Host "--- Top 5 CPU Consuming Processes ---"
Get-Process | Sort-Object -Property CPU -Descending | Select-Object -First 5 Id, ProcessName, CPU, WS | Format-Table -AutoSize

# Top 5 Memory (Working Set) consuming processes
Write-Host "--- Top 5 Memory Consuming Processes ---"
Get-Process | Sort-Object -Property WS -Descending | Select-Object -First 5 Id, ProcessName, WS, CPU | Format-Table -AutoSize
```

## Guiding Principles in PowerShell

*   **Portability:** Many `Get-Process` and `Stop-Process` functionalities are available in PowerShell Core, ensuring cross-platform compatibility. However, some detailed properties or `Get-WmiObject`/`Get-CimInstance` for remote management are more Windows-centric.
*   **Efficiency:** PowerShell cmdlets are highly optimized as they interact directly with OS APIs. Operations are generally fast and efficient.
*   **Minimal Dependencies:** PowerShell scripts rely only on the PowerShell runtime and its built-in modules. No external binaries or libraries are typically needed.
*   **CLI-centric:** PowerShell is a command-line shell; scripts are executed directly from the console and leverage its robust argument parsing.
*   **Structured Data Handling:** The greatest advantage is that cmdlets return objects, not raw text. This simplifies filtering, sorting, and reporting, and allows for direct conversion to formats like JSON or CSV.

## Conclusion

PowerShell provides an exceptionally powerful and intuitive environment for process management and automation. Its object-oriented pipeline simplifies the retrieval, filtering, and manipulation of process data, enabling administrators to quickly diagnose issues, enforce policies, and automate routine tasks. The structured nature of its output makes it ideal for integrating process monitoring into larger automation frameworks. The next step is to apply this knowledge in practical exercises.
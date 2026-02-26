# PowerShell Exercise: System Information Snapshot and Reporting

## Objective

This exercise challenges you to apply your PowerShell scripting skills to collect a comprehensive snapshot of essential system information. You will use standard PowerShell cmdlets and WMI/CIM to gather operating system details, hardware specifications, resource usage, and active processes, demonstrating proficiency in system diagnostics and structured reporting.

## Framework Alignment

This exercise on "**System Information Snapshot and Reporting**" using **PowerShell** provides practical experience in building and utilizing tools for a "Cross-Platform Baseline Hardening & Auditing Framework." By completing these tasks, you will learn how to audit and manage system configuration data, ensuring that system baselines are maintained and identifying unauthorized changes—essential steps in maintaining a secure and auditable environment.


## Scenario

You are tasked with performing an audit of a Windows (or PowerShell Core-enabled Linux/macOS) server's current configuration and state. Your goal is to gather key system metrics as structured objects and present them in a clear, human-readable format, suitable for a system report or for initial troubleshooting. Additionally, you should be able to output this data in a machine-readable format like JSON.

## Tasks

Using only standard PowerShell cmdlets (e.g., `Get-CimInstance`, `Get-ComputerInfo`, `Get-Process`, `Get-NetAdapter`, `Get-NetIPAddress`), provide the command-line solution or a simple script for each of the following tasks.

1.  **Operating System Summary:**
    *   Display the OS Caption (e.g., "Microsoft Windows Server 2022 Standard"), OS Architecture, Version, and the last boot-up time.

2.  **CPU Information:**
    *   Show the CPU `Name`, `NumberOfCores`, and `NumberOfLogicalProcessors`.

3.  **Memory Usage Overview:**
    *   Display the `TotalPhysicalMemory` and `FreePhysicalMemory` in gigabytes (GB). Calculate and display the used memory in GB as well.

4.  **Disk Space Usage (Local Fixed Disks):**
    *   List all local fixed disks (DriveType 3), showing their `DeviceID`, `FileSystem`, `Size` (in GB), and `FreeSpace` (in GB).

5.  **Active Network Interfaces and IP Addresses:**
    *   List all active network adapters, showing their `Name`, `MacAddress`, and `Status`.
    *   For each network adapter, list its associated IPv4 addresses (`IPAddress` and `PrefixLength`).

6.  **Top 5 CPU-Consuming Processes:**
    *   Display the top 5 processes consuming the most CPU, showing their `ProcessName`, `Id`, and `CPU` usage.

7.  **System Uptime:**
    *   Display the system's uptime in a human-readable format (e.g., "Days.Hours:Minutes:Seconds").

8.  **Comprehensive System Report to JSON:**
    *   Combine the collected information from tasks 1-7 (or a selection of key data points) into a single PowerShell object or hashtable, and then convert this object to a JSON string, printing it to the console.

## Deliverables

For each task, provide the single PowerShell command-line pipeline or concise script snippet that produces the required output. For Task 8, provide a script that generates the JSON output.

## Reflection Questions

1.  How does PowerShell's object pipeline (`|`) facilitate collecting and formatting diverse system information compared to text-based parsing in Bash?
2.  Describe the role of `Get-CimInstance` (or `Get-WmiObject`) in collecting detailed system data. What are its advantages over running external commands?
3.  How did you handle unit conversions (e.g., bytes to GB) for memory and disk information?
4.  If you needed to track system changes over time, how would you store the JSON output from Task 8 for later analysis?
5.  What are the advantages and disadvantages of using PowerShell for system information collection and reporting compared to Bash or a database like SQLite?

---
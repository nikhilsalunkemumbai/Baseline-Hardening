# PowerShell Tutorial: System Information Snapshot and Reporting

## Introduction

PowerShell excels at gathering system information due to its object-oriented nature and deep integration with underlying operating system APIs, particularly WMI (Windows Management Instrumentation) and CIM (Common Information Model). This tutorial will guide you through using PowerShell cmdlets to collect comprehensive system configuration and state data, focusing on methods that are efficient, produce structured output, and adhere to our principles of portability (especially with PowerShell Core) and minimal dependencies.

## Framework Alignment

This tutorial on "**System Information Snapshot and Reporting**" using **PowerShell** demonstrates the foundational skills necessary to implement components of a "Cross-Platform Baseline Hardening & Auditing Framework." The techniques learned here for collecting and reporting system configuration data are directly applicable to auditing systems against defined security baselines and ensuring compliance.


## Core PowerShell Cmdlets for System Information

PowerShell provides a rich set of cmdlets for retrieving system information:

*   **`Get-ComputerInfo`**: (PowerShell Core 6.0+ & Windows Server 2012 R2+) Retrieves properties of a local or remote computer, including operating system, hardware, and network configuration.
*   **`Get-CimInstance`** (or `Get-WmiObject` for Windows PowerShell 5.1 and earlier): The primary cmdlet for querying WMI/CIM classes, which expose a vast amount of system information. This is often the most powerful and versatile tool.
    *   Common classes: `Win32_OperatingSystem`, `Win32_Processor`, `Win32_ComputerSystem`, `Win32_LogicalDisk`, `Win32_NetworkAdapterConfiguration`, `Win32_Process`, `Win32_Product` (for installed software).
*   **`Get-PSDrive`**: Gets information about the drives on a computer (physical, logical, network, etc.).
*   **`Get-Process`**: Gets the processes that are running on the local computer.
*   **`Get-NetAdapter`**: (Windows-specific, but concepts apply) Gets the basic properties of a network adapter.
*   **`Get-NetIPAddress`**: (Windows-specific) Gets IP address information.
*   **`Get-Uptime`**: (PowerShell Core) Gets the system uptime.

## Implementing Core Functionality with PowerShell

### 1. Operating System Information

```powershell
Write-Host "--- OS Information ---"
# Cross-platform with Get-ComputerInfo (PS Core)
if ($PSVersionTable.PSEdition -eq 'Core') {
    (Get-ComputerInfo).OsName, (Get-ComputerInfo).OsVersion, (Get-ComputerInfo).OsOperatingSystemSKU, (Get-ComputerInfo).CsSystemType | Format-List
}
# Windows-specific via CIM
Get-CimInstance -ClassName Win32_OperatingSystem | Select-Object Caption, OSArchitecture, Version, BuildNumber, Manufacturer, CSDVersion, LastBootUpTime, NumberOfProcesses, FreePhysicalMemory, TotalPhysicalMemory
Write-Host ""
```

### 2. CPU Information

```powershell
Write-Host "--- CPU Information ---"
Get-CimInstance -ClassName Win32_Processor | Select-Object Name, NumberOfCores, NumberOfLogicalProcessors, Architecture, MaxClockSpeed
Write-Host ""
```

### 3. Memory Information

```powershell
Write-Host "--- Memory Information ---"
$OS = Get-CimInstance -ClassName Win32_OperatingSystem
$TotalPhysicalMemoryGB = [Math]::Round($OS.TotalPhysicalMemory / 1GB, 2)
$FreePhysicalMemoryGB = [Math]::Round($OS.FreePhysicalMemory / 1GB, 2)

[PSCustomObject]@{
    TotalMemoryGB = $TotalPhysicalMemoryGB
    FreeMemoryGB = $FreePhysicalMemoryGB
    UsedMemoryGB = [Math]::Round($TotalPhysicalMemoryGB - $FreePhysicalMemoryGB, 2)
} | Format-List
Write-Host ""
```

### 4. Disk Information

```powershell
Write-Host "--- Disk Information ---"
Get-CimInstance -ClassName Win32_LogicalDisk | Where-Object {$_.DriveType -eq 3} | # DriveType 3 = Local Disk
Select-Object DeviceID, FileSystem, @{Name="SizeGB";Expression={[Math]::Round($_.Size / 1GB, 2)}}, @{Name="FreeSpaceGB";Expression={[Math]::Round($_.FreeSpace / 1GB, 2)}}, VolumeName
Write-Host ""
```

### 5. Network Information

```powershell
Write-Host "--- Network Information ---"
# Get basic network adapter info (PowerShell Core / Windows)
Get-NetAdapter | Select-Object Name, MacAddress, Status, LinkSpeed

# Get IP addresses (PowerShell Core / Windows)
Get-NetIPAddress | Where-Object { $_.AddressFamily -eq 'IPv4' -and $_.InterfaceAlias -notlike 'Loopback*' } |
Select-Object InterfaceAlias, IPAddress, PrefixLength, AddressFamily
Write-Host ""
```

### 6. Running Processes (Basic)

```powershell
Write-Host "--- Running Processes (Top 10 by CPU) ---"
Get-Process | Sort-Object -Property CPU -Descending | Select-Object -First 10 ProcessName, Id, CPU, WorkingSet | Format-Table
Write-Host ""
```

### 7. System Uptime

```powershell
Write-Host "--- System Uptime ---"
if ($PSVersionTable.PSEdition -eq 'Core') {
    Get-Uptime | Select-Object -ExpandProperty Uptime
} else {
    # Windows PowerShell
    $OS = Get-CimInstance -ClassName Win32_OperatingSystem
    $Uptime = (Get-Date) - $OS.LastBootUpTime
    Write-Host ("{0:dd\.hh\:mm\:ss}" -f $Uptime)
}
Write-Host ""
```

### 8. Logged-in Users

```powershell
Write-Host "--- Logged-in Users ---"
# This can be tricky cross-platform. For Windows:
if ($PSVersionTable.PSEdition -ne 'Core' -and $env:OS -like 'Windows*') {
    Get-CimInstance -ClassName Win32_LoggedOnUser | Select-Object -ExpandProperty Antecedent | Select-Object Name, Caption | Format-Table -AutoSize
} else {
    Write-Host "Logged-in user information via WMI/CIM is primarily Windows-specific. On Linux/macOS, use 'who' or 'w' in Bash."
}
Write-Host ""
```

## Guiding Principles in PowerShell

*   **Portability:** Many cmdlets (e.g., `Get-Process`, `Get-PSDrive`) and the core `Get-CimInstance` approach are available in PowerShell Core, making scripts largely cross-platform. Platform-specific cmdlets (like `Get-NetAdapter`) require conditional logic or platform-specific modules.
*   **Efficiency:** PowerShell cmdlets are often optimized as they interact directly with system APIs. Querying WMI/CIM is efficient for retrieving structured data.
*   **Minimal Dependencies:** PowerShell scripts rely on the PowerShell runtime and its built-in modules. No external binaries or libraries are typically needed beyond what the OS provides for WMI/CIM.
*   **CLI-centric:** PowerShell is a command-line shell. Scripts are executed directly from the console and produce structured output that can be easily piped, converted (e.g., to JSON or CSV), or formatted.
*   **Structured Data Handling:** PowerShell's object pipeline is a major advantage. All collected data is returned as objects, simplifying filtering, sorting, and reporting compared to parsing raw text.

## Creating a Full System Snapshot Script

You can combine these snippets into a single PowerShell script to generate a comprehensive system information report.

```powershell
# sys_snapshot.ps1

Write-Host "### System Information Snapshot ###"
Write-Host "Report generated on: $(Get-Date)"
Write-Host ""

# OS Information
Write-Host "--- OS Information ---"
if ($PSVersionTable.PSEdition -eq 'Core') {
    (Get-ComputerInfo).OsName, (Get-ComputerInfo).OsVersion, (Get-ComputerInfo).OsOperatingSystemSKU, (Get-ComputerInfo).CsSystemType | Format-List
}
Get-CimInstance -ClassName Win32_OperatingSystem | Select-Object Caption, OSArchitecture, Version, BuildNumber, Manufacturer, CSDVersion, LastBootUpTime, NumberOfProcesses, FreePhysicalMemory, TotalPhysicalMemory
Write-Host ""

# CPU Information
Write-Host "--- CPU Information ---"
Get-CimInstance -ClassName Win32_Processor | Select-Object Name, NumberOfCores, NumberOfLogicalProcessors, Architecture, MaxClockSpeed
Write-Host ""

# Memory Information
Write-Host "--- Memory Information ---"
$OS = Get-CimInstance -ClassName Win32_OperatingSystem
$TotalPhysicalMemoryGB = [Math]::Round($OS.TotalPhysicalMemory / 1GB, 2)
$FreePhysicalMemoryGB = [Math]::Round($OS.FreePhysicalMemory / 1GB, 2)

[PSCustomObject]@{
    TotalMemoryGB = $TotalPhysicalMemoryGB
    FreeMemoryGB = $FreePhysicalMemoryGB
    UsedMemoryGB = [Math]::Round($TotalPhysicalMemoryGB - $FreePhysicalMemoryGB, 2)
} | Format-List
Write-Host ""

# Disk Information
Write-Host "--- Disk Information ---"
Get-CimInstance -ClassName Win32_LogicalDisk | Where-Object {$_.DriveType -eq 3} |
Select-Object DeviceID, FileSystem, @{Name="SizeGB";Expression={[Math]::Round($_.Size / 1GB, 2)}}, @{Name="FreeSpaceGB";Expression={[Math]::Round($_.FreeSpace / 1GB, 2)}}, VolumeName
Write-Host ""

# Network Information
Write-Host "--- Network Information ---"
Get-NetAdapter | Select-Object Name, MacAddress, Status, LinkSpeed
Get-NetIPAddress | Where-Object { $_.AddressFamily -eq 'IPv4' -and $_.InterfaceAlias -notlike 'Loopback*' } |
Select-Object InterfaceAlias, IPAddress, PrefixLength, AddressFamily
Write-Host ""

# Running Processes (Top 10 by CPU)
Write-Host "--- Running Processes (Top 10 by CPU) ---"
Get-Process | Sort-Object -Property CPU -Descending | Select-Object -First 10 ProcessName, Id, CPU, WorkingSet | Format-Table
Write-Host ""

# System Uptime
Write-Host "--- System Uptime ---"
if ($PSVersionTable.PSEdition -eq 'Core') {
    Get-Uptime | Select-Object -ExpandProperty Uptime
} else {
    $OS = Get-CimInstance -ClassName Win32_OperatingSystem
    $Uptime = (Get-Date) - $OS.LastBootUpTime
    Write-Host ("{0:dd\.hh\:mm\:ss}" -f $Uptime)
}
Write-Host ""

# Logged-in Users
Write-Host "--- Logged-in Users ---"
if ($PSVersionTable.PSEdition -ne 'Core' -and $env:OS -like 'Windows*') {
    Get-CimInstance -ClassName Win32_LoggedOnUser | Select-Object -ExpandProperty Antecedent | Select-Object Name, Caption | Format-Table -AutoSize
} else {
    Write-Host "Logged-in user information via WMI/CIM is primarily Windows-specific. On Linux/macOS, use 'who' or 'w' in Bash."
}
Write-Host ""

# Example output to JSON
# Get-CimInstance -ClassName Win32_OperatingSystem | ConvertTo-Json -Depth 3
```

To run this script: `./sys_snapshot.ps1 > system_report.txt`

## Conclusion

PowerShell provides an incredibly powerful and flexible platform for gathering detailed system information. Its object-oriented nature and deep integration with WMI/CIM allow for the collection of rich, structured data that is easy to filter, format, and export. By leveraging cmdlets and understanding the nuances of PowerShell Core for cross-platform compatibility, you can create highly effective system information reporting tools. The next step is to apply this knowledge in practical exercises.
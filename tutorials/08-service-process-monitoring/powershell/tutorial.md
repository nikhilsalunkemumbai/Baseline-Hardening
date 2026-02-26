# PowerShell Tutorial: Service/Process Monitoring and Health Check

## Introduction

PowerShell offers a rich set of cmdlets for comprehensive monitoring and management of system processes and services. Its object-oriented nature simplifies data extraction and manipulation, making it powerful for creating automated health checks and reporting tools. This tutorial will guide you through using PowerShell for local and remote monitoring, leveraging native cmdlets for structured output and efficient operations, aligning with our principles of minimal dependencies and CLI-centric automation.

## Framework Alignment

This tutorial on "**Service/Process Monitoring and Health Check**" using **PowerShell** demonstrates the foundational skills necessary to implement components of a "Cross-Platform Baseline Hardening & Auditing Framework." The techniques learned here for monitoring services and processes are directly applicable to auditing systems against defined security baselines and ensuring compliance.


## Core PowerShell Cmdlets for Monitoring

*   **`Get-Process`**: Gets the processes that are running on the local computer or a remote computer.
*   **`Stop-Process`**: Stops one or more running processes.
*   **`Start-Process`**: Starts one or more processes on the local computer.
*   **`Get-Service`**: Gets objects that represent the services on a computer.
*   **`Stop-Service`**: Stops a running service or services.
*   **`Start-Service`**: Starts one or more stopped services.
*   **`Restart-Service`**: Stops and then starts one or more services.
*   **`Test-NetConnection`**: Displays diagnostic information for a connection to a target computer by using Ping, TCP Connect, or Traceroute. Essential for port checks.
*   **`Invoke-WebRequest`** (alias `iwr`): Sends HTTP and HTTPS requests to a web page or web service. Used for HTTP/S health checks.
*   **`Get-Counter`**: Gets performance counter data from local and remote computers. (Windows-specific, but concepts apply to cross-platform performance monitoring).

## Implementing Core Functionality with PowerShell

### 1. Process Monitoring

#### a. Check if a process is running by name

```powershell
$processName = "notepad" # Example: try with a running Notepad instance

if (Get-Process -Name $processName -ErrorAction SilentlyContinue) {
    Write-Host "$processName is running."
} else {
    Write-Host "$processName is not running."
}
```

#### b. Get PID, CPU, Memory for a process

```powershell
$processName = "notepad" # Example

Get-Process -Name $processName -ErrorAction SilentlyContinue | Select-Object Name, Id, @{Name="CPU(s)";Expression={$_.CPU}}, @{Name="Memory(MB)";Expression={($_.WorkingSet / 1MB).ToString("N2")}}
```

#### c. Start a process

```powershell
# Example: Start Notepad
# Start-Process -FilePath "notepad.exe" -WindowStyle Hidden # -WindowStyle Hidden to run in background
Write-Host "Start-Process command is commented out for safety."
```

#### d. Stop a process

```powershell
$processName = "notepad" # Example

# Get all notepad processes and stop them
# Get-Process -Name $processName -ErrorAction SilentlyContinue | Stop-Process -Force -Confirm:$false
Write-Host "Stop-Process command is commented out for safety."
```

### 2. Service Monitoring (Windows Focus, but `Get-Service` is cross-platform in PS Core)

#### a. Check service status

```powershell
$serviceName = "Spooler" # Example: Print Spooler on Windows
# $serviceName = "sshd" # Example on Linux/macOS with SSH service

$service = Get-Service -Name $serviceName -ErrorAction SilentlyContinue

if ($service) {
    Write-Host "$serviceName status: $($service.Status)"
    Write-Host "$serviceName Start Type: $($service.StartType)"
    Write-Host "$serviceName Can Pause And Continue: $($service.CanPauseAndContinue)"
} else {
    Write-Host "Service $serviceName not found."
}
```

#### b. Start/Stop/Restart a service

```powershell
$serviceName = "Spooler" # Example

# Example: Restart the Spooler service
# Restart-Service -Name $serviceName -ErrorAction SilentlyContinue -PassThru | Select-Object Name, Status
Write-Host "Service control commands (Start-Service, Stop-Service, Restart-Service) are commented out for safety."
```

#### c. Set service start type

```powershell
$serviceName = "Spooler" # Example

# Example: Set Spooler service to Manual
# Set-Service -Name $serviceName -StartupType Manual
# Get-Service -Name $serviceName | Select-Object Name, StartType
Write-Host "Set-Service command is commented out for safety."
```

### 3. Health Checks

#### a. Check if a TCP port is listening

```powershell
$targetHost = "localhost"
$targetPort = 80 # Example: Common web server port

$tncResult = Test-NetConnection -ComputerName $targetHost -Port $targetPort -InformationLevel Detailed -ErrorAction SilentlyContinue

if ($tncResult -and $tncResult.TcpTestSucceeded) {
    Write-Host "Port $targetPort on $targetHost is open and listening."
} else {
    Write-Host "Port $targetPort on $targetHost is NOT listening."
    if ($tncResult) {
        Write-Host "Details: $($tncResult | Out-String)"
    }
}
```

#### b. Perform an HTTP GET request and check status code

```powershell
$url = "http://localhost:80" # Replace with a valid local web service URL
$expectedStatusCode = 200

try {
    $response = Invoke-WebRequest -Uri $url -Method Get -UseBasicParsing -ErrorAction Stop
    if ($response.StatusCode -eq $expectedStatusCode) {
        Write-Host "HTTP Health Check PASSED for $url (Status: $($response.StatusCode))."
        # Optionally, check content
        if ($response.Content -match "<html>") {
            Write-Host "  Content check PASSED: HTML content found."
        }
    } else {
        Write-Host "HTTP Health Check FAILED for $url (Status: $($response.StatusCode)), expected $expectedStatusCode."
    }
}
catch {
    Write-Host "HTTP Health Check FAILED for $url: $($_.Exception.Message)"
}
```

#### c. Check for specific performance counters (Windows-specific, for illustrative purposes)

```powershell
# Get current CPU usage for a process (e.g., powershell)
# $processCpu = (Get-Counter "\Process(powershell)\% Processor Time").CounterSamples[0].CookedValue
# Write-Host "PowerShell process CPU usage: $($processCpu.ToString("N2"))%"

# Get total physical memory free
# $memoryFree = (Get-Counter "\Memory\Available MBytes").CounterSamples[0].CookedValue
# Write-Host "Available Memory: $($memoryFree.ToString("N2")) MB"
```

### 4. Consolidated Monitoring Script Example

```powershell
# monitor_system_health.ps1

function Get-HealthReport {
    param (
        [string]$ProcessName = "notepad",
        [string]$ServiceName = "Spooler", # Or "sshd" for Linux
        [string]$WebAppUrl = "http://localhost:80",
        [int]$WebAppPort = 80
    )

    $report = [System.Collections.Generic.List[object]]::new()
    $timestamp = (Get-Date).ToUniversalTime().ToString("yyyy-MM-ddTHH:mm:ssZ")

    # 1. Check Process Status
    $processStatus = @{
        Check = "Process Status"
        Target = $ProcessName
        Status = "FAIL"
        Message = "$ProcessName is not running."
    }
    if (Get-Process -Name $ProcessName -ErrorAction SilentlyContinue) {
        $proc = Get-Process -Name $ProcessName
        $processStatus.Status = "PASS"
        $processStatus.Message = "$ProcessName is running (PID: $($proc.Id), CPU: $($proc.CPU), Mem: $(($proc.WorkingSet / 1MB).ToString("N2")) MB)"
    }
    $report.Add($processStatus)

    # 2. Check Service Status
    $serviceStatus = @{
        Check = "Service Status"
        Target = $ServiceName
        Status = "FAIL"
        Message = "$ServiceName is not running or not found."
    }
    $service = Get-Service -Name $ServiceName -ErrorAction SilentlyContinue
    if ($service) {
        $serviceStatus.Status = "PASS"
        $serviceStatus.Message = "$ServiceName status: $($service.Status), StartType: $($service.StartType)"
    }
    $report.Add($serviceStatus)

    # 3. Check Web App Port Listening
    $portCheck = @{
        Check = "Port Listening"
        Target = "$WebAppUrl:$WebAppPort"
        Status = "FAIL"
        Message = "Port $WebAppPort on localhost is NOT listening."
    }
    $tncResult = Test-NetConnection -ComputerName "localhost" -Port $WebAppPort -InformationLevel Detailed -ErrorAction SilentlyContinue
    if ($tncResult -and $tncResult.TcpTestSucceeded) {
        $portCheck.Status = "PASS"
        $portCheck.Message = "Port $WebAppPort on localhost is open and listening."
    }
    $report.Add($portCheck)

    # 4. Check Web App HTTP Endpoint
    $httpCheck = @{
        Check = "HTTP Endpoint"
        Target = $WebAppUrl
        Status = "FAIL"
        Message = "HTTP Health Check FAILED for $WebAppUrl."
    }
    try {
        $response = Invoke-WebRequest -Uri $WebAppUrl -Method Get -UseBasicParsing -ErrorAction Stop
        if ($response.StatusCode -eq 200) {
            $httpCheck.Status = "PASS"
            $httpCheck.Message = "HTTP Health Check PASSED for $WebAppUrl (Status: $($response.StatusCode))."
        } else {
            $httpCheck.Message = "HTTP Health Check FAILED for $WebAppUrl (Status: $($response.StatusCode)), expected 200."
        }
    }
    catch {
        $httpCheck.Message = "HTTP Health Check FAILED for $WebAppUrl: $($_.Exception.Message)"
    }
    $report.Add($httpCheck)

    # Output as JSON
    $output = @{
        Timestamp = $timestamp
        Host = $env:COMPUTERNAME
        HealthChecks = $report
    }
    return $output | ConvertTo-Json -Depth 100
}

# Example usage:
# Get-HealthReport -ProcessName "explorer" -WebAppUrl "http://localhost:80" | Write-Host
Write-Host "This script will generate a JSON report of system health."
Write-Host "Example Usage: Get-HealthReport -ProcessName 'explorer' -ServiceName 'WinRM' -WebAppUrl 'http://localhost:8080' -WebAppPort 8080"
```

## Guiding Principles in PowerShell

*   **Portability:** PowerShell Core enables many of these cmdlets (`Get-Process`, `Get-Service`, `Test-NetConnection`, `Invoke-WebRequest`) to run cross-platform on Linux, macOS, and Windows.
*   **Efficiency:** Native cmdlets are optimized for performance, especially when dealing with system objects. The object pipeline minimizes text parsing.
*   **Minimal Dependencies:** Relies primarily on the PowerShell runtime and its built-in modules. No external executables or libraries beyond the OS itself are typically needed.
*   **CLI-centric:** Scripts are executed from the command line, accepting parameters for flexible monitoring.
*   **Structured Data Handling:** The object pipeline is PowerShell's greatest strength for this task. System information is returned as rich objects, allowing for easy filtering, sorting, and conversion to structured formats like JSON.

## Conclusion

PowerShell provides a powerful, versatile, and cross-platform environment for robust service and process monitoring and health checking. Its native cmdlets and object pipeline significantly simplify the task of gathering system information, performing various health checks, and generating structured reports for automation and integration. The ability to manage services and processes directly within the script further enhances its utility for maintaining system resilience. The next step is to apply this knowledge in practical exercises.
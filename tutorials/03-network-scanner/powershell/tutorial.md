# PowerShell Tutorial: Network Connectivity and Port Scanning

## Introduction

PowerShell provides robust and native cmdlets for network diagnostics, making it an excellent choice for performing network connectivity checks and basic port scanning. Its object-oriented pipeline allows for structured output that can be easily filtered, processed, and reported. This tutorial will focus on using `Test-Connection` and `Test-NetConnection` to implement our network scanner design, emphasizing PowerShell's strengths in producing actionable, structured data.

## Framework Alignment

This tutorial on "**Network Connectivity and Port Scanner**" using **PowerShell** demonstrates the foundational skills necessary to implement components of a "Cross-Platform Baseline Hardening & Auditing Framework." The techniques learned here for auditing network connectivity and port accessibility are directly applicable to auditing systems against defined security baselines and ensuring compliance.


## Core PowerShell Cmdlets for Network Scanning

*   **`Test-Connection`**: Sends ICMP echo request packets (pings) to one or more computers. It's the PowerShell equivalent of the `ping` command, but returns structured objects.
    *   `-Count`: Specifies the number of echo requests to send.
    *   `-Delay`: Specifies the interval between echo requests.
    *   `-BufferSize`: Specifies the size of the echo request packet.
    *   `-Quiet`: Returns only a Boolean value (True/False) indicating success.
*   **`Test-NetConnection`**: Displays diagnostic information for a connection. It can perform ping, TCP port tests, and route tracing. This is the primary cmdlet for TCP port scanning in PowerShell.
    *   `-ComputerName`: Specifies the target computer.
    *   `-Port`: Specifies the TCP port number to test.
    *   `-InformationLevel "Detailed"`: Provides more verbose output with connection details.
    *   `-CommonTCPPort`: Tests common ports like HTTP, HTTPS, RDP, SMB.

## Implementing Core Functionality with PowerShell

### 1. Host Reachability Check (`Test-Connection`)

```powershell
function Test-HostReachability {
    param (
        [string]$ComputerName,
        [int]$Count = 1,
        [int]$TimeoutSeconds = 1
    )

    Write-Host "Pinging $ComputerName..."
    $result = Test-Connection -ComputerName $ComputerName -Count $Count -ErrorAction SilentlyContinue -WarningAction SilentlyContinue

    if ($result) {
        Write-Host "Host $ComputerName is UP"
        return $true
    } else {
        Write-Host "Host $ComputerName is DOWN or UNREACHABLE"
        return $false
    }
}

# Example Usage:
# Test-HostReachability -ComputerName "8.8.8.8"
# Test-HostReachability -ComputerName "nonexistent.example.com"
```

### 2. Basic Port Scanning (`Test-NetConnection`)

`Test-NetConnection` provides a direct way to check TCP port status.

```powershell
function Test-PortStatus {
    param (
        [string]$ComputerName,
        [int]$Port,
        [int]$TimeoutSeconds = 1 # Test-NetConnection uses internal timeouts, this is conceptual
    )

    Write-Host "Scanning $ComputerName:$Port..."
    $result = Test-NetConnection -ComputerName $ComputerName -Port $Port -InformationLevel Detailed -ErrorAction SilentlyContinue -WarningAction SilentlyContinue

    if ($result.TcpTestSucceeded) {
        Write-Host "Port $Port on $ComputerName: Open"
        return "Open"
    } elseif ($result.PingSucceeded -and -not $result.TcpTestSucceeded) {
        # Host is up, but TCP test failed (port closed or filtered)
        Write-Host "Port $Port on $ComputerName: Closed/Filtered"
        return "Closed/Filtered"
    } else {
        # Host unreachable or other general failure
        Write-Host "Port $Port on $ComputerName: Unreachable/Filtered"
        return "Unreachable/Filtered"
    }
}

# Example Usage:
# Test-PortStatus -ComputerName "google.com" -Port 80
# Test-PortStatus -ComputerName "127.0.0.1" -Port 22 # Assuming SSH is running locally
# Test-PortStatus -ComputerName "127.0.0.1" -Port 65535
```

### 3. Scanning Multiple Ports and Hosts

Combine functions to scan multiple ports on one or more hosts.

```powershell
function Invoke-NetworkScan {
    param (
        [string[]]$ComputerNames,
        [int[]]$Ports,
        [int]$PingTimeoutSeconds = 1,
        [int]$PortScanTimeoutSeconds = 1
    )

    $scanResults = @()

    foreach ($computer in $ComputerNames) {
        $hostResult = [PSCustomObject]@{
            Host = $computer
            HostStatus = "Unknown"
            Ports = @()
        }

        Write-Host "`n--- Scanning Host: $computer ---"
        if (Test-HostReachability -ComputerName $computer -TimeoutSeconds $PingTimeoutSeconds) {
            $hostResult.HostStatus = "Up"
            foreach ($port in $Ports) {
                $portStatus = Test-PortStatus -ComputerName $computer -Port $port -TimeoutSeconds $PortScanTimeoutSeconds
                $hostResult.Ports += [PSCustomObject]@{
                    Port = $port
                    Status = $portStatus
                }
            }
        } else {
            $hostResult.HostStatus = "Down"
        }
        $scanResults += $hostResult
    }
    return $scanResults
}

# Example Usage:
# $hosts = "127.0.0.1", "google.com"
# $ports = 22, 80, 443
# Invoke-NetworkScan -ComputerNames $hosts -Ports $ports | ConvertTo-Json -Depth 3
```

### 4. JSON Output

PowerShell's `ConvertTo-Json` cmdlet makes it trivial to generate structured JSON output from objects.

```powershell
# Example: Convert a single port scan result to JSON
Test-PortStatus -ComputerName "google.com" -Port 443 | ConvertTo-Json

# Example: Convert the full scan results from Invoke-NetworkScan
# Invoke-NetworkScan -ComputerNames $hosts -Ports $ports | ConvertTo-Json -Depth 3
```

## Guiding Principles in PowerShell

*   **Portability:** `Test-Connection` and `Test-NetConnection` (in PowerShell Core) are cross-platform. Ensure you are using PowerShell Core for maximum compatibility across Windows, Linux, and macOS.
*   **Efficiency:** These cmdlets are highly optimized as they utilize native OS network APIs. Performing multiple `Test-NetConnection` operations sequentially is generally efficient enough for basic scanning.
*   **Minimal Dependencies:** PowerShell scripts rely only on the PowerShell runtime and its built-in modules, requiring no additional external installations.
*   **CLI-centric:** PowerShell is a command-line shell; scripts are executed directly from the console and leverage its robust argument parsing.
*   **Structured Data Handling:** The greatest advantage is that cmdlets return objects, not raw text. This eliminates the need for complex text parsing (regex, `awk`) and allows for direct manipulation, filtering, and conversion to formats like JSON or CSV.

## Conclusion

PowerShell provides powerful and native capabilities for network connectivity and port scanning through its `Test-Connection` and `Test-NetConnection` cmdlets. Its object-oriented pipeline simplifies data processing and reporting, allowing for the creation of robust, efficient, and highly readable network diagnostic tools. The ability to easily output structured data like JSON makes it ideal for integrating with automation workflows and other systems. The next step is to apply this knowledge in practical exercises.
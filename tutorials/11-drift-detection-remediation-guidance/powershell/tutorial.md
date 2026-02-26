# PowerShell Tutorial: Configuration Drift Detection and Remediation Guidance

## Introduction

PowerShell's object-oriented nature makes it incredibly efficient for **Drift Detection**. Because data is handled as objects rather than text, we can use the `Compare-Object` cmdlet to find differences between a "Golden Image" baseline and the current system state. This tutorial will demonstrate how to identify these deltas and how to generate actionable PowerShell remediation commands based on audit failures.

## Framework Alignment

This tutorial on "**Configuration Drift Detection and Remediation Guidance**" using **PowerShell** demonstrates the foundational skills necessary to implement components of a "Cross-Platform Baseline Hardening & Auditing Framework." The techniques learned here for comparing objects and providing remediation advice are essential for maintaining Windows system integrity and security.

## Core PowerShell Logic for Drift Detection

### 1. Using `Compare-Object`

The most powerful way to find drift in PowerShell is comparing two arrays of objects.

```powershell
# Define Baseline (Golden Snapshot)
$Baseline = @(
    [PSCustomObject]@{Name="SSH"; Status="Running"}
    [PSCustomObject]@{Name="Firewall"; Status="Active"}
    [PSCustomObject]@{Name="GuestAccount"; Status="Disabled"}
)

# Define Current State (Actual)
$Current = @(
    [PSCustomObject]@{Name="SSH"; Status="Stopped"} # DRIFT!
    [PSCustomObject]@{Name="Firewall"; Status="Active"}
    [PSCustomObject]@{Name="GuestAccount"; Status="Disabled"}
    [PSCustomObject]@{Name="UnauthorizedApp"; Status="Running"} # NEW!
)

# Identify Differences
# => means the item is in the Current state but not the Baseline (Added)
# <= means the item is in the Baseline but missing from Current (Removed/Modified)
$Drift = Compare-Object -ReferenceObject $Baseline -DifferenceObject $Current -Property Name, Status -PassThru

$Drift | ForEach-Object {
    if ($_.SideIndicator -eq "=>") {
        Write-Host "ADDED or MODIFIED TO: $($_.Name) is $($_.Status)" -ForegroundColor Red
    } else {
        Write-Host "REMOVED or MODIFIED FROM: $($_.Name) was $($_.Status)" -ForegroundColor Yellow
    }
}
```

### 2. Generating Remediation Guidance

To provide "panache" to your tools, generate the exact command the admin needs to run.

```powershell
function Get-Remediation {
    param (
        [string]$ControlID,
        [string]$CurrentValue
    )

    # In a real tool, this would be looked up in a JSON/YAML policy
    switch ($ControlID) {
        "1.1" { return "Set-Service -Name 'sshd' -Status Running" }
        "1.2" { return "Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled True" }
        "1.3" { return "Disable-LocalUser -Name 'Guest'" }
        Default { return "Review system hardening guidelines." }
    }
}

# Example Usage
$AuditFailure = "1.1"
$Fix = Get-Remediation -ControlID $AuditFailure
Write-Host "FAIL: SSH service is not running." -ForegroundColor Red
Write-Host "SUGGESTED FIX [PowerShell]: $Fix" -ForegroundColor Green
```

### 3. Actionable Advisory Report

```powershell
$Report = @()
$Check = @{ID="1.3"; Title="Guest Account Disabled"; Actual="Enabled"; Expected="Disabled"}

if ($Check.Actual -ne $Check.Expected) {
    $Fix = Get-Remediation -ControlID $Check.ID
    $Report += [PSCustomObject]@{
        ID = $Check.ID
        Status = "FAIL"
        Message = "The Guest account should be disabled for security."
        Remediation = $Fix
    }
}

$Report | ConvertTo-Json
```

## Guiding Principles in PowerShell

*   **Object Comparison:** Always compare properties of objects (using `-Property`) rather than converting them to strings, to ensure accuracy.
*   **Advisory Strings:** Ensure remediation guidance uses standard PowerShell cmdlets that work in both Windows PowerShell and PowerShell Core where possible.
*   **Structured Output:** Include the `Remediation` property in your JSON output so it can be consumed by automation engines or orchestration platforms.

## Conclusion

By leveraging PowerShell's comparison engine, you can build tools that not only tell an administrator what is wrong but exactly how to fix it using the shell they are already in. This makes your baseline auditing framework an indispensable part of their daily operations. The next step is to apply these techniques in hands-on exercises.

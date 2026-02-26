# PowerShell Exercise: Configuration Drift Detection and Remediation Guidance

## Objective

This exercise challenges you to apply your PowerShell skills to detect drift between a baseline set of system services and their current state. You will use `Compare-Object` to identify differences and then generate an advisory report that provides the specific PowerShell commands needed to remediate any unauthorized changes.

## Framework Alignment

This exercise on "**Configuration Drift Detection and Remediation Guidance**" using **PowerShell** provides practical experience in building and utilizing tools for a "Cross-Platform Baseline Hardening & Auditing Framework." By completing these tasks, you will learn how to automate the detection of unauthorized system changes and provide actionable advice to administrators—essential steps in maintaining a secure and auditable environment.

## Scenario

You are a Windows administrator tasked with ensuring that critical security services are always running and that no unauthorized services have been started. You have a "Golden Snapshot" of the required services. Your task is to identify any deviations and provide the "Fix" for each one.

## Setup

Define your baseline and current state objects in your PowerShell console:

```powershell
# 1. Define Baseline (Known Good)
$BaselineServices = @(
    [PSCustomObject]@{Name="WinDefend"; Status="Running"}
    [PSCustomObject]@{Name="W32Time"; Status="Running"}
    [PSCustomObject]@{Name="Spooler"; Status="Stopped"}
)

# 2. Define Current State (Simulated Scan)
$CurrentServices = @(
    [PSCustomObject]@{Name="WinDefend"; Status="Stopped"} # DRIFT: Critical service stopped!
    [PSCustomObject]@{Name="W32Time"; Status="Running"}   # NO DRIFT
    [PSCustomObject]@{Name="UnauthorizedApp"; Status="Running"} # DRIFT: New item!
)
```

## Tasks

Using only standard PowerShell cmdlets, provide the solution for each of the following:

1.  **Detect All Drift:**
    *   Use `Compare-Object` to find all differences between `$BaselineServices` and `$CurrentServices` based on the `Name` and `Status` properties.

2.  **Filter for "Modified" Services:**
    *   Identify the service that exists in both lists but has a different status (e.g., `WinDefend`).

3.  **Generate Remediation Commands:**
    *   Create a script block or function that takes a failed service name and outputs the correct command to fix it (e.g., `Start-Service -Name "WinDefend"`).

4.  **Export Advisory JSON Report:**
    *   Combine your findings into a list of custom objects containing: `ServiceName`, `IssueType` (Modified/Added/Removed), and `RemediationFix`.
    *   Output this list as a JSON string.

## Deliverables

Provide the PowerShell code used to accomplish the tasks and the final JSON output.

## Reflection Questions

1.  How does PowerShell's ability to compare whole objects (properties and all) prevent the errors common in text-based line matching?
2.  Explain the use of the `SideIndicator` property returned by `Compare-Object`. What does `=>` versus `<=` tell you in this context?
3.  Why is it better to provide a specific cmdlet (e.g., `Start-Service`) in the remediation guidance rather than just saying "Service Stopped"?
4.  How could you adapt this exercise to monitor Registry Keys instead of Services?
5.  What are the advantages of using PowerShell for drift detection compared to a standalone security scanner?

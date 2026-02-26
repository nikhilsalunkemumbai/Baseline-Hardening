# PowerShell Exercise: Service/Process Monitoring and Health Check

## Objective

This exercise challenges you to apply your PowerShell scripting and cmdlet skills to monitor the operational status and health of critical Windows system components. You will use standard PowerShell cmdlets to check process existence, service states, network port listening, and perform basic application health checks, demonstrating proficiency in system observability and automated reporting.

## Framework Alignment

This exercise on "**Service/Process Monitoring and Health Check**" using **PowerShell** provides practical experience in building and utilizing tools for a "Cross-Platform Baseline Hardening & Auditing Framework." By completing these tasks, you will learn how to audit and manage service and process health, ensuring that critical security and operational services are running as expected—essential steps in maintaining a secure and auditable environment.


## Scenario

You are a system administrator responsible for a Windows web server. The server hosts an application relying on the `World Wide Web Publishing Service` (W3SVC) and has a critical background process running. Your task is to create a PowerShell script to periodically check the health of these components and provide a structured report.

## Setup

For this exercise, you will need access to a Windows environment.

### A. Simulating a Background Process (if no specific process is available)

If you don't have a specific long-running process like `iisexpress.exe` to monitor, you can simulate one using a PowerShell background job.

Create a file named `dummy_process.ps1` with the following content:

```powershell
# dummy_process.ps1
while ($true) {
    Start-Sleep -Seconds 5
    # This process just stays alive, simulating a long-running background task.
}
```

Start this script as a background job:

```powershell
Start-Job -FilePath "dummy_process.ps1" -Name "DummyMonitorJob"
Write-Host "Dummy background process started. Use Get-Job to see its status."
```
**Remember to stop and remove this job (`Stop-Job -Name "DummyMonitorJob"; Remove-Job -Name "DummyMonitorJob"`) when you are done with the exercise.**

## Tasks

Using only standard PowerShell cmdlets, provide the command-line solution or a simple script for each of the following tasks.

### Part 1: Individual Checks

1.  **Check W3SVC Service Status:**
    *   Determine if the `World Wide Web Publishing Service` (W3SVC) is currently running. Also, display its `StartType`.

2.  **Check Specific Process Status:**
    *   Determine if a process named `notepad` (or the `dummy_process.ps1` job, if you used it, by finding `powershell` process and filtering by job name or command line) is running.

3.  **Get CPU and Memory for a Process:**
    *   If a process (e.g., `notepad` or your `dummy_process.ps1`) is running, retrieve its `Id` (PID), `CPU` usage, and `WorkingSet` (memory usage in MB).

4.  **Check Web Server Port Listening:**
    *   Verify if TCP port `80` is currently listening on `localhost` using `Test-NetConnection`.

5.  **Perform HTTP Health Check:**
    *   Make an HTTP GET request to `http://localhost` (assuming a web server is running on port 80).
    *   Check if the HTTP status code returned is `200 OK`.

### Part 2: Consolidated Health Check Script

Create a single PowerShell script (`windows_server_health.ps1`) that performs the following:

1.  **Check W3SVC Service:** Report if `W3SVC` is running or not, and its start type.
2.  **Check a Specific Process:** Report if your chosen process (e.g., `notepad` or the `dummy_process.ps1` job) is running or not, and if running, display its PID, CPU, and memory usage.
3.  **Check Port 80:** Report if TCP port 80 is listening on `localhost`.
4.  **Check HTTP Endpoint:** Report if the `http://localhost` endpoint returns `200 OK`.
5.  **Generate Structured JSON Report:**
    *   Collect all individual check results into an array of PowerShell custom objects.
    *   Include a `Timestamp` and `Hostname` in the overall report.
    *   Finally, convert this array into a JSON string and print it to the console.

## Deliverables

For Part 1, provide the exact PowerShell command-line solution for each task. For Part 2, provide the complete PowerShell script file (`windows_server_health.ps1`).

## Reflection Questions

1.  How does PowerShell's object pipeline simplify extracting information from cmdlets like `Get-Process` and `Get-Service` compared to text parsing?
2.  Describe how you would differentiate between a `powershell.exe` process that is a user's console session versus one that is a background job from `dummy_process.ps1`.
3.  Explain the advantages of outputting the health check report as a structured JSON object. How can this report be consumed by other systems or monitoring dashboards?
4.  If you needed to automatically restart the `W3SVC` service if it's found to be stopped, how would you integrate `Restart-Service` into your script, and what considerations would you have for automated remediation?
5.  What are the cross-platform capabilities of PowerShell Core for service/process monitoring, and what are its limitations when comparing Windows-specific cmdlets to Linux equivalents?

---
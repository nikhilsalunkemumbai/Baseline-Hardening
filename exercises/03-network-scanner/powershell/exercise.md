# PowerShell Exercise: Network Connectivity and Basic Port Scanning

## Objective

This exercise challenges you to apply your PowerShell scripting skills to perform fundamental network connectivity checks and basic TCP port scanning. You will use standard PowerShell cmdlets to determine host reachability, identify the status of common ports on a target, and generate structured output, demonstrating proficiency in network diagnostics.

## Framework Alignment

This exercise on "**Network Connectivity and Port Scanner**" using **PowerShell** provides practical experience in building and utilizing tools for a "Cross-Platform Baseline Hardening & Auditing Framework." By completing these tasks, you will learn how to audit and manage network connectivity and port accessibility, ensuring that only authorized services are exposed—an essential step in maintaining a secure and auditable environment.


## Scenario

You need to quickly assess the network presence and exposed services of a remote host. Your task is to use PowerShell to perform an ICMP echo request (`ping`) to check if the host is online, and then attempt to connect to a few common service ports to see if they are open. You should aim to produce structured output that can be easily consumed by other scripts or for reporting.

## Target Information

*   **Target Host:** `scanme.nmap.org` (This host is provided by the Nmap project specifically for testing. Please do NOT scan other public IPs without explicit permission.)
*   **Target Ports:** `22` (SSH), `80` (HTTP), `443` (HTTPS), `3389` (RDP - likely closed/filtered on scanme.nmap.org)

## Tasks

Using only standard PowerShell cmdlets (e.g., `Test-Connection`, `Test-NetConnection`), provide the command-line solution or a simple script for each of the following tasks. Implement appropriate timeouts for network operations.

1.  **Check Host Reachability:**
    *   Determine if `scanme.nmap.org` is currently reachable via an ICMP echo request using `Test-Connection`. Your command should output whether the host is "UP" or "DOWN".

2.  **Scan for Open Ports (`Test-NetConnection`):**
    *   For each of the `Target Ports` (`22, 80, 443, 3389`), use `Test-NetConnection` to test connectivity to the specified port on `scanme.nmap.org`. For each port, report if it is "Open", "Closed", or "Filtered". Include a suitable timeout for each port test.

3.  **Combined Scan Script with Structured Output:**
    *   Create a single PowerShell script that defines a target host and a list of ports.
    *   The script should first check the host's reachability.
    *   If the host is up, it should then scan each specified port using `Test-NetConnection`, collecting the results (Host, Port, Status) into PowerShell objects.
    *   Finally, the script should output all collected host and port scan data as a single JSON string to the console.

## Deliverables

For tasks 1-2, provide the exact PowerShell command-line solution. For task 3, provide the complete PowerShell script file.

## Reflection Questions

1.  How did `Test-Connection` and `Test-NetConnection` simplify network diagnostic tasks compared to using external executables or parsing raw text output?
2.  Explain how PowerShell's object pipeline helped in creating structured results for your combined scan script.
3.  What are the advantages of outputting scan results as JSON in PowerShell, especially when integrating with other tools or automation workflows?
4.  Consider performance when scanning a large number of hosts or ports. How might you approach parallelizing port scans in PowerShell? (Hint: Consider `ForEach-Object -Parallel` or background jobs).
5.  What security considerations should be kept in mind when performing port scans, even basic ones?

---
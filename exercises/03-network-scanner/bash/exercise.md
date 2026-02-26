# Bash Exercise: Network Connectivity and Basic Port Scanning

## Objective

This exercise challenges you to apply your Bash scripting skills to perform fundamental network connectivity checks and basic TCP port scanning. You will use standard command-line utilities to determine host reachability and identify the status of common ports on a target, demonstrating proficiency in network diagnostics with minimal dependencies.

## Framework Alignment

This exercise on "**Network Connectivity and Port Scanner**" using **Bash** provides practical experience in building and utilizing tools for a "Cross-Platform Baseline Hardening & Auditing Framework." By completing these tasks, you will learn how to audit and manage network connectivity and port accessibility, ensuring that only authorized services are exposed—an essential step in maintaining a secure and auditable environment.


## Scenario

You need to quickly assess the network presence and exposed services of a remote host. Your task is to use Bash to perform an ICMP echo request (ping) to check if the host is online, and then attempt to connect to a few common service ports to see if they are open.

## Target Information

*   **Target Host:** `scanme.nmap.org` (This host is provided by the Nmap project specifically for testing. Please do NOT scan other public IPs without explicit permission.)
*   **Target Ports:** `22` (SSH), `80` (HTTP), `443` (HTTPS), `3389` (RDP - likely closed/filtered on scanme.nmap.org)

## Tasks

Using only standard Bash commands and utilities (e.g., `ping`, `nc` (netcat), `telnet`, or `/dev/tcp`), provide the command-line solution or a simple script for each of the following tasks. Implement appropriate timeouts for network operations.

1.  **Check Host Reachability:**
    *   Determine if `scanme.nmap.org` is currently reachable via an ICMP echo request (ping). Your command should output whether the host is "UP" or "DOWN".

2.  **Scan for Open Ports (Netcat `nc`):**
    *   For each of the `Target Ports` (`22, 80, 443, 3389`), use `nc` (netcat) to test if the port is "Open", "Closed", or "Filtered" on `scanme.nmap.org`. Include a 1-second timeout for each port scan attempt.

3.  **Scan for Open Ports (`/dev/tcp`):**
    *   For each of the `Target Ports`, use the Bash built-in `/dev/tcp` (if available on your system) to test if the port is "Open" or "Closed/Filtered" on `scanme.nmap.org`. Include a 1-second timeout for each attempt.

4.  **Combined Scan Script:**
    *   Create a single Bash script that takes a target host and a comma-separated list of ports as arguments.
    *   The script should first check the host's reachability.
    *   If the host is up, it should then scan each specified port using either `nc` or `/dev/tcp` (choose one and stick with it), reporting the status of each port.
    *   The output should be clear and human-readable.

## Deliverables

For tasks 1-3, provide the exact Bash command-line solution. For task 4, provide the complete Bash script file.

## Reflection Questions

1.  Compare the output and behavior of `nc` versus `/dev/tcp` for port scanning. What are the advantages and disadvantages of each method?
2.  How did you handle timeouts for `ping` and port scan attempts? Why are timeouts crucial for network utilities?
3.  Consider using your combined scan script on a local network segment (e.g., `192.168.1.0/24`). How could you modify the script to scan an entire range of IP addresses?
4.  What are the limitations of using these basic Bash tools for port scanning compared to a dedicated tool like Nmap (which we are intentionally avoiding for this exercise)?
5.  What security considerations should be kept in mind when performing port scans, even basic ones?

---
# Python Exercise: Network Connectivity and Basic Port Scanning

## Objective

This exercise challenges you to apply your Python scripting skills to perform fundamental network connectivity checks and basic TCP port scanning. You will use Python's standard library modules (`socket`, `subprocess`, `json`) to determine host reachability and identify the status of common ports on a target, demonstrating proficiency in cross-platform network diagnostics and structured reporting.

## Framework Alignment

This exercise on "**Network Connectivity and Port Scanner**" using **Python** provides practical experience in building and utilizing tools for a "Cross-Platform Baseline Hardening & Auditing Framework." By completing these tasks, you will learn how to audit and manage network connectivity and port accessibility, ensuring that only authorized services are exposed—an essential step in maintaining a secure and auditable environment.


## Scenario

You need to develop a portable Python script that can quickly assess the network presence and exposed services of a remote host. The script should perform an ICMP echo request (`ping`) to check host liveness, and then attempt to establish TCP connections to a few common service ports. All collected data should be output in a structured JSON format.

## Target Information

*   **Target Host:** `scanme.nmap.org` (This host is provided by the Nmap project specifically for testing. Please do NOT scan other public IPs without explicit permission.)
*   **Target Ports:** `22` (SSH), `80` (HTTP), `443` (HTTPS), `3389` (RDP - likely closed/filtered on scanme.nmap.org)

## Tasks

Write a Python script (`network_scanner.py` or similar name) that, when executed, performs the following tasks and outputs the aggregated data as a single JSON object to standard output. Your script should handle potential errors gracefully (e.g., host unreachable, connection timeouts).

1.  **Host Reachability Check:**
    *   Implement a function `check_host_reachability(host, timeout)` that uses the `ping` command (via `subprocess`) to determine if the target host is "Up" or "Down". Include a timeout for the ping command.

2.  **Basic TCP Port Scan:**
    *   Implement a function `scan_port(host, port, timeout)` that uses the `socket` module to attempt a TCP connection to the specified port on the host. It should return "Open", "Closed", or "Filtered" (e.g., if it times out or other socket errors occur). Include a timeout for the socket connection.

3.  **Combine and Scan:**
    *   The main part of your script should:
        *   Take the target host and list of ports.
        *   First, call `check_host_reachability`.
        *   If the host is "Up", then iterate through the `Target Ports` and call `scan_port` for each.
        *   Collect all results into a structured Python dictionary/list.

4.  **Output as JSON:**
    *   Serialize the final structured results (including host status and all scanned ports with their statuses) into a JSON string and print it to standard output. The JSON output should be well-formatted (e.g., using `indent=2`).

5.  **(Optional) Implement Basic Parallelism:**
    *   Modify your script to use `concurrent.futures.ThreadPoolExecutor` to scan multiple ports on a host concurrently. This should significantly speed up the port scanning phase.

## Deliverables

Provide the complete Python script file (`network_scanner.py`) that implements all the above tasks and prints the final JSON report to `stdout`.

## Reflection Questions

1.  How did the `socket` module simplify the process of establishing TCP connections and checking port status compared to parsing `nc` or `telnet` output?
2.  Explain how you handled timeouts for both the `ping` command and the `socket` connection attempts. Why is robust timeout handling critical for network utilities?
3.  If you implemented parallelism, how did `concurrent.futures.ThreadPoolExecutor` improve the efficiency of your port scanner? What are the considerations when choosing the number of `max_workers`?
4.  What are the advantages of outputting scan results as JSON in Python, especially when integrating with other tools or automation workflows?
5.  What are the limitations of this Python-based "connect scan" compared to more advanced scanning techniques (e.g., SYN scan, UDP scan) found in tools like Nmap?

---
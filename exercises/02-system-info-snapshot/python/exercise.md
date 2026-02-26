# Python Exercise: System Information Snapshot and Reporting

## Objective

This exercise challenges you to apply your Python scripting skills to collect a comprehensive snapshot of essential system information. You will use Python's standard library modules (e.g., `platform`, `os`, `subprocess`, `json`, `re`) to gather operating system details, hardware specifications, resource usage, and active processes, demonstrating proficiency in cross-platform system diagnostics and structured reporting.

## Framework Alignment

This exercise on "**System Information Snapshot and Reporting**" using **Python** provides practical experience in building and utilizing tools for a "Cross-Platform Baseline Hardening & Auditing Framework." By completing these tasks, you will learn how to audit and manage system configuration data, ensuring that system baselines are maintained and identifying unauthorized changes—essential steps in maintaining a secure and auditable environment.


## Scenario

You are tasked with developing a portable Python script that can collect key system metrics from various operating systems (Linux, macOS, Windows). The script should output the collected data in a structured JSON format, making it easy for automated systems or other programs to consume and analyze.

## Tasks

Write a Python script (`sys_snapshot.py` or similar name) that, when executed, performs the following tasks and outputs the aggregated data as a single JSON object to standard output. Your script should handle potential errors gracefully (e.g., commands not found) by reporting them in the output.

1.  **Collect Operating System Information:**
    *   Include OS name, version, architecture, node name (hostname), and machine type. For Linux, try to get distribution-specific names and versions (e.g., "Ubuntu 22.04 LTS").

2.  **Collect CPU Information:**
    *   Include CPU model name, total number of logical processors (cores), and architecture.

3.  **Collect Memory Information:**
    *   Include total physical memory and free physical memory, preferably in a human-readable format (e.g., "32 GB").

4.  **Collect Disk Space Usage:**
    *   For each mounted filesystem (excluding temporary/virtual filesystems), include the filesystem identifier, type, total size, used space, available space, and mount point.

5.  **Collect Network Interface Information:**
    *   For each active network interface, include its name, MAC address, status (up/down), and all assigned IPv4 addresses.

6.  **Collect Top 5 CPU-Consuming Processes:**
    *   List the top 5 processes currently consuming the most CPU. For each, include the user, PID, %CPU, and command.

7.  **Collect System Uptime:**
    *   Report how long the system has been running.

8.  **Aggregate and Output as JSON:**
    *   Combine all collected data into a single Python dictionary and then serialize it to a JSON string. The JSON output should be well-formatted (e.g., using `indent=2`).

## Deliverables

Provide the complete Python script file (`sys_snapshot.py`) that implements all the above tasks and prints the final JSON report to `stdout`.

## Reflection Questions

1.  How did Python's `platform` module aid in making your script cross-platform, and how did you handle platform-specific commands or parsing challenges?
2.  Explain your approach to executing external system commands and capturing their output in Python. What are the advantages of using `subprocess.run` over older methods like `os.system`?
3.  Describe how you structured the collected data in Python before converting it to JSON. Why is this structured approach beneficial for system reporting?
4.  If you wanted to extend this script to collect other metrics (e.g., currently listening network ports, installed software list), what modules or external commands would you consider using?
5.  What are the advantages and disadvantages of using Python for system information collection and reporting compared to Bash, PowerShell, or a database like SQLite?

---
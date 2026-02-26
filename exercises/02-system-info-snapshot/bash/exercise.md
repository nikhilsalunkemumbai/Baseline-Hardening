# Bash Exercise: System Information Snapshot and Reporting

## Objective

This exercise challenges you to apply your Bash scripting skills to collect a snapshot of essential system information. You will use standard command-line utilities to gather operating system details, hardware specifications, resource usage, and active processes, demonstrating proficiency in system diagnostics and reporting with minimal dependencies.

## Framework Alignment

This exercise on "**System Information Snapshot and Reporting**" using **Bash** provides practical experience in building and utilizing tools for a "Cross-Platform Baseline Hardening & Auditing Framework." By completing these tasks, you will learn how to audit and manage system configuration data, ensuring that system baselines are maintained and identifying unauthorized changes—essential steps in maintaining a secure and auditable environment.


## Scenario

You are tasked with performing a quick audit of a Linux (or macOS) server's current configuration and state. Your goal is to gather key system metrics and present them in a concise, human-readable format, suitable for a system report or for initial troubleshooting.

## Tasks

Using only standard Bash commands and utilities (e.g., `uname`, `lscpu`, `free`, `df`, `ip`, `ps`, `uptime`, `who`), provide the command-line solution or a simple script for each of the following tasks. Combine commands using pipes (`|`) and redirection (`>`) where appropriate.

1.  **Operating System Summary:**
    *   Display the kernel name, kernel release, machine hardware name (architecture), and the full name of the operating system (e.g., "Ubuntu 22.04 LTS").

2.  **CPU Core and Model Information:**
    *   Show the CPU model name and the total number of CPU cores (logical processors).

3.  **Memory Usage Overview:**
    *   Display the total, used, and free physical memory in a human-readable format (e.g., "31G", "11G").

4.  **Disk Space Usage (Non-Temporary Filesystems):**
    *   List all mounted filesystems (excluding `tmpfs`, `udev`, `devtmpfs`) showing their size, used space, available space, and mount point in a human-readable format.

5.  **Active Network Interfaces and IP Addresses:**
    *   List all active network interfaces and their primary IPv4 addresses.

6.  **Top 5 CPU-Consuming Processes:**
    *   Display the top 5 processes consuming the most CPU, showing their User, PID, %CPU, and Command.

7.  **System Uptime and Load Average:**
    *   Show how long the system has been running and its load average.

## Deliverables

For each task, provide the Bash command-line pipeline or script snippet that produces the required output. For tasks 1-7, you can optionally combine them into a single Bash script that prints all requested information.

## Reflection Questions

1.  Which standard Bash commands did you find most effective for retrieving hardware-related information (CPU, Memory, Disk)?
2.  How did you ensure the output was "human-readable" for tasks like memory and disk usage?
3.  Consider the challenge of cross-platform compatibility for a Bash script (e.g., between Linux and macOS). Which commands might behave differently or require alternatives?
4.  If you needed to collect this data from 100 remote servers, how would you automate the execution of your script and collection of its output?
5.  What are the limitations of using Bash for system information collection if you needed to store this data in a structured format for historical analysis?

---
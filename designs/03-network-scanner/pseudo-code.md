# Design Concept: Network Connectivity and Port Scanner

## I. Overview

This utility is designed to verify network reachability of target hosts and identify the status (open, closed, filtered) of specified TCP ports. It aims to provide a basic, cross-platform, and lightweight tool for network diagnostics, reconnaissance, and troubleshooting, adhering to the principle of minimal external dependencies.

## Framework Alignment

This design for "**Network Connectivity and Port Scanner**" provides a platform-agnostic blueprint for building and utilizing tools for a "Cross-Platform Baseline Hardening & Auditing Framework." The core functionalities of auditing network connectivity and port accessibility are essential for ensuring compliance with security baselines and identifying unauthorized network exposures across diverse operating environments.


## II. Core Functionality

### A. Target Specification

1.  **Host(s):**
    *   Single IP address or hostname.
    *   List of IP addresses or hostnames (e.g., comma-separated, space-separated).
    *   CIDR notation (e.g., `192.168.1.0/24`) for scanning a range of IP addresses (optional, as it adds complexity, prioritize single/list first).

2.  **Port(s):**
    *   Single port number (e.g., `80`).
    *   List of port numbers (e.g., `22,80,443`).
    *   Range of port numbers (e.g., `1-1024`).
    *   Predefined "common ports" (e.g., SSH, HTTP, HTTPS) as an option.

### B. Connectivity Check (Host Reachability)

1.  **Mechanism:** Perform a basic ICMP Echo Request (ping) to determine if the host is reachable before attempting port scans.
2.  **Outcome:** Report "Host Up" or "Host Down/Unreachable".
3.  **Timeout:** Configurable timeout for ping requests.

### C. Port Scanning Logic

1.  **Mechanism (TCP Connect Scan):** Attempt to establish a full TCP connection to each specified port on the target host.
    *   This is the simplest and most widely supported method, although it can be slower and more easily detected than SYN scans.
    *   Upon successful connection, the port is considered "Open".
    *   Upon connection refused (RST packet), the port is considered "Closed".
    *   Upon timeout (no response), the port is considered "Filtered" or "Closed" (depending on interpretation and timeout duration).
2.  **Timeout:** Configurable timeout for individual port connection attempts.
3.  **Concurrency (Optional/Basic):** For performance, allow a limited number of parallel connection attempts (e.g., 5-10 ports simultaneously) without introducing complex threading frameworks. This can be achieved with non-blocking sockets or simple process/thread pooling in some languages. Prioritize sequential scanning for minimal dependency if concurrency adds too much complexity.

### D. Output

1.  **Standard Output (Human-readable Text):**
    *   Clear, concise summary for each host and port.
    *   Example:
        ```
        Host: 192.168.1.1 (Up)
          Port 22: Open
          Port 80: Closed
          Port 443: Filtered (Timeout)
        Host: 192.168.1.2 (Down)
        ```
2.  **JSON Output:**
    *   Structured representation of all scan results for programmatic consumption.
    *   Example:
        ```json
        [
          {
            "host": "192.168.1.1",
            "host_status": "Up",
            "ports": [
              {"port": 22, "status": "Open"},
              {"port": 80, "status": "Closed"},
              {"port": 443, "status": "Filtered"}
            ]
          },
          {
            "host": "192.168.1.2",
            "host_status": "Down",
            "ports": []
          }
        ]
        ```

### E. Error Handling

*   Invalid IP address/hostname format.
*   Invalid port range.
*   Permission errors (e.g., for ICMP on some OS).
*   Network unreachable.

## III. Data Structures

*   The scan results can be represented as a list of dictionaries/objects.
*   Each host entry in the list would contain: `host` (string), `host_status` (string: "Up", "Down"), and a `ports` list.
*   Each port entry in the `ports` list would contain: `port` (integer), `status` (string: "Open", "Closed", "Filtered").

## IV. Guiding Principles (from cosolitdations.txt)

*   **Portability:** Implementations should strive for maximum compatibility across Windows, Linux, and macOS. Avoid low-level OS-specific network APIs unless absolutely necessary and provide clear alternatives.
*   **Efficiency:** Scanning should be reasonably fast for common use cases (e.g., scanning a few ports on a single host). Focus on connection-based methods rather than slower application-layer probes.
*   **Minimal Dependencies:** Solutions must primarily use standard language utilities and built-in networking capabilities (e.g., Python's `socket` module, PowerShell's `Test-NetConnection`, Bash's `nc` or `telnet`). Avoid external, specialized network scanning tools or libraries like Nmap.
*   **CLI-centric:** The tool should be primarily controlled via command-line arguments, making it suitable for scripting and integration into automation workflows.
*   **Safe Scanning:** Emphasize that this is a basic diagnostic tool. Avoid techniques that could be construed as intrusive or malicious. Respect network policies.

---
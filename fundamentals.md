# Fundamentals: Hardening & Auditing Framework

The **Baseline-Hardening** library provides technical building blocks for infrastructure resilience and security automation in restricted environments.

## 1. The Hardening Lifecycle
The framework automates the loop between policy and state:
*   **Policy as Code**: Security standards are defined in YAML/JSON.
*   **Continuous Audit**: Snippets verify if the system adheres to the baseline.
*   **Remediation Guidance**: Failures trigger actionable CLI commands to restore compliance.

## 2. Core Technical Principles
*   **Minimal Dependencies**: Favor standard libraries and built-in OS tools (`grep`, `systemctl`, `net user`) over third-party agents.
*   **Portability**: Scripts must function across Windows, Linux, and macOS.
*   **Structured Output**: Every utility is designed to output **JSON**. This makes results "pipable" into SQLite or Python logic.

## 3. The 11 Design Concepts
The library is structured into 11 domains required for a robust baseline:
1.  **Log Analysis**: Extracting events from SSH/Web logs.
2.  **System Snapshot**: Capturing OS, CPU, and Memory state.
3.  **Network Scanner**: Port scanning and connectivity checks.
4.  **File Integrity**: Cryptographic hashing and change detection.
5.  **Process Management**: Monitoring and controlling active tasks.
6.  **User/Group Audit**: Identifying privileged or weak accounts.
7.  **Config Validation**: Verifying settings in INI, JSON, and YAML.
8.  **Service Monitoring**: Health checks for system daemons.
9.  **Scheduled Tasks**: Auditing Cron and Windows Task Scheduler.
10. **Cryptography**: Safe encoding (Base64/Hex) and hashing.
11. **Drift Detection**: Differential analysis between snapshots.

## 4. Multi-Technology Strategy
*   **Bash**: High-speed text processing on Unix-like systems.
*   **PowerShell**: Native object-oriented management for Windows.
*   **Python**: Complexity handling and framework integration.
*   **SQLite**: Persistent historical storage and SQL-based auditing.

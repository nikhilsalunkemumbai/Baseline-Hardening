# Design Concept: Scheduled Task/Job Management

## I. Overview

This utility is designed to provide a standardized, cross-platform interface for managing scheduled tasks or jobs. It allows for programmatically listing existing tasks, creating new ones, modifying their properties, and deleting them. The primary goal is to facilitate automated system maintenance, administrative tasks, and recurring operational procedures, emphasizing minimal dependencies and command-line utility for seamless integration into scripts and automation workflows.

## Framework Alignment

This design for "**Scheduled Task/Job Management**" provides a platform-agnostic blueprint for building and utilizing tools for a "Cross-Platform Baseline Hardening & Auditing Framework." The core functionalities of listing, creating, and modifying scheduled tasks are essential for auditing system configurations against defined security baselines and ensuring compliance across diverse operating environments.


## II. Core Functionality

### A. List Scheduled Tasks/Jobs

1.  **Retrieve All Tasks:** Get a comprehensive list of all scheduled tasks or cron jobs configured on the system.
2.  **Filter Tasks:** Allow filtering of tasks/jobs by various criteria:
    *   By Name (exact match or pattern)
    *   By Status (e.g., enabled, disabled, running, failed)
    *   By User/Owner
    *   By Command/Script being executed
3.  **Display Details:** For each task/job, provide relevant information:
    *   Unique Name/Identifier
    *   Description (if available)
    *   Command or Script to execute
    *   User context under which the task runs
    *   Working Directory
    *   Full Schedule details (e.g., cron expression, frequency, specific time)
    *   Last Run Time and Status
    *   Next Run Time
    *   Enabled/Disabled Status

### B. Create Scheduled Task/Job

1.  **Define Task Parameters:** Allow specifying all necessary parameters for a new task:
    *   **`Name`**: A unique identifier for the task.
    *   **`Description`**: A brief explanation of the task's purpose.
    *   **`Command`**: The executable or script to be run (including arguments).
    *   **`User`**: The user account under which the task should execute.
    *   **`WorkingDirectory`**: The directory in which the command will be executed.
    *   **`Schedule`**:
        *   **Frequency:** (e.g., `once`, `daily`, `weekly`, `monthly`, `on_boot`, `on_logon`, `interval_minutes`).
        *   **Time/Date:** Specific time of day, day of week/month, or interval.
        *   **Cron-like Expression:** For advanced scheduling (especially on Unix-like systems).
    *   **`Error Handling`**: (Optional) Configuration for logging output, retries on failure, or notifications.
    *   **`Enabled`**: Initial state of the task (enabled by default).

### C. Modify Scheduled Task/Job

1.  **Update Parameters:** Given a task/job name, allow updating any of its configurable parameters (e.g., change the command, alter the schedule, update the user context).

### D. Delete Scheduled Task/Job

1.  **Remove Task by Name:** Permanently remove a scheduled task/job from the system by its unique name.

### E. Enable/Disable Scheduled Task/Job

1.  **Toggle State:** Change the enabled/disabled status of a specific task/job without deleting it, effectively pausing or resuming its execution.

### F. Run Scheduled Task/Job (On Demand)

1.  **Trigger Immediately:** Manually initiate the execution of a scheduled task/job, bypassing its defined schedule. This is useful for testing or immediate administrative actions.

### G. Reporting and Output

1.  **Standard Output:** Human-readable text output summarizing task details or confirmation of actions.
2.  **Structured Output (JSON):** Generate machine-readable reports of listed tasks or the results of creation/modification/deletion operations. Ideal for integration with automation systems.

### H. Error Handling

*   Task/Job Not Found (for modify, delete, enable/disable, run operations).
*   Invalid Schedule Definition.
*   Permission Denied (e.g., attempting to manage tasks for another user).
*   Invalid Command/Script Path.
*   Duplicate Task Name (for creation).

## III. Data Structures

*   **Task/Job Details Object:**
    ```
    {
        "name": "backup_daily",
        "description": "Daily backup of /var/www",
        "command": "/usr/local/bin/backup_script.sh",
        "user": "root",
        "working_directory": "/opt/backup",
        "schedule": {
            "type": "cron",
            "expression": "0 2 * * *" # At 02:00 daily
        },
        "status": "enabled",
        "last_run": "2026-02-25T02:00:00Z",
        "next_run": "2026-02-26T02:00:00Z",
        "platform_specific_id": "GUID_OR_CRONTAB_ENTRY_ID"
    }
    ```
*   **Schedule Definition Object (within Task/Job Details):**
    *   Can vary based on complexity, e.g., for `daily`: `{"type": "daily", "time": "02:00"}`
    *   For `weekly`: `{"type": "weekly", "day_of_week": "Sunday", "time": "03:00"}`

## IV. Guiding Principles (from cosolitdations.txt)

*   **Portability:** Leverage OS-native command-line tools (`crontab`, `schtasks`) or standard language-specific APIs. Abstract common scheduling patterns for cross-platform consistency where possible.
*   **Efficiency:** Operations should be quick and directly interact with the OS scheduler without unnecessary overhead.
*   **Minimal Dependencies:** Prioritize built-in OS commands and standard language libraries. Avoid reliance on complex, external job scheduling frameworks unless they are explicitly part of the target environment.
*   **CLI-centric:** Designed for command-line execution and seamless integration into automated scripts and larger orchestration systems.
*   **Automation Focus:** Directly supports automating routine system administration, maintenance, security operations, and any other recurring tasks, thereby enhancing operational efficiency and reliability.

---
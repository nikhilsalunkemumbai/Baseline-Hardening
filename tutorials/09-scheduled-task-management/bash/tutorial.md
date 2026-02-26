# Bash Tutorial: Scheduled Task/Job Management

## Introduction

In Linux/Unix environments, `cron` is the most common and powerful utility for scheduling recurring tasks. `crontab` (cron table) files allow users to define commands to be executed automatically at specific intervals. For one-time tasks, the `at` command provides a simple scheduling mechanism. Modern Linux distributions often leverage `systemd` timers as an alternative, offering more flexibility and better integration with system services. This tutorial will guide you through managing scheduled tasks using these Bash-centric tools, adhering to our principles of minimal dependencies and CLI-centric operation.

## Framework Alignment

This tutorial on "**Scheduled Task/Job Management**" using **Bash** demonstrates the foundational skills necessary to implement components of a "Cross-Platform Baseline Hardening & Auditing Framework." The techniques learned here for managing scheduled tasks are directly applicable to auditing systems against defined security baselines and ensuring compliance.


## Core Bash Utilities for Scheduled Task Management

*   **`crontab`**: Used to install, uninstall or list the tables used to drive the cron daemon. Each user has their own crontab.
    *   `crontab -l`: List current user's crontab entries.
    *   `crontab -e`: Edit current user's crontab entries.
    *   `crontab -r`: Remove current user's crontab.
*   **`cron` daemon**: The background process that executes scheduled commands.
*   **`at`**: Schedules commands to be executed once, at a specific time.
    *   `at now + 5 minutes`: Execute in 5 minutes.
    *   `at -l`: List pending `at` jobs.
    *   `at -r <job_number>`: Remove an `at` job.
*   **`systemctl`** (for `systemd` timers):
    *   `systemctl list-timers`: List active systemd timers.
    *   `systemctl start <timer_name>.timer`: Start a timer unit.
    *   `systemctl enable <timer_name>.timer`: Enable a timer unit to start on boot.
*   **`grep`** / **`sed`** / **`awk`**: For parsing and manipulating crontab content if managing programmatically.

## Understanding Cron Syntax

A cron entry (or "cronjob") has six fields:

```
┌───────────── minute (0 - 59)
│ ┌───────────── hour (0 - 23)
│ │ ┌───────────── day of month (1 - 31)
│ │ │ ┌───────────── month (1 - 12)
│ │ │ │ ┌───────────── day of week (0 - 6) (0 and 7 are Sunday, or use names)
│ │ │ │ │
* * * * *  command to execute
```

*   `*`: Any value (e.g., `*` in the minute field means "every minute").
*   `,`: List separator (e.g., `10,30` in the minute field means "at minute 10 and 30").
*   `-`: Range (e.g., `9-17` in the hour field means "hours 9 through 17").
*   `/`: Step values (e.g., `*/15` in the minute field means "every 15 minutes").

### Special Strings (convenience shortcuts):

*   `@reboot`: Run once at startup.
*   `@yearly`, `@annually`: Run once a year.
*   `@monthly`: Run once a month.
*   `@weekly`: Run once a week.
*   `@daily`, `@midnight`: Run once a day.
*   `@hourly`: Run once an hour.

## Implementing Core Functionality with Bash

### 1. Listing Scheduled Tasks (Cron)

```bash
#!/bin/bash

echo "Current user's cron jobs:"
crontab -l 2>/dev/null || echo "No cron jobs found for current user."
echo ""

echo "System-wide cron jobs (from /etc/cron.* directories):"
ls -l /etc/cron.{daily,hourly,monthly,weekly} /etc/cron.d/ 2>/dev/null
```

### 2. Creating a Cron Job

The recommended way to create or modify a cron job programmatically is to first export the current crontab, modify it, and then import it. This avoids issues with concurrent edits.

```bash
#!/bin/bash

# Define a temporary file for crontab editing
CRONTAB_TEMP_FILE="/tmp/my_new_crontab_$$" # $$ for unique process ID

# 1. Export current crontab
crontab -l > "$CRONTAB_TEMP_FILE" 2>/dev/null || true

# 2. Add new job (e.g., log current date to a file every minute)
NEW_CRON_JOB="* * * * * echo "$(date)" >> /tmp/cron_test.log 2>&1"

# Check if the job already exists to avoid duplicates
if ! grep -qF "$NEW_CRON_JOB" "$CRONTAB_TEMP_FILE"; then
    echo "Adding new cron job: $NEW_CRON_JOB"
    echo "$NEW_CRON_JOB" >> "$CRONTAB_TEMP_FILE"
    # 3. Import the modified crontab
    crontab "$CRONTAB_TEMP_FILE"
    echo "Cron job added. Verify with 'crontab -l'"
else
    echo "Cron job already exists: $NEW_CRON_JOB"
fi

# Clean up temporary file
rm "$CRONTAB_TEMP_FILE"
```
**Caution:** Be careful when using `crontab` programmatically. Always back up the original and validate your changes.

### 3. Deleting a Cron Job

```bash
#!/bin/bash

CRONTAB_TEMP_FILE="/tmp/my_crontab_del_$$"
JOB_TO_DELETE="* * * * * echo "$(date)" >> /tmp/cron_test.log 2>&1"

# 1. Export current crontab
crontab -l > "$CRONTAB_TEMP_FILE" 2>/dev/null || true

# 2. Filter out the job to delete
# Using 'grep -v' to remove lines matching the job
if grep -qF "$JOB_TO_DELETE" "$CRONTAB_TEMP_FILE"; then
    echo "Deleting cron job: $JOB_TO_DELETE"
    grep -vF "$JOB_TO_DELETE" "$CRONTAB_TEMP_FILE" > "$CRONTAB_TEMP_FILE.new"
    # 3. Import the modified crontab
    mv "$CRONTAB_TEMP_FILE.new" "$CRONTAB_TEMP_FILE"
    crontab "$CRONTAB_TEMP_FILE"
    echo "Cron job deleted. Verify with 'crontab -l'"
else
    echo "Cron job not found: $JOB_TO_DELETE"
fi

# Clean up temporary file
rm "$CRONTAB_TEMP_FILE"
```

### 4. Scheduling a One-Time Task (`at`)

```bash
#!/bin/bash

# Schedule a command to run in 1 minute
echo "echo 'One-time task executed at $(date)' >> /tmp/at_test.log" | at now + 1 minute

echo "One-time task scheduled. List pending jobs with 'at -l'."
sleep 5 # Wait for a bit
at -l

# To remove a pending 'at' job, find its job number with 'at -l' and then:
# at -r <job_number>
```

### 5. Managing Systemd Timers (Modern Linux)

Systemd timers (`.timer` units) are an alternative to cron for scheduling tasks. They are often preferred for their integration with systemd's logging, dependencies, and resource management.

#### a. Listing systemd timers

```bash
#!/bin/bash

echo "Systemd timers:"
systemctl list-timers --all --no-pager
```

#### b. Creating a Systemd Timer (conceptual outline)

Creating a systemd timer involves two unit files: a `.service` file defining what to run, and a `.timer` file defining when to run it.

**Example `mybackup.service` (in `/etc/systemd/system/`):**
```
[Unit]
Description=My daily backup service

[Service]
Type=oneshot
ExecStart=/usr/local/bin/daily_backup.sh
```

**Example `mybackup.timer` (in `/etc/systemd/system/`):**
```
[Unit]
Description=Run daily backup every day at 3 AM

[Timer]
OnCalendar=*-*-* 03:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

After creating these files:
```bash
sudo systemctl daemon-reload           # Reload systemd manager configuration
sudo systemctl enable mybackup.timer   # Enable the timer to start on boot
sudo systemctl start mybackup.timer    # Start the timer immediately
systemctl list-timers mybackup.timer   # Verify it's active
```
**Note:** Managing systemd timers programmatically from a Bash script involves writing these files and executing `systemctl` commands, which usually requires `sudo` privileges.

### 6. Consolidated Script Example (Conceptual for Cron)

A simple script to list cron jobs for the current user and check for a specific job.

```bash
#!/bin/bash

CRON_JOB_TO_CHECK="* * * * * echo "Check me" >> /tmp/check_log.log"

echo "--- Cron Job Status Report ---"
echo ""

# List all cron jobs for the current user
echo "All current user cron jobs:"
crontab -l 2>/dev/null || echo "  (None)"
echo ""

# Check for a specific job
if crontab -l 2>/dev/null | grep -qF "$CRON_JOB_TO_CHECK"; then
    echo "Specific job found: '$CRON_JOB_TO_CHECK' is PRESENT."
else
    echo "Specific job check: '$CRON_JOB_TO_CHECK' is ABSENT."
fi
echo ""

echo "--- Report End ---"
```

## Guiding Principles in Bash

*   **Portability:** `crontab` and `at` are standard on virtually all Unix-like systems. `systemd` timers are specific to Systemd-based Linux but are now very common.
*   **Efficiency:** Direct interaction with the cron daemon or systemd ensures minimal overhead.
*   **Minimal Dependencies:** Relies entirely on core system utilities.
*   **CLI-centric:** All operations are command-line based, ideal for scripting and quick management.
*   **Automation Focus:** Directly manages the scheduling of automated tasks, forming the backbone of many system automation processes.

## Conclusion

Bash, through tools like `crontab`, `at`, and `systemctl` (for timers), provides powerful and flexible capabilities for managing scheduled tasks and jobs on Linux/Unix systems. While programmatic manipulation of `crontab` requires care, these tools are indispensable for automating routine administrative duties and ensuring the timely execution of critical tasks. The next step is to apply this knowledge in practical exercises.
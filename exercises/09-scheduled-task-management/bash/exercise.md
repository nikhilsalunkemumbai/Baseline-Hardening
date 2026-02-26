# Bash Exercise: Scheduled Task/Job Management

## Objective

This exercise challenges you to apply your Bash scripting and `crontab` skills to manage scheduled tasks (cron jobs) on a Linux system. You will learn to create, list, verify, modify, and delete recurring jobs, demonstrating proficiency in automating system administration tasks.

## Framework Alignment

This exercise on "**Scheduled Task/Job Management**" using **Bash** provides practical experience in building and utilizing tools for a "Cross-Platform Baseline Hardening & Auditing Framework." By completing these tasks, you will learn how to audit and manage scheduled tasks, ensuring compliance with security policies and preventing unauthorized persistence mechanisms—essential steps in maintaining a secure and auditable environment.


## Scenario

You are a system administrator responsible for maintaining a Linux server. You need to automate two key tasks:
1.  A daily system health check script.
2.  Ensuring a critical backup script runs weekly.

You will use `crontab` to manage these recurring jobs and `at` for a one-time task.

## Setup

1.  **Create a dummy script for your cron job:**
    Create a file named `daily_health_check.sh` with the following content:
    ```bash
    #!/bin/bash
    echo "Health check ran at $(date)" >> /tmp/health_check.log
    ```
    Make it executable: `chmod +x daily_health_check.sh`

2.  **Create another dummy script for the backup:**
    Create a file named `weekly_backup.sh` with the following content:
    ```bash
    #!/bin/bash
    echo "Weekly backup ran at $(date)" >> /tmp/weekly_backup.log
    ```
    Make it executable: `chmod +x weekly_backup.sh`

## Tasks

Using only standard Bash commands and utilities (`crontab`, `at`, `grep`, `sed`), provide the command-line solution or a simple script for each of the following tasks.

### Part 1: Managing Cron Jobs

1.  **List Current Cron Jobs:**
    *   Display all cron jobs currently configured for your user.

2.  **Create Daily Health Check Job:**
    *   Add a new cron job that executes `~/daily_health_check.sh` every day at `03:00 AM`.

3.  **Verify New Cron Job:**
    *   List your cron jobs again and use `grep` to confirm that the `daily_health_check.sh` job has been successfully added.

4.  **Create or Update Weekly Backup Job:**
    *   Ensure that `~/weekly_backup.sh` runs every Sunday at `02:00 AM`. If such a job already exists, ensure its time/schedule is updated. If not, create it. (Hint: You might need to combine `crontab -l`, `grep -v`, `echo`, and `crontab -` in a script.)

5.  **Disable Daily Health Check Job (by commenting it out):**
    *   Modify your crontab to comment out the `daily_health_check.sh` job. The entry should still be there, but preceded by a `#`.

6.  **Delete Weekly Backup Job:**
    *   Remove the `weekly_backup.sh` cron job entirely from your crontab.

### Part 2: One-Time Task with `at`

1.  **Schedule a One-Time Alert:**
    *   Schedule a task using `at` that displays "Time for a break!" on your terminal in `5 minutes` from now. (Hint: The command needs to be properly quoted and echoed into `at`).

2.  **List Pending `at` Jobs:**
    *   Display all jobs currently queued with `at`.

3.  **Cancel One-Time Alert:**
    *   Identify the job number for your alert and cancel it using `at -r`.

## Deliverables

For each task, provide the exact Bash command-line solution. For tasks involving creation/modification/deletion of cron jobs, provide the sequence of commands or a small script that achieves the desired state.

## Reflection Questions

1.  What are the advantages and disadvantages of using `crontab -e` for interactive editing versus programmatically manipulating the crontab using a script (export, modify, import)?
2.  Explain the purpose of `crontab -l`, `crontab -r`, and `crontab -` when used in conjunction with redirection.
3.  How does the `at` command differ from `cron` in terms of scheduling capabilities and use cases?
4.  If you needed to manage system-wide cron jobs (e.g., in `/etc/cron.d/`), how would your approach differ from managing a user's crontab?
5.  What security considerations should be kept in mind when creating or modifying cron jobs, especially for jobs running as `root`?

---
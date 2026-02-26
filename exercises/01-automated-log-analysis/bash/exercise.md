# Bash Exercise: Automated Log File Analysis - Failed SSH Attempts and Apache Errors

## Objective

This exercise challenges you to apply your Bash text processing skills to analyze a sample log file (`sample_log.log`). You will use standard command-line utilities to extract specific information, filter events, and count occurrences, demonstrating proficiency in automated log file analysis.

## Framework Alignment

This exercise on "**Automated Log File Analysis and Event Extraction**" using **Bash** provides practical experience in building and utilizing tools for a "Cross-Platform Baseline Hardening & Auditing Framework." By completing these tasks, you will learn how to audit and analyze log data, identifying security events and ensuring that system logging policies are maintained—essential steps in maintaining a secure and auditable environment.


## Scenario

You are a system administrator investigating unusual activity on a server. You suspect there might be an increase in failed SSH login attempts and recurring web server errors. Your task is to analyze the provided `sample_log.log` to identify and quantify these events.

## Input Data

You will be working with the `sample_log.log` file located in the same directory as this exercise.

```
Feb 25 10:00:01 host-a sshd[1234]: Accepted password for user1 from 192.168.1.10 port 54321 ssh2
Feb 25 10:00:05 host-b kernel: [  123.456] usb 1-1: new high-speed USB device number 2 using ehci-pci
Feb 25 10:00:10 host-a sudo[5678]:    user1 : TTY=pts/0 ; PWD=/home/user1 ; USER=root ; COMMAND=/usr/bin/apt update
Feb 25 10:00:15 host-c CRON[9876]: (root) CMD (command -v lsb_release >/dev/null && lsb_release -cs || echo trusty)
Feb 25 10:00:20 host-a sshd[1235]: Failed password for invalid user from 192.168.1.11 port 54322 ssh2
Feb 25 10:00:25 host-d systemd[1]: Starting Cleanup of Temporary Directories...
Feb 25 10:00:30 host-a sshd[1236]: Failed password for user2 from 192.168.1.10 port 54323 ssh2
Feb 25 10:00:35 host-e kernel: [  123.457] ata1: link is slow to respond, please be patient (PHY changed)
Feb 25 10:00:40 host-a systemd[1]: Started Session 1 of user user1.
Feb 25 10:00:45 host-f apache2[1000]: [error] [client 10.0.0.5] File does not exist: /var/www/html/nonexistent.html
Feb 25 10:00:50 host-a sshd[1237]: Accepted password for user3 from 192.168.1.12 port 54324 ssh2
Feb 25 10:00:55 host-a sudo[5679]:    user3 : TTY=pts/0 ; PWD=/home/user3 ; USER=root ; COMMAND=/usr/bin/cat /etc/shadow
Feb 25 10:01:00 host-g kernel: [  123.458] NOHZ: local_softirq_pending 08
Feb 25 10:01:05 host-a sshd[1238]: Failed password for user4 from 192.168.1.11 port 54325 ssh2
Feb 25 10:01:10 host-h systemd[1]: Reached target Network.
Feb 25 10:01:15 host-a sshd[1239]: Accepted password for user5 from 192.168.1.13 port 54326 ssh2
Feb 25 10:01:20 host-a apache2[1001]: [error] [client 10.0.0.6] File does not exist: /var/www/html/another_nonexistent.html
Feb 25 10:01:25 host-a sshd[1240]: Failed password for invalid user from 192.168.1.10 port 54327 ssh2
```

## Tasks

Using only standard Bash commands (`grep`, `awk`, `sed`, `cut`, `sort`, `uniq`, etc.), provide the command-line solution for each of the following tasks.

1.  **Find All Failed SSH Login Attempts:**
    *   Display all lines from `sample_log.log` that indicate a "Failed password" attempt.

2.  **Extract Unique Source IPs of Failed SSH Attempts:**
    *   From the failed SSH login attempts, extract and list all unique source IP addresses.

3.  **Count Failed SSH Attempts Per Source IP:**
    *   Count how many "Failed password" attempts originated from each unique source IP address. Sort the results from most frequent to least frequent.

4.  **Identify Top 3 IPs for Failed SSH Attempts:**
    *   Based on the previous task, display only the top 3 source IP addresses with the most "Failed password" attempts and their respective counts.

5.  **Find All Apache Error Messages:**
    *   Display all lines from `sample_log.log` that contain the string `apache2` and `[error]`.

6.  **Extract Unique Non-Existent File Paths from Apache Errors:**
    *   From the Apache error messages, extract and list the unique file paths that "do not exist" (e.g., `/var/www/html/nonexistent.html`).

## Deliverables

For each task, provide the single Bash command-line pipeline that produces the required output.

## Reflection Questions

1.  Which Bash utilities were most useful for filtering versus extracting specific data?
2.  How did piping multiple commands together simplify the tasks?
3.  Consider the efficiency of your commands for very large log files (gigabytes). Are there any commands that might become bottlenecks?
4.  How would you adapt your solutions if the log file format changed slightly (e.g., a different delimiter or a new field)?
5.  What are the advantages and disadvantages of using Bash for log analysis compared to a more structured scripting language like Python or a database like SQLite?

---
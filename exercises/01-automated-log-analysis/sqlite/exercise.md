# SQLite Exercise: Automated Log File Analysis - Failed SSH Attempts and Apache Errors

## Objective

This exercise challenges you to apply your SQL querying skills using SQLite to analyze pre-structured log data. You will import a CSV file into an SQLite database and then use SQL commands to extract specific information, filter events, and count occurrences, demonstrating proficiency in structured log file analysis.

## Framework Alignment

This exercise on "**Automated Log File Analysis and Event Extraction**" using **SQLite** provides practical experience in building and utilizing tools for a "Cross-Platform Baseline Hardening & Auditing Framework." By completing these tasks, you will learn how to audit and analyze log data, identifying security events and ensuring that system logging policies are maintained—essential steps in maintaining a secure and auditable environment.


## Scenario

You are a system administrator investigating unusual activity on a server. Log data has been pre-parsed and is available as a CSV file. Your task is to import this data into an SQLite database and then use SQL queries to identify and quantify failed SSH login attempts and recurring web server errors.

## Input Data

You will be working with the `parsed_log.csv` file located in the same directory as this exercise. This file contains structured log entries.

```csv
timestamp,hostname,program,pid,message,level,source_ip,user
2026-02-25 10:00:01,host-a,sshd,1234,"Accepted password for user1 from 192.168.1.10 port 54321 ssh2",INFO,192.168.1.10,user1
2026-02-25 10:00:05,host-b,kernel,,"usb 1-1: new high-speed USB device number 2 using ehci-pci",INFO,,
2026-02-25 10:00:10,host-a,sudo,5678,"user1 : TTY=pts/0 ; PWD=/home/user1 ; USER=root ; COMMAND=/usr/bin/apt update",INFO,,user1
2026-02-25 10:00:15,host-c,CRON,9876,"(root) CMD (command -v lsb_release >/dev/null && lsb_release -cs || echo trusty)",INFO,,root
2026-02-25 10:00:20,host-a,sshd,1235,"Failed password for invalid user from 192.168.1.11 port 54322 ssh2",ERROR,192.168.1.11,invalid user
2026-02-25 10:00:25,host-d,systemd,1,"Starting Cleanup of Temporary Directories...",INFO,,
2026-02-25 10:00:30,host-a,sshd,1236,"Failed password for user2 from 192.168.1.10 port 54323 ssh2",ERROR,192.168.1.10,user2
2026-02-25 10:00:35,host-e,kernel,,"ata1: link is slow to respond, please be patient (PHY changed)",INFO,,
2026-02-25 10:00:40,host-a,systemd,1,"Started Session 1 of user user1.",INFO,,user1
2026-02-25 10:00:45,host-f,apache2,1000,"[error] [client 10.0.0.5] File does not exist: /var/www/html/nonexistent.html",ERROR,10.0.0.5,
2026-02-25 10:00:50,host-a,sshd,1237,"Accepted password for user3 from 192.168.1.12 port 54324 ssh2",INFO,192.168.1.12,user3
2026-02-25 10:00:55,host-a,sudo,5679,"user3 : TTY=pts/0 ; PWD=/home/user3 ; USER=root ; COMMAND=/usr/bin/cat /etc/shadow",INFO,,user3
2026-02-25 10:01:00,host-g,kernel,,"NOHZ: local_softirq_pending 08",INFO,,
2026-02-25 10:01:05,host-a,sshd,1238,"Failed password for user4 from 192.168.1.11 port 54325 ssh2",ERROR,192.168.1.11,user4
2026-02-25 10:01:10,host-h,systemd,1,"Reached target Network.",INFO,,
2026-02-25 10:01:15,host-a,sshd,1239,"Accepted password for user5 from 192.168.1.13 port 54326 ssh2",INFO,192.168.1.13,user5
2026-02-25 10:01:20,host-a,apache2,1001,"[error] [client 10.0.0.6] File does not exist: /var/www/html/another_nonexistent.html",ERROR,10.0.0.6,
2026-02-25 10:01:25,host-a,sshd,1240,"Failed password for invalid user from 192.168.1.10 port 54327 ssh2",ERROR,192.168.1.10,invalid user
```

## Setup

1.  **Create an SQLite Database:** Open your terminal and create a new SQLite database file.
    ```bash
    sqlite3 logs.db
    ```
2.  **Create a Table:** Inside the `sqlite3` prompt, create a table named `log_entries` with appropriate columns. Pay attention to data types.
    ```sql
    CREATE TABLE log_entries (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        timestamp TEXT NOT NULL,
        hostname TEXT,
        program TEXT,
        pid INTEGER,
        message TEXT NOT NULL,
        level TEXT NOT NULL,
        source_ip TEXT,
        user TEXT
    );
    ```
3.  **Import Data:** Import the `parsed_log.csv` file into the `log_entries` table. Remember to handle the header row.
    ```bash
    # (Inside sqlite3 prompt)
    .mode csv
    .import parsed_log.csv log_entries
    -- IMPORTANT: If your sqlite3 version doesn't support --skip, you will need to delete the header row:
    -- DELETE FROM log_entries WHERE timestamp = 'timestamp';
    ```
    *(If using a scripting language like Python to import, refer to the Python SQLite tutorial.)*

## Tasks

Using SQL queries within the `sqlite3` prompt, provide the solution for each of the following tasks.

1.  **Retrieve All Error Level Log Entries:**
    *   Display all log entries where the `level` is 'ERROR'.

2.  **Count Log Entries by Program:**
    *   Count how many log entries there are for each unique `program` (e.g., 'sshd', 'apache2', 'kernel').

3.  **Find Unique Source IPs for Failed SSH Attempts:**
    *   List all unique `source_ip` addresses associated with log entries where the `program` is 'sshd' and the `message` contains 'Failed password'.

4.  **Count Failed SSH Attempts Per Source IP:**
    *   Count the number of 'Failed password' attempts for each unique `source_ip` where the `program` is 'sshd'. Order the results by the count in descending order.

5.  **Identify Top 3 IPs for Failed SSH Attempts:**
    *   Based on the previous task, retrieve only the top 3 `source_ip` addresses with the most 'Failed password' attempts and their counts.

6.  **Find All Apache Error Messages with Client IP:**
    *   Display the `timestamp`, `message`, and `source_ip` for all entries where the `program` is 'apache2' and the `level` is 'ERROR'.

7.  **Count Apache Errors per Client IP:**
    *   Count how many 'ERROR' messages originated from each unique `source_ip` for the 'apache2' program.

## Deliverables

For each task, provide the SQL query and the resulting output.

## Reflection Questions

1.  How does using a structured database like SQLite simplify complex queries (e.g., counting unique occurrences, grouping data) compared to text-processing tools?
2.  What are the advantages of having log data stored in a persistent, queryable format like SQLite for incident response or historical analysis?
3.  Discuss the role of indexing in SQLite for performance when querying very large log datasets.
4.  If the log structure changed (e.g., a new field was added), how easily could you adapt your SQLite schema and import process?
5.  What are the limitations of using SQLite for real-time log analysis compared to continuous parsing with Bash or Python scripts?

---
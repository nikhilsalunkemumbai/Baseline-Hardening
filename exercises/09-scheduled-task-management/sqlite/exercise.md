# SQLite Exercise: Scheduled Task/Job Management Analysis

## Objective

This exercise challenges you to apply your SQL querying skills using SQLite to store, manage, and analyze historical scheduled task data from multiple systems. You will create an SQLite database schema, ingest provided JSON task reports, and then perform various SQL queries to audit task configurations, identify security risks, and track changes across your infrastructure.

## Framework Alignment

This exercise on "**Scheduled Task/Job Management**" using **SQLite** provides practical experience in building and utilizing tools for a "Cross-Platform Baseline Hardening & Auditing Framework." By completing these tasks, you will learn how to audit and manage scheduled tasks, ensuring compliance with security policies and preventing unauthorized persistence mechanisms—essential steps in maintaining a secure and auditable environment.


## Scenario

You are a security auditor responsible for ensuring that scheduled tasks across your organization's diverse server fleet (Linux and Windows) comply with security policies. You receive periodic reports of scheduled tasks from various systems in JSON format. Your task is to consolidate these reports into a central SQLite database to easily query, audit, and detect suspicious or non-compliant tasks.

## Input Data

You will be working with the following JSON task report files, located in the same directory as this exercise:

*   `task_report_linux_t1.json`: Task report for `linux-server-01` at an initial time.
*   `task_report_linux_t2.json`: Task report for `linux-server-01` at a later time, showing some changes and a new task.
*   `task_report_windows_t1.json`: Task report for `windows-server-01` at an initial time.

## Setup

You will need an SQLite database. You will use a Python script to perform the setup and ingestion.

### 1. Create `ingest_task_data.py`

Create a Python script named `ingest_task_data.py` in the same directory as the exercise and JSON files. This script will create the necessary database schema and ingest the JSON data.

```python
import sqlite3
import json
import os
from datetime import datetime

DB_FILE = 'scheduled_tasks_audits.db'
REPORT_FILES = [
    'task_report_linux_t1.json',
    'task_report_linux_t2.json',
    'task_report_windows_t1.json'
]

def create_schema(conn):
    cursor = conn.cursor()

    cursor.execute('''
        CREATE TABLE IF NOT EXISTS task_snapshots (
            snapshot_id INTEGER PRIMARY KEY AUTOINCREMENT,
            timestamp TEXT NOT NULL,
            hostname TEXT NOT NULL,
            os_type TEXT NOT NULL
        )
    ''')

    cursor.execute('''
        CREATE TABLE IF NOT EXISTS scheduled_tasks (
            task_id INTEGER PRIMARY KEY AUTOINCREMENT,
            snapshot_id INTEGER NOT NULL,
            task_name TEXT NOT NULL,
            command TEXT NOT NULL,
            schedule_type TEXT,
            schedule_details TEXT,
            run_as_user TEXT NOT NULL,
            status TEXT NOT NULL,
            description TEXT,
            last_run_time TEXT,
            next_run_time TEXT,
            FOREIGN KEY (snapshot_id) REFERENCES task_snapshots (snapshot_id)
        )
    ''')
    
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_task_snapshots_hostname ON task_snapshots (hostname)')
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_scheduled_tasks_snapshot_id ON scheduled_tasks (snapshot_id)')
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_scheduled_tasks_task_name ON scheduled_tasks (task_name)')
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_scheduled_tasks_run_as_user ON scheduled_tasks (run_as_user)')
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_scheduled_tasks_status ON scheduled_tasks (status)')

    conn.commit()
    print("Database schema created/updated.")

def ingest_task_report(conn, report_data):
    cursor = conn.cursor()

    cursor.execute('''
        INSERT INTO task_snapshots (timestamp, hostname, os_type)
        VALUES (?, ?, ?)
    ''', (report_data['timestamp'], report_data['hostname'], report_data['os_type']))
    snapshot_id = cursor.lastrowid

    for task in report_data.get('tasks', []):
        cursor.execute('''
            INSERT INTO scheduled_tasks (snapshot_id, task_name, command, schedule_type, schedule_details, run_as_user, status, description, last_run_time, next_run_time)
            VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
        ''', (snapshot_id, task['task_name'], task['command'], task.get('schedule_type'),
              task.get('schedule_details'), task['run_as_user'], task['status'],
              task.get('description'), task.get('last_run_time'), task.get('next_run_time')))

    conn.commit()
    print(f"Task report for {report_data['hostname']} at {report_data['timestamp']} (ID: {snapshot_id}) ingested successfully.")

def main():
    if os.path.exists(DB_FILE):
        os.remove(DB_FILE)
    
    conn = sqlite3.connect(DB_FILE)
    create_schema(conn)

    for report_file in REPORT_FILES:
        try:
            with open(report_file, 'r') as f:
                json_data = json.load(f)
            ingest_task_report(conn, json_data)
        except FileNotFoundError:
            print(f"Error: Report file '{report_file}' not found.")
        except json.JSONDecodeError:
            print(f"Error: Malformed JSON in '{report_file}'.")
        except Exception as e:
            print(f"An error occurred while processing '{report_file}': {e}")
    conn.close()
    print(f"
All task reports ingested into {DB_FILE}.")

if __name__ == '__main__':
    main()
```

### 2. Run the Ingestion Script

Execute the Python script to create the database and load the data:
```bash
python ingest_task_data.py
```
This will create `scheduled_tasks_audits.db` and populate it.

## Tasks

Using SQL queries within the `sqlite3` prompt, provide the solution for each of the following tasks. Start by opening the database:
```bash
sqlite3 scheduled_tasks_audits.db
```
*(All following SQL commands are executed within the `sqlite3` prompt)*

Assume:
*   `snapshot_id = 1`: `linux-server-01` (first snapshot)
*   `snapshot_id = 2`: `linux-server-01` (second snapshot)
*   `snapshot_id = 3`: `windows-server-01`

### 1. List All Scheduled Tasks (Across All Snapshots and Hosts)

```sql
SELECT
    ts.hostname,
    ts.os_type,
    st.task_name,
    st.command,
    st.run_as_user,
    st.status,
    ts.timestamp AS snapshot_time
FROM
    task_snapshots ts
JOIN
    scheduled_tasks st ON ts.snapshot_id = st.snapshot_id
ORDER BY
    ts.hostname, ts.timestamp, st.task_name;
```

### 2. Identify All Disabled Tasks

```sql
SELECT
    ts.hostname,
    st.task_name,
    st.command,
    ts.timestamp AS snapshot_time
FROM
    task_snapshots ts
JOIN
    scheduled_tasks st ON ts.snapshot_id = st.snapshot_id
WHERE
    st.status = 'disabled';
```

### 3. Find Tasks Running as Privileged Users (`root` or `SYSTEM`)

```sql
SELECT
    ts.hostname,
    ts.os_type,
    st.task_name,
    st.command,
    st.run_as_user,
    ts.timestamp AS snapshot_time
FROM
    task_snapshots ts
JOIN
    scheduled_tasks st ON ts.snapshot_id = st.snapshot_id
WHERE
    st.run_as_user = 'root' OR st.run_as_user = 'SYSTEM';
```

### 4. Identify Potentially Malicious or Suspicious Tasks (containing 'evil' or 'backdoor')

```sql
SELECT
    ts.hostname,
    st.task_name,
    st.command,
    ts.timestamp AS snapshot_time
FROM
    task_snapshots ts
JOIN
    scheduled_tasks st ON ts.snapshot_id = st.snapshot_id
WHERE
    st.command LIKE '%evil%' OR st.command LIKE '%backdoor%' OR st.description LIKE '%malicious%';
```

### 5. Track Changes to the `daily_backup` Task on `linux-server-01` Over Time

```sql
SELECT
    ts.timestamp,
    st.command,
    st.schedule_details,
    st.description
FROM
    task_snapshots ts
JOIN
    scheduled_tasks st ON ts.snapshot_id = st.snapshot_id
WHERE
    ts.hostname = 'linux-server-01' AND st.task_name = 'daily_backup'
ORDER BY
    ts.timestamp ASC;
```

### 6. Count the Number of Scheduled Tasks Per Host for the Latest Snapshot of Each Host

```sql
SELECT
    ts.hostname,
    COUNT(st.task_id) AS total_tasks
FROM
    task_snapshots ts
JOIN
    scheduled_tasks st ON ts.snapshot_id = st.snapshot_id
WHERE
    ts.snapshot_id IN (
        SELECT MAX(snapshot_id) FROM task_snapshots GROUP BY hostname
    )
GROUP BY
    ts.hostname;
```

### 7. Find Tasks that have been Disabled (comparing snapshots)

*(This query requires comparing different snapshots for the same task. The following query finds tasks that were enabled in `snapshot_id = 1` and disabled in `snapshot_id = 2` on `linux-server-01`)*

```sql
SELECT
    st1.task_name,
    st1.status AS status_t1,
    st2.status AS status_t2
FROM
    scheduled_tasks st1
JOIN
    scheduled_tasks st2 ON st1.task_name = st2.task_name
WHERE
    st1.snapshot_id = 1 AND st1.status = 'enabled' AND
    st2.snapshot_id = 2 AND st2.status = 'disabled';
```

## Deliverables

For each task, provide the SQL query and the resulting output.

## Reflection Questions

1.  How does creating a centralized SQLite database for scheduled task data enhance the ability to audit and ensure compliance across a heterogeneous IT environment?
2.  Explain the importance of the `snapshot_id` and `timestamp` fields in the `task_snapshots` table for historical analysis and tracking changes.
3.  Discuss how SQL queries (e.g., using `LIKE`, `WHERE`, `JOIN`) are invaluable for identifying suspicious patterns or non-compliant configurations within scheduled tasks.
4.  If you needed to identify tasks that were deleted between two snapshots for a specific host, what kind of SQL query logic would you employ?
5.  What are the advantages and disadvantages of using SQLite for storing scheduled task audit data compared to directly querying each system or using a full-fledged Configuration Management Database (CMDB) solution?

---
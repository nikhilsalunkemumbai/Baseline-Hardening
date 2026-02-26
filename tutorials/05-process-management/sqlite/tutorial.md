# SQLite Tutorial: Process Management and Automation Analysis

## Introduction

While Bash, PowerShell, and Python are excellent for *collecting* and *managing* processes in real-time, SQLite provides an ideal solution for persistently *storing* and *querying* historical process information. By centralizing process snapshots in an SQLite database, you can perform long-term monitoring, audit process activity, detect unusual process behavior, and track resource usage trends over time with the full power of SQL. This tutorial will guide you through designing an SQLite schema for process data and querying that data effectively.

## Framework Alignment

This tutorial on "**Process Management and Automation**" using **SQLite** demonstrates the foundational skills necessary to implement components of a "Cross-Platform Baseline Hardening & Auditing Framework." The techniques learned here for storing and querying process snapshot data are directly applicable to auditing systems against defined security baselines and ensuring compliance.


## Why SQLite for Process Analysis?

*   **Historical Tracking:** Store multiple process snapshots over time to track process lifecycles, resource consumption, and unexpected process launches/terminations.
*   **Centralized Repository:** Consolidate process data from various systems (if collected remotely) into a single, queryable database.
*   **Powerful SQL Analysis:** Use SQL to filter processes by various criteria, identify resource hogs, find processes active during specific periods, and generate custom audit reports.
*   **Security Auditing:** Easily query for unauthorized processes, detect process injection attempts (by comparing against baselines), or identify malware persistence mechanisms.
*   **Portable & Serverless:** The entire database is a single file, easily transferable and usable without a dedicated database server.
*   **Minimal Dependencies:** The `sqlite3` command-line tool and Python's `sqlite3` module are standard.

## Workflow: From Collection to SQLite

The typical workflow involves using a scripting language to:
1.  **Collect Process Data:** Use Bash, PowerShell, or Python scripts (as demonstrated in previous tutorials) to gather process lists and their metadata.
2.  **Structure Data:** Format the collected process data into a structured format (e.g., JSON).
3.  **Ingest into SQLite:** Use a script (e.g., Python with `sqlite3` module) to read the structured data and insert it into an SQLite database.

For this tutorial, we will focus on the SQLite schema design and querying. We'll provide examples of how you might ingest data from a Python script that generates JSON process lists.

## Schema Design for Process Auditing

A robust schema will typically involve two main tables: one for the snapshots themselves (metadata about when and from where the snapshot was taken) and one for the individual processes within each snapshot.

### Proposed Schema

1.  **`process_snapshots` Table:** Stores metadata about each snapshot (when it was taken, from which host).
2.  **`processes` Table:** Stores details about each process observed in a snapshot, linking to the `process_snapshots` table.

### 1. Creating the Database and Tables

We'll use a Python script to create the database and tables.

```python
import sqlite3
import json
from datetime import datetime

DB_FILE = 'process_audits.db'

def create_schema(conn):
    cursor = conn.cursor()

    # process_snapshots table: stores metadata for each snapshot
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS process_snapshots (
            snapshot_id INTEGER PRIMARY KEY AUTOINCREMENT,
            timestamp TEXT NOT NULL,
            hostname TEXT NOT NULL
        )
    ''')

    # processes table: stores details for each process within a snapshot
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS processes (
            process_id INTEGER PRIMARY KEY AUTOINCREMENT,
            snapshot_id INTEGER NOT NULL,
            pid INTEGER NOT NULL,
            ppid INTEGER, -- Parent PID
            name TEXT NOT NULL,
            command TEXT,
            user TEXT,
            cpu_percent REAL,
            mem_percent REAL,
            start_time TEXT,
            FOREIGN KEY (snapshot_id) REFERENCES process_snapshots (snapshot_id)
        )
    ''')
    
    # Add indexes for faster lookups
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_processes_snapshot_id ON processes (snapshot_id)')
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_processes_pid ON processes (pid)')
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_processes_name ON processes (name)')
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_processes_user ON processes (user)')

    conn.commit()
    print("Database schema created/updated.")

# Example Usage:
# conn = sqlite3.connect(DB_FILE)
# create_schema(conn)
# conn.close()
```

### 2. Data Ingestion (Python Example)

Assuming you have JSON process list files (`snapshot_host1_t1.json`, `snapshot_host1_t2.json`) generated by the Python `process_manager.py` script.

**`snapshot_host1_t1.json` (simplified example):**
```json
{
  "timestamp": "2026-03-01T09:00:00Z",
  "hostname": "server-alpha",
  "processes": [
    {
      "user": "root",
      "pid": 1,
      "cpu_percent": 0.0,
      "mem_percent": 0.1,
      "command": "/sbin/init",
      "ppid": 0,
      "name": "init"
    },
    {
      "user": "user1",
      "pid": 1234,
      "cpu_percent": 1.5,
      "mem_percent": 2.3,
      "command": "/usr/bin/python3 myapp.py",
      "ppid": 1,
      "name": "python3"
    },
    {
      "user": "user1",
      "pid": 1235,
      "cpu_percent": 0.2,
      "mem_percent": 0.5,
      "command": "/bin/bash",
      "ppid": 1234,
      "name": "bash"
    }
  ]
}
```

**`snapshot_host1_t2.json` (myapp.py terminated, new editor process):**
```json
{
  "timestamp": "2026-03-01T09:10:00Z",
  "hostname": "server-alpha",
  "processes": [
    {
      "user": "root",
      "pid": 1,
      "cpu_percent": 0.0,
      "mem_percent": 0.1,
      "command": "/sbin/init",
      "ppid": 0,
      "name": "init"
    },
    {
      "user": "user1",
      "pid": 1236,
      "cpu_percent": 0.8,
      "mem_percent": 1.2,
      "command": "/usr/bin/vim report.txt",
      "ppid": 1,
      "name": "vim"
    },
    {
      "user": "user1",
      "pid": 1235,
      "cpu_percent": 0.2,
      "mem_percent": 0.5,
      "command": "/bin/bash",
      "ppid": 1,
      "name": "bash"
    }
  ]
}
```

```python
def ingest_process_snapshot(conn, snapshot_data):
    cursor = conn.cursor()

    # Insert into process_snapshots table
    cursor.execute('''
        INSERT INTO process_snapshots (timestamp, hostname)
        VALUES (?, ?)
    ''', (snapshot_data['timestamp'], snapshot_data['hostname']))
    snapshot_id = cursor.lastrowid

    # Iterate through processes
    for proc in snapshot_data['processes']:
        cursor.execute('''
            INSERT INTO processes (snapshot_id, pid, ppid, name, command, user, cpu_percent, mem_percent, start_time)
            VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?)
        ''', (snapshot_id, proc.get('pid'), proc.get('ppid'), proc.get('name'),
              proc.get('command'), proc.get('user'), proc.get('cpu_percent'),
              proc.get('mem_percent'), proc.get('start_time'))) # start_time might need to be parsed from original ps output

    conn.commit()
    print(f"Process snapshot for {snapshot_data['hostname']} at {snapshot_data['timestamp']} (ID: {snapshot_id}) ingested successfully.")

# Example Main function for ingestion:
# if __name__ == '__main__':
#     DB_FILE = 'process_audits.db'
#     if os.path.exists(DB_FILE):
#         os.remove(DB_FILE)
#     conn = sqlite3.connect(DB_FILE)
#     create_schema(conn)

#     # Ingest snapshot_host1_t1.json
#     with open('snapshot_host1_t1.json', 'r') as f:
#         data1 = json.load(f)
#     ingest_process_snapshot(conn, data1)

#     # Ingest snapshot_host1_t2.json
#     with open('snapshot_host1_t2.json', 'r') as f:
#         data2 = json.load(f)
#     ingest_process_snapshot(conn, data2)
    
#     conn.close()
```

## Basic SQL Queries for Process Analysis

Once data is in SQLite, you can perform powerful analyses using the `sqlite3` command-line tool.

```bash
sqlite3 process_audits.db
```
*(All following SQL commands are executed within the `sqlite3` prompt)*

Assume `snapshot_id = 1` is for `snapshot_host1_t1.json` and `snapshot_id = 2` is for `snapshot_host1_t2.json`.

### 1. List All Snapshots

```sql
SELECT snapshot_id, timestamp, hostname FROM process_snapshots;
```

### 2. List All Processes from a Specific Snapshot (e.g., `snapshot_id = 1`)

```sql
SELECT pid, name, command, user, cpu_percent, mem_percent
FROM processes
WHERE snapshot_id = 1
ORDER BY cpu_percent DESC;
```

### 3. Find Processes by Name Across All Snapshots

```sql
SELECT ps.timestamp, ps.hostname, p.pid, p.name, p.command, p.user
FROM process_snapshots ps
JOIN processes p ON ps.snapshot_id = p.snapshot_id
WHERE p.name LIKE '%python3%'
ORDER BY ps.timestamp;
```

### 4. Identify Top 5 CPU Consumers in the Latest Snapshot (assume `snapshot_id = 2` is latest)

```sql
SELECT p.pid, p.name, p.command, p.cpu_percent
FROM processes p
WHERE p.snapshot_id = 2
ORDER BY p.cpu_percent DESC
LIMIT 5;
```

### 5. Detect Newly Launched Processes Between Two Snapshots

```sql
-- Processes in snapshot 2 that were not in snapshot 1 (based on name and command, ignoring PID changes)
SELECT
    p2.name,
    p2.command,
    p2.user,
    p2.pid
FROM
    processes p2
WHERE
    p2.snapshot_id = 2
    AND NOT EXISTS (
        SELECT 1
        FROM processes p1
        WHERE p1.snapshot_id = 1
          AND p1.name = p2.name
          AND p1.command = p2.command
          -- AND p1.user = p2.user -- Optional: be more strict about user
    );
```

### 6. Detect Terminated Processes Between Two Snapshots

```sql
-- Processes in snapshot 1 that are no longer in snapshot 2 (based on name and command, ignoring PID changes)
SELECT
    p1.name,
    p1.command,
    p1.user,
    p1.pid
FROM
    processes p1
WHERE
    p1.snapshot_id = 1
    AND NOT EXISTS (
        SELECT 1
        FROM processes p2
        WHERE p2.snapshot_id = 2
          AND p2.name = p1.name
          AND p2.command = p1.command
          -- AND p2.user = p1.user -- Optional: be more strict about user
    );
```

### 7. Track Resource Usage Trend for a Specific Process

```sql
SELECT
    ps.timestamp,
    p.cpu_percent,
    p.mem_percent
FROM
    process_snapshots ps
JOIN
    processes p ON ps.snapshot_id = p.snapshot_id
WHERE
    ps.hostname = 'server-alpha' AND p.name = 'python3' -- or p.command LIKE '%myapp.py%'
ORDER BY
    ps.timestamp ASC;
```

## Guiding Principles with SQLite

*   **Portability:** The SQLite database file (`.db`) is fully portable. The SQL queries are standard.
*   **Efficiency:** SQLite is highly optimized for read operations. Proper indexing ensures fast querying even with many snapshots and processes.
*   **Minimal Dependencies:** Python's built-in `sqlite3` module and the `sqlite3` CLI tool are standard components.
*   **CLI-centric:** The `sqlite3` command-line tool provides a robust interface for interacting with the database.
*   **Structured Data & Actionable Output:** Storing process information in a relational database transforms ephemeral system state into structured, queryable data, enabling powerful historical analysis, anomaly detection, and security auditing.

## Conclusion

SQLite provides a robust and indispensable component for managing and analyzing historical process data. It moves beyond real-time snapshots to long-term trend analysis, allowing you to track process lifecycles, identify resource hogs, and detect suspicious activity that might indicate a security compromise. By integrating SQLite with your chosen scripting language for data collection and ingestion, you create a comprehensive and flexible process monitoring and auditing framework. The next step is to apply this knowledge in practical exercises.
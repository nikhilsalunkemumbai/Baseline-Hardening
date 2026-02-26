# SQLite Exercise: Process Management and Automation Analysis

## Objective

This exercise challenges you to apply your SQL querying skills using SQLite to store, manage, and analyze historical process snapshots. You will create an SQLite database schema, ingest provided JSON process snapshot data, and then perform various SQL queries to identify changes in running processes, detect resource hogs, and audit process activity over time.

## Framework Alignment

This exercise on "**Process Management and Automation**" using **SQLite** provides practical experience in building and utilizing tools for a "Cross-Platform Baseline Hardening & Auditing Framework." By completing these tasks, you will learn how to audit and manage running processes, ensuring that only authorized services are active and identifying potential security risks or unauthorized activity—essential steps in maintaining a secure and auditable environment.


## Scenario

You are a system administrator or security analyst responsible for monitoring process activity on critical servers. You have implemented a system that periodically captures snapshots of running processes in JSON format. Your task is to consolidate these snapshots into an SQLite database and use SQL to report on process behavior, detect suspicious activity, and track resource usage trends.

## Input Data

You will be working with the following JSON snapshot files, located in the same directory as this exercise:

*   `proc_snapshot_t1.json`: Represents a process snapshot from `server-alpha` at an initial time.
*   `proc_snapshot_t2.json`: Represents a process snapshot from `server-alpha` at a later time, after some simulated process changes.

## Setup

You will need an SQLite database. You will use a Python script to perform the setup and ingestion.

### 1. Create `ingest_process_snapshots.py`

Create a Python script named `ingest_process_snapshots.py` in the same directory as the exercise and JSON files. This script will create the necessary database schema and ingest the JSON data.

```python
import sqlite3
import json
import os
from datetime import datetime

DB_FILE = 'process_audits.db'
SNAPSHOT_FILES = [
    'proc_snapshot_t1.json',
    'proc_snapshot_t2.json'
]

def create_schema(conn):
    cursor = conn.cursor()

    cursor.execute('''
        CREATE TABLE IF NOT EXISTS process_snapshots (
            snapshot_id INTEGER PRIMARY KEY AUTOINCREMENT,
            timestamp TEXT NOT NULL,
            hostname TEXT NOT NULL
        )
    ''')

    cursor.execute('''
        CREATE TABLE IF NOT EXISTS processes (
            process_id INTEGER PRIMARY KEY AUTOINCREMENT,
            snapshot_id INTEGER NOT NULL,
            pid INTEGER NOT NULL,
            ppid INTEGER,
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
              proc.get('mem_percent'), proc.get('start_time')))

    conn.commit()
    print(f"Process snapshot for {snapshot_data['hostname']} at {snapshot_data['timestamp']} (ID: {snapshot_id}) ingested successfully.")

def main():
    if os.path.exists(DB_FILE):
        os.remove(DB_FILE) # Clean up previous DB for fresh run
    
    conn = sqlite3.connect(DB_FILE)
    create_schema(conn)

    for snapshot_file in SNAPSHOT_FILES:
        try:
            with open(snapshot_file, 'r') as f:
                json_data = json.load(f)
            ingest_process_snapshot(conn, json_data)
        except FileNotFoundError:
            print(f"Error: Snapshot file '{snapshot_file}' not found.")
        except json.JSONDecodeError:
            print(f"Error: Malformed JSON in '{snapshot_file}'.")
        except Exception as e:
            print(f"An error occurred while processing '{snapshot_file}': {e}")
    conn.close()
    print(f"
All process snapshots ingested into {DB_FILE}.")

if __name__ == '__main__':
    main()
```
### 2. Run the Ingestion Script

Execute the Python script to create the database and load the data:
```bash
python ingest_process_snapshots.py
```
This will create `process_audits.db` and populate it.

## Tasks

Using SQL queries within the `sqlite3` prompt, provide the solution for each of the following tasks. Start by opening the database:
```bash
sqlite3 process_audits.db
```
*(All following SQL commands are executed within the `sqlite3` prompt)*

Assume `snapshot_id = 1` refers to `proc_snapshot_t1.json` and `snapshot_id = 2` refers to `proc_snapshot_t2.json`.

1.  **List All Process Snapshots:**
    *   Retrieve the `snapshot_id`, `timestamp`, and `hostname` for all recorded process snapshots.

2.  **List All Processes from a Specific Snapshot (e.g., `snapshot_id = 1`):**
    *   Retrieve the `pid`, `name`, `command`, `user`, `cpu_percent`, and `mem_percent` for all processes in `snapshot_id = 1`, ordered by `cpu_percent` descending.

3.  **Find Processes by Name Across All Snapshots:**
    *   List the `timestamp`, `hostname`, `pid`, `name`, and `command` for all processes whose `name` is 'tail' across all snapshots.

4.  **Identify Top 3 CPU Consumers in the Latest Snapshot (`snapshot_id = 2`):**
    *   Find the top 3 processes with the highest `cpu_percent` in `snapshot_id = 2`. Display their `pid`, `name`, `command`, and `cpu_percent`.

5.  **Detect Newly Launched Processes:**
    *   Identify processes that appear in `snapshot_id = 2` but were NOT present in `snapshot_id = 1` (based on `name` and `command`). Display their `name`, `command`, and `user`.

6.  **Detect Terminated Processes:**
    *   Identify processes that were present in `snapshot_id = 1` but are NO longer present in `snapshot_id = 2` (based on `name` and `command`). Display their `name`, `command`, and `user`.

7.  **Track Resource Usage Trend for a Specific Process:**
    *   For the 'tail' process (`name = 'tail'`), show its `timestamp`, `cpu_percent`, and `mem_percent` across all snapshots, ordered chronologically.

## Deliverables

For each task, provide the SQL query and the resulting output.

## Reflection Questions

1.  How does storing process snapshots in a relational database (SQLite) enable the detection of newly launched or terminated processes over time, which would be difficult with isolated text files?
2.  Discuss the advantages of normalizing the schema (using separate tables for snapshots and processes) for managing process audit data.
3.  How would you extend this schema to include more detailed process information, such as parent PID, open files, or network connections, if that data were available in your JSON snapshots?
4.  If you needed to create an alert system for suspicious processes, how could the SQL queries be used by an external script to identify, for example, processes with high CPU usage for an extended period or unauthorized executables?
5.  What are the advantages and disadvantages of using SQLite for historical process analysis compared to a real-time process monitoring tool or a simple text-based logging approach?

---
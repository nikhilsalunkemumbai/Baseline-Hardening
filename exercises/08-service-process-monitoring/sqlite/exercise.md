# SQLite Exercise: Service/Process Monitoring and Health Check Analysis

## Objective

This exercise challenges you to apply your SQL querying skills using SQLite to store, manage, and analyze historical monitoring data for services and processes. You will create an SQLite database schema, ingest provided JSON monitoring reports, and then perform various SQL queries to identify issues, track performance trends, and analyze service availability over time.

## Framework Alignment

This exercise on "**Service/Process Monitoring and Health Check**" using **SQLite** provides practical experience in building and utilizing tools for a "Cross-Platform Baseline Hardening & Auditing Framework." By completing these tasks, you will learn how to audit and manage service and process health, ensuring that critical security and operational services are running as expected—essential steps in maintaining a secure and auditable environment.


## Scenario

You are a system administrator tasked with monitoring several critical servers. Periodic monitoring scripts generate health reports in JSON format. Your goal is to consolidate these reports into a central SQLite database to facilitate long-term analysis, identify performance bottlenecks, and pinpoint service outages across your infrastructure.

## Input Data

You will be working with the following JSON monitoring report files, located in the same directory as this exercise:

*   `monitor_report_server1_t1.json`: Monitoring report for `web-server-01` at an initial time.
*   `monitor_report_server1_t2.json`: Monitoring report for `web-server-01` at a later time, showing a service issue.
*   `monitor_report_server2_t1.json`: Monitoring report for `db-server-01` at an initial time.

## Setup

You will need an SQLite database. You will use a Python script to perform the setup and ingestion.

### 1. Create `ingest_monitoring_data.py`

Create a Python script named `ingest_monitoring_data.py` in the same directory as the exercise and JSON files. This script will create the necessary database schema and ingest the JSON data.

```python
import sqlite3
import json
import os
from datetime import datetime

DB_FILE = 'monitoring_data.db'
REPORT_FILES = [
    'monitor_report_server1_t1.json',
    'monitor_report_server1_t2.json',
    'monitor_report_server2_t1.json'
]

def create_schema(conn):
    cursor = conn.cursor()

    cursor.execute('''
        CREATE TABLE IF NOT EXISTS monitoring_snapshots (
            snapshot_id INTEGER PRIMARY KEY AUTOINCREMENT,
            timestamp TEXT NOT NULL,
            hostname TEXT NOT NULL
        )
    ''')

    cursor.execute('''
        CREATE TABLE IF NOT EXISTS service_process_status (
            item_id INTEGER PRIMARY KEY AUTOINCREMENT,
            snapshot_id INTEGER NOT NULL,
            name TEXT NOT NULL,
            type TEXT NOT NULL, -- 'process' or 'service'
            status TEXT NOT NULL, -- 'running', 'stopped', 'failed', etc.
            pid INTEGER,
            cpu_percent REAL,
            memory_mb REAL,
            owner TEXT,
            uptime_seconds INTEGER,
            FOREIGN KEY (snapshot_id) REFERENCES monitoring_snapshots (snapshot_id)
        )
    ''')
    
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS health_check_results (
            check_id INTEGER PRIMARY KEY AUTOINCREMENT,
            snapshot_id INTEGER NOT NULL,
            check_type TEXT NOT NULL, -- 'port', 'http_url', etc.
            target TEXT NOT NULL,
            status TEXT NOT NULL, -- 'PASS', 'FAIL', 'ERROR'
            message TEXT,
            latency_ms INTEGER,
            http_status_code INTEGER,
            FOREIGN KEY (snapshot_id) REFERENCES monitoring_snapshots (snapshot_id)
        )
    ''')
    
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_monitoring_snapshots_hostname ON monitoring_snapshots (hostname)')
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_sps_snapshot_id ON service_process_status (snapshot_id)')
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_sps_name_type ON service_process_status (name, type)')
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_hcr_snapshot_id ON health_check_results (snapshot_id)')
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_hcr_target_type ON health_check_results (target, check_type)')
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_hcr_status ON health_check_results (status)')

    conn.commit()
    print("Database schema created/updated.")

def ingest_monitoring_report(conn, report_data):
    cursor = conn.cursor()

    cursor.execute('''
        INSERT INTO monitoring_snapshots (timestamp, hostname)
        VALUES (?, ?)
    ''', (report_data['timestamp'], report_data['hostname']))
    snapshot_id = cursor.lastrowid

    for item in report_data.get('service_process_status', []):
        cursor.execute('''
            INSERT INTO service_process_status (snapshot_id, name, type, status, pid, cpu_percent, memory_mb, owner, uptime_seconds)
            VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?)
        ''', (snapshot_id, item['name'], item['type'], item['status'],
              item.get('pid'), item.get('cpu_percent'), item.get('memory_mb'),
              item.get('owner'), item.get('uptime_seconds')))

    for check in report_data.get('health_check_results', []):
        cursor.execute('''
            INSERT INTO health_check_results (snapshot_id, check_type, target, status, message, latency_ms, http_status_code)
            VALUES (?, ?, ?, ?, ?, ?, ?)
        ''', (snapshot_id, check['check_type'], check['target'], check['status'],
              check.get('message'), check.get('latency_ms'), check.get('http_status_code')))

    conn.commit()
    print(f"Monitoring report for {report_data['hostname']} at {report_data['timestamp']} (ID: {snapshot_id}) ingested successfully.")

def main():
    if os.path.exists(DB_FILE):
        os.remove(DB_FILE)
    
    conn = sqlite3.connect(DB_FILE)
    create_schema(conn)

    for report_file in REPORT_FILES:
        try:
            with open(report_file, 'r') as f:
                json_data = json.load(f)
            ingest_monitoring_report(conn, json_data)
        except FileNotFoundError:
            print(f"Error: Report file '{report_file}' not found.")
        except json.JSONDecodeError:
            print(f"Error: Malformed JSON in '{report_file}'.")
        except Exception as e:
            print(f"An error occurred while processing '{report_file}': {e}")
    conn.close()
    print(f"
All monitoring reports ingested into {DB_FILE}.")

if __name__ == '__main__':
    main()
```

### 2. Run the Ingestion Script

Execute the Python script to create the database and load the data:
```bash
python ingest_monitoring_data.py
```
This will create `monitoring_data.db` and populate it.

## Tasks

Using SQL queries within the `sqlite3` prompt, provide the solution for each of the following tasks. Start by opening the database:
```bash
sqlite3 monitoring_data.db
```
*(All following SQL commands are executed within the `sqlite3` prompt)*

Assume:
*   `snapshot_id = 1`: `monitor_report_server1_t1.json`
*   `snapshot_id = 2`: `monitor_report_server1_t2.json`
*   `snapshot_id = 3`: `monitor_report_server2_t1.json`

1.  **List All Monitoring Snapshots:**
    *   Retrieve the `snapshot_id`, `timestamp`, and `hostname` for all recorded snapshots.

2.  **Get the Latest Status of `nginx` on `web-server-01`:**
    *   Find the `timestamp`, `status`, `cpu_percent`, and `memory_mb` of the `nginx` service (type 'service') for `web-server-01` from its most recent snapshot.

3.  **Identify All Health Check Failures:**
    *   List the `hostname`, `timestamp`, `check_type`, `target`, `status`, and `message` for all health checks that resulted in a 'FAIL' or 'ERROR'.

4.  **Track CPU Usage of `php-fpm` on `web-server-01` Over Time:**
    *   Show the `timestamp` and `cpu_percent` for the `php-fpm` process (type 'process') on `web-server-01` across all snapshots, ordered chronologically.

5.  **Identify Hosts Where `nginx` was `stopped`:**
    *   List the `hostname` and `timestamp` for any snapshot where the `nginx` service (type 'service') had a `status` of 'stopped'.

6.  **Calculate Average HTTP Health Check Latency:**
    *   Calculate the average `latency_ms` for all `http_url` checks.

7.  **Identify Processes with High Memory Usage:**
    *   List `hostname`, `timestamp`, `process_name`, and `memory_mb` for any process (type 'process') that consumed more than `200 MB` of memory in any snapshot. Order by `memory_mb` descending.

## Deliverables

For each task, provide the SQL query and the resulting output.

## Reflection Questions

1.  How does storing monitoring data in a relational database (SQLite) facilitate long-term trend analysis and identifying intermittent issues compared to relying on ephemeral logs or real-time dashboards?
2.  Discuss the advantages of normalizing monitoring data across three tables (`monitoring_snapshots`, `service_process_status`, `health_check_results`).
3.  If you wanted to calculate the approximate uptime percentage for a service (e.g., `nginx`) on a host (`web-server-01`) over a given period, what kind of SQL queries would be involved? What additional data points might be useful for more accurate calculations?
4.  How could this SQLite database be integrated into an alerting system to notify administrators of persistent service failures or unusual resource consumption patterns?
5.  What are the trade-offs of using SQLite for storing monitoring data compared to a more powerful time-series database (like InfluxDB) or a full-fledged relational database (like PostgreSQL) for very large-scale or high-frequency monitoring?

---
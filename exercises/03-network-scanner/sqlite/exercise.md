# SQLite Exercise: Network Connectivity and Port Scan Analysis

## Objective

This exercise challenges you to apply your SQL querying skills using SQLite to store, manage, and analyze historical network scan results. You will create an SQLite database schema, ingest provided JSON scan data, and then perform various SQL queries to identify open ports, track changes, and audit network configurations over time.

## Framework Alignment

This exercise on "**Network Connectivity and Port Scanner**" using **SQLite** provides practical experience in building and utilizing tools for a "Cross-Platform Baseline Hardening & Auditing Framework." By completing these tasks, you will learn how to audit and manage network connectivity and port accessibility, ensuring that only authorized services are exposed—an essential step in maintaining a secure and auditable environment.


## Scenario

You are responsible for monitoring the network security posture of your organization. You regularly run basic network scans to check host availability and identify open services. To effectively track changes and analyze trends, you need a centralized, queryable repository for these scan results. You have been provided with several network scan outputs in JSON format, collected from different hosts at different times.

## Input Data

You will be working with the following JSON snapshot files, located in the same directory as this exercise:

*   `scan_host1_day1.json`: Scan of `192.168.1.100` on Feb 25, 2026.
*   `scan_host1_day2.json`: Scan of `192.168.1.100` on Feb 26, 2026.
*   `scan_host2_day1.json`: Scan of `192.168.1.101` on Feb 25, 2026.

## Setup

You will need an SQLite database. You can either use the `sqlite3` command-line tool or a Python script to perform the setup and ingestion. For this exercise, we will provide a Python ingestion script to simplify data loading.

### 1. Create `ingest_scans.py`

Create a Python script named `ingest_scans.py` in the same directory as the exercise and JSON files. This script will create the necessary database schema and ingest the JSON data.

```python
import sqlite3
import json
import os
from datetime import datetime

DB_FILE = 'network_audits.db'
SCAN_FILES = [
    'scan_host1_day1.json',
    'scan_host1_day2.json',
    'scan_host2_day1.json'
]

def create_schema(conn):
    cursor = conn.cursor()

    cursor.execute('''
        CREATE TABLE IF NOT EXISTS scans (
            scan_id INTEGER PRIMARY KEY AUTOINCREMENT,
            timestamp TEXT NOT NULL,
            scan_duration_seconds REAL,
            targets_scanned TEXT,
            ports_scanned TEXT
        )
    ''')

    cursor.execute('''
        CREATE TABLE IF NOT EXISTS scan_hosts (
            host_result_id INTEGER PRIMARY KEY AUTOINCREMENT,
            scan_id INTEGER NOT NULL,
            host TEXT NOT NULL,
            host_status TEXT NOT NULL,
            FOREIGN KEY (scan_id) REFERENCES scans (scan_id)
        )
    ''')

    cursor.execute('''
        CREATE TABLE IF NOT EXISTS scan_ports (
            port_result_id INTEGER PRIMARY KEY AUTOINCREMENT,
            host_result_id INTEGER NOT NULL,
            port INTEGER NOT NULL,
            status TEXT NOT NULL,
            FOREIGN KEY (host_result_id) REFERENCES scan_hosts (host_result_id)
        )
    ''')
    
    # Add indexes for faster lookups
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_scan_hosts_scan_id ON scan_hosts (scan_id)')
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_scan_hosts_host ON scan_hosts (host)')
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_scan_ports_host_result_id ON scan_ports (host_result_id)')
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_scan_ports_port ON scan_ports (port)')
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_scan_ports_status ON scan_ports (status)')

    conn.commit()
    print("Database schema created/updated.")

def ingest_scan_results(conn, scan_data):
    cursor = conn.cursor()

    # Insert into scans table
    cursor.execute('''
        INSERT INTO scans (timestamp, scan_duration_seconds, targets_scanned, ports_scanned)
        VALUES (?, ?, ?, ?)
    ''', (scan_data['timestamp'], scan_data.get('scan_duration_seconds'),
          scan_data.get('targets_scanned'), scan_data.get('ports_scanned')))
    scan_id = cursor.lastrowid

    # Iterate through host results
    for host_result in scan_data['results']:
        cursor.execute('''
            INSERT INTO scan_hosts (scan_id, host, host_status)
            VALUES (?, ?, ?)
        ''', (scan_id, host_result['host'], host_result['host_status']))
        host_result_id = cursor.lastrowid

        # Iterate through port results for each host
        for port_data in host_result['ports']:
            cursor.execute('''
                INSERT INTO scan_ports (host_result_id, port, status)
                VALUES (?, ?, ?)
            ''', (host_result_id, port_data['port'], port_data['status']))

    conn.commit()
    print(f"Scan results from {scan_data['timestamp']} ingested.")

def main():
    if os.path.exists(DB_FILE):
        os.remove(DB_FILE) # Clean up previous DB for fresh run
    
    conn = sqlite3.connect(DB_FILE)
    create_schema(conn)

    for scan_file in SCAN_FILES:
        try:
            with open(scan_file, 'r') as f:
                json_data = json.load(f)
            ingest_scan_results(conn, json_data)
        except FileNotFoundError:
            print(f"Error: Scan file '{scan_file}' not found.")
        except json.JSONDecodeError:
            print(f"Error: Malformed JSON in '{scan_file}'.")
        except Exception as e:
            print(f"An error occurred while processing '{scan_file}': {e}")
    conn.close()
    print(f"
All scan files ingested into {DB_FILE}.")

if __name__ == '__main__':
    main()
```
### 2. Run the Ingestion Script

Execute the Python script to create the database and load the data:
```bash
python ingest_scans.py
```
This will create `network_audits.db` and populate it.

## Tasks

Using SQL queries within the `sqlite3` prompt, provide the solution for each of the following tasks. Start by opening the database:
```bash
sqlite3 network_audits.db
```
*(All following SQL commands are executed within the `sqlite3` prompt)*

1.  **List All Scans:**
    *   Retrieve the `scan_id`, `timestamp`, and `targets_scanned` for all recorded scans.

2.  **Find All Hosts with Open Port 80 (HTTP):**
    *   List all unique `host` IP addresses that had `port` 80 reported as `Open` in any scan.

3.  **Count Open Ports Per Host (for the latest scan of a host):**
    *   For each host (`192.168.1.100` and `192.168.1.101`), find its most recent `scan_id`. Then, for each host, count the number of ports reported as `Open` in their latest scan.

4.  **Track Port 22 (SSH) Status for `192.168.1.100` Over Time:**
    *   Show the `timestamp` and `status` of `port` 22 for host `192.168.1.100` across all scans, ordered chronologically.

5.  **Identify Hosts with No Open Ports:**
    *   List all `host` IP addresses that had no `Open` ports in their latest scan.

6.  **Find Hosts That Were Down:**
    *   List all unique `host` IP addresses that had a `host_status` of 'Down' in any scan.

## Deliverables

For each task, provide the SQL query and the resulting output.

## Reflection Questions

1.  How does storing network scan results in a relational database (SQLite) enable more powerful analysis and trend tracking compared to parsing individual text or JSON files?
2.  Discuss the advantages of normalizing the schema (using separate tables for scans, hosts, and ports) for managing network scan data.
3.  If you wanted to track changes in services (e.g., if a port that was "Open" became "Closed" or "Filtered"), what kind of SQL queries would you use?
4.  How could this SQLite database be used in an automated system to trigger alerts when a new, unexpected open port is detected on a critical server?
5.  What are the advantages and disadvantages of using SQLite for network scan analysis compared to a full-fledged enterprise database or a simple file-based approach?

---
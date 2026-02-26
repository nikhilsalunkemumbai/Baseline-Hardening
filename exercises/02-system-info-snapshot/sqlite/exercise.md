# SQLite Exercise: System Information Snapshot Analysis

## Objective

This exercise challenges you to apply your SQL querying skills using SQLite to store, manage, and analyze historical system information snapshots. You will create an SQLite database schema, ingest provided JSON snapshot data, and then perform various SQL queries to extract insights, track changes, and compare system configurations over time.

## Framework Alignment

This exercise on "**System Information Snapshot and Reporting**" using **SQLite** provides practical experience in building and utilizing tools for a "Cross-Platform Baseline Hardening & Auditing Framework." By completing these tasks, you will learn how to audit and manage system configuration data, ensuring that system baselines are maintained and identifying unauthorized changes—essential steps in maintaining a secure and auditable environment.


## Scenario

As a system auditor or security analyst, you regularly collect system information snapshots from various servers. You need a robust method to store this data and query it efficiently to identify configuration drifts, track resource utilization trends, and answer questions about your infrastructure's historical state. You have been provided with several system snapshots in JSON format, collected from different hosts and at different times.

## Input Data

You will be working with the following JSON snapshot files, located in the same directory as this exercise:

*   `snapshot_host1_day1.json`: Snapshot from `server-alpha` on Feb 25, 2026.
*   `snapshot_host1_day2.json`: Snapshot from `server-alpha` on Feb 26, 2026.
*   `snapshot_host2_day1.json`: Snapshot from `server-beta` on Feb 25, 2026.

## Setup

You will need an SQLite database. You can either use the `sqlite3` command-line tool or a Python script to perform the setup and ingestion. For this exercise, we will provide the Python ingestion script to simplify data loading.

### 1. Create `ingest_snapshots.py`

Create a Python script named `ingest_snapshots.py` in the same directory as the exercise and JSON files. This script will create the necessary database schema and ingest the JSON data.

```python
import sqlite3
import json
import os
from datetime import datetime

DB_FILE = 'system_audits.db'
SNAPSHOT_FILES = [
    'snapshot_host1_day1.json',
    'snapshot_host1_day2.json',
    'snapshot_host2_day1.json'
]

def create_schema(conn):
    cursor = conn.cursor()

    cursor.execute('''
        CREATE TABLE IF NOT EXISTS snapshots (
            snapshot_id INTEGER PRIMARY KEY AUTOINCREMENT,
            timestamp TEXT NOT NULL,
            hostname TEXT NOT NULL
        )
    ''')

    cursor.execute('''
        CREATE TABLE IF NOT EXISTS os_info (
            os_info_id INTEGER PRIMARY KEY AUTOINCREMENT,
            snapshot_id INTEGER,
            system TEXT,
            node_name TEXT,
            release TEXT,
            version TEXT,
            architecture TEXT,
            FOREIGN KEY (snapshot_id) REFERENCES snapshots (snapshot_id)
        )
    ''')

    cursor.execute('''
        CREATE TABLE IF NOT EXISTS cpu_info (
            cpu_info_id INTEGER PRIMARY KEY AUTOINCREMENT,
            snapshot_id INTEGER,
            model_name TEXT,
            cpus_total INTEGER,
            FOREIGN KEY (snapshot_id) REFERENCES snapshots (snapshot_id)
        )
    ''')

    cursor.execute('''
        CREATE TABLE IF NOT EXISTS memory_info (
            memory_info_id INTEGER PRIMARY KEY AUTOINCREMENT,
            snapshot_id INTEGER,
            total_memory_gb REAL,
            free_memory_gb REAL,
            FOREIGN KEY (snapshot_id) REFERENCES snapshots (snapshot_id)
        )
    ''')

    cursor.execute('''
        CREATE TABLE IF NOT EXISTS disks (
            disk_id INTEGER PRIMARY KEY AUTOINCREMENT,
            snapshot_id INTEGER,
            mount_point TEXT,
            size_gb REAL,
            free_space_gb REAL,
            FOREIGN KEY (snapshot_id) REFERENCES snapshots (snapshot_id)
        )
    ''')
    
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS network_interfaces (
            interface_id INTEGER PRIMARY KEY AUTOINCREMENT,
            snapshot_id INTEGER,
            name TEXT,
            mac_address TEXT,
            ipv4_addresses TEXT, -- Stored as JSON string
            FOREIGN KEY (snapshot_id) REFERENCES snapshots (snapshot_id)
        )
    ''')

    conn.commit()
    print("Database schema created/updated.")

def ingest_snapshot(conn, json_data):
    cursor = conn.cursor()

    snapshot_ts = json_data['timestamp']
    hostname = json_data['system_info']['os']['node_name']

    # Insert into snapshots table and get snapshot_id
    cursor.execute('INSERT INTO snapshots (timestamp, hostname) VALUES (?, ?)',
                   (snapshot_ts, hostname))
    snapshot_id = cursor.lastrowid

    # Insert into os_info
    os_data = json_data['system_info']['os']
    cursor.execute('''
        INSERT INTO os_info (snapshot_id, system, node_name, release, version, architecture)
        VALUES (?, ?, ?, ?, ?, ?)
    ''', (snapshot_id, os_data.get('system'), os_data.get('node_name'),
          os_data.get('release'), os_data.get('version'), os_data.get('architecture')))

    # Insert into cpu_info
    cpu_data = json_data['system_info']['cpu']
    cursor.execute('''
        INSERT INTO cpu_info (snapshot_id, model_name, cpus_total)
        VALUES (?, ?, ?)
    ''', (snapshot_id, cpu_data.get('model_name'), cpu_data.get('cpus_total')))
          
    # Insert into memory_info
    mem_data = json_data['system_info']['memory']
    cursor.execute('''
        INSERT INTO memory_info (snapshot_id, total_memory_gb, free_memory_gb)
        VALUES (?, ?, ?)
    ''', (snapshot_id, float(mem_data.get('total_memory_gb').split()[0]), float(mem_data.get('free_memory_gb').split()[0]))) # Convert "X.YY GB" to float

    # Insert into disks
    for disk in json_data['system_info']['disks']:
        cursor.execute('''
            INSERT INTO disks (snapshot_id, mount_point, size_gb, free_space_gb)
            VALUES (?, ?, ?, ?)
        ''', (snapshot_id, disk.get('mount_point'), float(disk.get('size_gb').split()[0]), float(disk.get('free_space_gb').split()[0])))
              
    # Insert into network_interfaces
    for net_if in json_data['system_info']['network_interfaces']:
        cursor.execute('''
            INSERT INTO network_interfaces (snapshot_id, name, mac_address, ipv4_addresses)
            VALUES (?, ?, ?, ?)
        ''', (snapshot_id, net_if.get('name'), net_if.get('mac_address'), json.dumps(net_if.get('ipv4_addresses', []))))

    conn.commit()
    print(f"Snapshot for {hostname} at {snapshot_ts} ingested.")

def main():
    if os.path.exists(DB_FILE):
        os.remove(DB_FILE) # Clean up previous DB for fresh run
    
    conn = sqlite3.connect(DB_FILE)
    create_schema(conn)

    for snapshot_file in SNAPSHOT_FILES:
        try:
            with open(snapshot_file, 'r') as f:
                json_data = json.load(f)
            ingest_snapshot(conn, json_data)
        except FileNotFoundError:
            print(f"Error: Snapshot file '{snapshot_file}' not found.")
        except json.JSONDecodeError:
            print(f"Error: Malformed JSON in '{snapshot_file}'.")
        except Exception as e:
            print(f"An error occurred while processing '{snapshot_file}': {e}")
    conn.close()
    print(f"
All snapshots ingested into {DB_FILE}.")

if __name__ == '__main__':
    main()
```
### 2. Run the Ingestion Script

Execute the Python script to create the database and load the data:
```bash
python ingest_snapshots.py
```
This will create `system_audits.db` and populate it.

## Tasks

Using SQL queries within the `sqlite3` prompt, provide the solution for each of the following tasks. Start by opening the database:
```bash
sqlite3 system_audits.db
```
*(All following SQL commands are executed within the `sqlite3` prompt)*

1.  **List All Snapshots:**
    *   Retrieve the `snapshot_id`, `timestamp`, and `hostname` for all recorded snapshots.

2.  **OS Details for a Specific Host:**
    *   Display the OS `system`, `version`, and `architecture` for snapshots taken from `server-alpha`.

3.  **CPU Core Count by Hostname:**
    *   For each unique `hostname`, show its CPU `model_name` and `cpus_total`.

4.  **Memory Usage Trend for a Host:**
    *   For `server-alpha`, show the `timestamp`, `total_memory_gb`, and `free_memory_gb` for each snapshot, ordered chronologically.

5.  **Disk Free Space Alert:**
    *   Identify any disk partitions (mount points) from any host where `free_space_gb` is less than 50 GB. Display the `hostname`, `timestamp`, `mount_point`, and `free_space_gb`.

6.  **Count Systems by OS Version:**
    *   Count the number of distinct `hostname`s associated with each unique OS `version`.

7.  **Identify Network Interfaces with Specific MAC Address:**
    *   Find the `hostname` and `name` of network interfaces that have the MAC address `00:11:22:33:44:55`. (Hint: `ipv4_addresses` and `ipv6_addresses` are stored as JSON strings. You might need to use SQLite's JSON functions if available, or simpler string matching if not.)

## Deliverables

For each task, provide the SQL query and the resulting output.

## Reflection Questions

1.  How does storing system information in a relational database (SQLite) facilitate complex queries and historical analysis compared to using individual JSON or text files?
2.  Discuss the trade-offs between a normalized schema (multiple tables, foreign keys) versus a de-normalized schema (e.g., all info in one large table) for this type of data.
3.  How would you extend this schema to include information about installed software or running services?
4.  If you needed to identify systems that changed their OS version between two snapshots, what kind of SQL query would you construct?
5.  What are the advantages and disadvantages of using SQLite for system information auditing compared to a full-fledged enterprise database or a simple file-based approach?

---
# SQLite Tutorial: Network Connectivity and Port Scanner Results Analysis

## Introduction

After performing network connectivity checks and port scans with tools like Bash, PowerShell, or Python, the results are often ephemeral or stored in simple text/JSON files. SQLite provides an ideal solution for persistently storing these scan results, enabling powerful historical analysis, trend tracking, and auditing of network configurations. This tutorial will guide you through designing an SQLite schema for network scan data and querying that data to extract meaningful insights.

## Framework Alignment

This tutorial on "**Network Connectivity and Port Scanner**" using **SQLite** demonstrates the foundational skills necessary to implement components of a "Cross-Platform Baseline Hardening & Auditing Framework." The techniques learned here for storing and querying network scan results are directly applicable to auditing systems against defined security baselines and ensuring compliance.


## Why SQLite for Network Scan Results?

*   **Historical Tracking:** Store multiple scan results over time to observe changes in network topology, port statuses, or host reachability.
*   **Centralized Repository:** Consolidate scan data from various sources (hosts, subnets) into a single, queryable database.
*   **Powerful SQL Analysis:** Use SQL to identify open ports, track changes, filter by host/port, and generate custom reports.
*   **Auditing & Compliance:** Easily query for systems with unexpected open ports or verify adherence to network security policies.
*   **Portable & Serverless:** The entire database is a single file, easily transferable and usable without a dedicated database server.
*   **Minimal Dependencies:** The `sqlite3` command-line tool and Python's `sqlite3` module are standard.

## Workflow: From Scan to SQLite

The typical workflow involves using a scripting language to:
1.  **Perform Scan:** Use Bash, PowerShell, or Python scripts (as demonstrated in previous tutorials) to gather network scan data.
2.  **Structure Data:** Format the collected scan data into a structured format (e.g., JSON).
3.  **Ingest into SQLite:** Use a script (e.g., Python with `sqlite3` module) to read the structured data and insert it into an SQLite database.

For this tutorial, we will focus on the SQLite schema design and querying. We'll provide examples of how you might ingest data that has been output as JSON by a Python scan script.

## Schema Design for Network Scan Results

A robust schema will typically involve a few tables to normalize the data.

### Proposed Schema

1.  **`scans` Table:** Stores metadata about each scan execution (when it was run, what targets were scanned, etc.).
2.  **`scan_hosts` Table:** Stores details about each host involved in a scan (IP/hostname, overall reachability).
3.  **`scan_ports` Table:** Stores the results for each port scan on a specific host.

### 1. Creating the Database and Tables

We'll use a Python script to create the database and tables, as it's the most common way to programmatically manage SQLite.

```python
import sqlite3
import json
from datetime import datetime

DB_FILE = 'network_scans.db'

def create_schema():
    conn = sqlite3.connect(DB_FILE)
    cursor = conn.cursor()

    # scans table: stores metadata for each scan run
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS scans (
            scan_id INTEGER PRIMARY KEY AUTOINCREMENT,
            timestamp TEXT NOT NULL,
            scan_duration_seconds REAL,
            targets_scanned TEXT, -- e.g., "192.168.1.0/24", "host1,host2"
            ports_scanned TEXT    -- e.g., "22,80,443", "1-1024"
        )
    ''')

    # scan_hosts table: stores results for each host within a scan
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS scan_hosts (
            host_result_id INTEGER PRIMARY KEY AUTOINCREMENT,
            scan_id INTEGER NOT NULL,
            host TEXT NOT NULL,
            host_status TEXT NOT NULL, -- "Up" or "Down"
            FOREIGN KEY (scan_id) REFERENCES scans (scan_id)
        )
    ''')

    # scan_ports table: stores results for each port on a host
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS scan_ports (
            port_result_id INTEGER PRIMARY KEY AUTOINCREMENT,
            host_result_id INTEGER NOT NULL,
            port INTEGER NOT NULL,
            status TEXT NOT NULL, -- "Open", "Closed", "Filtered"
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
    conn.close()
    print(f"Database schema created/updated in {DB_FILE}")

# Example Usage:
# create_schema()
```

### 2. Data Ingestion (Python Example)

Assuming you have a JSON file (`scan_result.json`) output by the Python `network_scanner.py` script:

```json
[
  {
    "host": "192.168.1.1",
    "host_status": "Up",
    "ports": [
      {"port": 22, "status": "Open"},
      {"port": 80, "status": "Closed"},
      {"port": 443, "status": "Open"}
    ]
  },
  {
    "host": "192.168.1.2",
    "host_status": "Down",
    "ports": []
  },
  {
    "host": "192.168.1.100",
    "host_status": "Up",
    "ports": [
      {"port": 22, "status": "Filtered"},
      {"port": 80, "status": "Open"},
      {"port": 443, "status": "Open"}
    ]
  }
]
```

```python
def ingest_scan_results(scan_metadata, json_results):
    conn = sqlite3.connect(DB_FILE)
    cursor = conn.cursor()

    # 1. Insert into scans table
    cursor.execute('''
        INSERT INTO scans (timestamp, scan_duration_seconds, targets_scanned, ports_scanned)
        VALUES (?, ?, ?, ?)
    ''', (scan_metadata['timestamp'], scan_metadata.get('scan_duration_seconds'),
          scan_metadata.get('targets_scanned'), scan_metadata.get('ports_scanned')))
    scan_id = cursor.lastrowid

    # 2. Iterate through host results
    for host_result in json_results:
        cursor.execute('''
            INSERT INTO scan_hosts (scan_id, host, host_status)
            VALUES (?, ?, ?)
        ''', (scan_id, host_result['host'], host_result['host_status']))
        host_result_id = cursor.lastrowid

        # 3. Iterate through port results for each host
        for port_data in host_result['ports']:
            cursor.execute('''
                INSERT INTO scan_ports (host_result_id, port, status)
                VALUES (?, ?, ?)
            ''', (host_result_id, port_data['port'], port_data['status']))

    conn.commit()
    conn.close()
    print(f"Scan results from {scan_metadata['timestamp']} ingested successfully.")

# Example Usage:
# if __name__ == '__main__':
#     create_schema()
#     # Mock scan metadata (would come from argparse/script execution)
#     mock_scan_metadata = {
#         "timestamp": datetime.now().isoformat(),
#         "scan_duration_seconds": 12.5,
#         "targets_scanned": "192.168.1.0/24",
#         "ports_scanned": "22,80,443"
#     }
#     with open('scan_result.json', 'r') as f:
#         sample_scan_results = json.load(f)
#     ingest_scan_results(mock_scan_metadata, sample_scan_results)
```

## Basic SQL Queries for Network Scan Analysis

Once data is in SQLite, you can perform powerful analyses using the `sqlite3` command-line tool.

```bash
sqlite3 network_scans.db
```
*(All following SQL commands are executed within the `sqlite3` prompt)*

### 1. List All Scans

```sql
SELECT scan_id, timestamp, targets_scanned, ports_scanned FROM scans;
```

### 2. Find All Hosts Scanned in a Specific Scan

```sql
SELECT s.timestamp, sh.host, sh.host_status
FROM scans s
JOIN scan_hosts sh ON s.scan_id = sh.scan_id
WHERE s.scan_id = 1; -- Replace 1 with the actual scan_id
```

### 3. List All Open Ports for a Specific Host

```sql
SELECT sh.host, sp.port, sp.status
FROM scan_hosts sh
JOIN scan_ports sp ON sh.host_result_id = sp.host_result_id
WHERE sh.host = '192.168.1.1' AND sp.status = 'Open';
```

### 4. Count Open Ports Per Host (for the latest scan)

```sql
-- Find the latest scan ID
WITH LatestScan AS (
    SELECT MAX(scan_id) as latest_id FROM scans
)
SELECT
    sh.host,
    COUNT(sp.port) AS open_port_count
FROM
    scan_hosts sh
JOIN
    scan_ports sp ON sh.host_result_id = sp.host_result_id
WHERE
    sh.scan_id = (SELECT latest_id FROM LatestScan) AND sp.status = 'Open'
GROUP BY
    sh.host
ORDER BY
    open_port_count DESC;
```

### 5. Identify Hosts with Specific Open Ports (e.g., Port 22)

```sql
SELECT DISTINCT sh.host
FROM scan_hosts sh
JOIN scan_ports sp ON sh.host_result_id = sp.host_result_id
WHERE sp.port = 22 AND sp.status = 'Open';
```

### 6. Track Port Status Changes for a Host Over Time

This is more complex and typically involves joining `scans` and `scan_ports` and ordering by timestamp.

```sql
SELECT
    s.timestamp,
    sp.port,
    sp.status
FROM
    scans s
JOIN
    scan_hosts sh ON s.scan_id = sh.scan_id
JOIN
    scan_ports sp ON sh.host_result_id = sp.host_result_id
WHERE
    sh.host = '192.168.1.1' AND sp.port = 80
ORDER BY
    s.timestamp ASC;
```

### 7. Find All Hosts That Were Down in Any Scan

```sql
SELECT DISTINCT sh.host
FROM scan_hosts sh
WHERE sh.host_status = 'Down';
```

## Guiding Principles with SQLite

*   **Portability:** The SQLite database file (`.db`) is fully portable. The SQL queries are standard.
*   **Efficiency:** SQLite is highly optimized for read operations. Proper indexing ensures fast querying even with many scan results.
*   **Minimal Dependencies:** Python's built-in `sqlite3` module and the `sqlite3` CLI tool are standard components.
*   **CLI-centric:** The `sqlite3` command-line tool provides a robust interface for interacting with the database, suitable for scripting and automation.
*   **Structured Data & Actionable Output:** Storing network scan data in a relational database transforms raw output into structured, queryable information, enabling powerful analysis, reporting, and automation.

## Conclusion

SQLite serves as an invaluable backend for managing and analyzing network connectivity and port scan results. It provides the necessary structure and querying capabilities to move beyond one-off scans to comprehensive network auditing, security posture assessments, and historical change tracking. Integrating SQLite with your chosen scripting language for data collection and ingestion creates a robust and flexible framework for network intelligence. The next step is to apply this knowledge in practical exercises.
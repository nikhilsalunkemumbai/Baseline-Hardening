# SQLite Tutorial: Service/Process Monitoring and Health Check Analysis

## Introduction

While Bash, PowerShell, and Python are excellent for *collecting* service and process monitoring data and *performing* health checks, SQLite provides an ideal solution for persistently *storing* and *analyzing* this historical information. By centralizing monitoring snapshots in an SQLite database, you can perform long-term trend analysis, identify intermittent issues, establish performance baselines, and audit service uptime/downtime. This tutorial will guide you through designing an SQLite schema for monitoring data and effectively querying that data for insights.

## Framework Alignment

This tutorial on "**Service/Process Monitoring and Health Check**" using **SQLite** demonstrates the foundational skills necessary to implement components of a "Cross-Platform Baseline Hardening & Auditing Framework." The techniques learned here for storing and querying service and process monitoring data are directly applicable to auditing systems against defined security baselines and ensuring compliance.


## Why SQLite for Monitoring Data?

*   **Historical Trends:** Store periodic snapshots of process/service states and health checks to visualize performance over time, detect gradual degradations, or identify recurring problems.
*   **Performance Baselining:** Establish normal operating parameters for CPU, memory, etc., to quickly spot deviations.
*   **Uptime/Downtime Auditing:** Track service availability and quickly query periods of service disruption.
*   **Troubleshooting Intermittent Issues:** Query for patterns of short-lived failures that might be missed by real-time alerts alone.
*   **Portable & Serverless:** The entire database is a single file, easily transferable and usable without a dedicated database server, ideal for embedded or distributed monitoring agents.
*   **Minimal Dependencies:** The `sqlite3` command-line tool and Python's `sqlite3` module are standard.

## Workflow: From Collection to SQLite

The typical workflow involves using a scripting language to:
1.  **Collect Monitoring Data:** Use Bash, PowerShell, or Python scripts (as demonstrated in previous tutorials) to gather process status, service status, and health check results.
2.  **Structure Data:** Format the collected monitoring data into a structured format (e.g., JSON).
3.  **Ingest into SQLite:** Use a script (e.g., Python with `sqlite3` module) to read the structured data and insert it into an SQLite database.

For this tutorial, we will focus on the SQLite schema design and querying. We'll provide examples of how you might ingest data from a Python script that generates JSON reports.

## Schema Design for Monitoring Data

A robust schema for monitoring will typically involve tables for snapshots, detailed process/service status, and health check results.

### Proposed Schema

1.  **`monitoring_snapshots` Table:** Stores metadata about each monitoring run (when it was taken, from which host).
2.  **`service_process_status` Table:** Stores detailed information about each monitored service or process, linked to a specific snapshot.
3.  **`health_check_results` Table:** Stores the outcome of each individual health check (port, HTTP, etc.), linked to a specific snapshot.

### 1. Creating the Database and Tables

We'll use a Python script to create the database and tables.

```python
import sqlite3
import json
from datetime import datetime

DB_FILE = 'monitoring_data.db'

def create_schema(conn):
    cursor = conn.cursor()

    # monitoring_snapshots table: stores metadata for each monitoring run
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS monitoring_snapshots (
            snapshot_id INTEGER PRIMARY KEY AUTOINCREMENT,
            timestamp TEXT NOT NULL,
            hostname TEXT NOT NULL
        )
    ''')

    # service_process_status table: stores details about processes/services
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS service_process_status (
            item_id INTEGER PRIMARY KEY AUTOINCREMENT,
            snapshot_id INTEGER NOT NULL,
            name TEXT NOT NULL,
            type TEXT NOT NULL, -- 'process' or 'service'
            status TEXT NOT NULL, -- 'running', 'stopped', 'failed', etc.
            pid INTEGER, -- NULL for services, if not applicable
            cpu_percent REAL,
            memory_mb REAL,
            owner TEXT,
            uptime_seconds INTEGER,
            FOREIGN KEY (snapshot_id) REFERENCES monitoring_snapshots (snapshot_id)
        )
    ''')
    
    # health_check_results table: stores results of various health checks
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
    
    # Add indexes for faster lookups
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_monitoring_snapshots_hostname ON monitoring_snapshots (hostname)')
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_sps_snapshot_id ON service_process_status (snapshot_id)')
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_sps_name_type ON service_process_status (name, type)')
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_hcr_snapshot_id ON health_check_results (snapshot_id)')
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_hcr_target_type ON health_check_results (target, check_type)')
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_hcr_status ON health_check_results (status)')

    conn.commit()
    print("Database schema created/updated.")

# Example Usage:
# conn = sqlite3.connect(DB_FILE)
# create_schema(conn)
# conn.close()
```

### 2. Data Ingestion (Python Example)

Assuming a JSON output format from a Python monitoring script that includes `hostname`, `timestamp`, `service_process_status_list`, and `health_check_results_list`.

**`monitor_report_host1_t1.json` (simplified example):**
```json
{
  "timestamp": "2026-03-01T10:00:00Z",
  "hostname": "server-alpha",
  "service_process_status": [
    {
      "name": "nginx", "type": "service", "status": "running", "pid": 1234, "cpu_percent": 1.5, "memory_mb": 128
    },
    {
      "name": "apache2", "type": "service", "status": "stopped", "pid": null, "cpu_percent": 0.0, "memory_mb": 0.0
    },
    {
      "name": "python", "type": "process", "status": "running", "pid": 5678, "cpu_percent": 3.2, "memory_mb": 64
    }
  ],
  "health_check_results": [
    {
      "check_type": "port", "target": "server-alpha:80", "status": "PASS", "message": "Port is open", "latency_ms": 5
    },
    {
      "check_type": "http_url", "target": "http://server-alpha/health", "status": "FAIL", "message": "HTTP 503 Service Unavailable", "http_status_code": 503
    }
  ]
}
```

```python
def ingest_monitoring_report(conn, report_data):
    cursor = conn.cursor()

    # 1. Insert into monitoring_snapshots table
    cursor.execute('''
        INSERT INTO monitoring_snapshots (timestamp, hostname)
        VALUES (?, ?)
    ''', (report_data['timestamp'], report_data['hostname']))
    snapshot_id = cursor.lastrowid

    # 2. Insert into service_process_status
    for item in report_data.get('service_process_status', []):
        cursor.execute('''
            INSERT INTO service_process_status (snapshot_id, name, type, status, pid, cpu_percent, memory_mb, owner, uptime_seconds)
            VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?)
        ''', (snapshot_id, item['name'], item['type'], item['status'],
              item.get('pid'), item.get('cpu_percent'), item.get('memory_mb'),
              item.get('owner'), item.get('uptime_seconds')))

    # 3. Insert into health_check_results
    for check in report_data.get('health_check_results', []):
        cursor.execute('''
            INSERT INTO health_check_results (snapshot_id, check_type, target, status, message, latency_ms, http_status_code)
            VALUES (?, ?, ?, ?, ?, ?, ?)
        ''', (snapshot_id, check['check_type'], check['target'], check['status'],
              check.get('message'), check.get('latency_ms'), check.get('http_status_code')))

    conn.commit()
    print(f"Monitoring report for {report_data['hostname']} at {report_data['timestamp']} (ID: {snapshot_id}) ingested successfully.")

# Example Main function for ingestion:
# if __name__ == '__main__':
#     DB_FILE = 'monitoring_data.db'
#     if os.path.exists(DB_FILE):
#         os.remove(DB_FILE)
#     conn = sqlite3.connect(DB_FILE)
#     create_schema(conn)

#     # Assuming you have monitor_report_host1_t1.json
#     with open('monitor_report_host1_t1.json', 'r') as f:
#         data1 = json.load(f)
#     ingest_monitoring_report(conn, data1)
    
#     conn.close()
```

## Basic SQL Queries for Monitoring Analysis

Once data is in SQLite, you can perform powerful analyses using the `sqlite3` command-line tool.

```bash
sqlite3 monitoring_data.db
```
*(All following SQL commands are executed within the `sqlite3` prompt)*

Assume `snapshot_id = 1` for `server-alpha`.

### 1. List All Monitoring Snapshots

```sql
SELECT snapshot_id, timestamp, hostname FROM monitoring_snapshots;
```

### 2. Get the Latest Status of a Specific Service (`nginx`) on `server-alpha`

```sql
SELECT
    ms.timestamp,
    sps.status,
    sps.cpu_percent,
    sps.memory_mb
FROM
    monitoring_snapshots ms
JOIN
    service_process_status sps ON ms.snapshot_id = sps.snapshot_id
WHERE
    ms.hostname = 'server-alpha' AND sps.name = 'nginx' AND sps.type = 'service'
ORDER BY
    ms.timestamp DESC
LIMIT 1;
```

### 3. Find All Health Check Failures

```sql
SELECT
    ms.hostname,
    ms.timestamp,
    hcr.check_type,
    hcr.target,
    hcr.message,
    hcr.http_status_code
FROM
    monitoring_snapshots ms
JOIN
    health_check_results hcr ON ms.snapshot_id = hcr.snapshot_id
WHERE
    hcr.status = 'FAIL' OR hcr.status = 'ERROR'
ORDER BY
    ms.hostname, ms.timestamp;
```

### 4. Track CPU Usage for a Specific Process (`python`) on `server-alpha` Over Time

```sql
SELECT
    ms.timestamp,
    sps.cpu_percent
FROM
    monitoring_snapshots ms
JOIN
    service_process_status sps ON ms.snapshot_id = sps.snapshot_id
WHERE
    ms.hostname = 'server-alpha' AND sps.name = 'python' AND sps.type = 'process'
ORDER BY
    ms.timestamp ASC;
```

### 5. Calculate Service Uptime/Downtime (Conceptual - requires more data points for robust calculation)

To calculate actual uptime percentage, you'd need continuous data. Here's how to identify downtime events.

```sql
-- Find all 'stopped' events for nginx on server-alpha
SELECT
    ms.timestamp,
    sps.status
FROM
    monitoring_snapshots ms
JOIN
    service_process_status sps ON ms.snapshot_id = sps.snapshot_id
WHERE
    ms.hostname = 'server-alpha' AND sps.name = 'nginx' AND sps.type = 'service' AND sps.status = 'stopped'
ORDER BY
    ms.timestamp ASC;
```

### 6. Identify Hosts with any Health Check Failures for a specific HTTP endpoint

```sql
SELECT DISTINCT
    ms.hostname
FROM
    monitoring_snapshots ms
JOIN
    health_check_results hcr ON ms.snapshot_id = hcr.snapshot_id
WHERE
    hcr.target LIKE '%/health' AND (hcr.status = 'FAIL' OR hcr.status = 'ERROR');
```

## Guiding Principles with SQLite

*   **Portability:** The SQLite database file (`.db`) is fully portable. Standard SQL is used for queries.
*   **Efficiency:** SQLite is highly optimized for storage and retrieval, making it suitable for large volumes of monitoring data. Indexes are crucial for query performance.
*   **Minimal Dependencies:** Python's built-in `sqlite3` module and the `sqlite3` CLI tool are standard components.
*   **CLI-centric:** The `sqlite3` command-line tool provides a robust interface for interacting with the database.
*   **Structured Data & Actionable Output:** Storing monitoring data in a relational database transforms raw observations into structured, queryable information, enabling powerful historical analysis, performance trending, and proactive identification of issues.

## Conclusion

SQLite is an incredibly valuable tool for persisting and analyzing service/process monitoring and health check data. It elevates raw monitoring outputs into a rich historical dataset, allowing for in-depth analysis of system behavior over time. By integrating SQLite with your chosen scripting language for data collection and ingestion, you build a comprehensive, efficient, and cross-platform monitoring data analysis framework. The next step is to apply this knowledge in practical exercises.
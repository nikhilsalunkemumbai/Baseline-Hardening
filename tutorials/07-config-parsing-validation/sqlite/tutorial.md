# SQLite Tutorial: Configuration File Parsing and Validation Analysis

## Introduction

While Bash, PowerShell, and Python are excellent for *parsing* configuration files and *performing* validation checks, SQLite provides an ideal solution for persistently *storing* and *querying* these configurations and their validation results. By centralizing configuration snapshots and audit reports in an SQLite database, you can perform long-term compliance monitoring, track configuration drifts, identify systems with non-compliant settings, and manage large sets of configuration data with the full power of SQL. This tutorial will guide you through designing an SQLite schema for configuration data and querying that data effectively.

## Framework Alignment

This tutorial on "**Configuration File Parsing and Validation**" using **SQLite** demonstrates the foundational skills necessary to implement components of a "Cross-Platform Baseline Hardening & Auditing Framework." The techniques learned here for storing and querying configuration settings and validation results are directly applicable to auditing systems against defined security baselines and ensuring compliance.


## Why SQLite for Configuration Auditing?

*   **Historical Tracking:** Store multiple configuration snapshots and their validation results over time to track changes, identify when a setting deviated, or prove compliance history.
*   **Centralized Repository:** Consolidate configuration data and audit findings from various systems into a single, queryable database.
*   **Powerful SQL Analysis:** Use SQL to filter configurations by value, identify non-compliant settings, aggregate validation results, and generate custom audit reports.
*   **Compliance & Security:** Easily audit configurations against security policies (e.g., all web servers must enforce TLSv1.2+, no root login enabled).
*   **Portable & Serverless:** The entire database is a single file, easily transferable and usable without a dedicated database server.
*   **Minimal Dependencies:** The `sqlite3` command-line tool and Python's `sqlite3` module are standard.

## Workflow: From Collection to SQLite

The typical workflow involves using a scripting language to:
1.  **Parse & Validate Configs:** Use Bash, PowerShell, or Python scripts (as demonstrated in previous tutorials) to read configuration files, extract settings, and perform validation checks.
2.  **Structure Data:** Format the parsed configuration data and validation results into a structured format (e.g., JSON).
3.  **Ingest into SQLite:** Use a script (e.g., Python with `sqlite3` module) to read the structured data and insert it into an SQLite database.

For this tutorial, we will focus on the SQLite schema design and querying. We'll provide examples of how you might ingest data from a Python script that generates JSON reports.

## Schema Design for Configuration Auditing

A robust schema will typically involve tables for snapshots, configuration settings, and validation results.

### Proposed Schema

1.  **`config_snapshots` Table:** Stores metadata about each configuration snapshot (when it was taken, from which host, config file path).
2.  **`config_settings` Table:** Stores individual key-value pairs from a parsed configuration file, linked to a specific snapshot. This allows for flexible storage of various config formats.
3.  **`validation_results` Table:** Stores the outcome of each validation check performed on a snapshot.

### 1. Creating the Database and Tables

We'll use a Python script to create the database and tables.

```python
import sqlite3
import json
from datetime import datetime

DB_FILE = 'config_audits.db'

def create_schema(conn):
    cursor = conn.cursor()

    # config_snapshots table: stores metadata for each configuration snapshot
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS config_snapshots (
            snapshot_id INTEGER PRIMARY KEY AUTOINCREMENT,
            timestamp TEXT NOT NULL,
            hostname TEXT NOT NULL,
            config_file_path TEXT NOT NULL,
            config_type TEXT NOT NULL
        )
    ''')

    # config_settings table: stores individual key-value pairs from the config
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS config_settings (
            setting_id INTEGER PRIMARY KEY AUTOINCREMENT,
            snapshot_id INTEGER NOT NULL,
            key_path TEXT NOT NULL, -- e.g., "database.port" or "PermitRootLogin"
            value TEXT,
            FOREIGN KEY (snapshot_id) REFERENCES config_snapshots (snapshot_id)
        )
    ''')
    
    # validation_results table: stores results of policy checks
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS validation_results (
            validation_id INTEGER PRIMARY KEY AUTOINCREMENT,
            snapshot_id INTEGER NOT NULL,
            rule_name TEXT NOT NULL,
            key_path TEXT NOT NULL,
            status TEXT NOT NULL, -- "PASS", "FAIL", "ERROR"
            message TEXT,
            expected_value TEXT,
            actual_value TEXT,
            FOREIGN KEY (snapshot_id) REFERENCES config_snapshots (snapshot_id)
        )
    ''')
    
    # Add indexes for faster lookups
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_config_snapshots_hostname ON config_snapshots (hostname)')
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_config_settings_snapshot_id ON config_settings (snapshot_id)')
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_config_settings_key_path ON config_settings (key_path)')
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_validation_results_snapshot_id ON validation_results (snapshot_id)')
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_validation_results_status ON validation_results (status)')

    conn.commit()
    print("Database schema created/updated.")

# Example Usage:
# conn = sqlite3.connect(DB_FILE)
# create_schema(conn)
# conn.close()
```

### 2. Data Ingestion (Python Example)

Assuming a JSON output format from a Python config validator script that includes both the parsed config and validation results.

**`config_audit_report_t1.json` (simplified example):**
```json
{
  "timestamp": "2026-03-01T12:00:00Z",
  "hostname": "web-server-01",
  "config_file": "/etc/httpd/conf/httpd.conf",
  "config_type": "apache_conf_like",
  "parsed_config": {
    "ServerRoot": "/etc/httpd",
    "Listen": "80",
    "PermitRootLogin": "no",
    "LogLevel": "info"
  },
  "validation_results": [
    {
      "key": "PermitRootLogin",
      "status": "PASS",
      "message": "Value is 'no' as expected."
    },
    {
      "key": "Listen",
      "status": "FAIL",
      "message": "Expected '443', found '80'."
    }
  ]
}
```

```python
def ingest_config_audit_report(conn, report_data):
    cursor = conn.cursor()

    # 1. Insert into config_snapshots table
    cursor.execute('''
        INSERT INTO config_snapshots (timestamp, hostname, config_file_path, config_type)
        VALUES (?, ?, ?, ?)
    ''', (report_data['timestamp'], report_data['hostname'],
          report_data['config_file'], report_data['config_type']))
    snapshot_id = cursor.lastrowid

    # 2. Insert into config_settings
    for key_path, value in report_data['parsed_config'].items():
        cursor.execute('''
            INSERT INTO config_settings (snapshot_id, key_path, value)
            VALUES (?, ?, ?)
        ''', (snapshot_id, key_path, str(value))) # Store all values as TEXT for simplicity

    # 3. Insert into validation_results
    for validation in report_data['validation_results']:
        cursor.execute('''
            INSERT INTO validation_results (snapshot_id, rule_name, key_path, status, message, expected_value, actual_value)
            VALUES (?, ?, ?, ?, ?, ?, ?)
        ''', (snapshot_id, validation.get('rule_name', 'Unnamed Rule'), validation['key'],
              validation['status'], validation.get('message'),
              str(validation.get('expected', '')), str(validation.get('actual', ''))))

    conn.commit()
    print(f"Config audit report for {report_data['hostname']} at {report_data['timestamp']} (ID: {snapshot_id}) ingested successfully.")

# Example Main function for ingestion:
# if __name__ == '__main__':
#     DB_FILE = 'config_audits.db'
#     if os.path.exists(DB_FILE):
#         os.remove(DB_FILE)
#     conn = sqlite3.connect(DB_FILE)
#     create_schema(conn)

#     # Assuming you have config_audit_report_t1.json
#     with open('config_audit_report_t1.json', 'r') as f:
#         data1 = json.load(f)
#     ingest_config_audit_report(conn, data1)
    
#     conn.close()
```

## Basic SQL Queries for Configuration Auditing

Once data is in SQLite, you can perform powerful analyses using the `sqlite3` command-line tool.

```bash
sqlite3 config_audits.db
```
*(All following SQL commands are executed within the `sqlite3` prompt)*

Assume `snapshot_id = 1` is for `config_audit_report_t1.json`.

### 1. List All Configuration Snapshots

```sql
SELECT snapshot_id, timestamp, hostname, config_file_path FROM config_snapshots;
```

### 2. Retrieve All Settings from a Specific Snapshot (`snapshot_id = 1`)

```sql
SELECT key_path, value
FROM config_settings
WHERE snapshot_id = 1
ORDER BY key_path;
```

### 3. Find Systems with a Specific Configuration Value (e.g., `Listen` port `80`)

```sql
SELECT cs.hostname, cs.timestamp, cs.config_file_path
FROM config_snapshots cs
JOIN config_settings cset ON cs.snapshot_id = cset.snapshot_id
WHERE cset.key_path = 'Listen' AND cset.value = '80';
```

### 4. Identify All Failed Validation Checks Across All Snapshots

```sql
SELECT cs.hostname, cs.timestamp, vr.key_path, vr.message, vr.expected_value, vr.actual_value
FROM config_snapshots cs
JOIN validation_results vr ON cs.snapshot_id = vr.snapshot_id
WHERE vr.status = 'FAIL'
ORDER BY cs.hostname, cs.timestamp;
```

### 5. Track Changes in a Specific Configuration Setting Over Time

Assume you have multiple snapshots for 'web-server-01' and 'PermitRootLogin' key.

```sql
-- This query requires having multiple snapshots for the same host and key
SELECT
    cs.timestamp,
    cset.value AS PermitRootLogin_Value
FROM
    config_snapshots cs
JOIN
    config_settings cset ON cs.snapshot_id = cset.snapshot_id
WHERE
    cs.hostname = 'web-server-01' AND cset.key_path = 'PermitRootLogin'
ORDER BY
    cs.timestamp;
```

### 6. Count Systems with Non-Compliant `Listen` Port (where `status = FAIL` for `Listen` checks)

```sql
SELECT
    cs.hostname,
    cs.config_file_path,
    vr.message
FROM
    config_snapshots cs
JOIN
    validation_results vr ON cs.snapshot_id = vr.snapshot_id
WHERE
    vr.key_path = 'Listen' AND vr.status = 'FAIL'
GROUP BY
    cs.hostname, cs.config_file_path;
```

## Guiding Principles with SQLite

*   **Portability:** The SQLite database file (`.db`) is fully portable. The SQL queries are standard.
*   **Efficiency:** SQLite is highly optimized for read operations. Proper indexing ensures fast querying for configuration settings and validation results.
*   **Minimal Dependencies:** Python's built-in `sqlite3` module and the `sqlite3` CLI tool are standard components.
*   **CLI-centric:** The `sqlite3` command-line tool provides a robust interface for interacting with the database.
*   **Structured Data & Actionable Output:** Storing parsed configuration and validation results in a relational database transforms raw audit data into structured, queryable information, enabling powerful historical analysis, compliance reporting, and security investigations.

## Conclusion

SQLite provides a robust and indispensable backbone for managing configuration files and their validation results. It transforms transient configuration states and audit findings into persistent, queryable knowledge, enabling in-depth compliance monitoring, historical change tracking, and the rapid identification of misconfigurations or policy violations. By integrating SQLite with your chosen scripting language for data collection and ingestion, you create a comprehensive and flexible configuration management and auditing framework. The next step is to apply this knowledge in practical exercises.
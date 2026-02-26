# SQLite Exercise: Configuration File Parsing and Validation Analysis

## Objective

This exercise challenges you to apply your SQL querying skills using SQLite to store, manage, and analyze historical configuration settings and their validation results. You will create an SQLite database schema, ingest provided JSON configuration audit reports, and then perform various SQL queries to identify non-compliant systems, track configuration drifts, and audit security settings over time.

## Framework Alignment

This exercise on "**Configuration File Parsing and Validation**" using **SQLite** provides practical experience in building and utilizing tools for a "Cross-Platform Baseline Hardening & Auditing Framework." By completing these tasks, you will learn how to audit and manage configuration settings, ensuring compliance with security policies and identifying unauthorized changes—essential steps in maintaining a secure and auditable environment.


## Scenario

You are a compliance officer or security architect responsible for ensuring that all systems adhere to strict configuration policies. Automated scripts periodically collect configuration files and validate them against a set of rules, generating structured reports. Your task is to consolidate these reports into a central SQLite database to facilitate querying, historical analysis, and identifying systems that require attention.

## Input Data

You will be working with the following JSON audit report files, located in the same directory as this exercise:

*   `config_report_server1_t1.json`: Audit report for `web-server-01` at an initial time.
*   `config_report_server1_t2.json`: Audit report for `web-server-01` at a later time, after some configuration changes.
*   `config_report_server2_t1.json`: Audit report for `db-server-01` at an initial time.

## Setup

You will need an SQLite database. You will use a Python script to perform the setup and ingestion.

### 1. Create `ingest_config_audits.py`

Create a Python script named `ingest_config_audits.py` in the same directory as the exercise and JSON files. This script will create the necessary database schema and ingest the JSON data.

```python
import sqlite3
import json
import os
from datetime import datetime

DB_FILE = 'config_audits.db'
REPORT_FILES = [
    'config_report_server1_t1.json',
    'config_report_server1_t2.json',
    'config_report_server2_t1.json'
]

def create_schema(conn):
    cursor = conn.cursor()

    cursor.execute('''
        CREATE TABLE IF NOT EXISTS config_snapshots (
            snapshot_id INTEGER PRIMARY KEY AUTOINCREMENT,
            timestamp TEXT NOT NULL,
            hostname TEXT NOT NULL,
            config_file_path TEXT NOT NULL,
            config_type TEXT NOT NULL
        )
    ''')

    cursor.execute('''
        CREATE TABLE IF NOT EXISTS config_settings (
            setting_id INTEGER PRIMARY KEY AUTOINCREMENT,
            snapshot_id INTEGER NOT NULL,
            key_path TEXT NOT NULL,
            value TEXT,
            FOREIGN KEY (snapshot_id) REFERENCES config_snapshots (snapshot_id)
        )
    ''')
    
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS validation_results (
            validation_id INTEGER PRIMARY KEY AUTOINCREMENT,
            snapshot_id INTEGER NOT NULL,
            rule_name TEXT, -- Optional, can derive from key_path or message
            key_path TEXT NOT NULL,
            status TEXT NOT NULL, -- "PASS", "FAIL", "ERROR"
            message TEXT,
            expected_value TEXT,
            actual_value TEXT,
            FOREIGN KEY (snapshot_id) REFERENCES config_snapshots (snapshot_id)
        )
    ''')
    
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_config_snapshots_hostname ON config_snapshots (hostname)')
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_config_settings_snapshot_id ON config_settings (snapshot_id)')
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_config_settings_key_path ON config_settings (key_path)')
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_validation_results_snapshot_id ON validation_results (snapshot_id)')
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_validation_results_status ON validation_results (status)')

    conn.commit()
    print("Database schema created/updated.")

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
    # The parsed_config can be nested, so flatten it for key_path.
    # For simplicity, this example assumes parsed_config is mostly flat key-value
    # or a shallow nest like config.ini sections.
    # A more robust solution would recursively flatten nested JSON/YAML.
    for key_or_section, value in report_data['parsed_config'].items():
        if isinstance(value, dict): # Handle sections or nested JSON objects
            for nested_key, nested_value in value.items():
                cursor.execute('''
                    INSERT INTO config_settings (snapshot_id, key_path, value)
                    VALUES (?, ?, ?)
                ''', (snapshot_id, f"{key_or_section}.{nested_key}", str(nested_value)))
        else: # Simple key-value
            cursor.execute('''
                INSERT INTO config_settings (snapshot_id, key_path, value)
                VALUES (?, ?, ?)
            ''', (snapshot_id, key_or_section, str(value)))

    # 3. Insert into validation_results
    for validation in report_data['validation_results']:
        # rule_name is optional, can use key_path if not provided
        rule_name = validation.get('rule_name', validation['key']) 
        cursor.execute('''
            INSERT INTO validation_results (snapshot_id, rule_name, key_path, status, message, expected_value, actual_value)
            VALUES (?, ?, ?, ?, ?, ?, ?)
        ''', (snapshot_id, rule_name, validation['key'], validation['status'], validation.get('message'),
              str(validation.get('expected', '')), str(validation.get('actual', ''))))

    conn.commit()
    print(f"Config audit report for {report_data['hostname']} at {report_data['timestamp']} (ID: {snapshot_id}) ingested successfully.")

def main():
    if os.path.exists(DB_FILE):
        os.remove(DB_FILE)
    
    conn = sqlite3.connect(DB_FILE)
    create_schema(conn)

    for report_file in REPORT_FILES:
        try:
            with open(report_file, 'r') as f:
                json_data = json.load(f)
            ingest_config_audit_report(conn, json_data)
        except FileNotFoundError:
            print(f"Error: Report file '{report_file}' not found.")
        except json.JSONDecodeError:
            print(f"Error: Malformed JSON in '{report_file}'.")
        except Exception as e:
            print(f"An error occurred while processing '{report_file}': {e}")
    conn.close()
    print(f"
All configuration audit reports ingested into {DB_FILE}.")

if __name__ == '__main__':
    main()
```
### 2. Run the Ingestion Script

Execute the Python script to create the database and load the data:
```bash
python ingest_config_audits.py
```
This will create `config_audits.db` and populate it.

## Tasks

Using SQL queries within the `sqlite3` prompt, provide the solution for each of the following tasks. Start by opening the database:
```bash
sqlite3 config_audits.db
```
*(All following SQL commands are executed within the `sqlite3` prompt)*

Assume:
*   `snapshot_id = 1`: `config_report_server1_t1.json`
*   `snapshot_id = 2`: `config_report_server1_t2.json`
*   `snapshot_id = 3`: `config_report_server2_t1.json`

1.  **List All Configuration Snapshots:**
    *   Retrieve the `snapshot_id`, `timestamp`, `hostname`, and `config_file_path` for all recorded snapshots.

2.  **Retrieve All Settings for `web-server-01` from its Latest Snapshot (`snapshot_id = 2`):**
    *   Display the `key_path` and `value` for all configuration settings from `snapshot_id = 2`.

3.  **Identify All Non-Compliant `hostname`s and `config_file_path`s:**
    *   List the `hostname` and `config_file_path` for every snapshot that has at least one `FAIL` status in its `validation_results`. Ensure unique host/path combinations.

4.  **Track Changes in `PermitRootLogin` for `web-server-01` Over Time:**
    *   Show the `timestamp` and `value` of the `security.permit_root_login` setting for `web-server-01` across all its snapshots, ordered chronologically.

5.  **Identify Hosts with Specific Policy Failures:**
    *   List the `hostname` and `config_file_path` for any snapshot where the `database.port` validation `status` is 'FAIL'.

6.  **Count Total Validation Failures per Host:**
    *   For each `hostname`, count how many `FAIL` statuses were recorded across all its snapshots. Display `hostname` and `total_failures`.

7.  **Find All Settings Where `LogLevel` is 'DEBUG':**
    *   List `hostname`, `timestamp`, and the `value` of the `logging.level` setting where `value` is 'DEBUG'.

## Deliverables

For each task, provide the SQL query and the resulting output.

## Reflection Questions

1.  How does storing configuration settings and validation results in a relational database (SQLite) facilitate historical analysis and compliance reporting compared to individual JSON files?
2.  Discuss the advantages of using a normalized schema (separate tables for snapshots, settings, and validation results) for managing configuration audit data.
3.  If you wanted to identify configurations where a security setting (e.g., `enable_https`) changed from `false` to `true` between two snapshots, what kind of SQL query would you construct?
4.  How could this SQLite database be used in an automated system to trigger alerts when a critical configuration policy fails validation on any system?
5.  What are the advantages and disadvantages of using SQLite for configuration audit analysis compared to directly parsing configuration files or using a dedicated Configuration Management Database (CMDB)?

---
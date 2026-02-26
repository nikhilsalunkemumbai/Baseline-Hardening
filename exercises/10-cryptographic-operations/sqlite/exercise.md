# SQLite Exercise: Basic Cryptographic Operations Analysis

## Objective

This exercise challenges you to apply your SQL querying skills using SQLite to store, manage, and analyze cryptographic checksums for file integrity monitoring (FIM). You will create an SQLite database schema, ingest provided JSON file hash reports, and then perform various SQL queries to establish baselines, detect file changes, and identify potential security incidents across your infrastructure.

## Framework Alignment

This exercise on "**Basic Cryptographic Operations (Hashing/Encoding)**" using **SQLite** provides practical experience in building and utilizing tools for a "Cross-Platform Baseline Hardening & Auditing Framework." By completing these tasks, you will learn how to verify data integrity and detect unauthorized modifications—essential steps in maintaining a secure and auditable environment.


## Scenario

You are a security analyst tasked with maintaining the integrity of critical system files across several servers. You have a system in place that periodically scans important directories and generates JSON reports containing file paths, sizes, and their cryptographic hashes. Your goal is to consolidate these reports into a central SQLite database to automate FIM, detect unauthorized modifications, and provide an audit trail.

## Input Data

You will be working with the following JSON file hash report files, located in the same directory as this exercise:

*   `file_hash_report_server1_t1.json`: File hash report for `web-server-01` at an initial baseline time.
*   `file_hash_report_server1_t2.json`: File hash report for `web-server-01` at a later time, where `index.html` has been modified.
*   `file_hash_report_server2_t1.json`: File hash report for `db-server-01` at an initial baseline time.

## Setup

You will need an SQLite database. You will use a Python script to perform the setup and ingestion.

### 1. Create `ingest_hash_data.py`

Create a Python script named `ingest_hash_data.py` in the same directory as the exercise and JSON files. This script will create the necessary database schema and ingest the JSON data.

```python
import sqlite3
import json
import os
from datetime import datetime

DB_FILE = 'crypto_audits.db'
REPORT_FILES = [
    'file_hash_report_server1_t1.json',
    'file_hash_report_server1_t2.json',
    'file_hash_report_server2_t1.json'
]

def create_schema(conn):
    cursor = conn.cursor()

    cursor.execute('''
        CREATE TABLE IF NOT EXISTS scan_snapshots (
            snapshot_id INTEGER PRIMARY KEY AUTOINCREMENT,
            timestamp TEXT NOT NULL,
            hostname TEXT NOT NULL
        )
    ''')

    cursor.execute('''
        CREATE TABLE IF NOT EXISTS file_checksums (
            checksum_id INTEGER PRIMARY KEY AUTOINCREMENT,
            snapshot_id INTEGER NOT NULL,
            file_path TEXT NOT NULL,
            file_size INTEGER,
            md5_hash TEXT,
            sha256_hash TEXT,
            sha512_hash TEXT,
            FOREIGN KEY (snapshot_id) REFERENCES scan_snapshots (snapshot_id)
        )
    ''')
    
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_scan_snapshots_hostname ON scan_snapshots (hostname)')
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_file_checksums_snapshot_id ON file_checksums (snapshot_id)')
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_file_checksums_filepath ON file_checksums (file_path)')
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_file_checksums_sha256 ON file_checksums (sha256_hash)')
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_file_checksums_md5 ON file_checksums (md5_hash)')

    conn.commit()
    print("Database schema created/updated.")

def ingest_hash_report(conn, report_data):
    cursor = conn.cursor()

    cursor.execute('''
        INSERT INTO scan_snapshots (timestamp, hostname)
        VALUES (?, ?)
    ''', (report_data['timestamp'], report_data['hostname']))
    snapshot_id = cursor.lastrowid

    for file_data in report_data.get('files', []):
        cursor.execute('''
            INSERT INTO file_checksums (snapshot_id, file_path, file_size, md5_hash, sha256_hash, sha512_hash)
            VALUES (?, ?, ?, ?, ?, ?)
        ''', (snapshot_id, file_data['file_path'], file_data.get('file_size'),
              file_data.get('md5_hash'), file_data.get('sha256_hash'), file_data.get('sha512_hash')))

    conn.commit()
    print(f"Hash report for {report_data['hostname']} at {report_data['timestamp']} (ID: {snapshot_id}) ingested successfully.")

def main():
    if os.path.exists(DB_FILE):
        os.remove(DB_FILE)
    
    conn = sqlite3.connect(DB_FILE)
    create_schema(conn)

    for report_file in REPORT_FILES:
        try:
            with open(report_file, 'r') as f:
                json_data = json.load(f)
            ingest_hash_report(conn, json_data)
        except FileNotFoundError:
            print(f"Error: Report file '{report_file}' not found.")
        except json.JSONDecodeError:
            print(f"Error: Malformed JSON in '{report_file}'.")
        except Exception as e:
            print(f"An error occurred while processing '{report_file}': {e}")
    conn.close()
    print(f"
All hash reports ingested into {DB_FILE}.")

if __name__ == '__main__':
    main()
```

### 2. Run the Ingestion Script

Execute the Python script to create the database and load the data:
```bash
python ingest_hash_data.py
```
This will create `crypto_audits.db` and populate it.

## Tasks

Using SQL queries within the `sqlite3` prompt, provide the solution for each of the following tasks. Start by opening the database:
```bash
sqlite3 crypto_audits.db
```
*(All following SQL commands are executed within the `sqlite3` prompt)*

Assume:
*   `snapshot_id = 1`: `web-server-01` (first scan)
*   `snapshot_id = 2`: `web-server-01` (second scan, `index.html` modified)
*   `snapshot_id = 3`: `db-server-01` (first scan)

### 1. List All Files and Their SHA256 Hashes from the Latest Scan for `web-server-01`

```sql
SELECT
    fc.file_path,
    fc.sha256_hash,
    ss.timestamp
FROM
    file_checksums fc
JOIN
    scan_snapshots ss ON fc.snapshot_id = ss.snapshot_id
WHERE
    ss.hostname = 'web-server-01'
ORDER BY
    ss.timestamp DESC
LIMIT (SELECT COUNT(*) FROM file_checksums WHERE snapshot_id = (SELECT MAX(snapshot_id) FROM scan_snapshots WHERE hostname = 'web-server-01'));
```

### 2. Identify Files That Have Changed (SHA256 Mismatch) on `web-server-01` Between Two Scans (`snapshot_id=1` and `snapshot_id=2`)

```sql
SELECT
    fc1.file_path,
    fc1.sha256_hash AS sha256_t1,
    fc2.sha256_hash AS sha256_t2,
    ss1.timestamp AS timestamp_t1,
    ss2.timestamp AS timestamp_t2
FROM
    file_checksums fc1
JOIN
    scan_snapshots ss1 ON fc1.snapshot_id = ss1.snapshot_id
JOIN
    file_checksums fc2 ON fc1.file_path = fc2.file_path
JOIN
    scan_snapshots ss2 ON fc2.snapshot_id = ss2.snapshot_id
WHERE
    ss1.hostname = 'web-server-01' AND ss2.hostname = 'web-server-01'
    AND fc1.snapshot_id = 1 AND fc2.snapshot_id = 2
    AND fc1.sha256_hash != fc2.sha256_hash;
```

### 3. Find All Files with a Specific SHA256 Hash Across All Snapshots and Hosts

```sql
SELECT
    ss.hostname,
    ss.timestamp,
    fc.file_path,
    fc.sha256_hash
FROM
    file_checksums fc
JOIN
    scan_snapshots ss ON fc.snapshot_id = ss.snapshot_id
WHERE
    fc.sha256_hash = 'e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855';
```

### 4. Identify Duplicate Files Across All Hosts and Scans (by SHA256 Hash)

```sql
SELECT
    fc.sha256_hash,
    GROUP_CONCAT(ss.hostname || ':' || fc.file_path || ' (snapshot_id:' || fc.snapshot_id || ')') AS locations,
    COUNT(*) AS duplicate_count
FROM
    file_checksums fc
JOIN
    scan_snapshots ss ON fc.snapshot_id = ss.snapshot_id
GROUP BY
    fc.sha256_hash
HAVING
    COUNT(*) > 1;
```

### 5. Count the Total Number of Files Scanned per Host per Snapshot

```sql
SELECT
    ss.hostname,
    ss.timestamp,
    COUNT(fc.checksum_id) AS total_files_scanned
FROM
    scan_snapshots ss
JOIN
    file_checksums fc ON ss.snapshot_id = fc.snapshot_id
GROUP BY
    ss.hostname, ss.timestamp
ORDER BY
    ss.hostname, ss.timestamp;
```

### 6. Identify Files That Were Present in `snapshot_id=1` but Deleted in `snapshot_id=2` on `web-server-01`

```sql
SELECT
    fc1.file_path
FROM
    file_checksums fc1
WHERE
    fc1.snapshot_id = 1
    AND NOT EXISTS (
        SELECT 1
        FROM file_checksums fc2
        WHERE fc2.snapshot_id = 2 AND fc2.file_path = fc1.file_path
    )
    AND EXISTS (SELECT 1 FROM scan_snapshots WHERE snapshot_id = 1 AND hostname = 'web-server-01');
```

### 7. Identify New Files in `snapshot_id=2` That Were Not Present in `snapshot_id=1` on `web-server-01`

```sql
SELECT
    fc2.file_path
FROM
    file_checksums fc2
WHERE
    fc2.snapshot_id = 2
    AND NOT EXISTS (
        SELECT 1
        FROM file_checksums fc1
        WHERE fc1.snapshot_id = 1 AND fc1.file_path = fc2.file_path
    )
    AND EXISTS (SELECT 1 FROM scan_snapshots WHERE snapshot_id = 2 AND hostname = 'web-server-01');
```

## Deliverables

For each task, provide the SQL query and the resulting output.

## Reflection Questions

1.  How does an SQLite database act as a foundational component for a File Integrity Monitoring (FIM) solution?
2.  Explain the importance of cryptographic hashing in FIM and how detecting hash mismatches indicates potential file tampering.
3.  Discuss the role of `snapshot_id` and `timestamp` in the `scan_snapshots` table for historical analysis and establishing a chain of custody for file changes.
4.  How would you extend this database schema to include information about who last modified a file or when it was accessed, assuming this data could be collected by the scanning script?
5.  What are the advantages of using SQLite for storing FIM data compared to simply logging hash results to text files or using a more complex relational database like PostgreSQL for very large-scale FIM?

---
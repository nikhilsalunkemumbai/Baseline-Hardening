# SQLite Tutorial: Basic Cryptographic Operations Analysis

## Introduction

While Bash, PowerShell, and Python are excellent for *performing* basic cryptographic operations like hashing and encoding, SQLite provides an ideal solution for persistently *storing* and *analyzing* the results of these operations, especially for file integrity monitoring. By centralizing cryptographic checksums and related metadata in an SQLite database, you can perform long-term file integrity monitoring, detect unauthorized changes, identify duplicate files, and maintain an audit trail of file states. This tutorial will guide you through designing an SQLite schema for cryptographic data and effectively querying that data for security and data management insights.

## Framework Alignment

This tutorial on "**Basic Cryptographic Operations (Hashing/Encoding)**" using **SQLite** demonstrates the foundational skills necessary to implement components of a "Cross-Platform Baseline Hardening & Auditing Framework." The techniques learned here for storing and querying cryptographic data are directly applicable to auditing systems against defined security baselines and ensuring compliance.


## Why SQLite for Cryptographic Data?

*   **File Integrity Monitoring (FIM):** Establish a baseline of file hashes and periodically compare current hashes against the baseline to detect any modifications.
*   **Change Detection:** Track when and how files have changed by comparing their hashes over time.
*   **Audit Trails:** Maintain a historical record of file states, including who might have accessed or modified them (if combined with other logs).
*   **Duplicate File Identification:** Easily find identical files across a system or multiple systems by querying for matching hash values.
*   **Compliance & Forensics:** Provide verifiable proof of file integrity or evidence of tampering.
*   **Portable & Serverless:** The entire database is a single file, easily transferable and usable without a dedicated database server.
*   **Minimal Dependencies:** The `sqlite3` command-line tool and Python's `sqlite3` module are standard.

## Workflow: From Collection to SQLite

The typical workflow involves using a scripting language to:
1.  **Perform Cryptographic Operations:** Use Bash, PowerShell, or Python scripts (as demonstrated in previous tutorials) to hash files or perform encoding operations.
2.  **Structure Data:** Format the results (e.g., file path, hash value, algorithm, timestamp) into a structured format (e.g., JSON).
3.  **Ingest into SQLite:** Use a script (e.g., Python with `sqlite3` module) to read the structured data and insert it into an SQLite database.

For this tutorial, we will focus on the SQLite schema design and querying, primarily for file hashing. We'll provide examples of how you might ingest data from a Python script that generates JSON reports.

## Schema Design for Cryptographic Data

A robust schema for cryptographic data, especially for FIM, will typically involve tables for snapshots of scanned files and their calculated checksums.

### Proposed Schema

1.  **`scan_snapshots` Table:** Stores metadata about each file integrity scan run (when it was taken, from which host).
2.  **`file_checksums` Table:** Stores details about each file (path, size, hash values), linked to a specific scan snapshot.

### 1. Creating the Database and Tables

We'll use a Python script to create the database and tables.

```python
import sqlite3
import json
from datetime import datetime

DB_FILE = 'crypto_audits.db'

def create_schema(conn):
    cursor = conn.cursor()

    # scan_snapshots table: stores metadata for each file integrity scan
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS scan_snapshots (
            snapshot_id INTEGER PRIMARY KEY AUTOINCREMENT,
            timestamp TEXT NOT NULL,
            hostname TEXT NOT NULL
        )
    ''')

    # file_checksums table: stores details about each file and its checksums
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
    
    # Add indexes for faster lookups
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_scan_snapshots_hostname ON scan_snapshots (hostname)')
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_file_checksums_snapshot_id ON file_checksums (snapshot_id)')
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_file_checksums_filepath ON file_checksums (file_path)')
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_file_checksums_sha256 ON file_checksums (sha256_hash)')
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_file_checksums_md5 ON file_checksums (md5_hash)')

    conn.commit()
    print("Database schema created/updated.")

# Example Usage:
# conn = sqlite3.connect(DB_FILE)
# create_schema(conn)
# conn.close()
```

### 2. Data Ingestion (Python Example)

Assuming a JSON output format from a Python hashing script that includes `hostname`, `timestamp`, and a list of `files`.

**`file_hash_report_server1_t1.json` (simplified example):**
```json
{
  "timestamp": "2026-03-01T10:00:00Z",
  "hostname": "web-server-01",
  "files": [
    {
      "file_path": "/var/www/html/index.html",
      "file_size": 1024,
      "md5_hash": "d41d8cd98f00b204e9800998ecf8427e",
      "sha256_hash": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855"
    },
    {
      "file_path": "/etc/nginx/nginx.conf",
      "file_size": 512,
      "md5_hash": "a1a2a3a4a5a6a7a8a9a0b1b2b3b4b5b6",
      "sha256_hash": "b0c1c2c3c4c5c6c7c8c9d0d1d2d3d4d5e6e7e8e9f0f1f2f3f4f5f6f7f8f9g0g1"
    }
  ]
}
```

**`file_hash_report_server1_t2.json` (index.html modified):**
```json
{
  "timestamp": "2026-03-01T10:05:00Z",
  "hostname": "web-server-01",
  "files": [
    {
      "file_path": "/var/www/html/index.html",
      "file_size": 1025,
      "md5_hash": "11111111111111111111111111111111",
      "sha256_hash": "2222222222222222222222222222222222222222222222222222222222222222"
    },
    {
      "file_path": "/etc/nginx/nginx.conf",
      "file_size": 512,
      "md5_hash": "a1a2a3a4a5a6a7a8a9a0b1b2b3b4b5b6",
      "sha256_hash": "b0c1c2c3c4c5c6c7c8c9d0d1d2d3d4d5e6e7e8e9f0f1f2f3f4f5f6f7f8f9g0g1"
    }
  ]
}
```

```python
def ingest_hash_report(conn, report_data):
    cursor = conn.cursor()

    # 1. Insert into scan_snapshots table
    cursor.execute('''
        INSERT INTO scan_snapshots (timestamp, hostname)
        VALUES (?, ?)
    ''', (report_data['timestamp'], report_data['hostname']))
    snapshot_id = cursor.lastrowid

    # 2. Insert into file_checksums
    for file_data in report_data.get('files', []):
        cursor.execute('''
            INSERT INTO file_checksums (snapshot_id, file_path, file_size, md5_hash, sha256_hash, sha512_hash)
            VALUES (?, ?, ?, ?, ?, ?)
        ''', (snapshot_id, file_data['file_path'], file_data.get('file_size'),
              file_data.get('md5_hash'), file_data.get('sha256_hash'), file_data.get('sha512_hash')))

    conn.commit()
    print(f"Hash report for {report_data['hostname']} at {report_data['timestamp']} (ID: {snapshot_id}) ingested successfully.")

# Example Main function for ingestion:
# if __name__ == '__main__':
#     DB_FILE = 'crypto_audits.db'
#     if os.path.exists(DB_FILE):
#         os.remove(DB_FILE)
#     conn = sqlite3.connect(DB_FILE)
#     create_schema(conn)

#     with open('file_hash_report_server1_t1.json', 'r') as f:
#         data1 = json.load(f)
#     ingest_hash_report(conn, data1)

#     with open('file_hash_report_server1_t2.json', 'r') as f:
#         data2 = json.load(f)
#     ingest_hash_report(conn, data2)
    
#     conn.close()
```

## Basic SQL Queries for Cryptographic Data Analysis

Once data is in SQLite, you can perform powerful analyses using the `sqlite3` command-line tool.

```bash
sqlite3 crypto_audits.db
```
*(All following SQL commands are executed within the `sqlite3` prompt)*

Assume:
*   `snapshot_id = 1`: `web-server-01` (first scan)
*   `snapshot_id = 2`: `web-server-01` (second scan)

### 1. List All Files and Their Hashes from the Latest Scan for a Host

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
LIMIT 2; -- Adjust limit to show all from latest scan
```

### 2. Identify Files That Have Changed Between Two Specific Snapshots

Here, we compare `snapshot_id=1` and `snapshot_id=2` for `web-server-01`.

```sql
SELECT
    fc1.file_path,
    fc1.sha256_hash AS sha256_t1,
    fc2.sha256_hash AS sha256_t2
FROM
    file_checksums fc1
JOIN
    file_checksums fc2 ON fc1.file_path = fc2.file_path
WHERE
    fc1.snapshot_id = 1 AND fc2.snapshot_id = 2
    AND fc1.sha256_hash != fc2.sha256_hash;
```

### 3. Find All Files with a Specific SHA256 Hash

```sql
SELECT
    ss.hostname,
    ss.timestamp,
    fc.file_path
FROM
    file_checksums fc
JOIN
    scan_snapshots ss ON fc.snapshot_id = ss.snapshot_id
WHERE
    fc.sha256_hash = 'e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855';
```

### 4. Identify Duplicate Files Across All Scans (by SHA256)

```sql
SELECT
    fc.sha256_hash,
    GROUP_CONCAT(ss.hostname || ':' || fc.file_path) AS locations,
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

### 5. Count the Number of Files Scanned per Host per Snapshot

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

## Guiding Principles with SQLite

*   **Portability:** The SQLite database file (`.db`) is fully portable. Standard SQL is used for queries.
*   **Efficiency:** SQLite is highly optimized for storage and retrieval, making it suitable for managing large volumes of file integrity data. Indexes are crucial for query performance.
*   **Minimal Dependencies:** Python's built-in `sqlite3` module and the `sqlite3` CLI tool are standard components.
*   **CLI-centric:** The `sqlite3` command-line tool provides a robust interface for interacting with the database.
*   **Structured Data & Actionable Output:** Storing cryptographic data in a relational database transforms raw outputs into structured, queryable information, enabling powerful FIM, change detection, and audit capabilities.

## Conclusion

SQLite is an exceptionally valuable tool for persisting and analyzing the results of cryptographic operations, particularly for file integrity monitoring. It transforms ephemeral hash values into a rich historical dataset, enabling proactive security monitoring, forensic analysis, and ensuring data trustworthiness. By integrating SQLite with your chosen scripting language for data collection and ingestion, you build a comprehensive, efficient, and cross-platform file integrity monitoring framework. The next step is to apply this knowledge in practical exercises.
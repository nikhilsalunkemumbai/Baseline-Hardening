# SQLite Exercise: File System Integrity Analysis with Baselines

## Objective

This exercise challenges you to apply your SQL querying skills using SQLite to store, manage, and analyze file system integrity baselines. You will create an SQLite database schema, ingest provided JSON baseline data, and then perform various SQL queries to identify modified, new, and deleted files between different snapshots, demonstrating proficiency in file integrity monitoring (FIM).

## Framework Alignment

This exercise on "**File System Search and Integrity Check**" using **SQLite** provides practical experience in building and utilizing tools for a "Cross-Platform Baseline Hardening & Auditing Framework." By completing these tasks, you will learn how to audit and manage file system integrity, ensuring that critical system files and configurations have not been unauthorizedly modified—an essential step in maintaining a secure and auditable environment.


## Scenario

You are a security analyst responsible for monitoring critical system directories for unauthorized changes. You have implemented a system that periodically generates file integrity baselines in JSON format. Your task is to consolidate these baselines into an SQLite database and use SQL to report on changes over time, specifically detecting additions, deletions, and modifications.

## Input Data

You will be working with the following JSON baseline files, located in the same directory as this exercise:

*   `baseline_v1.json`: Represents the initial "known good" state of a `test_dir` directory.
*   `baseline_v2.json`: Represents a later state of the `test_dir` directory, after some simulated changes.

## Setup

You will need an SQLite database. You will use a Python script to perform the setup and ingestion.

### 1. Create `ingest_baselines.py`

Create a Python script named `ingest_baselines.py` in the same directory as the exercise and JSON files. This script will create the necessary database schema and ingest the JSON data.

```python
import sqlite3
import json
import os
from datetime import datetime

DB_FILE = 'fim_audits.db'
BASELINE_FILES = [
    'baseline_v1.json',
    'baseline_v2.json'
]

def create_schema(conn):
    cursor = conn.cursor()

    cursor.execute('''
        CREATE TABLE IF NOT EXISTS baselines (
            baseline_id INTEGER PRIMARY KEY AUTOINCREMENT,
            timestamp TEXT NOT NULL,
            root_path TEXT NOT NULL,
            algorithm TEXT NOT NULL
        )
    ''')

    cursor.execute('''
        CREATE TABLE IF NOT EXISTS files (
            file_id INTEGER PRIMARY KEY AUTOINCREMENT,
            baseline_id INTEGER NOT NULL,
            path TEXT NOT NULL,
            name TEXT NOT NULL,
            size INTEGER,
            last_modified TEXT,
            hash TEXT NOT NULL,
            FOREIGN KEY (baseline_id) REFERENCES baselines (baseline_id)
        )
    ''')
    
    # Add indexes for faster lookups
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_files_baseline_id ON files (baseline_id)')
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_files_path ON files (path)')
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_files_hash ON files (hash)')

    conn.commit()
    print("Database schema created/updated.")

def ingest_baseline(conn, baseline_data):
    cursor = conn.cursor()

    # Insert into baselines table
    cursor.execute('''
        INSERT INTO baselines (timestamp, root_path, algorithm)
        VALUES (?, ?, ?)
    ''', (baseline_data['timestamp'], baseline_data['root_path'], baseline_data['algorithm']))
    baseline_id = cursor.lastrowid

    # Iterate through file data
    for file_entry in baseline_data['files']:
        cursor.execute('''
            INSERT INTO files (baseline_id, path, name, size, last_modified, hash)
            VALUES (?, ?, ?, ?, ?, ?)
        ''', (baseline_id, file_entry['path'], file_entry['name'], file_entry['size'],
              file_entry['mtime'], file_entry['hash']))

    conn.commit()
    print(f"Baseline for '{baseline_data['root_path']}' at {baseline_data['timestamp']} (ID: {baseline_id}) ingested successfully.")

def main():
    if os.path.exists(DB_FILE):
        os.remove(DB_FILE) # Clean up previous DB for fresh run
    
    conn = sqlite3.connect(DB_FILE)
    create_schema(conn)

    for baseline_file in BASELINE_FILES:
        try:
            with open(baseline_file, 'r') as f:
                json_data = json.load(f)
            ingest_baseline(conn, json_data)
        except FileNotFoundError:
            print(f"Error: Baseline file '{baseline_file}' not found.")
        except json.JSONDecodeError:
            print(f"Error: Malformed JSON in '{baseline_file}'.")
        except Exception as e:
            print(f"An error occurred while processing '{baseline_file}': {e}")
    conn.close()
    print(f"
All baseline files ingested into {DB_FILE}.")

if __name__ == '__main__':
    main()
```
### 2. Run the Ingestion Script

Execute the Python script to create the database and load the data:
```bash
python ingest_baselines.py
```
This will create `fim_audits.db` and populate it.

## Tasks

Using SQL queries within the `sqlite3` prompt, provide the solution for each of the following tasks. Start by opening the database:
```bash
sqlite3 fim_audits.db
```
*(All following SQL commands are executed within the `sqlite3` prompt)*

Assume `baseline_id = 1` refers to `baseline_v1.json` and `baseline_id = 2` refers to `baseline_v2.json`.

1.  **List All Baselines:**
    *   Retrieve the `baseline_id`, `timestamp`, `root_path`, and `algorithm` for all recorded baselines.

2.  **Find All Files in a Specific Baseline:**
    *   List the `path`, `name`, `size`, and `hash` for all files in `baseline_id = 1`.

3.  **Identify Files by Hash:**
    *   Find all files (from any baseline) that have the SHA256 hash `a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0a1b2`.

4.  **Detect Modified Files Between Baselines:**
    *   Find files that exist in both `baseline_id = 1` and `baseline_id = 2` but have different `hash` values. Display their `path`, `hash` from `baseline_id = 1`, and `hash` from `baseline_id = 2`.

5.  **Detect New Files Added:**
    *   Find files that are present in `baseline_id = 2` but were NOT present in `baseline_id = 1`. Display their `path` and `hash`.

6.  **Detect Deleted Files:**
    *   Find files that were present in `baseline_id = 1` but are NO longer present in `baseline_id = 2`. Display their `path` and `hash`.

7.  **Count Files by Extension in Latest Baseline:**
    *   For the latest `baseline_id` (assume `baseline_id = 2` is the latest), count the number of files for each unique file extension (e.g., `.txt`, `.conf`, `.sh`). Files without an extension can be grouped as "no_extension".

## Deliverables

For each task, provide the SQL query and the resulting output.

## Reflection Questions

1.  How does storing file integrity baselines in a relational database (SQLite) facilitate detecting changes over time compared to performing `diff` operations on raw JSON or text files?
2.  Discuss the advantages of normalizing the schema (using separate tables for baselines and files) for managing FIM data.
3.  How would you extend this schema to include information about file permissions, owner, or creation time for more granular auditing?
4.  If you needed to set up an automated FIM system, how could the SQL queries be used by an external script to generate alerts for specific types of changes?
5.  What are the advantages and disadvantages of using SQLite for file integrity analysis compared to a dedicated FIM solution or just simple scripting?

---
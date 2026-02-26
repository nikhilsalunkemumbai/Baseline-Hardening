# SQLite Exercise: Basic User/Group Management and Audit Analysis

## Objective

This exercise challenges you to apply your SQL querying skills using SQLite to store, manage, and analyze historical user and group configurations. You will create an SQLite database schema, ingest provided JSON user/group snapshot data, and then perform various SQL queries to identify changes in user accounts and group memberships, audit for privileged access, and detect policy violations over time.

## Framework Alignment

This exercise on "**User/Group Management and Audit**" using **SQLite** provides practical experience in building and utilizing tools for a "Cross-Platform Baseline Hardening & Auditing Framework." By completing these tasks, you will learn how to audit and manage user accounts and group memberships, ensuring compliance with security policies and identifying unauthorized privilege escalations—essential steps in maintaining a secure and auditable environment.


## Scenario

You are a security auditor responsible for maintaining user access control policies and identifying any deviations from security baselines. You periodically collect snapshots of user and group configurations from critical systems. Your task is to consolidate these snapshots into an SQLite database and use SQL to report on account status, group memberships, and changes in configuration.

## Input Data

You will be working with the following JSON snapshot files, located in the same directory as this exercise:

*   `ug_snapshot_t1.json`: Represents a user/group configuration snapshot from `server-alpha` at an initial time.
*   `ug_snapshot_t2.json`: Represents a user/group configuration snapshot from `server-alpha` at a later time, after some simulated changes (e.g., new user, deleted user, changed group memberships).

## Setup

You will need an SQLite database. You will use a Python script to perform the setup and ingestion.

### 1. Create `ingest_ug_snapshots.py`

Create a Python script named `ingest_ug_snapshots.py` in the same directory as the exercise and JSON files. This script will create the necessary database schema and ingest the JSON data.

```python
import sqlite3
import json
import os
from datetime import datetime

DB_FILE = 'user_group_audits.db'
SNAPSHOT_FILES = [
    'ug_snapshot_t1.json',
    'ug_snapshot_t2.json'
]

def create_schema(conn):
    cursor = conn.cursor()

    cursor.execute('''
        CREATE TABLE IF NOT EXISTS config_snapshots (
            snapshot_id INTEGER PRIMARY KEY AUTOINCREMENT,
            timestamp TEXT NOT NULL,
            hostname TEXT NOT NULL
        )
    ''')

    cursor.execute('''
        CREATE TABLE IF NOT EXISTS users (
            user_id INTEGER PRIMARY KEY AUTOINCREMENT,
            snapshot_id INTEGER NOT NULL,
            username TEXT NOT NULL,
            uid INTEGER,
            gid INTEGER,
            home_dir TEXT,
            shell TEXT,
            account_active INTEGER, -- 1 for active, 0 for disabled/locked
            password_expires TEXT,
            FOREIGN KEY (snapshot_id) REFERENCES config_snapshots (snapshot_id)
        )
    ''')
    
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS groups (
            group_id INTEGER PRIMARY KEY AUTOINCREMENT,
            snapshot_id INTEGER NOT NULL,
            groupname TEXT NOT NULL,
            gid INTEGER,
            FOREIGN KEY (snapshot_id) REFERENCES config_snapshots (snapshot_id)
        )
    ''')

    cursor.execute('''
        CREATE TABLE IF NOT EXISTS group_members (
            member_id INTEGER PRIMARY KEY AUTOINCREMENT,
            snapshot_id INTEGER NOT NULL,
            username TEXT NOT NULL,
            groupname TEXT NOT NULL,
            FOREIGN KEY (snapshot_id) REFERENCES config_snapshots (snapshot_id)
        )
    ''')
    
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_users_snapshot_id ON users (snapshot_id)')
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_users_username ON users (username)')
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_groups_snapshot_id ON groups (snapshot_id)')
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_groups_groupname ON groups (groupname)')
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_group_members_snapshot_id ON group_members (snapshot_id)')
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_group_members_username_groupname ON group_members (username, groupname)')

    conn.commit()
    print("Database schema created/updated.")

def ingest_ug_snapshot(conn, snapshot_data):
    cursor = conn.cursor()

    cursor.execute('''
        INSERT INTO config_snapshots (timestamp, hostname)
        VALUES (?, ?)
    ''', (snapshot_data['timestamp'], snapshot_data['hostname']))
    snapshot_id = cursor.lastrowid

    for user_data in snapshot_data['users']:
        cursor.execute('''
            INSERT INTO users (snapshot_id, username, uid, gid, home_dir, shell, account_active, password_expires)
            VALUES (?, ?, ?, ?, ?, ?, ?, ?)
        ''', (snapshot_id, user_data.get('username'), user_data.get('uid'), user_data.get('gid'),
              user_data.get('home_dir'), user_data.get('shell'), 1 if user_data.get('account_active') else 0,
              user_data.get('password_expires')))

    for group_data in snapshot_data['groups']:
        cursor.execute('''
            INSERT INTO groups (snapshot_id, groupname, gid)
            VALUES (?, ?, ?)
        ''', (snapshot_id, group_data.get('groupname'), group_data.get('gid')))
        
        for member_username in group_data.get('members', []):
            cursor.execute('''
                INSERT INTO group_members (snapshot_id, username, groupname)
                VALUES (?, ?, ?)
            ''', (snapshot_id, member_username, group_data.get('groupname')))

    conn.commit()
    print(f"User/Group snapshot for {snapshot_data['hostname']} at {snapshot_data['timestamp']} (ID: {snapshot_id}) ingested successfully.")

def main():
    if os.path.exists(DB_FILE):
        os.remove(DB_FILE)
    
    conn = sqlite3.connect(DB_FILE)
    create_schema(conn)

    for snapshot_file in SNAPSHOT_FILES:
        try:
            with open(snapshot_file, 'r') as f:
                json_data = json.load(f)
            ingest_ug_snapshot(conn, json_data)
        except FileNotFoundError:
            print(f"Error: Snapshot file '{snapshot_file}' not found.")
        except json.JSONDecodeError:
            print(f"Error: Malformed JSON in '{snapshot_file}'.")
        except Exception as e:
            print(f"An error occurred while processing '{snapshot_file}': {e}")
    conn.close()
    print(f"
All user/group snapshots ingested into {DB_FILE}.")

if __name__ == '__main__':
    main()
```
### 2. Run the Ingestion Script

Execute the Python script to create the database and load the data:
```bash
python ingest_ug_snapshots.py
```
This will create `user_group_audits.db` and populate it.

## Tasks

Using SQL queries within the `sqlite3` prompt, provide the solution for each of the following tasks. Start by opening the database:
```bash
sqlite3 user_group_audits.db
```
*(All following SQL commands are executed within the `sqlite3` prompt)*

Assume `snapshot_id = 1` refers to `ug_snapshot_t1.json` and `snapshot_id = 2` refers to `ug_snapshot_t2.json`.

1.  **List All Configuration Snapshots:**
    *   Retrieve the `snapshot_id`, `timestamp`, and `hostname` for all recorded configuration snapshots.

2.  **List All Users from the Latest Snapshot (`snapshot_id = 2`):**
    *   Retrieve the `username`, `uid`, `home_dir`, and `account_active` status for all users in `snapshot_id = 2`, ordered by `username`.

3.  **Identify Privileged Users (UID 0):**
    *   List the `username` and `uid` of all users who have `uid = 0` in any snapshot, along with the `hostname` and `timestamp` of that snapshot.

4.  **Find Users in the 'developers' Group in the Latest Snapshot:**
    *   List the `username` of all users who are members of the 'developers' group in `snapshot_id = 2`.

5.  **Detect New Users Added:**
    *   Identify users who are present in `snapshot_id = 2` but were NOT present in `snapshot_id = 1`. Display their `username`, `uid`, and `hostname`.

6.  **Detect Deleted Users:**
    *   Identify users who were present in `snapshot_id = 1` but are NO longer present in `snapshot_id = 2`. Display their `username`, `uid`, and `hostname`.

7.  **Detect Changes in Group Membership for 'user1':**
    *   List all `groupname`s that 'user1' was a member of across all snapshots, along with the `timestamp` of the snapshot. Order chronologically.

8.  **Count Active vs. Disabled Users in Latest Snapshot:**
    *   For `snapshot_id = 2`, count the number of users where `account_active` is 1 (active) and where it is 0 (disabled).

## Deliverables

For each task, provide the SQL query and the resulting output.

## Reflection Questions

1.  How does storing user and group configuration snapshots in a relational database (SQLite) facilitate tracking changes and auditing compared to managing individual text or JSON files?
2.  Discuss the advantages of normalizing the schema (using separate tables for snapshots, users, groups, and group_members) for managing user/group audit data.
3.  If you needed to identify a user whose primary GID changed between two snapshots, what kind of SQL query would you construct?
4.  How could this SQLite database be used in an automated system to alert on unauthorized user creations, deletions, or privilege escalations?
5.  What are the advantages and disadvantages of using SQLite for historical user/group analysis compared to directly querying system files or using a directory service like Active Directory?

---
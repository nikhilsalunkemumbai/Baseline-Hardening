# SQLite Tutorial: Basic User/Group Management and Audit Analysis

## Introduction

While Bash, PowerShell, and Python are excellent for *collecting* and *managing* user and group accounts, SQLite provides an invaluable solution for persistently *storing* and *querying* historical user and group configurations. By centralizing user/group snapshots in an SQLite database, you can perform long-term auditing, track access control changes, identify policy deviations, and detect suspicious account activity with the full power of SQL. This tutorial will guide you through designing an SQLite schema for user/group data and querying that data effectively.

## Framework Alignment

This tutorial on "**User/Group Management and Audit**" using **SQLite** demonstrates the foundational skills necessary to implement components of a "Cross-Platform Baseline Hardening & Auditing Framework." The techniques learned here for storing and querying user account and group membership data are directly applicable to auditing systems against defined security baselines and ensuring compliance.


## Why SQLite for User/Group Auditing?

*   **Historical Tracking:** Store multiple snapshots of user/group configurations over time to track changes in permissions, new accounts, deleted accounts, or altered group memberships.
*   **Centralized Repository:** Consolidate user/group data from various systems (if collected remotely) into a single, queryable database.
*   **Powerful SQL Analysis:** Use SQL to filter users/groups by various criteria, identify privileged accounts, find orphaned accounts, and generate custom audit reports.
*   **Compliance & Security:** Easily audit configurations against security policies (e.g., no users with UID 0 other than root, all users in a specific group).
*   **Portable & Serverless:** The entire database is a single file, easily transferable and usable without a dedicated database server.
*   **Minimal Dependencies:** The `sqlite3` command-line tool and Python's `sqlite3` module are standard.

## Workflow: From Collection to SQLite

The typical workflow involves using a scripting language to:
1.  **Collect User/Group Data:** Use Bash, PowerShell, or Python scripts (as demonstrated in previous tutorials) to gather user and group lists and their metadata.
2.  **Structure Data:** Format the collected data into a structured format (e.g., JSON).
3.  **Ingest into SQLite:** Use a script (e.g., Python with `sqlite3` module) to read the structured data and insert it into an SQLite database.

For this tutorial, we will focus on the SQLite schema design and querying. We'll provide examples of how you might ingest data from a Python script that generates JSON user/group reports.

## Schema Design for User/Group Auditing

A robust schema will typically involve several tables: one for the snapshots themselves, one for users, one for groups, and a linking table for user-to-group memberships.

### Proposed Schema

1.  **`config_snapshots` Table:** Stores metadata about each configuration snapshot (when it was taken, from which host).
2.  **`users` Table:** Stores details about each user account observed in a snapshot.
3.  **`groups` Table:** Stores details about each group observed in a snapshot.
4.  **`group_members` Table:** Links users to the groups they are members of for a specific snapshot.

### 1. Creating the Database and Tables

We'll use a Python script to create the database and tables.

```python
import sqlite3
import json
from datetime import datetime

DB_FILE = 'user_group_audits.db'

def create_schema(conn):
    cursor = conn.cursor()

    # config_snapshots table: stores metadata for each configuration snapshot
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS config_snapshots (
            snapshot_id INTEGER PRIMARY KEY AUTOINCREMENT,
            timestamp TEXT NOT NULL,
            hostname TEXT NOT NULL
        )
    ''')

    # users table: stores details for each user within a snapshot
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS users (
            user_id INTEGER PRIMARY KEY AUTOINCREMENT,
            snapshot_id INTEGER NOT NULL,
            username TEXT NOT NULL,
            uid INTEGER,
            gid INTEGER, -- Primary GID
            home_dir TEXT,
            shell TEXT,
            account_active INTEGER, -- 1 for active, 0 for disabled/locked
            password_expires TEXT,
            FOREIGN KEY (snapshot_id) REFERENCES config_snapshots (snapshot_id)
        )
    ''')
    
    # groups table: stores details for each group within a snapshot
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS groups (
            group_id INTEGER PRIMARY KEY AUTOINCREMENT,
            snapshot_id INTEGER NOT NULL,
            groupname TEXT NOT NULL,
            gid INTEGER,
            FOREIGN KEY (snapshot_id) REFERENCES config_snapshots (snapshot_id)
        )
    ''')

    # group_members table: links users to groups
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS group_members (
            member_id INTEGER PRIMARY KEY AUTOINCREMENT,
            snapshot_id INTEGER NOT NULL,
            username TEXT NOT NULL,
            groupname TEXT NOT NULL,
            FOREIGN KEY (snapshot_id) REFERENCES config_snapshots (snapshot_id)
        )
    ''')
    
    # Add indexes for faster lookups
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_users_snapshot_id ON users (snapshot_id)')
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_users_username ON users (username)')
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_groups_snapshot_id ON groups (snapshot_id)')
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_groups_groupname ON groups (groupname)')
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_group_members_snapshot_id ON group_members (snapshot_id)')
    cursor.execute('CREATE INDEX IF NOT EXISTS idx_group_members_username_groupname ON group_members (username, groupname)')

    conn.commit()
    print("Database schema created/updated.")

# Example Usage:
# conn = sqlite3.connect(DB_FILE)
# create_schema(conn)
# conn.close()
```

### 2. Data Ingestion (Python Example)

Assuming you have JSON user/group snapshot files (`ug_snapshot_t1.json`, `ug_snapshot_t2.json`) generated by an auditing script.

**`ug_snapshot_t1.json` (simplified example):**
```json
{
  "timestamp": "2026-03-01T10:00:00Z",
  "hostname": "server-alpha",
  "users": [
    {
      "username": "root",
      "uid": 0,
      "gid": 0,
      "home_dir": "/root",
      "shell": "/bin/bash",
      "account_active": true
    },
    {
      "username": "user1",
      "uid": 1001,
      "gid": 1001,
      "home_dir": "/home/user1",
      "shell": "/bin/bash",
      "account_active": true
    },
    {
      "username": "guest",
      "uid": 1002,
      "gid": 1002,
      "home_dir": "/home/guest",
      "shell": "/bin/bash",
      "account_active": false
    }
  ],
  "groups": [
    {
      "groupname": "root",
      "gid": 0,
      "members": ["root"]
    },
    {
      "groupname": "user1",
      "gid": 1001,
      "members": ["user1"]
    },
    {
      "groupname": "developers",
      "gid": 2000,
      "members": ["user1"]
    }
  ]
}
```

**`ug_snapshot_t2.json` (user 'admin' added, 'guest' deleted, 'user1' added to 'ops'):**
```json
{
  "timestamp": "2026-03-01T11:00:00Z",
  "hostname": "server-alpha",
  "users": [
    {
      "username": "root",
      "uid": 0,
      "gid": 0,
      "home_dir": "/root",
      "shell": "/bin/bash",
      "account_active": true
    },
    {
      "username": "user1",
      "uid": 1001,
      "gid": 1001,
      "home_dir": "/home/user1",
      "shell": "/bin/bash",
      "account_active": true
    },
    {
      "username": "admin",
      "uid": 1003,
      "gid": 1003,
      "home_dir": "/home/admin",
      "shell": "/bin/bash",
      "account_active": true
    }
  ],
  "groups": [
    {
      "groupname": "root",
      "gid": 0,
      "members": ["root"]
    },
    {
      "groupname": "user1",
      "gid": 1001,
      "members": ["user1"]
    },
    {
      "groupname": "developers",
      "gid": 2000,
      "members": ["user1"]
    },
    {
      "groupname": "ops",
      "gid": 2001,
      "members": ["admin", "user1"]
    }
  ]
}
```

```python
def ingest_ug_snapshot(conn, snapshot_data):
    cursor = conn.cursor()

    # Insert into config_snapshots table
    cursor.execute('''
        INSERT INTO config_snapshots (timestamp, hostname)
        VALUES (?, ?)
    ''', (snapshot_data['timestamp'], snapshot_data['hostname']))
    snapshot_id = cursor.lastrowid

    # Iterate through users
    for user_data in snapshot_data['users']:
        cursor.execute('''
            INSERT INTO users (snapshot_id, username, uid, gid, home_dir, shell, account_active, password_expires)
            VALUES (?, ?, ?, ?, ?, ?, ?, ?)
        ''', (snapshot_id, user_data.get('username'), user_data.get('uid'), user_data.get('gid'),
              user_data.get('home_dir'), user_data.get('shell'), 1 if user_data.get('account_active') else 0,
              user_data.get('password_expires')))

    # Iterate through groups
    for group_data in snapshot_data['groups']:
        cursor.execute('''
            INSERT INTO groups (snapshot_id, groupname, gid)
            VALUES (?, ?, ?)
        ''', (snapshot_id, group_data.get('groupname'), group_data.get('gid')))
        
        # Insert group members
        for member_username in group_data.get('members', []):
            cursor.execute('''
                INSERT INTO group_members (snapshot_id, username, groupname)
                VALUES (?, ?, ?)
            ''', (snapshot_id, member_username, group_data.get('groupname')))

    conn.commit()
    print(f"User/Group snapshot for {snapshot_data['hostname']} at {snapshot_data['timestamp']} (ID: {snapshot_id}) ingested successfully.")

# Example Main function for ingestion:
# if __name__ == '__main__':
#     DB_FILE = 'user_group_audits.db'
#     if os.path.exists(DB_FILE):
#         os.remove(DB_FILE)
#     conn = sqlite3.connect(DB_FILE)
#     create_schema(conn)

#     # Ingest ug_snapshot_t1.json
#     with open('ug_snapshot_t1.json', 'r') as f:
#         data1 = json.load(f)
#     ingest_ug_snapshot(conn, data1)

#     # Ingest ug_snapshot_t2.json
#     with open('ug_snapshot_t2.json', 'r') as f:
#         data2 = json.load(f)
#     ingest_ug_snapshot(conn, data2)
    
#     conn.close()
```

## Basic SQL Queries for User/Group Audit

Once data is in SQLite, you can perform powerful analyses using the `sqlite3` command-line tool.

```bash
sqlite3 user_group_audits.db
```
*(All following SQL commands are executed within the `sqlite3` prompt)*

Assume `snapshot_id = 1` is for `ug_snapshot_t1.json` and `snapshot_id = 2` is for `ug_snapshot_t2.json`.

### 1. List All Config Snapshots

```sql
SELECT snapshot_id, timestamp, hostname FROM config_snapshots;
```

### 2. List All Users from a Specific Snapshot (`snapshot_id = 1`)

```sql
SELECT username, uid, gid, home_dir, shell, account_active
FROM users
WHERE snapshot_id = 1
ORDER BY username;
```

### 3. Identify Privileged Users (UID 0) in Any Snapshot

```sql
SELECT DISTINCT cs.hostname, cs.timestamp, u.username, u.uid
FROM config_snapshots cs
JOIN users u ON cs.snapshot_id = u.snapshot_id
WHERE u.uid = 0;
```

### 4. Find Users in a Specific Group in the Latest Snapshot (`snapshot_id = 2`)

```sql
SELECT gm.username, gm.groupname
FROM group_members gm
WHERE gm.snapshot_id = 2 AND gm.groupname = 'developers';
```

### 5. Detect New Users Added Between Two Snapshots

```sql
-- Users present in snapshot 2 that were not in snapshot 1
SELECT
    u2.username,
    u2.uid
FROM
    users u2
WHERE
    u2.snapshot_id = 2
    AND NOT EXISTS (
        SELECT 1
        FROM users u1
        WHERE u1.snapshot_id = 1
          AND u1.username = u2.username
    );
```

### 6. Detect Deleted Users Between Two Snapshots

```sql
-- Users present in snapshot 1 that are no longer in snapshot 2
SELECT
    u1.username,
    u1.uid
FROM
    users u1
WHERE
    u1.snapshot_id = 1
    AND NOT EXISTS (
        SELECT 1
        FROM users u2
        WHERE u2.snapshot_id = 2
          AND u2.username = u1.username
    );
```

### 7. Detect Changes in Group Membership for a Specific User (e.g., 'user1')

```sql
SELECT
    cs.timestamp,
    gm.groupname
FROM
    config_snapshots cs
JOIN
    group_members gm ON cs.snapshot_id = gm.snapshot_id
WHERE
    gm.username = 'user1'
ORDER BY
    cs.timestamp, gm.groupname;
```

### 8. Identify Disabled Accounts in the Latest Snapshot (`snapshot_id = 2`)

```sql
SELECT username, account_active
FROM users
WHERE snapshot_id = 2 AND account_active = 0;
```

## Guiding Principles with SQLite

*   **Portability:** The SQLite database file (`.db`) is fully portable. The SQL queries are standard.
*   **Efficiency:** SQLite is highly optimized for read operations. Proper indexing ensures fast querying even with many snapshots and users/groups.
*   **Minimal Dependencies:** Python's built-in `sqlite3` module and the `sqlite3` CLI tool are standard components.
*   **CLI-centric:** The `sqlite3` command-line tool provides a robust interface for interacting with the database.
*   **Structured Data & Actionable Output:** Storing user/group configuration in a relational database transforms ephemeral system state into structured, queryable data, enabling powerful historical analysis, compliance auditing, and security investigations.

## Conclusion

SQLite provides a robust and indispensable component for managing and analyzing historical user and group configuration data. It moves beyond real-time snapshots to long-term trend analysis, allowing you to track access control changes, identify privileged users, detect new/deleted accounts, and audit group memberships. By integrating SQLite with your chosen scripting language for data collection and ingestion, you create a comprehensive and flexible user/group management and auditing framework. The next step is to apply this knowledge in practical exercises.
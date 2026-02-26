# SQLite Tutorial: System Information Snapshot and Reporting

## Introduction

While Bash, PowerShell, and Python are excellent for *collecting* system information, SQLite provides a robust, lightweight, and incredibly versatile solution for *storing*, *managing*, and *querying* these system snapshots. By centralizing collected system data in an SQLite database, you can perform historical trend analysis, auditing, compliance checks, and cross-system comparisons with the full power of SQL. This tutorial focuses on how to design an SQLite schema for system information and how to query that data effectively.

## Framework Alignment

This tutorial on "**System Information Snapshot and Reporting**" using **SQLite** demonstrates the foundational skills necessary to implement components of a "Cross-Platform Baseline Hardening & Auditing Framework." The techniques learned here for storing and querying system configuration data are directly applicable to auditing systems against defined security baselines and ensuring compliance.


## Why SQLite for System Information Snapshots?

*   **Historical Data:** Store multiple snapshots over time to track changes in configuration, resource usage, or installed software.
*   **Powerful Querying:** Use SQL to filter, aggregate, and join system data in ways that are difficult with flat files.
*   **Auditing & Compliance:** Easily query for systems not meeting specific configurations or security baselines.
*   **Cross-System Comparison:** Aggregate data from multiple systems into a single database for comparative analysis.
*   **Portable & Serverless:** The entire database is a single file, easily transferable and usable without a dedicated database server.
*   **Minimal Dependencies:** The `sqlite3` command-line tool and Python's `sqlite3` module are standard.

## Workflow: From Collection to SQLite

The typical workflow involves using a scripting language to:
1.  **Collect System Info:** Use Bash, PowerShell, or Python scripts (as demonstrated in previous tutorials) to gather system data.
2.  **Structure Data:** Format the collected data into a structured format (e.g., JSON, CSV).
3.  **Ingest into SQLite:** Use a script (e.g., Python with `sqlite3` module) to read the structured data and insert it into an SQLite database.

For this tutorial, we will focus on the SQLite schema design and querying. We'll provide examples of how you might ingest data that has been output as JSON by a Python script, for instance.

## Schema Design for System Information

A robust schema will involve multiple tables to normalize the data and prevent redundancy, especially for items that appear multiple times (like network interfaces or disk partitions).

We'll consider a simplified schema focusing on key aspects.

### Proposed Schema

1.  **`snapshots` Table:** Stores metadata about each snapshot (when it was taken, from which host).
2.  **`os_info` Table:** Stores operating system details.
3.  **`cpu_info` Table:** Stores CPU details.
4.  **`memory_info` Table:** Stores memory details.
5.  **`disks` Table:** Stores information about logical disks/partitions.
6.  **`network_interfaces` Table:** Stores details about each network interface.
7.  **`processes` Table:** Stores a list of running processes for a given snapshot.

### 1. Creating the Database and Tables

Let's assume our Python script (from the previous tutorial) collects system information and outputs it as JSON. We'll use a Python script to create the database and tables.

```python
import sqlite3
import json
from datetime import datetime

DB_FILE = 'system_snapshots.db'

def create_schema():
    conn = sqlite3.connect(DB_FILE)
    cursor = conn.cursor()

    # snapshots table
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS snapshots (
            snapshot_id INTEGER PRIMARY KEY AUTOINCREMENT,
            timestamp TEXT NOT NULL,
            hostname TEXT NOT NULL UNIQUE
        )
    ''')

    # os_info table (one-to-one with snapshots, or one-to-many if tracking OS changes per host)
    # For simplicity, we'll link directly to snapshot_id
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS os_info (
            os_info_id INTEGER PRIMARY KEY AUTOINCREMENT,
            snapshot_id INTEGER,
            system TEXT,
            node_name TEXT,
            release TEXT,
            version TEXT,
            machine TEXT,
            processor TEXT,
            os_name TEXT,
            architecture TEXT,
            distro_name TEXT,
            distro_version TEXT,
            FOREIGN KEY (snapshot_id) REFERENCES snapshots (snapshot_id)
        )
    ''')

    # cpu_info table
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS cpu_info (
            cpu_info_id INTEGER PRIMARY KEY AUTOINCREMENT,
            snapshot_id INTEGER,
            model_name TEXT,
            cpus_total INTEGER,
            cores_per_socket INTEGER,
            architecture TEXT,
            FOREIGN KEY (snapshot_id) REFERENCES snapshots (snapshot_id)
        )
    ''')

    # memory_info table
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS memory_info (
            memory_info_id INTEGER PRIMARY KEY AUTOINCREMENT,
            snapshot_id INTEGER,
            total_memory TEXT,
            used_memory TEXT,
            free_memory TEXT,
            FOREIGN KEY (snapshot_id) REFERENCES snapshots (snapshot_id)
        )
    ''')

    # disks table (one-to-many with snapshots)
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS disks (
            disk_id INTEGER PRIMARY KEY AUTOINCREMENT,
            snapshot_id INTEGER,
            filesystem TEXT,
            type TEXT,
            size TEXT,
            used TEXT,
            available TEXT,
            mount_point TEXT,
            FOREIGN KEY (snapshot_id) REFERENCES snapshots (snapshot_id)
        )
    ''')
    
    # network_interfaces table (one-to-many with snapshots)
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS network_interfaces (
            interface_id INTEGER PRIMARY KEY AUTOINCREMENT,
            snapshot_id INTEGER,
            name TEXT,
            mac_address TEXT,
            state TEXT,
            ipv4_addresses TEXT, -- Storing as JSON string or comma-separated for simplicity
            ipv6_addresses TEXT, -- Storing as JSON string or comma-separated for simplicity
            FOREIGN KEY (snapshot_id) REFERENCES snapshots (snapshot_id)
        )
    ''')

    # processes table (one-to-many with snapshots)
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS processes (
            process_id INTEGER PRIMARY KEY AUTOINCREMENT,
            snapshot_id INTEGER,
            user TEXT,
            pid INTEGER,
            cpu_percent REAL,
            mem_percent REAL,
            command TEXT,
            FOREIGN KEY (snapshot_id) REFERENCES snapshots (snapshot_id)
        )
    ''')
    
    # Add an index to the snapshots table for faster hostname lookups
    cursor.execute('CREATE UNIQUE INDEX IF NOT EXISTS idx_hostname ON snapshots (hostname)')

    conn.commit()
    conn.close()
    print(f"Database schema created/updated in {DB_FILE}")

# Example Usage:
# create_schema()
```

### 2. Data Ingestion (Python Example)

Assuming you have a JSON file (`snapshot.json`) output by the Python `sys_snapshot.py` script:

```json
{
  "timestamp": "2026-02-25T16:00:00.123456",
  "system_info": {
    "os": {
      "system": "Linux",
      "node_name": "my-host",
      "release": "5.15.0-89-generic",
      "version": "#99-Ubuntu SMP Mon Nov 6 16:09:41 UTC 2023",
      "machine": "x86_64",
      "processor": "x86_64",
      "os_name": "Linux-5.15.0-89-generic-x86_64-with-glibc2.35",
      "architecture": "64bit",
      "name": "Ubuntu",
      "pretty_name": "Ubuntu 22.04.3 LTS",
      "id": "ubuntu"
    },
    "cpu": {
      "model_name": "Intel(R) Core(TM) i7-8700K CPU @ 3.70GHz",
      "cpus_total": 12,
      "cores_per_socket": 6,
      "architecture": "x86_64",
      "sockets": 2
    },
    "memory": {
      "total_memory": "31G",
      "used_memory": "11G",
      "free_memory": "18G"
    },
    "disks": [
      {
        "filesystem": "/dev/sda1",
        "type": "ext4",
        "size": "450G",
        "used": "150G",
        "available": "280G",
        "mount_point": "/"
      }
    ],
    "network": [
      {
        "name": "eth0",
        "mac_address": "00:11:22:33:44:55",
        "state": "UP",
        "ipv4_addresses": ["192.168.1.100"],
        "ipv6_addresses": []
      }
    ],
    "processes": [
        {"user": "root", "pid": "1", "cpu_percent": "0.0", "mem_percent": "0.1", "command": "/sbin/init"},
        {"user": "user", "pid": "1234", "cpu_percent": "1.5", "mem_percent": "2.3", "command": "firefox"}
    ],
    "uptime": {"uptime": "1 day, 5 hours, 30 minutes"},
    "logged_in_users": [{"user": "user", "tty": "pts/0", "login_time": "Feb 25 10:00"}]
  }
}
```

```python
def ingest_snapshot(json_data):
    conn = sqlite3.connect(DB_FILE)
    cursor = conn.cursor()

    snapshot_ts = json_data['timestamp']
    hostname = json_data['system_info']['os']['node_name']

    # Insert into snapshots table
    cursor.execute('INSERT OR REPLACE INTO snapshots (timestamp, hostname) VALUES (?, ?)',
                   (snapshot_ts, hostname))
    snapshot_id = cursor.lastrowid # Get the ID of the newly inserted snapshot

    # Insert into os_info
    os_data = json_data['system_info']['os']
    cursor.execute('''
        INSERT INTO os_info (snapshot_id, system, node_name, release, version, machine, processor, os_name, architecture, distro_name, distro_version)
        VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
    ''', (snapshot_id, os_data.get('system'), os_data.get('node_name'), os_data.get('release'),
          os_data.get('version'), os_data.get('machine'), os_data.get('processor'),
          os_data.get('os_name'), os_data.get('architecture'), os_data.get('name'), os_data.get('version')))

    # Insert into cpu_info
    cpu_data = json_data['system_info']['cpu']
    cursor.execute('''
        INSERT INTO cpu_info (snapshot_id, model_name, cpus_total, cores_per_socket, architecture)
        VALUES (?, ?, ?, ?, ?)
    ''', (snapshot_id, cpu_data.get('model_name'), cpu_data.get('cpus_total'),
          cpu_data.get('cores_per_socket'), cpu_data.get('architecture')))
          
    # Insert into memory_info
    mem_data = json_data['system_info']['memory']
    cursor.execute('''
        INSERT INTO memory_info (snapshot_id, total_memory, used_memory, free_memory)
        VALUES (?, ?, ?, ?)
    ''', (snapshot_id, mem_data.get('total_memory'), mem_data.get('used_memory'), mem_data.get('free_memory')))

    # Insert into disks
    for disk in json_data['system_info']['disks']:
        cursor.execute('''
            INSERT INTO disks (snapshot_id, filesystem, type, size, used, available, mount_point)
            VALUES (?, ?, ?, ?, ?, ?, ?)
        ''', (snapshot_id, disk.get('filesystem'), disk.get('type'), disk.get('size'),
              disk.get('used'), disk.get('available'), disk.get('mount_point')))
              
    # Insert into network_interfaces
    for net_if in json_data['system_info']['network']:
        cursor.execute('''
            INSERT INTO network_interfaces (snapshot_id, name, mac_address, state, ipv4_addresses, ipv6_addresses)
            VALUES (?, ?, ?, ?, ?, ?)
        ''', (snapshot_id, net_if.get('name'), net_if.get('mac_address'), net_if.get('state'),
              json.dumps(net_if.get('ipv4_addresses', [])), json.dumps(net_if.get('ipv6_addresses', [])))) # Store lists as JSON strings

    # Insert into processes
    for proc in json_data['system_info']['processes']:
        cursor.execute('''
            INSERT INTO processes (snapshot_id, user, pid, cpu_percent, mem_percent, command)
            VALUES (?, ?, ?, ?, ?, ?)
        ''', (snapshot_id, proc.get('user'), proc.get('pid'), proc.get('cpu_percent'),
              proc.get('mem_percent'), proc.get('command')))

    conn.commit()
    conn.close()
    print(f"Snapshot for {hostname} at {snapshot_ts} ingested successfully.")

# Example Usage:
# if __name__ == '__main__':
#     create_schema()
#     with open('snapshot.json', 'r') as f:
#         sample_snapshot_data = json.load(f)
#     ingest_snapshot(sample_snapshot_data)
```

### 3. Querying System Information

Now, let's use the `sqlite3` command-line tool to query the data.

```bash
sqlite3 system_snapshots.db
```
*(All following SQL commands are executed within the `sqlite3` prompt)*

### a. Retrieve OS Information for a Host

```sql
SELECT
    s.hostname,
    o.system,
    o.os_name,
    o.version,
    o.architecture
FROM
    snapshots s
JOIN
    os_info o ON s.snapshot_id = o.snapshot_id
WHERE
    s.hostname = 'my-host';
```

### b. Find Systems with Low Free Memory (e.g., < 20GB)

This requires some string parsing in SQL if memory is stored as TEXT with units, or converting units during ingestion. Assuming `free_memory` is stored as `XXG`:

```sql
SELECT
    s.hostname,
    m.total_memory,
    m.free_memory
FROM
    snapshots s
JOIN
    memory_info m ON s.snapshot_id = m.snapshot_id
WHERE
    CAST(REPLACE(m.free_memory, 'G', '') AS INTEGER) < 20;
-- Note: REPLACE and CAST might not be robust for all string formats (e.g., "1.5G"). Better to store as INTEGER/REAL in ingestion.
```

### c. List Disks and Their Usage for a Specific Host

```sql
SELECT
    s.hostname,
    d.filesystem,
    d.mount_point,
    d.size,
    d.used,
    d.available
FROM
    snapshots s
JOIN
    disks d ON s.snapshot_id = d.snapshot_id
WHERE
    s.hostname = 'my-host';
```

### d. Count Systems by OS Distribution

```sql
SELECT
    o.distro_name,
    COUNT(DISTINCT s.hostname) AS system_count
FROM
    snapshots s
JOIN
    os_info o ON s.snapshot_id = o.snapshot_id
GROUP BY
    o.distro_name;
```

### e. Find Processes Running on a Specific Host

```sql
SELECT
    s.hostname,
    p.user,
    p.pid,
    p.command
FROM
    snapshots s
JOIN
    processes p ON s.snapshot_id = p.snapshot_id
WHERE
    s.hostname = 'my-host'
ORDER BY p.pid;
```

### f. Compare Total Memory of a Host Across Different Snapshots (Historical Trending)

```sql
SELECT
    s.timestamp,
    s.hostname,
    m.total_memory
FROM
    snapshots s
JOIN
    memory_info m ON s.snapshot_id = m.snapshot_id
WHERE
    s.hostname = 'my-host'
ORDER BY
    s.timestamp;
```

## Guiding Principles with SQLite

*   **Portability:** The SQLite database file (`.db`) is fully portable across operating systems. The SQL queries are standard.
*   **Efficiency:** SQLite is highly optimized for read operations, especially with proper indexing. Storing structured data allows for very efficient querying.
*   **Minimal Dependencies:** Python's built-in `sqlite3` module and the `sqlite3` CLI tool are standard components.
*   **CLI-centric:** The `sqlite3` command-line tool provides a robust interface for interacting with the database, suitable for scripting and automation.
*   **Structured Data & Actionable Output:** By moving system information into a relational database, you transform raw data into a highly structured and queryable form, making it immensely actionable for analysis, reporting, and automation tasks.

## Conclusion

SQLite provides a powerful and indispensable component in a system information snapshot and reporting solution. It transforms transient system data into persistent, queryable knowledge, enabling in-depth analysis, historical tracking, and robust auditing capabilities that are crucial for modern IT operations and security. By integrating SQLite with your chosen scripting language for data collection and ingestion, you create a comprehensive and flexible system monitoring framework. The next step is to apply this knowledge in practical exercises.
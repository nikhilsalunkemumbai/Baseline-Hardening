# SQLite Tutorial: Configuration Drift Detection and Remediation Guidance

## Introduction

SQLite's relational engine is uniquely suited for **Drift Detection**. By storing system snapshots in tables, we can use set operations (`EXCEPT`, `INTERSECT`) and standard `JOIN` logic to instantly identify exactly what has been added, removed, or modified between two points in time. This tutorial will guide you through the SQL logic required to detect data drift and how to query remediation guidance stored in your database.

## Framework Alignment

This tutorial on "**Configuration Drift Detection and Remediation Guidance**" using **SQLite** demonstrates the foundational skills necessary to implement components of a "Cross-Platform Baseline Hardening & Auditing Framework." The techniques learned here for performing differential analysis on configuration data are essential for identifying unauthorized system changes and ensuring continuous compliance.

## SQL Logic for Drift Detection

Assume we have two tables: `snapshot_baseline` and `snapshot_current`, both with columns `item_name` and `item_value`.

### 1. Identifying Added Items

Items that exist in the **Current** snapshot but not in the **Baseline**.

```sql
SELECT item_name, item_value 
FROM snapshot_current
WHERE item_name NOT IN (SELECT item_name FROM snapshot_baseline);

-- OR using EXCEPT (Standard SQL)
-- SELECT item_name FROM snapshot_current EXCEPT SELECT item_name FROM snapshot_baseline;
```

### 2. Identifying Removed Items

Items that exist in the **Baseline** but are missing from the **Current** snapshot.

```sql
SELECT item_name, item_value 
FROM snapshot_baseline
WHERE item_name NOT IN (SELECT item_name FROM snapshot_current);
```

### 3. Identifying Modified Items (Value Drift)

Items that exist in both tables but have different values.

```sql
SELECT 
    b.item_name,
    b.item_value AS baseline_value,
    c.item_value AS current_value
FROM 
    snapshot_baseline b
JOIN 
    snapshot_current c ON b.item_name = c.item_name
WHERE 
    b.item_value <> c.item_value;
```

### 4. Querying Remediation Guidance

If you store your security policy in an SQLite table, you can join it directly to your audit results to produce an **Advisory Report**.

```sql
-- Assume table 'security_policy' with columns 'control_id', 'parameter_name', 'guidance'
-- Assume table 'audit_results' with columns 'parameter_name', 'status' (PASS/FAIL)

SELECT 
    a.parameter_name,
    a.status,
    p.guidance
FROM 
    audit_results a
LEFT JOIN 
    security_policy p ON a.parameter_name = p.parameter_name
WHERE 
    a.status = 'FAIL';
```

## Creating a Drift Detection View

A clean way to manage drift is to create a View that automatically identifies differences.

```sql
CREATE VIEW IF NOT EXISTS configuration_drift AS
SELECT 
    'ADDED' as drift_type,
    c.item_name,
    NULL as old_val,
    c.item_value as new_val
FROM snapshot_current c
WHERE c.item_name NOT IN (SELECT item_name FROM snapshot_baseline)

UNION ALL

SELECT 
    'REMOVED' as drift_type,
    b.item_name,
    b.item_value as old_val,
    NULL as new_val
FROM snapshot_baseline b
WHERE b.item_name NOT IN (SELECT item_name FROM snapshot_current)

UNION ALL

SELECT 
    'MODIFIED' as drift_type,
    b.item_name,
    b.item_value as old_val,
    c.item_value as new_val
FROM snapshot_baseline b
JOIN snapshot_current c ON b.item_name = c.item_name
WHERE b.item_value <> c.item_value;
```

## Guiding Principles with SQLite

*   **Snapshots as Tables:** Treat every audit run as a potential table (or a set of rows with a unique `run_id`) to enable comparison.
*   **Set Theory:** Use `UNION ALL`, `EXCEPT`, and `INTERSECT` to handle large-scale comparisons efficiently.
*   **Advisory Focus:** Store remediation commands as text in a policy table so they can be easily retrieved alongside failures.

## Conclusion

SQLite provides a high-performance way to perform differential analysis on system state. By using simple JOINs and Set operators, you can identify exactly how a system has drifted from its baseline and immediately provide the administrator with the guidance needed to restore security. The next step is to apply these queries in practical exercises.

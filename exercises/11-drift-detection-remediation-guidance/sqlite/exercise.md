# SQLite Exercise: Configuration Drift Detection and Remediation Guidance

## Objective

This exercise challenges you to apply your SQL querying skills using SQLite to identify configuration drift between two tables. You will perform differential analysis to detect added, removed, and modified settings, and then join your findings with a policy table to provide actionable remediation guidance.

## Framework Alignment

This exercise on "**Configuration Drift Detection and Remediation Guidance**" using **SQLite** provides practical experience in building and utilizing tools for a "Cross-Platform Baseline Hardening & Auditing Framework." By completing these tasks, you will learn how to automate the detection of unauthorized system changes and provide actionable advice to administrators—essential steps in maintaining a secure and auditable environment.

## Scenario

You are maintaining a security audit database. You have two tables: `baseline_state` (the "Known Good" configuration) and `current_state` (the results of a recent scan). You also have a `security_policy` table that contains remediation commands. Your goal is to use SQL to find every configuration item that has drifted and produce a "Fix-it" report.

## Setup

Run the following SQL commands in your `sqlite3` prompt to set up the scenario:

```sql
-- 1. Create Baseline Table
CREATE TABLE baseline_state (
    parameter TEXT PRIMARY KEY,
    val TEXT
);
INSERT INTO baseline_state VALUES ('SSH_ROOT_LOGIN', 'no');
INSERT INTO baseline_state VALUES ('PASSWORD_MAX_DAYS', '90');
INSERT INTO baseline_state VALUES ('TELNET_INSTALLED', 'no');

-- 2. Create Current State Table
CREATE TABLE current_state (
    parameter TEXT PRIMARY KEY,
    val TEXT
);
INSERT INTO current_state VALUES ('SSH_ROOT_LOGIN', 'yes'); -- MODIFIED (High Risk!)
INSERT INTO current_state VALUES ('PASSWORD_MAX_DAYS', '90'); -- UNCHANGED
INSERT INTO current_state VALUES ('UNAUTHORIZED_USER', 'guest'); -- ADDED

-- 3. Create Security Policy Table
CREATE TABLE security_policy (
    parameter TEXT PRIMARY KEY,
    remediation TEXT
);
INSERT INTO security_policy VALUES ('SSH_ROOT_LOGIN', 'sudo sed -i "s/PermitRootLogin yes/PermitRootLogin no/" /etc/ssh/sshd_config');
INSERT INTO security_policy VALUES ('PASSWORD_MAX_DAYS', 'chage --maxdays 90 <user>');
INSERT INTO security_policy VALUES ('TELNET_INSTALLED', 'sudo apt purge telnetd');
```

## Tasks

Using only SQL queries, provide the solution for each of the following:

1.  **Identify Modified Items:**
    *   Find all parameters that exist in both tables but have different values. Display the `parameter`, `baseline_val`, and `current_val`.

2.  **Identify Added Items:**
    *   Find all parameters that exist in `current_state` but are missing from `baseline_state`.

3.  **Identify Removed Items:**
    *   Find all parameters that exist in `baseline_state` but are missing from `current_state`.

4.  **Generate an Advisory Remediation Report:**
    *   Perform a JOIN between the **Modified** items and the `security_policy` table.
    *   The report should list the `parameter`, the `current_val`, and the `remediation` command for every item that differs from the baseline.

## Deliverables

Provide the SQL queries and the resulting output for each task.

## Reflection Questions

1.  How does the SQL `JOIN` operation simplify comparing thousands of configuration parameters compared to manual file diffing?
2.  What is the benefit of using `UNION ALL` to combine added, removed, and modified items into a single "Drift View"?
3.  Why is it useful to have the remediation command stored in the database alongside the policy definition?
4.  How could you use SQL to calculate a "Drift Score" (percentage of settings that differ from the baseline)?
5.  What are the advantages of using SQLite for drift analysis compared to using a simple text-based `diff` tool?

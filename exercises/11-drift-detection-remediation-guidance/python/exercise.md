# Python Exercise: Configuration Drift Detection and Remediation Guidance

## Objective

This exercise challenges you to apply your Python skills to detect "drift" between a baseline configuration and a current system state. You will write a script that identifies added, removed, and modified configuration items and maps any audit failures to remediation guidance found in a provided security policy.

## Framework Alignment

This exercise on "**Configuration Drift Detection and Remediation Guidance**" using **Python** provides practical experience in building and utilizing tools for a "Cross-Platform Baseline Hardening & Auditing Framework." By completing these tasks, you will learn how to automate the detection of unauthorized system changes and provide actionable advice to administrators—essential steps in maintaining a secure and auditable environment.

## Scenario

You are a security automation engineer. You have been given a "Golden Snapshot" (the desired state) and a "Current Snapshot" (the actual state) of a server's security settings. Your goal is to identify all differences and generate an **Advisory Audit Report** that includes the exact remediation steps for any non-compliant settings.

## Setup

Create two JSON files in your working directory to simulate the snapshots.

### `baseline.json`
```json
{
  "ENCRYPT_METHOD": "SHA512",
  "PASS_MAX_DAYS": "90",
  "IP_FORWARD": "0",
  "SSH_PORT": "22"
}
```

### `current_state.json`
```json
{
  "ENCRYPT_METHOD": "MD5",
  "PASS_MAX_DAYS": "90",
  "SSH_PORT": "22",
  "NEW_USER_ACCOUNT": "unauthorized_user"
}
```

## Tasks

Write a Python script (`drift_advisor.py`) that performs the following:

1.  **Detect Drift:**
    *   Compare `current_state.json` against `baseline.json`.
    *   Identify which key is **MODIFIED** (value differs).
    *   Identify which key is **REMOVED** (missing in current).
    *   Identify which key is **ADDED** (missing in baseline).

2.  **Provide Remediation Guidance:**
    *   Define a small policy dictionary inside your script that maps keys to remediation commands (e.g., `{"ENCRYPT_METHOD": "Set to SHA512 in /etc/login.defs"}`).
    *   For every **MODIFIED** or **REMOVED** item that represents a security failure, print the "Remediation Step."

3.  **Generate an Advisory Report:**
    *   Output a summary of all drifts.
    *   For the "ENCRYPT_METHOD" failure, output a specific "Fix" command.

## Deliverables

Provide the complete `drift_advisor.py` script.

## Reflection Questions

1.  How does programmatically comparing dictionaries (JSON) differ from comparing text files line-by-line?
2.  Why is it important to distinguish between an "Added" item (e.g., a new user) and a "Modified" item (e.g., a changed password policy)?
3.  How could you extend this script to handle nested JSON objects (e.g., a list of network interfaces within the snapshot)?
4.  Discuss the value of providing a "Remediation" string in an automated report compared to just stating "FAIL."
5.  What are the advantages of using Python for this task compared to using a specialized diff tool?

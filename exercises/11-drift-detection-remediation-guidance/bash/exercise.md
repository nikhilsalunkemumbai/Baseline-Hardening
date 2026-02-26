# Bash Exercise: Configuration Drift Detection and Remediation Guidance

## Objective

This exercise challenges you to apply your Bash text-processing skills to detect drift in a configuration file and generate copy-pasteable remediation commands. You will use `grep`, `awk`, and conditional logic to identify when a system parameter has deviated from its baseline and provide the exact `sed` command to fix it.

## Framework Alignment

This exercise on "**Configuration Drift Detection and Remediation Guidance**" using **Bash** provides practical experience in building and utilizing tools for a "Cross-Platform Baseline Hardening & Auditing Framework." By completing these tasks, you will learn how to automate the detection of unauthorized system changes and provide actionable advice to administrators—essential steps in maintaining a secure and auditable environment.

## Scenario

You are hardening a set of Linux servers. You need a script that checks two critical parameters in `/etc/login.defs`: `PASS_MAX_DAYS` and `ENCRYPT_METHOD`. If they don't match the baseline, your script should output a "Security Advisory" with the fix.

## Setup

Create a dummy configuration file to test your script:

```bash
cat <<EOL > mock_login.defs
# System Configuration
PASS_MAX_DAYS   999
ENCRYPT_METHOD  MD5
EOL
```

## Tasks

Write a Bash script (`check_drift.sh`) that performs the following:

1.  **Define Baseline Values:**
    *   Set variables for the expected values: `EXPECTED_DAYS=90` and `EXPECTED_HASH=SHA512`.

2.  **Extract Current Values:**
    *   Use `grep` and `awk` to extract the current values of `PASS_MAX_DAYS` and `ENCRYPT_METHOD` from `mock_login.defs`.

3.  **Detect Drift and Suggest Fixes:**
    *   Compare the current values to the expected ones.
    *   If `PASS_MAX_DAYS` is wrong, output: `[!] PASS_MAX_DAYS DRIFT DETECTED. FIX: sudo sed -i "s/^PASS_MAX_DAYS.*/PASS_MAX_DAYS 90/" /etc/login.defs`
    *   If `ENCRYPT_METHOD` is wrong, output: `[!] ENCRYPT_METHOD DRIFT DETECTED. FIX: sudo sed -i "s/^ENCRYPT_METHOD.*/ENCRYPT_METHOD SHA512/" /etc/login.defs`

4.  **Summary:**
    *   If no drift is found, output: `[OK] System configuration matches baseline.`

## Deliverables

Provide the complete `check_drift.sh` script.

## Reflection Questions

1.  Why is `awk` preferred over `cut` when parsing configuration files with varying amounts of whitespace?
2.  How does providing the exact `sed` command in the output help an administrator who is responding to an alert?
3.  What is the risk of using `sed -i` (in-place edit) in a real remediation script, and how can you make it safer?
4.  How would you extend this script to check for multiple files across the system?
5.  What are the advantages of using Bash for this task compared to a high-level configuration management tool like Ansible or Chef?

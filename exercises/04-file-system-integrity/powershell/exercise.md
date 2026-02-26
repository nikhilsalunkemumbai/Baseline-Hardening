# PowerShell Exercise: File System Search and Integrity Check

## Objective

This exercise challenges you to apply your PowerShell scripting skills to perform file system searches based on various criteria and to verify file integrity using cryptographic hashes. You will use standard PowerShell cmdlets to locate files, generate a baseline of their hashes, and detect changes (modifications, additions, deletions), demonstrating proficiency in security auditing and file management.

## Framework Alignment

This exercise on "**File System Search and Integrity Check**" using **PowerShell** provides practical experience in building and utilizing tools for a "Cross-Platform Baseline Hardening & Auditing Framework." By completing these tasks, you will learn how to audit and manage file system integrity, ensuring that critical system files and configurations have not been unauthorizedly modified—an essential step in maintaining a secure and auditable environment.


## Scenario

You are a system administrator tasked with monitoring a critical application directory for unauthorized changes. You need to be able to:
1.  Quickly find specific files.
2.  Generate a cryptographic baseline of the directory's contents.
3.  Periodically check the directory against this baseline to detect any tampering.

## Setup

Before starting the tasks, create the following directory structure and files in a temporary location (e.g., `C:	emp	est_dir`):

```powershell
# Create the directory structure
New-Item -Path "C:	emp	est_dir\config" -ItemType Directory -Force
New-Item -Path "C:	emp	est_dir\logs" -ItemType Directory -Force
New-Item -Path "C:	emp	est_dir\scripts" -ItemType Directory -Force

# Create app.conf
@"
[settings]
debug=true
port=8080
secret_key=YOUR_SECRET_KEY
"@ | Set-Content -Path "C:	emp	est_dir\config\app.conf"

# Create error.log
@"
2026-03-01 10:00:00 ERROR: Failed to connect to DB.
2026-03-01 10:05:00 INFO: User 'admin' logged in.
"@ | Set-Content -Path "C:	emp	est_dir\logs\error.log"

# Create backup.sh
@"
#!/bin/bash
# Simple backup script
tar -czf /tmp/backup_$(Get-Date -Format "yyyy-MM-dd").tar.gz /var/www/html
"@ | Set-Content -Path "C:	emp	est_dir\scripts\backup.sh"

# Create important.txt
@"
This is an important document.
Do not modify without authorization.
"@ | Set-Content -Path "C:	emp	est_dir\important.txt"

Write-Host "Test directory 'C:	emp	est_dir' created with sample files."
```

Navigate into the `test_dir` for the exercise: `Set-Location C:	emp	est_dir`

## Tasks

Using only standard PowerShell cmdlets (`Get-ChildItem`, `Where-Object`, `Select-String`, `Get-FileHash`, `Compare-Object`, etc.), provide the command-line solution or a simple script for each of the following tasks.

### Part 1: File Search

1.  **Find all `.conf` files:**
    *   List all files ending with `.conf` within `C:	emp	est_dir`.

2.  **Find files larger than 100 bytes:**
    *   List all regular files in `C:	emp	est_dir` that are larger than 100 bytes.

3.  **Find files containing "ERROR":**
    *   Recursively search for files within `C:	emp	est_dir` that contain the literal string "ERROR" (case-sensitive) and list their full paths.

4.  **Find files modified in the last 24 hours:**
    *   *(Skip this task if your setup created files just now, as all will match. If you want to test, manually modify an old file or use a temporary file with an old timestamp for demonstration purposes).*
    *   List files that were modified less than 24 hours ago.

### Part 2: File Integrity Baseline and Verification

For this part, perform the tasks in order.

1.  **Generate Initial Baseline:**
    *   Create a SHA256 hash baseline for all regular files within the current `C:	emp	est_dir` directory structure. The baseline should contain the full path, relative path (from `C:	emp	est_dir`), and SHA256 hash for each file.
    *   Export this baseline to a CSV file named `baseline_v1.csv` in the parent directory (e.g., `C:	emp\baseline_v1.csv`).

2.  **Verify Initial Baseline:**
    *   Write a script that loads `baseline_v1.csv`, re-calculates hashes for the files in `C:	emp	est_dir`, and reports if any files have hashes that do not match the baseline.

3.  **Simulate a File Modification:**
    *   Append the text "MALICIOUS_CODE" to the end of `C:	emp	est_dir\scripts\backup.sh`.
    *   Append the text "Important update." to `C:	emp	est_dir\important.txt`.

4.  **Detect Modified Files:**
    *   Re-run your integrity check script.
    *   Identify which files are now reported as "Modified" due to hash mismatch.

5.  **Simulate a New File Addition:**
    *   Create a new file: `C:	emp	est_dir
ew_threat.txt` with content "I am a new threat!".

6.  **Detect New Files:**
    *   Re-run your integrity check script or a separate comparison.
    *   Identify `new_threat.txt` as a new file (present in the current state but not in `baseline_v1.csv`).

7.  **Simulate a File Deletion:**
    *   Delete the `C:	emp	est_dir\logs\error.log` file.

8.  **Detect Deleted Files:**
    *   Re-run your integrity check script or a separate comparison.
    *   Identify `error.log` as a deleted file (present in `baseline_v1.csv` but not in the current state).

## Deliverables

For each task in Part 1, provide the PowerShell command-line pipeline. For Part 2, provide the PowerShell script file(s) you create for baseline generation and comparison.

## Reflection Questions

1.  How did PowerShell's object pipeline (`Get-ChildItem | Where-Object | Get-FileHash`) simplify the process of finding files and calculating their hashes compared to methods that require parsing text output?
2.  Explain how `Compare-Object` can be effectively used to detect new, deleted, and modified files when comparing a current state against a baseline.
3.  What are the advantages of storing the baseline in a structured format like CSV or JSON when working with PowerShell?
4.  If you needed to collect this data from 100 remote Windows servers, how would you adapt your PowerShell solution for remote execution and centralized reporting?
5.  What are the security implications of file integrity monitoring, and how does PowerShell contribute to building effective FIM solutions?

---
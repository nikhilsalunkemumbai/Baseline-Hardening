# Python Exercise: File System Search and Integrity Check

## Objective

This exercise challenges you to apply your Python scripting skills to perform file system searches based on various criteria and to verify file integrity using cryptographic hashes. You will use Python's standard library modules to locate files, generate a baseline of their hashes, and detect changes (modifications, additions, deletions), demonstrating proficiency in cross-platform security auditing and file management.

## Framework Alignment

This exercise on "**File System Search and Integrity Check**" using **Python** provides practical experience in building and utilizing tools for a "Cross-Platform Baseline Hardening & Auditing Framework." By completing these tasks, you will learn how to audit and manage file system integrity, ensuring that critical system files and configurations have not been unauthorizedly modified—an essential step in maintaining a secure and auditable environment.


## Scenario

You are a system administrator tasked with developing a portable Python script to monitor a critical application directory for unauthorized changes. Your script needs to:
1.  Provide flexible search capabilities for files.
2.  Generate a cryptographic baseline (SHA256) of the directory's contents.
3.  Compare the current state of the directory against this baseline to detect any tampering (new, modified, or deleted files).

## Setup

Before starting the tasks, create the following directory structure and files in a temporary location (e.g., `~/test_dir`):

```python
import os
from pathlib import Path

base_path = Path("test_dir")
base_path.mkdir(exist_ok=True)
(base_path / "config").mkdir(exist_ok=True)
(base_path / "logs").mkdir(exist_ok=True)
(base_path / "scripts").mkdir(exist_ok=True)

(base_path / "config" / "app.conf").write_text("""[settings]
debug=true
port=8080
secret_key=YOUR_SECRET_KEY
""")

(base_path / "logs" / "error.log").write_text("""2026-03-01 10:00:00 ERROR: Failed to connect to DB.
2026-03-01 10:05:00 INFO: User 'admin' logged in.
""")

(base_path / "scripts" / "backup.sh").write_text("""#!/bin/bash
# Simple backup script
tar -czf /tmp/backup_$(date +%F).tar.gz /var/www/html
""")

(base_path / "important.txt").write_text("""This is an important document.
Do not modify without authorization.
""")

print(f"Test directory '{base_path.resolve()}' created with sample files.")
```

Run this Python snippet to create the `test_dir` and its contents. Then, ensure your working directory is the parent of `test_dir`.

## Tasks

Write a Python script (`file_auditor.py` or similar name) that, when executed, can perform the following tasks. Your script should be structured with functions for clarity and output results in a structured JSON format where appropriate.

### Part 1: File Search

Implement functions in your script to perform the following searches within the `test_dir` directory.

1.  **Find all `.conf` files:**
    *   List all files ending with `.conf` (case-insensitive) within `test_dir`.

2.  **Find files larger than 100 bytes:**
    *   List all regular files in `test_dir` that are larger than 100 bytes.

3.  **Find files containing "ERROR":**
    *   Recursively search for files within `test_dir` that contain the literal string "ERROR" (case-sensitive) and list their full paths.

4.  **Find files modified in the last 24 hours:**
    *   *(If your setup just created files, all will match. For testing, manually modify an old file or use a temporary file with an old timestamp.)*
    *   List files that were modified less than 24 hours ago.

### Part 2: File Integrity Baseline and Verification

Implement functions in your script for file integrity.

1.  **Generate Initial Baseline:**
    *   Create a SHA256 hash baseline for all regular files within the `test_dir` directory structure.
    *   Save this baseline to a JSON file named `baseline_v1.json` in the parent directory. The JSON should map file paths to a dictionary containing `hash`, `size`, `mtime`, and `algorithm`.

2.  **Verify Initial Baseline (Implicit):**
    *   (This will be part of the `compare` function below).

3.  **Simulate a File Modification:**
    *   Append the text "MALICIOUS_CODE" to the end of `test_dir/scripts/backup.sh`.
    *   Append the text "Important update." to `test_dir/important.txt`.

4.  **Detect Changes (New, Modified, Deleted Files):**
    *   Implement a function `compare_baselines(current_dir, baseline_file, algorithm="sha256")` that compares the current state of `current_dir` against the `baseline_file`.
    *   This function should return a report (as a Python dictionary) detailing:
        *   `new_files`: Files present now but not in the baseline.
        *   `deleted_files`: Files in the baseline but not present now.
        *   `modified_files`: Files present in both, but with differing hashes.
        *   `unchanged_files`: Files present in both with matching hashes.
    *   Run this comparison using `baseline_v1.json` after modifications.

5.  **Simulate a New File Addition:**
    *   Create a new file: `test_dir/new_threat.txt` with content "I am a new threat!".
    *   Run your comparison function again to detect this new file.

6.  **Simulate a File Deletion:**
    *   Delete the `test_dir/logs/error.log` file.
    *   Run your comparison function again to detect this deleted file.

## Deliverables

Provide the complete Python script file (`file_auditor.py`) that implements all the above tasks. Ensure the script can execute different tasks based on command-line arguments (e.g., `python file_auditor.py search --name "*.conf" .` or `python file_auditor.py baseline generate . baseline_v1.json`).

## Reflection Questions

1.  How did Python's `os.walk` or `pathlib` simplify file system traversal compared to purely shell-based methods?
2.  Explain how `hashlib` is used to ensure the integrity of files. What are the advantages of using SHA256 over MD5 in a security context?
3.  Describe the logic you implemented to detect `new`, `deleted`, and `modified` files when comparing the current state against a baseline.
4.  How does outputting the search results and integrity reports as JSON make your Python script more versatile for integration with other tools or automation workflows?
5.  What are the advantages and disadvantages of using Python for file system search and integrity checks compared to Bash, PowerShell, or a database like SQLite?

---
# Bash Exercise: File System Search and Integrity Check

## Objective

This exercise challenges you to apply your Bash scripting skills to perform file system searches based on various criteria and to verify file integrity using cryptographic hashes. You will use standard command-line utilities to locate files, generate a baseline of their hashes, and detect changes (modifications, additions, deletions), demonstrating proficiency in security auditing and file management.

## Framework Alignment

This exercise on "**File System Search and Integrity Check**" using **Bash** provides practical experience in building and utilizing tools for a "Cross-Platform Baseline Hardening & Auditing Framework." By completing these tasks, you will learn how to audit and manage file system integrity, ensuring that critical system files and configurations have not been unauthorizedly modified—an essential step in maintaining a secure and auditable environment.


## Scenario

You are a system administrator tasked with monitoring a critical application directory for unauthorized changes. You need to be able to:
1.  Quickly find specific files.
2.  Generate a cryptographic baseline of the directory's contents.
3.  Periodically check the directory against this baseline to detect any tampering.

## Setup

Before starting the tasks, create the following directory structure and files in a temporary location (e.g., `~/test_dir`):

```
# Create the directory structure
mkdir -p test_dir/config
mkdir -p test_dir/logs
mkdir -p test_dir/scripts

# Create app.conf
cat <<EOL > test_dir/config/app.conf
[settings]
debug=true
port=8080
secret_key=YOUR_SECRET_KEY
EOL

# Create error.log
cat <<EOL > test_dir/logs/error.log
2026-03-01 10:00:00 ERROR: Failed to connect to DB.
2026-03-01 10:05:00 INFO: User 'admin' logged in.
EOL

# Create backup.sh
cat <<EOL > test_dir/scripts/backup.sh
#!/bin/bash
# Simple backup script
tar -czf /tmp/backup_\$(date +%F).tar.gz /var/www/html
EOL

# Create important.txt
cat <<EOL > test_dir/important.txt
This is an important document.
Do not modify without authorization.
EOL

echo "Test directory 'test_dir' created with sample files."
```

Navigate into the `test_dir` for the exercise: `cd test_dir`

## Tasks

Using only standard Bash commands (`find`, `grep`, `md5sum`, `sha256sum`, `xargs`, etc.), provide the command-line solution for each of the following tasks.

### Part 1: File Search

1.  **Find all `.conf` files:**
    *   List all files ending with `.conf` within `test_dir`.

2.  **Find files larger than 100 bytes:**
    *   List all regular files in `test_dir` that are larger than 100 bytes.

3.  **Find files containing "ERROR":**
    *   Recursively search for files within `test_dir` that contain the literal string "ERROR" (case-sensitive) and list their paths.

4.  **Find files modified in the last 24 hours:**
    *   *(Skip this task if your setup created files just now, as all will match. If you want to test, manually modify an old file or use a temporary file with an old timestamp for demonstration purposes).*
    *   List files that were modified less than 24 hours ago.

### Part 2: File Integrity Baseline and Verification

For this part, perform the tasks in order.

1.  **Generate Initial Baseline:**
    *   Create a SHA256 hash baseline for all regular files within the current `test_dir` directory structure.
    *   Save this baseline to a file named `baseline_v1.sha256` in the parent directory (e.g., `../baseline_v1.sha256`). The baseline file should contain one hash and filename per line.

2.  **Verify Initial Baseline:**
    *   Use the `sha256sum -c` command (or similar) to verify the integrity of the files against `baseline_v1.sha256`. Ensure all files are reported as "OK".

3.  **Simulate a File Modification:**
    *   Append the text "MALICIOUS_CODE" to the end of `test_dir/scripts/backup.sh`.
    *   Append the text "Important update." to `test_dir/important.txt`.

4.  **Detect Modified Files:**
    *   Run an integrity check again using `sha256sum -c baseline_v1.sha256`.
    *   Identify which files are now reported as "FAILED".

5.  **Simulate a New File Addition:**
    *   Create a new file: `test_dir/new_threat.txt` with content "I am a new threat!".

6.  **Detect New Files (Manual Comparison):**
    *   Perform a comparison that identifies `new_threat.txt` as a new file (i.e., present in the current state but not in `baseline_v1.sha256`). You'll need to generate current hashes and compare them.

7.  **Simulate a File Deletion:**
    *   Delete the `test_dir/logs/error.log` file.

8.  **Detect Deleted Files (Manual Comparison):**
    *   Perform a comparison that identifies `error.log` as a deleted file (i.e., present in `baseline_v1.sha256` but not in the current state).

## Deliverables

For each task, provide the single Bash command-line pipeline or script snippet that produces the required output. For task 2.1, provide the content of `baseline_v1.sha256`.

## Reflection Questions

1.  Which specific `find` options were most useful for filtering files by name, size, and modification time?
2.  How does `grep`'s recursive and content-based search functionality complement `find` for file system analysis?
3.  Describe the basic steps you would take to automate the integrity checking process using a Bash script (e.g., scheduling it with cron, alerting on discrepancies).
4.  What are the limitations of using a simple text file for storing a hash baseline when dealing with large, frequently changing file systems?
5.  Why is it important to use strong cryptographic hashing algorithms (like SHA256) for file integrity checks, especially in a security context?

---
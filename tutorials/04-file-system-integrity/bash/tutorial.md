# Bash Tutorial: File System Search and Integrity Check

## Introduction

Bash, through its powerful set of command-line utilities, offers robust capabilities for navigating file systems, searching for files based on various criteria, and performing basic integrity checks using cryptographic hashes. This tutorial will demonstrate how to leverage tools like `find`, `grep`, `md5sum`, and `sha256sum` to implement our file system search and integrity check design, emphasizing minimal dependencies and cross-platform compatibility across Unix-like systems.

## Framework Alignment

This tutorial on "**File System Search and Integrity Check**" using **Bash** demonstrates the foundational skills necessary to implement components of a "Cross-Platform Baseline Hardening & Auditing Framework." The techniques learned here for searching files and calculating cryptographic hashes are directly applicable to auditing systems against defined security baselines and ensuring compliance.


## Core Bash Utilities for File System Operations

*   **`find`**: Recursively searches for files and directories in a directory hierarchy based on criteria.
    *   `-name`, `-iname`: Search by filename (case-sensitive/insensitive).
    *   `-size`: Search by file size.
    *   `-mtime`, `-atime`, `-ctime`: Search by modification, access, or inode change time.
    *   `-type`: Search by file type (f for file, d for directory, l for symlink).
    *   `-exec`: Execute a command on found files.
*   **`grep`**: Searches for patterns in files. Essential for content-based searches.
    *   `-r`: Recursive.
    *   `-i`: Ignore case.
    *   `-l`: List filenames only.
    *   `-E`: Extended regex.
*   **`xargs`**: Builds and executes command lines from standard input. Useful for passing `find` results to other commands.
*   **`md5sum`**: Computes and checks MD5 message digests.
*   **`sha1sum`** / **`sha256sum`**: Compute and check SHA1 / SHA256 checksums. (`shasum` on macOS can be used with `-a` for different algorithms).
*   **`sort`**, **`uniq`**: For processing and filtering lists of files or hashes.

## Implementing Core Functionality with Bash

### 1. File System Search

#### a. By Name (`find -name`)

```bash
# Find all files named 'report.log' in the current directory and subdirectories
find . -name "report.log"

# Find all files with '.conf' extension (case-insensitive)
find /etc -iname "*.conf"
```

#### b. By Size (`find -size`)

```bash
# Find all files larger than 1MB (suffix 'M')
find . -type f -size +1M

# Find all files smaller than 10KB (suffix 'k')
find /var/log -type f -size -10k
```

#### c. By Modification Time (`find -mtime`)

```bash
# Find files modified in the last 24 hours (mtime 0 means modified less than 24 hours ago)
find . -type f -mtime 0

# Find files modified exactly 7 days ago
find . -type f -mtime 7

# Find files modified more than 30 days ago
find . -type f -mtime +30
```
*Note: `-mtime N` means N *24 hour periods* ago. For more precise time, use `find -newermt "YYYY-MM-DD HH:MM:SS"`.*

#### d. By Content (`grep -r`)

```bash
# Find files containing "ERROR" recursively in current directory (case-insensitive)
grep -ri "ERROR" .

# List only the names of files containing "secret_key"
grep -rl "secret_key" /home/user
```

### 2. Integrity Check (Hashing)

#### a. Generate Hash of a Single File

```bash
# Generate MD5 hash
md5sum my_document.txt

# Generate SHA256 hash
sha256sum /bin/bash
```

#### b. Generate Hashes for Multiple Files (e.g., from `find` output)

```bash
# Find all .sh files in current directory and generate their SHA256 hashes
find . -type f -name "*.sh" -print0 | xargs -0 sha256sum
# -print0 and xargs -0 handle filenames with spaces correctly.
```

#### c. Verify Hash of a Single File (against a known hash)

```bash
KNOWN_HASH="e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855" # SHA256 of empty file
FILE_TO_CHECK="empty.txt"

CURRENT_HASH=$(sha256sum "$FILE_TO_CHECK" | awk '{print $1}')

if [ "$CURRENT_HASH" = "$KNOWN_HASH" ]; then
    echo "Integrity check passed for $FILE_TO_CHECK."
else
    echo "Integrity check FAILED for $FILE_TO_CHECK. Hash mismatch!"
fi
```

### 3. Basic Baseline Management

#### a. Generate a Baseline File

```bash
#!/bin/bash

# baseline_gen.sh /path/to/monitor > baseline.sha256

TARGET_DIR="$1"
OUTPUT_FILE="$2"

if [ -z "$TARGET_DIR" ] || [ ! -d "$TARGET_DIR" ]; then
    echo "Usage: $0 <directory_to_monitor> > baseline.sha256"
    exit 1
fi

echo "Generating SHA256 baseline for $TARGET_DIR..."
# Find all regular files, then compute their SHA256 sum.
# Output format: HASH  PATH
find "$TARGET_DIR" -type f -print0 | xargs -0 sha256sum > "$OUTPUT_FILE"
echo "Baseline saved to $OUTPUT_FILE"

# Example Usage:
# bash baseline_gen.sh /etc > /tmp/etc_baseline.sha256
```

#### b. Compare Current State Against Baseline

```bash
#!/bin/bash

# baseline_check.sh baseline.sha256 /path/to/monitor

BASELINE_FILE="$1"
TARGET_DIR="$2"

if [ -z "$BASELINE_FILE" ] || [ ! -f "$BASELINE_FILE" ] || [ -z "$TARGET_DIR" ] || [ ! -d "$TARGET_DIR" ]; then
    echo "Usage: $0 <baseline_file.sha256> <directory_to_monitor>"
    exit 1
fi

echo "Checking integrity of $TARGET_DIR against $BASELINE_FILE..."

# Generate current hashes
CURRENT_HASHS_TEMP=$(mktemp)
find "$TARGET_DIR" -type f -print0 | xargs -0 sha256sum > "$CURRENT_HASHS_TEMP"

# Compare using diff. The 'sha256sum -c' command is also an option.
# diff -u "$BASELINE_FILE" "$CURRENT_HASHS_TEMP"
# If no output, no changes. If output, changes detected.

# More explicit check using 'sha256sum -c'
# This command expects a file with HASH FILENAME format.
# It will report "OK" or "FAILED" for each file.
# We then process its output to categorize changes.

# Run check on existing files (will report FAILED for modified, OK for unchanged)
echo "--- Checking known files for modification ---"
sha256sum -c "$BASELINE_FILE" 2>&1 | grep -E "FAILED|OK"

echo "--- Checking for new or deleted files ---"
# Find files in current state not in baseline (new files)
grep -Fvxf <(awk '{print $2}' "$BASELINE_FILE") <(awk '{print $2}' "$CURRENT_HASHS_TEMP") | sed 's/^/NEW: /'

# Find files in baseline not in current state (deleted files)
grep -Fvxf <(awk '{print $2}' "$CURRENT_HASHS_TEMP") <(awk '{print $2}' "$BASELINE_FILE") | sed 's/^/DELETED: /'

rm "$CURRENT_HASHS_TEMP"

# Example Usage:
# Create some changes, then run:
# bash baseline_check.sh /tmp/etc_baseline.sha256 /etc
```

## Guiding Principles in Bash

*   **Portability:** `find`, `grep`, `md5sum`/`sha256sum` are standard on virtually all Unix-like systems.
*   **Efficiency:** These utilities are typically compiled C programs, making them very fast. Piping efficiently streams data between them.
*   **Minimal Dependencies:** Relies entirely on core system utilities.
*   **CLI-centric:** All operations are command-line based, ideal for scripting.

## Conclusion

Bash provides a robust and efficient command-line environment for performing file system searches and integrity checks. By mastering utilities like `find`, `grep`, and the `*sum` tools, you can build powerful scripts to locate specific files, verify their integrity, and detect unauthorized modifications. While Bash is excellent for direct interaction and lightweight scripting, managing complex data structures or handling very large datasets might lead to more complex scripting logic. The next step is to apply this knowledge in practical exercises.
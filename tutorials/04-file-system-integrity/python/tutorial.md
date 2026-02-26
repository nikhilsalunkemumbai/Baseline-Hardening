# Python Tutorial: File System Search and Integrity Check

## Introduction

Python's comprehensive standard library provides powerful and cross-platform capabilities for interacting with the file system, searching for files based on various criteria, and performing cryptographic hashing for integrity checks. This tutorial will guide you through building a utility that can search files and verify their integrity, leveraging modules like `os`, `pathlib`, `hashlib`, and `re`, while adhering to our principles of minimal dependencies and structured output.

## Framework Alignment

This tutorial on "**File System Search and Integrity Check**" using **Python** demonstrates the foundational skills necessary to implement components of a "Cross-Platform Baseline Hardening & Auditing Framework." The techniques learned here for searching files and calculating cryptographic hashes are directly applicable to auditing systems against defined security baselines and ensuring compliance.


## Core Python Modules for File System Operations

*   **`os`**: Provides a way of using operating system dependent functionality. Essential for file system traversal (`os.walk`), path manipulation (`os.path`), and getting file metadata.
*   **`pathlib`**: (Python 3.4+) Offers an object-oriented approach to file system paths, making code more readable and robust.
*   **`hashlib`**: Provides various secure hash and message digest algorithms (MD5, SHA1, SHA256, etc.). Crucial for file integrity.
*   **`re`**: Regular expression operations. Useful for pattern-based searches in filenames or file content.
*   **`json`**: For serializing search results or hash baselines into JSON format.
*   **`csv`**: For reading/writing data in CSV format.

## Implementing Core Functionality with Python

### 1. File System Search

#### Helper Function for File Traversal

```python
import os
import hashlib
import json
import re
from datetime import datetime
from pathlib import Path

def get_file_metadata(filepath):
    """Returns basic metadata for a given file."""
    stat = os.stat(filepath)
    return {
        "path": str(filepath),
        "name": filepath.name,
        "size": stat.st_size,
        "last_modified": datetime.fromtimestamp(stat.st_mtime).isoformat(),
        "is_dir": filepath.is_dir(),
        "is_file": filepath.is_file(),
        "is_symlink": filepath.is_symlink(),
    }

def find_files_recursive(start_path):
    """Generator to yield Path objects for all files in a directory hierarchy."""
    for root, dirs, files in os.walk(start_path):
        for file in files:
            yield Path(root) / file
```

#### a. By Name (`pathlib` / `os.walk`)

```python
def search_by_name(start_dir, name_pattern, is_regex=False, case_sensitive=True):
    """Searches for files by name pattern."""
    results = []
    for filepath in find_files_recursive(start_dir):
        match = False
        if is_regex:
            if case_sensitive:
                match = re.search(name_pattern, filepath.name)
            else:
                match = re.search(name_pattern, filepath.name, re.IGNORECASE)
        else:
            if case_sensitive:
                match = (name_pattern in filepath.name)
            else:
                match = (name_pattern.lower() in filepath.name.lower())
        
        if match:
            results.append(get_file_metadata(filepath))
    return results

# Example Usage:
# print("--- Search by Name ---")
# found = search_by_name(".", "*.py", is_regex=False) # Not truly wildcard, just substring for is_regex=False
# print(json.dumps(found, indent=2))
```

#### b. By Size (`os.stat` / `pathlib.stat`)

```python
def search_by_size(start_dir, operator, size_bytes):
    """Searches for files by size (e.g., '>1000', '<500')."""
    results = []
    for filepath in find_files_recursive(start_dir):
        try:
            file_size = filepath.stat().st_size
            if operator == '>':
                if file_size > size_bytes: results.append(get_file_metadata(filepath))
            elif operator == '<':
                if file_size < size_bytes: results.append(get_file_metadata(filepath))
            elif operator == '=':
                if file_size == size_bytes: results.append(get_file_metadata(filepath))
        except OSError:
            # Handle permission errors or file not found during stat()
            pass
    return results

# Example Usage:
# print("--- Search by Size (>1KB) ---")
# found = search_by_size(".", ">", 1024)
# print(json.dumps(found, indent=2))
```

#### c. By Modification Time (`os.stat` / `pathlib.stat`)

```python
def search_by_mtime(start_dir, operator, target_datetime):
    """Searches for files by modification time (e.g., '> YYYY-MM-DD')."""
    results = []
    target_timestamp = target_datetime.timestamp()
    for filepath in find_files_recursive(start_dir):
        try:
            mtime = filepath.stat().st_mtime
            if operator == '>':
                if mtime > target_timestamp: results.append(get_file_metadata(filepath))
            elif operator == '<':
                if mtime < target_timestamp: results.append(get_file_metadata(filepath))
        except OSError:
            pass
    return results

# Example Usage:
# print("--- Search by Modification Time (after 2026-02-01) ---")
# target_dt = datetime(2026, 2, 1)
# found = search_by_mtime(".", ">", target_dt)
# print(json.dumps(found, indent=2))
```

#### d. By Content (`re` and file reading)

```python
def search_by_content(start_dir, content_pattern, is_regex=False, case_sensitive=True):
    """Searches for files containing a specific content pattern."""
    results = []
    for filepath in find_files_recursive(start_dir):
        try:
            with open(filepath, 'r', encoding='utf-8', errors='ignore') as f:
                content = f.read()
                match = False
                if is_regex:
                    if case_sensitive:
                        match = re.search(content_pattern, content)
                    else:
                        match = re.search(content_pattern, content, re.IGNORECASE)
                else:
                    if case_sensitive:
                        match = (content_pattern in content)
                    else:
                        match = (content_pattern.lower() in content.lower())
                
                if match:
                    results.append(get_file_metadata(filepath))
        except (OSError, UnicodeDecodeError):
            # Handle permission errors, binary files, etc.
            pass
    return results

# Example Usage (assuming you have some text files):
# print("--- Search by Content ('ERROR') ---")
# found = search_by_content(".", "ERROR", case_sensitive=False)
# print(json.dumps(found, indent=2))
```

### 2. Integrity Check (Hashing)

#### a. Generate File Hash

```python
def generate_file_hash(filepath, algorithm="sha256"):
    """Generates the hash of a file using the specified algorithm."""
    hash_func = hashlib.new(algorithm)
    try:
        with open(filepath, "rb") as f:
            for chunk in iter(lambda: f.read(4096), b""):
                hash_func.update(chunk)
        return hash_func.hexdigest()
    except OSError:
        return "ERROR: Could not read file."
    except Exception as e:
        return f"ERROR: Hashing failed: {e}"

# Example Usage:
# print("--- Generate File Hash ---")
# test_file = Path("test.txt")
# if not test_file.exists():
#     test_file.write_text("Hello World!")
# print(f"{test_file}: {generate_file_hash(test_file, 'sha256')}")
```

#### b. Generate Baseline File

```python
def generate_baseline(start_dir, output_file, algorithm="sha256"):
    """Generates a baseline file with hashes for all files in a directory."""
    baseline_data = {}
    for filepath in find_files_recursive(start_dir):
        file_hash = generate_file_hash(filepath, algorithm)
        if not file_hash.startswith("ERROR"):
            baseline_data[str(filepath)] = {
                "hash": file_hash,
                "size": filepath.stat().st_size,
                "mtime": datetime.fromtimestamp(filepath.stat().st_mtime).isoformat(),
                "algorithm": algorithm
            }
    
    with open(output_file, 'w') as f:
        json.dump(baseline_data, f, indent=2)
    print(f"Baseline saved to {output_file}")

# Example Usage:
# generate_baseline(".", "baseline.json", "sha256")
```

#### c. Compare Current State Against Baseline

```python
def compare_to_baseline(start_dir, baseline_file, algorithm="sha256"):
    """Compares current file system state against a saved baseline."""
    
    with open(baseline_file, 'r') as f:
        baseline = json.load(f)
    
    current_state = {}
    for filepath in find_files_recursive(start_dir):
        file_hash = generate_file_hash(filepath, algorithm)
        if not file_hash.startswith("ERROR"):
            current_state[str(filepath)] = {
                "hash": file_hash,
                "size": filepath.stat().st_size,
                "mtime": datetime.fromtimestamp(filepath.stat().st_mtime).isoformat(),
                "algorithm": algorithm
            }
    
    report = {
        "new_files": [],
        "deleted_files": [],
        "modified_files": [],
        "unchanged_files": []
    }

    # Check for modified or unchanged files
    for path, data in baseline.items():
        if path in current_state:
            if current_state[path]["hash"] != data["hash"]:
                report["modified_files"].append({
                    "path": path,
                    "baseline_hash": data["hash"],
                    "current_hash": current_state[path]["hash"]
                })
            else:
                report["unchanged_files"].append({"path": path, "hash": data["hash"]})
        else:
            report["deleted_files"].append({"path": path, "hash": data["hash"]})
    
    # Check for new files
    for path, data in current_state.items():
        if path not in baseline:
            report["new_files"].append({"path": path, "hash": data["hash"]})
            
    return report

# Example Usage:
# print("--- Compare to Baseline ---")
# # Create a new file or modify an existing one to see changes
# Path("new_file.txt").write_text("This is new.")
# report = compare_to_baseline(".", "baseline.json", "sha256")
# print(json.dumps(report, indent=2))
```

### 3. Full Script Structure (`filesystem_auditor.py`)

```python
#!/usr/bin/env python3

import os
import hashlib
import json
import re
import sys
import argparse
from datetime import datetime
from pathlib import Path

# --- Helper Functions (as defined above) ---

def get_file_metadata(filepath):
    # ... (same as above) ...
    stat = os.stat(filepath)
    return {
        "path": str(filepath), "name": filepath.name, "size": stat.st_size,
        "last_modified": datetime.fromtimestamp(stat.st_mtime).isoformat(),
        "is_dir": filepath.is_dir(), "is_file": filepath.is_file(), "is_symlink": filepath.is_symlink(),
    }

def find_files_recursive(start_path):
    # ... (same as above) ...
    for root, dirs, files in os.walk(start_path):
        for file in files: yield Path(root) / file

def generate_file_hash(filepath, algorithm="sha256"):
    # ... (same as above) ...
    hash_func = hashlib.new(algorithm)
    try:
        with open(filepath, "rb") as f:
            for chunk in iter(lambda: f.read(4096), b""): hash_func.update(chunk)
        return hash_func.hexdigest()
    except OSError: return "ERROR: Could not read file."
    except Exception as e: return f"ERROR: Hashing failed: {e}"

def generate_baseline(start_dir, output_file, algorithm="sha256"):
    # ... (same as above) ...
    baseline_data = {}
    for filepath in find_files_recursive(start_dir):
        file_hash = generate_file_hash(filepath, algorithm)
        if not file_hash.startswith("ERROR"):
            baseline_data[str(filepath)] = {
                "hash": file_hash, "size": filepath.stat().st_size,
                "mtime": datetime.fromtimestamp(filepath.stat().st_mtime).isoformat(),
                "algorithm": algorithm
            }
    with open(output_file, 'w') as f: json.dump(baseline_data, f, indent=2)
    print(f"Baseline saved to {output_file}")

def compare_to_baseline(start_dir, baseline_file, algorithm="sha256"):
    # ... (same as above) ...
    with open(baseline_file, 'r') as f: baseline = json.load(f)
    current_state = {}
    for filepath in find_files_recursive(start_dir):
        file_hash = generate_file_hash(filepath, algorithm)
        if not file_hash.startswith("ERROR"):
            current_state[str(filepath)] = {
                "hash": file_hash, "size": filepath.stat().st_size,
                "mtime": datetime.fromtimestamp(filepath.stat().st_mtime).isoformat(),
                "algorithm": algorithm
            }
    report = {"new_files": [], "deleted_files": [], "modified_files": [], "unchanged_files": []}
    for path, data in baseline.items():
        if path in current_state:
            if current_state[path]["hash"] != data["hash"]:
                report["modified_files"].append({"path": path, "baseline_hash": data["hash"], "current_hash": current_state[path]["hash"]})
            else: report["unchanged_files"].append({"path": path, "hash": data["hash"]})
        else: report["deleted_files"].append({"path": path, "hash": data["hash"]})
    for path, data in current_state.items():
        if path not in baseline: report["new_files"].append({"path": path, "hash": data["hash"]})
    return report

# --- Main Logic ---
def main():
    parser = argparse.ArgumentParser(description="File System Search and Integrity Check Utility.")
    subparsers = parser.add_subparsers(dest='command', help='Available commands')

    # Search command
    search_parser = subparsers.add_parser('search', help='Search files based on criteria.')
    search_parser.add_argument('path', type=str, help='Starting directory for the search.')
    search_parser.add_argument('--name', type=str, help='Search by file name (supports wildcards like * and ?).')
    search_parser.add_argument('--regex-name', type=str, help='Search by file name using regular expression.')
    search_parser.add_argument('--content', type=str, help='Search by content (substring match).')
    search_parser.add_argument('--regex-content', type=str, help='Search by content using regular expression.')
    search_parser.add_argument('--size-op', type=str, choices=['>', '<', '='], help='Operator for size comparison.')
    search_parser.add_argument('--size-val', type=int, help='Size value in bytes for comparison.')
    search_parser.add_argument('--mtime-op', type=str, choices=['>', '<'], help='Operator for modification time comparison.')
    search_parser.add_argument('--mtime-val', type=lambda s: datetime.strptime(s, '%Y-%m-%d'), help='Modification date (YYYY-MM-DD) for comparison.')
    search_parser.add_argument('--case-sensitive', action='store_true', help='Perform case-sensitive searches.')

    # Hash command
    hash_parser = subparsers.add_parser('hash', help='Generate hash for a file.')
    hash_parser.add_argument('file', type=str, help='File to hash.')
    hash_parser.add_argument('--algorithm', type=str, default='sha256', choices=hashlib.algorithms_available, help='Hashing algorithm (default: sha256).')

    # Baseline command
    baseline_parser = subparsers.add_parser('baseline', help='Generate or compare against a file integrity baseline.')
    baseline_parser.add_argument('path', type=str, help='Directory to monitor or compare.')
    baseline_parser.add_argument('action', type=str, choices=['generate', 'compare'], help='Action to perform: generate new baseline or compare against existing.')
    baseline_parser.add_argument('--file', type=str, required=True, help='Baseline file path (.json).')
    baseline_parser.add_argument('--algorithm', type=str, default='sha256', choices=hashlib.algorithms_available, help='Hashing algorithm (default: sha256).')

    args = parser.parse_args()

    if args.command == 'search':
        results = []
        for filepath_obj in find_files_recursive(args.path):
            file_meta = get_file_metadata(filepath_obj)
            match = True

            # Apply name filter
            if args.name:
                name_pattern = args.name.replace('*', '.*').replace('?', '.') # Convert wildcard to regex-like
                if not re.fullmatch(name_pattern, file_meta['name'], re.IGNORECASE if not args.case_sensitive else 0):
                    match = False
            elif args.regex_name:
                if not re.search(args.regex_name, file_meta['name'], re.IGNORECASE if not args.case_sensitive else 0):
                    match = False
            
            # Apply content filter
            if match and (args.content or args.regex_content):
                try:
                    with open(filepath_obj, 'r', encoding='utf-8', errors='ignore') as f:
                        content = f.read()
                        if args.content:
                            if not (args.content in content if args.case_sensitive else args.content.lower() in content.lower()):
                                match = False
                        elif args.regex_content:
                            if not re.search(args.regex_content, content, re.IGNORECASE if not args.case_sensitive else 0):
                                match = False
                except (OSError, UnicodeDecodeError):
                    match = False # Treat as no match if content cannot be read

            # Apply size filter
            if match and args.size_op and args.size_val is not None:
                if args.size_op == '>':
                    if not (file_meta['size'] > args.size_val): match = False
                elif args.size_op == '<':
                    if not (file_meta['size'] < args.size_val): match = False
                elif args.size_op == '=':
                    if not (file_meta['size'] == args.size_val): match = False

            # Apply mtime filter
            if match and args.mtime_op and args.mtime_val:
                file_mtime_ts = datetime.fromisoformat(file_meta['last_modified']).timestamp()
                target_mtime_ts = args.mtime_val.timestamp()
                if args.mtime_op == '>':
                    if not (file_mtime_ts > target_mtime_ts): match = False
                elif args.mtime_op == '<':
                    if not (file_mtime_ts < target_mtime_ts): match = False

            if match:
                results.append(file_meta)
        
        print(json.dumps(results, indent=2))

    elif args.command == 'hash':
        file_path_obj = Path(args.file)
        if not file_path_obj.is_file():
            print(f"Error: File '{args.file}' not found.", file=sys.stderr)
            sys.exit(1)
        file_hash = generate_file_hash(file_path_obj, args.algorithm)
        print(f"File: {args.file}
Algorithm: {args.algorithm}
Hash: {file_hash}")

    elif args.command == 'baseline':
        if args.action == 'generate':
            generate_baseline(args.path, args.file, args.algorithm)
        elif args.action == 'compare':
            if not Path(args.file).is_file():
                print(f"Error: Baseline file '{args.file}' not found.", file=sys.stderr)
                sys.exit(1)
            report = compare_to_baseline(args.path, args.file, args.algorithm)
            print(json.dumps(report, indent=2))

if __name__ == "__main__":
    main()
```

## Guiding Principles in Python

*   **Portability:** `os`, `pathlib`, `hashlib`, `re` are all part of Python's standard library, ensuring the script runs consistently across Windows, Linux, and macOS.
*   **Efficiency:** File system traversal is handled efficiently by `os.walk`. `hashlib` uses optimized hashing algorithms. Reading files in chunks (4096 bytes) prevents excessive memory consumption for large files during hashing.
*   **Minimal Dependencies:** The entire solution relies solely on Python's standard library. No external `pip` installations are required.
*   **CLI-centric:** `argparse` provides a professional and flexible command-line interface, allowing users to specify paths, criteria, and actions.
*   **Structured Data Handling:** All operations generate and consume structured Python objects (dictionaries, lists), which are then easily serialized to JSON for machine-readable output. This is crucial for integration with other tools and automated analysis.

## Conclusion

Python offers a powerful, flexible, and cross-platform environment for building sophisticated file system search and integrity checking utilities. By leveraging its rich standard library, you can implement complex search criteria and robust cryptographic integrity verification with ease. The ability to output structured data in formats like JSON makes Python tools highly valuable for security auditing, forensic analysis, and automated system monitoring. The next step is to apply this knowledge in practical exercises.
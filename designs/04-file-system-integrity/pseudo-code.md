# Design Concept: File System Search and Integrity Check

## I. Overview

This utility is designed to perform comprehensive searches across file systems and verify the integrity of files. It aims to provide a cross-platform, lightweight, and efficient tool crucial for security auditing, forensic analysis, compliance, and general system administration. The emphasis is on identifying files based on various criteria and detecting unauthorized modifications using cryptographic hashing, all with minimal external dependencies.

## Framework Alignment

This design for "**File System Search and Integrity Check**" provides a platform-agnostic blueprint for building and utilizing tools for a "Cross-Platform Baseline Hardening & Auditing Framework." The core functionalities of searching files and calculating cryptographic hashes are essential for auditing system configurations against defined security baselines and ensuring the integrity of critical system files across diverse operating environments.


## II. Core Functionality

### A. File System Search

1.  **Starting Path:** Specify one or more root directories from which the search will begin. The search should be recursive.
2.  **Search Criteria:**
    *   **Name:**
        *   Exact match.
        *   Wildcard patterns (e.g., `*.log`, `temp*`).
        *   Regular expression patterns.
        *   Case-sensitive/insensitive options.
    *   **Size:**
        *   Files larger than, smaller than, or exactly a specified size (e.g., `>1MB`, `<10KB`).
    *   **Time Attributes:**
        *   Last Modified Time (mtime): Files modified before/after a specific date/time or within a time range.
        *   Last Accessed Time (atime): Files accessed before/after a specific date/time.
        *   Creation Time (ctime): (Windows) Files created before/after a specific date/time. (Unix-like: ctime is inode change time).
    *   **Type:**
        *   Regular files, directories, symbolic links.
    *   **Permissions/Owner/Group (Unix-like systems):**
        *   Files with specific permissions (e.g., executable).
        *   Files owned by a specific user or group.
    *   **Content:**
        *   Files containing a specific string (exact match, case-insensitive).
        *   Files containing content matching a regular expression.
        *   *Note: Content search can be resource-intensive; warn users or make it an explicit option.*

3.  **Output for Search Results:**
    *   List of matching file paths.
    *   Optionally include additional metadata: file size, last modified date, permissions, owner, computed hash.
    *   Support for human-readable console output, CSV, or JSON.

### B. Integrity Check (Hashing)

1.  **Hashing Algorithm Selection:**
    *   Support for commonly used cryptographic hash algorithms: MD5, SHA1, SHA256. (SHA256 recommended for security; MD5/SHA1 for legacy/speed).
2.  **Hash Generation:**
    *   Compute the hash of a single specified file.
    *   Compute hashes for all files identified by a file system search.
3.  **Hash Verification:**
    *   Compare a newly computed hash against a previously recorded "known good" hash.
    *   Input for known good hash: direct string, or a baseline file.
4.  **Baseline Management (Basic):**
    *   **Generate Baseline:** Create a file (e.g., plain text, CSV, JSON) containing file paths and their hashes for a given directory structure at a specific point in time. This is the "known good" state.
    *   **Compare to Baseline:** Re-scan the same directory structure, compute current hashes, and compare them against the baseline file.
    *   **Discrepancy Reporting:** Output a report detailing:
        *   New files (present in current scan, not in baseline).
        *   Deleted files (present in baseline, not in current scan).
        *   Modified files (present in both, but hashes differ).
        *   Files with unchanged hashes.
    *   *Note: Baseline management can be complex if not handled carefully (e.g., handling expected changes). Keep it simple.*

### C. Output for Integrity Check

*   Standard Output (human-readable report of discrepancies or verification status).
*   JSON (structured report of all files and their status against a baseline).

### D. Error Handling

*   **Permission Denied:** Gracefully handle directories/files that cannot be accessed due to permissions.
*   **File Not Found:** Handle cases where files specified for hashing or verification do not exist.
*   **Invalid Path:** Report on invalid starting paths.
*   **I/O Errors:** Handle unexpected read/write errors during file operations.

## III. Data Structures

*   **Search Results:** A list of dictionaries/objects, each representing a matching file.
    `{"path": "/path/to/file.txt", "name": "file.txt", "size": 1234, "mtime": "2026-02-25T10:00:00Z", "hash_sha256": "..."}`
*   **Baseline Data:** A dictionary or list of dictionaries mapping canonical file paths to their hashes and possibly other metadata.
    `{"/path/to/file.txt": {"hash_sha256": "...", "size": 1234, "mtime": "..."}}`
*   **Discrepancy Report:** A list of dictionaries/objects detailing changes.
    `{"path": "/path/to/file.txt", "status": "Modified", "old_hash": "...", "new_hash": "..."}`

## IV. Guiding Principles (from cosolitdations.txt)

*   **Portability:** Implementations should use cross-platform file system functions and hashing libraries. Where OS-specific commands are used (e.g., `find`, `Get-ChildItem`), provide graceful fallback or alternatives.
*   **Efficiency:** File system traversal and hash computation can be resource-intensive. Optimize for speed (e.g., avoiding reading entire files into memory if only hash is needed, using efficient traversal methods).
*   **Minimal Dependencies:** Solutions should primarily use standard language utilities and built-in OS commands. Avoid large third-party file system abstraction layers or specialized integrity monitoring tools unless they are standard library components (e.g., Python's `hashlib`).
*   **CLI-centric:** The tool should be primarily controlled via command-line arguments, making it suitable for scripting and automation.
*   **Security Focus:** The integrity check aspect should be robust and clearly identify any unauthorized or unexpected changes to files, making it invaluable for security incident detection.

---
# Design Concept: Configuration Drift Detection and Remediation Guidance

## I. Overview

This utility is designed to identify "drift" in system configurations by comparing current system snapshots against a "Known Good" baseline (Golden Snapshot). Furthermore, it maps audit failures to specific, actionable remediation steps defined in security policies. This design completes the loop of the "Cross-Platform Baseline Hardening & Auditing Framework" by transforming raw audit data into advisory guidance.

## Framework Alignment

This design for "**Configuration Drift Detection and Remediation Guidance**" provides the advisory layer for the "Cross-Platform Baseline Hardening & Auditing Framework." It defines the logic required to detect unauthorized changes over time and provides administrators with the exact commands needed to bring systems back into compliance.

## II. Core Functionality

### A. Drift Detection (Snapshot Comparison)

1.  **Load Snapshots:** Retrieve two sets of structured data (JSON or SQLite tables):
    *   **Baseline (Golden Snapshot):** The reference state.
    *   **Current State:** The system's actual configuration at the moment of audit.
2.  **Differential Analysis:**
    *   **Added Items:** Identify elements present in the current state but missing from the baseline (e.g., new user, new open port).
    *   **Removed Items:** Identify elements present in the baseline but missing from the current state (e.g., a critical security service stopped).
    *   **Modified Items:** Identify elements present in both but with differing values (e.g., a file hash changed, a configuration parameter altered).
3.  **Severity Assignment:** Tag drifts based on their impact (e.g., a change in a root-owned file is "Critical").

### B. Remediation Guidance Mapping

1.  **Failure Identification:** Filter audit results for any check returning a `FAIL` status.
2.  **Guidance Retrieval:** For each failed check, look up the `remediation_guidance` field in the security policy (YAML/JSON).
3.  **Template Injection (Optional):** If the guidance contains placeholders (e.g., `<user>`), inject the relevant system variable identified during the audit.
4.  **Multi-Platform Guidance:** If the policy supports it, provide different fix commands for Bash (Linux) and PowerShell (Windows).

### C. Actionable Reporting

1.  **Comparison Report:** List all deltas identified during drift detection.
2.  **Advisory Audit Report:**
    *   State the Audit Result (`PASS`/`FAIL`).
    *   State the Rationale (Why it matters).
    *   State the **Remediation Fix** (How to fix it).
3.  **Output Formats:**
    *   **Human-Readable:** A "Fix-it" checklist for administrators.
    *   **JSON:** For integration into centralized management dashboards.

## III. Data Structures

*   **Drift Delta Object:**
    ```json
    {
        "item": "PermitRootLogin",
        "type": "Modification",
        "baseline": "no",
        "current": "yes",
        "impact": "High"
    }
    ```
*   **Remediation Guidance Object:**
    ```json
    {
        "control_id": "5.2.10",
        "status": "FAIL",
        "current_value": "MD5",
        "expected_value": "SHA512",
        "fix": "sudo sed -i 's/ENCRYPT_METHOD.*/ENCRYPT_METHOD SHA512/' /etc/login.defs"
    }
    ```

## IV. Guiding Principles

*   **Portability:** Use standard data comparison algorithms (e.g., set theory for lists, key-value comparison for dictionaries) that are easily implemented in Python, PowerShell, or SQL.
*   **Minimal Dependencies:** Avoid complex configuration management databases (CMDBs). Rely on local snapshot storage (JSON files or SQLite).
*   **Advisory, Not Active:** Focus on providing **guidance** to empower the administrator, rather than automated "fixing" which could break production systems without human review.
*   **CLI-centric:** Ensure the "Fix-it" strings are valid CLI commands that can be copy-pasted or piped into a shell.

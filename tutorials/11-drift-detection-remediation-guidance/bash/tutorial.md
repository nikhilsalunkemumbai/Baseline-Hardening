# Bash Tutorial: Configuration Drift Detection and Remediation Guidance

## Introduction

Bash, with its rich history of text processing, is highly effective for **Drift Detection** in line-oriented files (like `/etc/passwd` or configuration files). By using utilities like `diff`, `comm`, and `grep`, you can identify exactly how a file has drifted from its baseline state. This tutorial will demonstrate how to perform these comparisons and how to output copy-pasteable remediation commands based on audit findings.

## Framework Alignment

This tutorial on "**Configuration Drift Detection and Remediation Guidance**" using **Bash** demonstrates the foundational skills necessary to implement components of a "Cross-Platform Baseline Hardening & Auditing Framework." The techniques learned here for identifying configuration changes and providing command-line fixes are essential for maintaining Linux system security and compliance.

## Core Bash Logic for Drift Detection

### 1. Identifying Line-Level Drift (`diff` and `comm`)

To find what has changed between two sorted files (e.g., lists of users or open ports):

```bash
#!/bin/bash

baseline="baseline_users.txt"
current="current_users.txt"

# Sort both files before comparison
sort -o "$baseline" "$baseline"
sort -o "$current" "$current"

echo "--- DRIFT ANALYSIS ---"

# Items present in Baseline but MISSING in Current (Removed)
missing=$(comm -23 "$baseline" "$current")
if [[ -n "$missing" ]]; then
    echo "[!] REMOVED ITEMS:"
    echo "$missing"
fi

# Items present in Current but MISSING in Baseline (Added)
new_items=$(comm -13 "$baseline" "$current")
if [[ -n "$new_items" ]]; then
    echo "[!] ADDED ITEMS (Potential Drift):"
    echo "$new_items"
fi
```

### 2. Identifying Value Drift (`grep` and `awk`)

To check if a specific configuration parameter has changed value:

```bash
#!/bin/bash

config_file="/etc/login.defs"
param="PASS_MAX_DAYS"
expected="90"

# Extract current value
actual=$(grep "^$param" "$config_file" | awk '{print $2}')

if [[ "$actual" != "$expected" ]]; then
    echo "FAIL: $param is set to $actual (Expected: $expected)"
    # Remediation Guidance logic below
fi
```

### 3. Outputting Remediation Guidance

Provide the administrator with the exact `sed` or `systemctl` command needed to fix the issue.

```bash
#!/bin/bash

get_remediation() {
    local control_id="$1"
    case "$control_id" in
        "5.2.10") echo "sudo sed -i 's/^ENCRYPT_METHOD.*/ENCRYPT_METHOD SHA512/' /etc/login.defs" ;;
        "6.2.1")  echo "sudo sed -i 's/^PASS_MAX_DAYS.*/PASS_MAX_DAYS 90/' /etc/login.defs" ;;
        "3.4.1")  echo "sudo sysctl -w net.ipv4.ip_forward=0" ;;
        *)        echo "Consult Security Policy Documentation." ;;
    esac
}

# Example Usage
control="6.2.1"
fix=$(get_remediation "$control")

echo "-----------------------------------"
echo "AUDIT FAILURE detected for Control: $control"
echo "SUGGESTED FIX [Bash]: $fix"
echo "-----------------------------------"
```

## Guiding Principles in Bash

*   **Piping for Power:** Combine `grep`, `awk`, and `sed` to extract, compare, and suggest fixes in a single flow.
*   **Idempotency in Guidance:** Ensure your suggested fixes are safe to run even if already fixed (e.g., `sed` searching for the pattern correctly).
*   **Text-Centricity:** Since most Linux configurations are plain text, focusing on line-by-line comparison is the most reliable cross-platform method for Linux distributions.

## Conclusion

By using Bash to automate drift detection and provide remediation commands, you create tools that are highly operational and immediately useful for system administrators. Providing a "Fix-it" command alongside every failure removes friction from the hardening process and ensures security policies are enforced consistently. The next step is to apply these techniques in hands-on exercises.

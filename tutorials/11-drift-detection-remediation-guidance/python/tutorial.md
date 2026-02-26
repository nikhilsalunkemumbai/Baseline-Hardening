# Python Tutorial: Configuration Drift Detection and Remediation Guidance

## Introduction

Python's powerful dictionary and list manipulation capabilities, combined with its ability to handle structured data like JSON and YAML, make it an ideal language for detecting configuration drift and providing remediation guidance. This tutorial will demonstrate how to compare two system snapshots to identify changes and how to map audit failures to actionable "Fix-it" commands defined in a security policy.

## Framework Alignment

This tutorial on "**Configuration Drift Detection and Remediation Guidance**" using **Python** demonstrates the foundational skills necessary to implement components of a "Cross-Platform Baseline Hardening & Auditing Framework." The techniques learned here for comparing system states and providing remediation advice are essential for maintaining continuous compliance and security.

## Core Python Logic for Drift Detection

Drift detection involves identifying the differences between two Python dictionaries (representing the Baseline and the Current State).

### 1. Basic Dictionary Comparison

```python
def find_dictionary_drift(baseline, current):
    """
    Identifies additions, removals, and modifications between two dictionaries.
    """
    drift = {
        "added": {},
        "removed": {},
        "modified": {}
    }

    # Find Added and Modified items
    for key, value in current.items():
        if key not in baseline:
            drift["added"][key] = value
        elif baseline[key] != value:
            drift["modified"][key] = {
                "from": baseline[key],
                "to": value
            }

    # Find Removed items
    for key, value in baseline.items():
        if key not in current:
            drift["removed"][key] = value

    return drift

# Example Usage:
# baseline_snap = {"SSH": "Enabled", "Firewall": "Active", "User": "admin"}
# current_snap = {"SSH": "Disabled", "Firewall": "Active", "NewPort": 8080}
# print(find_dictionary_drift(baseline_snap, current_snap))
```

### 2. Remediation Guidance Mapping

To provide guidance, we map an audit failure to a remediation string found in our YAML policy.

```python
import yaml

def get_remediation(policy_file, control_id):
    """
    Retrieves the remediation guidance for a specific control ID from a YAML policy.
    """
    with open(policy_file, 'r') as f:
        policy = yaml.safe_load(f)
    
    for control in policy['controls']:
        if control['id'] == control_id:
            return control.get('remediation_guidance', "No guidance available.")
    return "Control ID not found."

# Example Usage:
# fix = get_remediation("cis_ubuntu_baseline.yaml", "5.2.10")
# print(f"FAIL: Password hashing is weak. FIX: {fix}")
```

### 3. Integrated Audit and Guidance Script

```python
import json
from datetime import datetime

def perform_advisory_audit(policy, system_state):
    """
    Audits a system and provides remediation guidance for failures.
    """
    audit_report = {
        "timestamp": datetime.now().isoformat(),
        "results": []
    }

    for control in policy['controls']:
        target = control['target']
        param = control['parameter']
        expected = control['expected_value']
        
        # Simple lookup logic (simulating Design 07/08)
        actual = system_state.get(param, "NOT_FOUND")
        
        status = "PASS" if str(actual) == str(expected) else "FAIL"
        
        result = {
            "id": control['id'],
            "title": control['title'],
            "status": status,
            "actual": actual,
            "expected": expected
        }
        
        if status == "FAIL":
            result["remediation"] = control.get('remediation_guidance', "Review system hardening guidelines.")
            
        audit_report["results"].append(result)
    
    return audit_report

# Example Policy Snippet
# policy = {
#     "controls": [
#         {"id": "1.1", "title": "SSH Port", "parameter": "port", "expected_value": "22", "remediation_guidance": "Edit /etc/ssh/sshd_config and set Port 22."}
#     ]
# }
# system_state = {"port": "2222"} # Drifted!
# report = perform_advisory_audit(policy, system_state)
# print(json.dumps(report, indent=2))
```

## Guiding Principles in Python

*   **Data Consistency:** Ensure that data types are compared correctly (e.g., strings vs. integers) during drift detection.
*   **Structured Advisory:** Output remediation as part of the JSON report to allow automated ticketing systems or UI dashboards to display the "Fix" next to the "Failure."
*   **Modular Comparison:** Build separate comparison functions for different data types (e.g., comparing lists of users vs. comparing a single configuration value).

## Conclusion

By using Python to automate drift detection and guidance retrieval, you transform your auditing tools into proactive assistants. This capability is critical for large-scale infrastructure management where identifying a problem is only half the battle—knowing how to fix it quickly and correctly is what ensures resilience. The next step is to apply these concepts in practical exercises.

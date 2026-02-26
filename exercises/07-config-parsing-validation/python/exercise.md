# Python Exercise: Configuration File Parsing and Validation

## Objective

This exercise challenges you to apply your Python scripting skills to parse different configuration file formats (INI-style and JSON) and validate their settings against security and operational policies. You will use Python's standard library modules (`configparser`, `json`, `re`) to extract specific values, perform checks, and report on compliance, demonstrating proficiency in automated configuration auditing.

## Framework Alignment

This exercise on "**Configuration File Parsing and Validation**" using **Python** provides practical experience in building and utilizing tools for a "Cross-Platform Baseline Hardening & Auditing Framework." By completing these tasks, you will learn how to audit and manage configuration settings, ensuring compliance with security policies and identifying unauthorized changes—essential steps in maintaining a secure and auditable environment.


## Scenario

You are a security engineer tasked with auditing the configuration files for a new application deployment. The application uses two main configuration files: `config.ini` for general server settings and `config.json` for database and high-level security parameters. Your goal is to develop a portable Python script that can read both files, extract critical settings, and verify their compliance with your organization's hardening guidelines.

## Setup

Before starting the tasks, create the following configuration files in your current directory:

### `config.ini`

```python
with open("config.ini", "w") as f:
    f.write("""[server]
port=8080
bind_ip=0.0.0.0

[security]
permit_root_login=yes
force_2fa=no

[logging]
level=DEBUG
log_file=/var/log/app.log
""")
print("Configuration file 'config.ini' created.")
```

### `config.json`

```python
import json
with open("config.json", "w") as f:
    json.dump({
      "database": {
        "host": "db.example.com",
        "port": 3306,
        "user": "appuser"
      },
      "security": {
        "enable_https": False,
        "min_tls_version": "TLSv1.1"
      },
      "performance": {
        "max_connections": 100
      }
    }, f, indent=2)
print("Configuration file 'config.json' created.")
```

Run these Python snippets to create the `config.ini` and `config.json` files.

## Tasks

Write a Python script (`config_auditor.py` or similar name) that, when executed, performs the following tasks. Your script should be structured with functions for clarity and output a structured JSON validation report to standard output.

### Part 1: Parsing Configuration Files

1.  **Parse `config.ini`:**
    *   Load `config.ini` using `configparser` into a Python object.

2.  **Parse `config.json`:**
    *   Load `config.json` using `json` into a Python object.

3.  **Extract `Server.Port` (from INI):**
    *   From the parsed `config.ini` object, extract the value for `port` in the `[server]` section.

4.  **Extract `Database.Port` (from JSON):**
    *   From the parsed `config.json` object, extract the value for `port` under the `database` section.

### Part 2: Configuration Validation

Your script should implement a validation mechanism. For each check, create a result object (e.g., a dictionary) containing `key_path`, `status` (`PASS`/`FAIL`), `message`, `expected_value`, and `actual_value`.

1.  **Policy Check (INI): `PermitRootLogin`:**
    *   Verify that `permit_root_login` in `[security]` section of `config.ini` is `no`. Report `FAIL` if it's `yes`.

2.  **Policy Check (INI): `Logging.Level`:**
    *   Verify that `level` in `[logging]` section of `config.ini` is NOT `DEBUG` (assuming production environment). Report `FAIL` if `DEBUG`.

3.  **Policy Check (JSON): `Database.Port`:**
    *   Verify that `database.port` in `config.json` is NOT `3306` (assuming `3306` is a default or insecure port). Report `FAIL` if `3306`.

4.  **Policy Check (JSON): `Security.EnableHttps`:**
    *   Verify that `security.enable_https` in `config.json` is `true`. Report `FAIL` if `false`.

5.  **Policy Check (JSON): `Security.MinTlsVersion`:**
    *   Verify that `security.min_tls_version` in `config.json` is `TLSv1.2` or higher (i.e., not `TLSv1.1`). You might need a regex or version comparison. Report `FAIL` if `TLSv1.1`.

6.  **Generate Consolidated Report:**
    *   Collect all validation results into a list of dictionaries.
    *   Output this list as a single JSON array to `stdout`.

## Deliverables

Provide the complete Python script file (`config_auditor.py`) that implements all the above tasks. Your script should be executable from the command line, taking the config file paths as arguments.

## Reflection Questions

1.  Compare using `configparser` for INI files versus `json` module for JSON files. What are the advantages of using dedicated parsing modules?
2.  How did you handle extracting nested values from the JSON configuration (e.g., `database.port`)?
3.  Explain the logic you used to validate string values (e.g., `PermitRootLogin`, `MinTlsVersion`) and boolean values (`EnableHttps`).
4.  Describe how outputting the validation results as JSON makes your Python script more versatile for integration with automated CI/CD pipelines or reporting dashboards.
5.  What are the advantages and disadvantages of using Python for configuration file parsing and validation compared to Bash, PowerShell, or a database like SQLite?

---
# Python Tutorial: Configuration File Parsing and Validation

## Introduction

Python's comprehensive standard library provides powerful and flexible modules for handling various configuration file formats, making it an excellent language for parsing and validating system or application settings. This tutorial will demonstrate how to read, extract, and validate configuration values from INI-style, JSON, YAML (with `PyYAML` as a common minimal dependency, or basic regex), and simple key-value files, all while adhering to our principles of minimal dependencies, cross-platform compatibility, and structured output.

## Framework Alignment

This tutorial on "**Configuration File Parsing and Validation**" using **Python** demonstrates the foundational skills necessary to implement components of a "Cross-Platform Baseline Hardening & Auditing Framework." The techniques learned here for parsing and validating configuration files are directly applicable to auditing systems against defined security baselines and ensuring compliance.


## Core Python Modules for Configuration Parsing

*   **`configparser`**: Implements a basic configuration language which provides a structure similar to what's found in Microsoft Windows INI files.
*   **`json`**: For working with JSON (JavaScript Object Notation) data.
*   **`yaml`**: (External library `PyYAML` - common minimal dependency) A parser for YAML. If strict standard library is required, basic YAML can be parsed with `re`.
*   **`re`**: Regular expression operations. Useful for parsing simple key-value files or basic regex-based validation.
*   **`os`**: For file system path manipulation and existence checks.
*   **`argparse`**: For parsing command-line arguments, making scripts user-friendly.

## Implementing Core Functionality with Python

### 1. Parsing INI-style Files (`configparser`)

Let's use a sample `app.conf`:
```ini
# Application Configuration
[database]
type=sqlite
host=localhost
port=5432
user=admin
password=securepassword
; This is another comment

[security]
enable_https=true
min_tls_version=TLSv1.2
permit_root_login=false
```

```python
import configparser
import json
import yaml # Requires pip install PyYAML
import re
import os
import sys
import argparse
from datetime import datetime
from pathlib import Path

def parse_ini_file(file_path):
    config = configparser.ConfigParser()
    config.read(file_path)
    # configparser can handle comments and sections directly
    return config

# Example Usage:
# config_file_ini = "app.conf"
# Path(config_file_ini).write_text("""... (content from above) ...""") # Create the file
# ini_config = parse_ini_file(config_file_ini)
# print("Database Port (INI):", ini_config['database']['port'])
# print("Permit Root Login (INI):", ini_config['security'].getboolean('permit_root_login'))
```

### 2. Parsing Simple Key-Value Pair Files (Custom)

Let's use a sample `.env` file:
```
DB_HOST=127.0.0.1
DB_USER=root
# DB_PASSWORD=secret
APP_DEBUG=true
```

```python
def parse_env_file(file_path):
    env_vars = {}
    with open(file_path, 'r') as f:
        for line in f:
            line = line.strip()
            if not line or line.startswith('#'):
                continue
            if '=' in line:
                key, value = line.split('=', 1)
                env_vars[key.strip()] = value.strip()
    return env_vars

# Example Usage:
# env_file = ".env"
# Path(env_file).write_text("""... (content from above) ...""") # Create the file
# env_config = parse_env_file(env_file)
# print("DB Host (.env):", env_config['DB_HOST'])
# print("App Debug (.env):", env_config['APP_DEBUG'])
```

### 3. Parsing JSON Files (`json`)

Let's use a sample `config.json`:
```json
{
  "database": {
    "type": "postgres",
    "port": 5432,
    "credentials": {
      "user": "appuser",
      "password": "securepassword"
    }
  },
  "security": {
    "enable_https": true,
    "min_tls_version": "TLSv1.3"
  },
  "logging": {
    "level": "INFO",
    "file": "/var/log/app.log"
  }
}
```

```python
def parse_json_file(file_path):
    with open(file_path, 'r') as f:
        return json.load(f)

# Example Usage:
# json_file = "config.json"
# Path(json_file).write_text("""... (content from above) ...""") # Create the file
# json_config = parse_json_file(json_file)
# print("DB Type (JSON):", json_config['database']['type'])
# print("Min TLS Version (JSON):", json_config['security']['min_tls_version'])
```

### 4. Parsing YAML Files (`PyYAML` or Basic Regex)

Let's use a sample `config.yaml`:
```yaml
database:
  type: mysql
  port: 3306
  credentials:
    user: webuser
    password: anothersecurepassword
security:
  enable_https: true
  min_tls_version: TLSv1.2
logging:
  level: DEBUG
  file: /var/log/web.log
```

```python
# Option 1: Using PyYAML (pip install PyYAML) - recommended for robust YAML parsing
try:
    import yaml
    def parse_yaml_file(file_path):
        with open(file_path, 'r') as f:
            return yaml.safe_load(f)
    # Example Usage with PyYAML:
    # yaml_file = "config.yaml"
    # Path(yaml_file).write_text("""... (content from above) ...""")
    # yaml_config = parse_yaml_file(yaml_file)
    # print("DB Type (YAML with PyYAML):", yaml_config['database']['type'])
except ImportError:
    print("Warning: PyYAML not installed. Basic YAML parsing might be limited.", file=sys.stderr)
    # Option 2: Basic regex parsing for simple key-value YAML (minimal dependency)
    # This is fragile and not recommended for complex YAML.
    def parse_simple_yaml_kv(file_path):
        data = {}
        with open(file_path, 'r') as f:
            for line in f:
                line = line.strip()
                if not line or line.startswith('#'): continue
                match = re.match(r'^(\S+):\s*(.*)$', line)
                if match:
                    key = match.group(1)
                    value = match.group(2).strip()
                    # Basic type conversion
                    if value.lower() == 'true': value = True
                    elif value.lower() == 'false': value = False
                    elif value.isdigit(): value = int(value)
                    data[key] = value
        return data
    
    # Example Usage with basic regex:
    # yaml_file = "config.yaml" # Assume this file is created
    # yaml_config_simple = parse_simple_yaml_kv(yaml_file)
    # print("DB Type (YAML via regex):", yaml_config_simple.get('type')) # Will only get top-level keys
```

### 5. Validation

```python
def validate_setting(config_data, key_path, expected_value=None, operator='equals', value_type=None, regex_pattern=None):
    """
    Validates a setting in the configuration.
    key_path can be 'section.key' for INI or 'path.to.value' for dicts.
    """
    status = "PASS"
    message = ""
    actual_value = None

    # Retrieve actual value
    if isinstance(config_data, configparser.ConfigParser):
        section, key = key_path.split('.', 1)
        actual_value = config_data.get(section, key, fallback=None)
    elif isinstance(config_data, dict):
        current_level = config_data
        for part in key_path.split('.'):
            if isinstance(current_level, dict) and part in current_level:
                current_level = current_level[part]
            else:
                actual_value = None
                break
        if actual_value is not None and isinstance(current_level, (str, int, bool, float)):
             actual_value = current_level
        else:
            actual_value = None
    
    if actual_value is None:
        status = "FAIL"
        message = f"Key '{key_path}' not found."
        return {"key": key_path, "status": status, "message": message}

    if value_type:
        try:
            if value_type == 'int': actual_value = int(actual_value)
            elif value_type == 'bool': actual_value = str(actual_value).lower() == 'true'
            # Add other types
        except ValueError:
            status = "FAIL"
            message = f"Value for '{key_path}' is not of expected type '{value_type}'."
            return {"key": key_path, "status": status, "message": message}

    if operator == 'equals' and expected_value is not None:
        if actual_value != expected_value:
            status = "FAIL"
            message = f"Expected '{expected_value}', found '{actual_value}'."
    elif operator == 'regex' and regex_pattern is not None:
        if not re.fullmatch(regex_pattern, str(actual_value)):
            status = "FAIL"
            message = f"Value '{actual_value}' does not match regex '{regex_pattern}'."
    elif operator == 'greater_than' and expected_value is not None and isinstance(actual_value, (int, float)):
        if not (actual_value > expected_value):
            status = "FAIL"
            message = f"Expected > '{expected_value}', found '{actual_value}'."
    # Add other operators like 'less_than', 'in_list', etc.

    if status == "PASS":
        message = f"Value is '{actual_value}' as expected."
    
    return {"key": key_path, "status": status, "message": message, "actual": actual_value, "expected": expected_value}

# Example Usage:
# print("--- Validation ---")
# # For JSON config:
# print(validate_setting(json_config, "security.min_tls_version", expected_value="TLSv1.3"))
# print(validate_setting(json_config, "database.port", expected_value=5432, operator="equals", value_type="int"))
# print(validate_setting(json_config, "security.enable_https", expected_value=True, operator="equals", value_type="bool"))
# print(validate_setting(json_config, "database.port", expected_value=1024, operator="greater_than", value_type="int"))

# # For INI config:
# print(validate_setting(ini_config, "security.permit_root_login", expected_value=False, operator="equals", value_type="bool"))
```

### 6. Full Script Structure (`config_validator.py`)

```python
#!/usr/bin/env python3

import configparser
import json
import yaml # Requires pip install PyYAML
import re
import os
import sys
import argparse
from datetime import datetime
from pathlib import Path

# --- Parsing Functions (as defined above) ---
def parse_ini_file(file_path):
    config = configparser.ConfigParser()
    config.read(file_path)
    return config

def parse_env_file(file_path):
    env_vars = {}
    with open(file_path, 'r') as f:
        for line in f:
            line = line.strip()
            if not line or line.startswith('#'): continue
            if '=' in line:
                key, value = line.split('=', 1)
                env_vars[key.strip()] = value.strip()
    return env_vars

def parse_json_file(file_path):
    with open(file_path, 'r') as f:
        return json.load(f)

# Use PyYAML if available, otherwise a placeholder or simple regex parser
try:
    import yaml
    def parse_yaml_file(file_path):
        with open(file_path, 'r') as f: return yaml.safe_load(f)
except ImportError:
    def parse_yaml_file(file_path):
        print("Warning: PyYAML not installed. Attempting basic regex parsing for YAML.", file=sys.stderr)
        data = {}
        with open(file_path, 'r') as f:
            for line in f:
                line = line.strip()
                if not line or line.startswith('#'): continue
                match = re.match(r'^(\S+):\s*(.*)$', line)
                if match:
                    key = match.group(1); value = match.group(2).strip()
                    if value.lower() == 'true': value = True
                    elif value.lower() == 'false': value = False
                    elif value.isdigit(): value = int(value)
                    data[key] = value
        return data

# --- Validation Function (as defined above) ---
def validate_setting(config_data, key_path, expected_value=None, operator='equals', value_type=None, regex_pattern=None):
    # ... (same as above) ...
    status = "PASS"; message = ""; actual_value = None
    if isinstance(config_data, configparser.ConfigParser):
        section, key = key_path.split('.', 1)
        actual_value = config_data.get(section, key, fallback=None)
    elif isinstance(config_data, dict):
        current_level = config_data
        for part in key_path.split('.'):
            if isinstance(current_level, dict) and part in current_level: current_level = current_level[part]
            else: actual_value = None; break
        if actual_value is not None and isinstance(current_level, (str, int, bool, float)): actual_value = current_level
        else: actual_value = None
    if actual_value is None:
        status = "FAIL"; message = f"Key '{key_path}' not found."
        return {"key": key_path, "status": status, "message": message}
    if value_type:
        try:
            if value_type == 'int': actual_value = int(actual_value)
            elif value_type == 'bool': actual_value = str(actual_value).lower() == 'true'
        except ValueError:
            status = "FAIL"; message = f"Value for '{key_path}' is not of expected type '{value_type}'."
            return {"key": key_path, "status": status, "message": message}
    if operator == 'equals' and expected_value is not None:
        if actual_value != expected_value: status = "FAIL"; message = f"Expected '{expected_value}', found '{actual_value}'."
    elif operator == 'regex' and regex_pattern is not None:
        if not re.fullmatch(regex_pattern, str(actual_value)): status = "FAIL"; message = f"Value '{actual_value}' does not match regex '{regex_pattern}'."
    elif operator == 'greater_than' and expected_value is not None and isinstance(actual_value, (int, float)):
        if not (actual_value > expected_value): status = "FAIL"; message = f"Expected > '{expected_value}', found '{actual_value}'."
    if status == "PASS": message = f"Value is '{actual_value}' as expected."
    return {"key": key_path, "status": status, "message": message, "actual": actual_value, "expected": expected_value}

# --- Main Logic ---
def main():
    parser = argparse.ArgumentParser(description="Configuration File Parser and Validator.")
    parser.add_argument("config_file", type=str, help="Path to the configuration file.")
    parser.add_argument("config_type", type=str, choices=['ini', 'json', 'yaml', 'env'], help="Type of configuration file.")
    parser.add_argument("--policy", type=str, help="Path to a JSON file containing validation policies.")
    parser.add_argument("-oJ", "--output-json", action="store_true", help="Output results in JSON format.")
    
    args = parser.parse_args()

    if not Path(args.config_file).is_file():
        print(f"Error: Configuration file '{args.config_file}' not found.", file=sys.stderr)
        sys.exit(1)

    parsed_config = None
    try:
        if args.config_type == 'ini':
            parsed_config = parse_ini_file(args.config_file)
        elif args.config_type == 'json':
            parsed_config = parse_json_file(args.config_file)
        elif args.config_type == 'yaml':
            parsed_config = parse_yaml_file(args.config_file)
        elif args.config_type == 'env':
            parsed_config = parse_env_file(args.config_file)
    except Exception as e:
        print(f"Error parsing {args.config_type} file '{args.config_file}': {e}", file=sys.stderr)
        sys.exit(1)

    if parsed_config is None:
        print(f"Error: Could not parse configuration file '{args.config_file}'.", file=sys.stderr)
        sys.exit(1)

    validation_results = []
    if args.policy:
        if not Path(args.policy).is_file():
            print(f"Error: Policy file '{args.policy}' not found.", file=sys.stderr)
            sys.exit(1)
        with open(args.policy, 'r') as f:
            policies = json.load(f)
        
        for policy in policies:
            result = validate_setting(
                parsed_config,
                policy['key_path'],
                expected_value=policy.get('expected_value'),
                operator=policy.get('operator', 'equals'),
                value_type=policy.get('value_type'),
                regex_pattern=policy.get('regex_pattern')
            )
            validation_results.append(result)
    else:
        # If no policy, just output the parsed config
        if args.output_json:
            if isinstance(parsed_config, configparser.ConfigParser):
                # Convert ConfigParser object to a dict for JSON output
                parsed_config_dict = {section: dict(parsed_config[section]) for section in parsed_config.sections()}
                print(json.dumps(parsed_config_dict, indent=2))
            else:
                print(json.dumps(parsed_config, indent=2))
        else:
            if isinstance(parsed_config, configparser.ConfigParser):
                parsed_config.write(sys.stdout) # Prints INI format
            else:
                print(json.dumps(parsed_config, indent=2)) # Default to JSON for dicts
        sys.exit(0)

    # Output validation results
    if args.output_json:
        print(json.dumps(validation_results, indent=2))
    else:
        print("--- Configuration Validation Report ---")
        for result in validation_results:
            status_color = "\033[92mPASS\033[0m" if result['status'] == "PASS" else "\033[91mFAIL\033[0m"
            print(f"{status_color}: {result['key']} - {result['message']}")

if __name__ == "__main__":
    main()
```

## Guiding Principles in Python

*   **Portability:** Python's standard library modules (`configparser`, `json`, `re`, `os`, `pathlib`) are inherently cross-platform. `subprocess` can be used for platform-specific tools if needed.
*   **Efficiency:** Python's parsing modules are efficient. For very large files, line-by-line processing can manage memory.
*   **Minimal Dependencies:** This tutorial primarily uses standard library modules. `PyYAML` is mentioned as a common *minimal* external dependency for robust YAML parsing.
*   **CLI-centric:** The script uses `argparse` to create a flexible command-line interface, accepting config file paths, types, and policy files.
*   **Structured Data Handling:** Parsed configurations are converted into native Python objects (dictionaries, `ConfigParser` objects), making extraction and validation programmatic and robust, without relying on fragile text parsing. Results are easily serialized to JSON.

## Conclusion

Python offers a powerful, flexible, and cross-platform environment for building robust configuration file parsing and validation utilities. Its rich standard library provides dedicated modules for various formats, simplifying data extraction and enabling policy-driven security checks. The ability to output structured data in JSON makes Python tools highly valuable for automated configuration management, security auditing, and compliance reporting. The next step is to apply this knowledge in practical exercises.
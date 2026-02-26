# Bash Tutorial: Configuration File Parsing and Validation

## Introduction

Bash, through its suite of powerful text processing utilities (`grep`, `sed`, `awk`, `cut`), is exceptionally well-suited for parsing and validating various configuration file formats, especially those that are line-oriented or semi-structured. This tutorial will guide you through using these tools to extract specific settings, check for expected values, and enforce configuration policies, all while adhering to our principles of minimal dependencies and CLI-centric operation.

## Framework Alignment

This tutorial on "**Configuration File Parsing and Validation**" using **Bash** demonstrates the foundational skills necessary to implement components of a "Cross-Platform Baseline Hardening & Auditing Framework." The techniques learned here for parsing and validating configuration files are directly applicable to auditing systems against defined security baselines and ensuring compliance.


## Core Bash Utilities for Configuration Parsing

*   **`grep`**: Searches for patterns in files. Excellent for finding specific lines or sections.
    *   `-E`: Extended regex.
    *   `-v`: Invert match.
    *   `-i`: Ignore case.
*   **`sed`**: Stream editor for filtering and transforming text. Useful for removing comments, extracting parts of a line, or making simple substitutions.
*   **`awk`**: A powerful pattern scanning and processing language. Ideal for field-based parsing and complex conditional logic.
    *   `-F`: Specify field separator.
*   **`cut`**: Removes sections from each line of files. Good for simple delimited files.
    *   `-d`: Specify delimiter.
    *   `-f`: Specify field number.
*   **`test`** / **`[`**: Evaluate conditional expressions.
*   **`if` / `elif` / `else`**: Conditional branching.
*   **`tr`**: Translate or delete characters.

## Implementing Core Functionality with Bash

### 1. Parsing INI-style Files

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

#### a. Extract a specific value from a section

```bash
#!/bin/bash

CONFIG_FILE="app.conf"

# Function to get a value from an INI-style file
get_ini_value() {
    local section="$1"
    local key="$2"
    local file="$3"

    # 1. Locate the section header
    # 2. Extract lines until next section or end of file
    # 3. Filter for the key=value line
    # 4. Remove key= and any comments
    # 5. Trim whitespace
    grep -A 1000 "\[$section\]" "$file" | 
    grep -E "^$key=" | 
    head -n 1 | 
    sed -E "s/^$key=([^;#]*).*/\1/" | 
    tr -d ' '
}

echo "Database Port: $(get_ini_value "database" "port" "$CONFIG_FILE")"
echo "Permit Root Login: $(get_ini_value "security" "permit_root_login" "$CONFIG_FILE")"
```

### 2. Parsing Simple Key-Value Pair Files

Let's use a sample `.env` file:
```
DB_HOST=127.0.0.1
DB_USER=root
# DB_PASSWORD=secret
APP_DEBUG=true
```

#### a. Extract a specific value by key

```bash
#!/bin/bash

ENV_FILE=".env"

# Function to get a value from a simple KEY=VALUE file
get_env_value() {
    local key="$1"
    local file="$2"

    grep -E "^$key=" "$file" | 
    head -n 1 | 
    cut -d'=' -f2- | 
    sed 's/^[[:space:]]*//;s/[[:space:]]*$//' # Trim whitespace
}

echo "DB Host: $(get_env_value "DB_HOST" "$ENV_FILE")"
echo "App Debug: $(get_env_value "APP_DEBUG" "$ENV_FILE")"
```

### 3. Parsing Custom Delimited Files (e.g., `/etc/passwd`-like)

Let's assume a simplified `users.csv` with `username:UID:home_dir:shell`
```
root:0:/root:/bin/bash
user1:1001:/home/user1:/bin/bash
sysuser:500:/var/empty:/sbin/nologin
```

#### a. Extract specific fields

```bash
#!/bin/bash

USERS_FILE="users.csv"

# Function to get shell for a username
get_user_shell() {
    local username="$1"
    local file="$2"

    grep "^$username:" "$file" | 
    awk -F':' '{print $4}' # Assuming shell is the 4th field
}

echo "root's shell: $(get_user_shell "root" "$USERS_FILE")"
echo "user1's home directory: $(grep "^user1:" "$USERS_FILE" | awk -F':' '{print $3}')"
```

### 4. Validation

#### a. Check if a key exists

```bash
# Check if 'enable_https' exists in [security] section of app.conf
if get_ini_value "security" "enable_https" "app.conf" > /dev/null; then
    echo "enable_https setting found."
else
    echo "enable_https setting NOT found."
fi
```

#### b. Validate a value against an expected value

```bash
EXPECTED_PORT="8080"
ACTUAL_PORT=$(get_ini_value "database" "port" "app.conf")

if [ "$ACTUAL_PORT" -eq "$EXPECTED_PORT" ]; then # Use -eq for numeric comparison
    echo "Database port is set correctly to $EXPECTED_PORT."
else
    echo "WARNING: Database port is '$ACTUAL_PORT', expected '$EXPECTED_PORT'."
fi
```

#### c. Policy Compliance Check (e.g., `PermitRootLogin no`)

```bash
PERMIT_ROOT_LOGIN_SETTING=$(get_ini_value "security" "permit_root_login" "app.conf")

if [ "$PERMIT_ROOT_LOGIN_SETTING" = "false" ]; then
    echo "Security check PASSED: PermitRootLogin is 'false'."
else
    echo "SECURITY ALERT: PermitRootLogin is '$PERMIT_ROOT_LOGIN_SETTING', should be 'false'!"
fi
```

### 5. Handling Comments and Whitespace

The `get_ini_value` and `get_env_value` functions already include basic comment and whitespace handling.
*   `grep -E "^$key="`: Ensures it matches line start and key directly, ignoring lines that are just comments.
*   `sed -E "s/^$key=([^;#]*).*/\1/"`: Extracts value up to a comment character (`#` or `;`).
*   `tr -d ' '` and `sed 's/^[[:space:]]*//;s/[[:space:]]*$//'`: Remove leading/trailing whitespace.

## Building a Configuration Validator Script

```bash
#!/bin/bash

CONFIG_FILE="app.conf"
VALIDATION_REPORT="validation_report.txt"

# Helper function to get INI values (as defined above)
get_ini_value() {
    local section="$1"; local key="$2"; local file="$3"
    grep -A 1000 "\[$section\]" "$file" | grep -E "^$key=" | head -n 1 | sed -E "s/^$key=([^;#]*).*/\1/" | tr -d ' '
}

echo "--- Configuration Validation Report for $CONFIG_FILE ---" > "$VALIDATION_REPORT"

# Check 1: Database port must be 5432
EXPECTED_DB_PORT="5432"
ACTUAL_DB_PORT=$(get_ini_value "database" "port" "$CONFIG_FILE")
if [ "$ACTUAL_DB_PORT" = "$EXPECTED_DB_PORT" ]; then
    echo "PASS: Database port is set to $EXPECTED_DB_PORT." >> "$VALIDATION_REPORT"
else
    echo "FAIL: Database port is '$ACTUAL_DB_PORT', expected '$EXPECTED_DB_PORT'." >> "$VALIDATION_REPORT"
fi

# Check 2: PermitRootLogin must be false
EXPECTED_PRL="false"
ACTUAL_PRL=$(get_ini_value "security" "permit_root_login" "$CONFIG_FILE")
if [ "$ACTUAL_PRL" = "$EXPECTED_PRL" ]; then
    echo "PASS: PermitRootLogin is 'false'." >> "$VALIDATION_REPORT"
else
    echo "FAIL: PermitRootLogin is '$ACTUAL_PRL', expected 'false'." >> "$VALIDATION_REPORT"
fi

# Check 3: enable_https must be true
EXPECTED_HTTPS="true"
ACTUAL_HTTPS=$(get_ini_value "security" "enable_https" "$CONFIG_FILE")
if [ "$ACTUAL_HTTPS" = "$EXPECTED_HTTPS" ]; then
    echo "PASS: HTTPS is enabled." >> "$VALIDATION_REPORT"
else
    echo "FAIL: HTTPS is '$ACTUAL_HTTPS', expected 'true'." >> "$VALIDATION_REPORT"
fi

echo "Validation complete. Report saved to $VALIDATION_REPORT"
cat "$VALIDATION_REPORT"
```

## Guiding Principles in Bash

*   **Portability:** `grep`, `sed`, `awk`, `cut`, `head`, `tr` are standard on virtually all Unix-like systems.
*   **Efficiency:** These utilities are compiled binaries, making them very fast for text processing. Piping efficiently streams data between them.
*   **Minimal Dependencies:** Relies entirely on core system utilities.
*   **CLI-centric:** All operations are command-line based, ideal for scripting and quick checks.
*   **Security Focus:** Bash is highly effective for parsing and validating plain-text configuration files, which are common for security-sensitive settings.

## Conclusion

Bash, through its powerful text processing tools, offers a flexible and efficient way to parse and validate configuration files. While it may require more intricate piping and regular expressions compared to dedicated parsing libraries in other languages, its ubiquitous presence on Unix-like systems and direct approach make it invaluable for quick checks, automation, and basic compliance auditing. The next step is to apply this knowledge in practical exercises.
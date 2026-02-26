# Troubleshooting: IT Hardening Framework

Common technical issues and their resolutions for the Cross-Platform IT Utility Snippet Library.

## 1. Script Execution Issues

### Bash: "Command not found" or `` errors
*   **Cause**: Running scripts edited on Windows in a Linux environment (CRLF line endings).
*   **Fix**: Convert line endings to LF. In VS Code, use the bottom-right status bar or run `dos2unix <filename>`.

### PowerShell: "Scripts are disabled"
*   **Cause**: Windows default execution policy.
*   **Fix**: Run `Set-ExecutionPolicy -ExecutionPolicy Bypass -Scope Process` for your current terminal session.

### Python: "ModuleNotFoundError"
*   **Cause**: Missing `PyYAML` or `psutil` in the current environment.
*   **Fix**: Ensure your virtual environment is active and run `pip install -r requirements.txt` (or install manually).

## 2. Audit & Permission Errors

### Permission Denied (Access Denied)
*   **Cause**: Auditing system files (`/etc/shadow`, Registry) or services requires elevated rights.
*   **Fix**: Run your terminal as **Administrator** (Windows) or use `sudo` (Linux).

### SQLite: "Database is Locked"
*   **Cause**: Another process has an uncommitted transaction on the `.db` file.
*   **Fix**: Close any database browsers or separate `sqlite3` CLI sessions.

## 3. Logic & Data Issues

### YAML Parsing Errors
*   **Cause**: Indentation errors in `cis_ubuntu_baseline.yaml`.
*   **Fix**: Use spaces instead of tabs. Validate your YAML using `python -m yaml.tool <file.yaml>`.

### CIDR Expansion Failure
*   **Cause**: Entering invalid network strings in the Network Scanner (03).
*   **Fix**: Ensure strings follow `x.x.x.x/yy` format. The Python `ipaddress` module is strict.

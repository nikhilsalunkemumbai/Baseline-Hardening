# Setup Guide: IT Utility Snippet Library

This guide ensures your environment is ready to implement, test, and audit the 11 core IT utility designs using Bash, PowerShell, Python, and SQLite.

## 1. Runtime Prerequisites

Ensure the following are installed and accessible in your system PATH:

*   **Python 3.9+**: Required for the master auditing framework and Python tutorials.
*   **PowerShell Core 7+**: Recommended for consistent behavior across Windows and Linux.
*   **Bash**: Native on Linux/macOS. Use **WSL** or **Git Bash** on Windows.
*   **SQLite 3**: Required for structured log storage and historical auditing.

## 2. Dependency Configuration

### Python Framework Integration
The `master_audit.py` and certain tutorials require the `PyYAML` library.

```bash
# Recommended: Use a virtual environment
python -m venv .venv
# Activate (Windows): .venv\Scripts\activate
# Activate (Linux): source .venv/bin/activate

pip install pyyaml watchdog psutil
```

### PowerShell Permissions
On Windows, you must allow the execution of local scripts:
```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

## 3. Repository Navigation
The library is organized to take you from concept to implementation:
1.  **`/designs`**: Technology-agnostic logic (Pseudo-code).
2.  **`/tutorials`**: Step-by-step implementation in your preferred language.
3.  **`/exercises`**: Hands-on challenges to test your implementations.
4.  **`/framework-integration`**: Examples of combining snippets into a unified auditor.

## 4. Environment Validation
Check your setup by running:
```bash
python --version
pwsh --version
sqlite3 --version
bash --version
```

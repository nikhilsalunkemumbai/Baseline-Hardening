# Python Tutorial: Basic User/Group Management and Audit

## Introduction

Python's standard library provides fundamental tools for interacting with the operating system, making it suitable for basic user and group management tasks and auditing. This tutorial will demonstrate how to list, inspect, and (via subprocess calls to system commands) manage users and groups. We will emphasize standard library modules and leverage external commands for modifications and additions, as direct Python APIs for these tasks vary significantly across operating systems or require additional, non-standard libraries. The focus remains on minimal dependencies and cross-platform compatibility.

## Framework Alignment

This tutorial on "**User/Group Management and Audit**" using **Python** demonstrates the foundational skills necessary to implement components of a "Cross-Platform Baseline Hardening & Auditing Framework." The techniques learned here for managing user accounts and group memberships are directly applicable to auditing systems against defined security baselines and ensuring compliance.


## Core Python Modules for User/Group Management

*   **`os`**: Provides functions for interacting with the operating system.
*   **`subprocess`**: Essential for running external system commands like `useradd`, `usermod`, `groupadd`, `net user`, `net localgroup`.
*   **`pwd`**: (Unix-like only) Access to the password database (user accounts).
*   **`grp`**: (Unix-like only) Access to the group database.
*   **`json`**: For serializing collected data into JSON format.
*   **`platform`**: To detect the operating system and run appropriate commands.
*   **`re`**: For robust parsing of text output from external commands.

## Implementing Core Functionality with Python

### Utility Function for Running Commands

```python
import os
import subprocess
import json
import re
import platform
import sys
import pwd # Unix-like
import grp # Unix-like
from datetime import datetime

def run_command(command, shell=False, check=True, input_data=None):
    """Runs a shell command and returns its stdout. Handles errors."""
    try:
        result = subprocess.run(
            command,
            shell=shell,
            capture_output=True,
            text=True,
            check=check, # Raise an exception for non-zero exit codes only if check=True
            input=input_data # Pass input to stdin of the command
        )
        return result.stdout.strip()
    except subprocess.CalledProcessError as e:
        return f"ERROR: Command '{' '.join(command) if isinstance(command, list) else command}' failed: {e.stderr.strip()}"
    except FileNotFoundError:
        return f"ERROR: Command '{command[0] if isinstance(command, list) else command.split()[0]}' not found."
    except Exception as e:
        return f"ERROR: An unexpected error occurred: {e}"

```

### 1. User Management

#### a. List All Local Users

```python
def list_local_users():
    users = []
    system = platform.system()

    if system == "Linux" or system == "Darwin": # Unix-like
        # Using pwd module for direct access to user database
        for p in pwd.getpwall():
            users.append({
                "username": p.pw_name,
                "uid": p.pw_uid,
                "gid": p.pw_gid,
                "home_dir": p.pw_dir,
                "shell": p.pw_shell,
                "full_name": p.pw_gecos,
            })
        # For password expiry, would need to parse `chage -l <user>` output
        # For locked status, would need to parse /etc/shadow or `passwd -S <user>`
    elif system == "Windows":
        # Using 'net user' command and parsing output
        net_user_output = run_command(["net", "user"])
        if not net_user_output.startswith("ERROR"):
            lines = net_user_output.splitlines()
            if len(lines) > 2: # Skip header and blank line
                user_list_line = lines[-1] # The last line usually contains usernames
                usernames = user_list_line.split()
                # Exclude built-in "Administrator" and "Guest" if desired, or handle for other accounts
                for username in usernames:
                    if username: # Ensure not empty string
                        user_details_output = run_command(["net", "user", username])
                        details = {}
                        for line in user_details_output.splitlines():
                            if ":" in line:
                                key, value = line.split(":", 1)
                                details[key.strip()] = value.strip()
                        users.append({
                            "username": username,
                            "full_name": details.get("Full Name"),
                            "comments": details.get("Comment"),
                            "account_active": True if details.get("Account active") == "Yes" else False,
                            "password_expires": details.get("Password expires"),
                            # More parsing needed for other details
                        })
    return users

# Example Usage:
# print("--- Local Users ---")
# print(json.dumps(list_local_users(), indent=2))
```

#### b. Add a New User (via `subprocess`)

```python
def add_user(username, password, system_user=False):
    system = platform.system()
    try:
        if system == "Linux":
            # Requires root
            command = ["sudo", "useradd"]
            if system_user: command.append("-r") # System user
            command.append(username)
            run_command(command)
            # Set password
            run_command(["sudo", "passwd", username], input_data=f"{password}
{password}
")
            return f"User '{username}' added."
        elif system == "Windows":
            # Requires administrator
            run_command(["net", "user", username, password, "/add"])
            return f"User '{username}' added."
        else:
            return "ERROR: User creation not implemented for this OS."
    except Exception as e:
        return f"ERROR adding user '{username}': {e}"

# Example Usage:
# print(add_user("testuser_py", "PyP@ssw0rd123"))
```

#### c. Modify User Attributes (via `subprocess`)

```python
def modify_user_password(username, new_password):
    system = platform.system()
    try:
        if system == "Linux":
            run_command(["sudo", "passwd", username], input_data=f"{new_password}
{new_password}
")
            return f"Password for '{username}' changed."
        elif system == "Windows":
            run_command(["net", "user", username, new_password])
            return f"Password for '{username}' changed."
        else:
            return "ERROR: Password modification not implemented for this OS."
    except Exception as e:
        return f"ERROR modifying password for '{username}': {e}"

def lock_user_account(username):
    system = platform.system()
    try:
        if system == "Linux":
            run_command(["sudo", "usermod", "-L", username])
            return f"User '{username}' locked."
        elif system == "Windows":
            run_command(["net", "user", username, "/active:no"])
            return f"User '{username}' disabled."
        else:
            return "ERROR: Account locking not implemented for this OS."
    except Exception as e:
        return f"ERROR locking user '{username}': {e}"

# Example Usage:
# print(modify_user_password("testuser_py", "NewP@ssw0rd123"))
# print(lock_user_account("testuser_py"))
```

#### d. Delete a User (via `subprocess`)

```python
def delete_user(username, delete_home=False):
    system = platform.system()
    try:
        if system == "Linux":
            command = ["sudo", "userdel"]
            if delete_home: command.append("-r")
            command.append(username)
            run_command(command)
            return f"User '{username}' deleted."
        elif system == "Windows":
            run_command(["net", "user", username, "/delete"])
            return f"User '{username}' deleted."
        else:
            return "ERROR: User deletion not implemented for this OS."
    except Exception as e:
        return f"ERROR deleting user '{username}': {e}"

# Example Usage:
# print(delete_user("testuser_py", delete_home=True))
```

### 2. Group Management

#### a. List All Local Groups

```python
def list_local_groups():
    groups = []
    system = platform.system()

    if system == "Linux" or system == "Darwin": # Unix-like
        for g in grp.getgrall():
            groups.append({
                "groupname": g.gr_name,
                "gid": g.gr_gid,
                "members": g.gr_mem # List of usernames
            })
    elif system == "Windows":
        # Using 'net localgroup' command and parsing output
        net_localgroup_output = run_command(["net", "localgroup"])
        if not net_localgroup_output.startswith("ERROR"):
            lines = net_localgroup_output.splitlines()
            group_names = []
            parsing_groups = False
            for line in lines:
                if "-------" in line: parsing_groups = True; continue
                if "The command completed successfully." in line: break
                if parsing_groups:
                    if line.strip(): group_names.append(line.strip())
            
            for groupname in group_names:
                if groupname:
                    group_details_output = run_command(["net", "localgroup", groupname])
                    members_found = False
                    members_list = []
                    for line in group_details_output.splitlines():
                        if "-------" in line: members_found = True; continue
                        if "The command completed successfully." in line: break
                        if members_found:
                            if line.strip(): members_list.append(line.strip())
                    groups.append({
                        "groupname": groupname,
                        "members": members_list
                        # GID is harder to get directly via net localgroup
                    })
    return groups

# Example Usage:
# print("--- Local Groups ---")
# print(json.dumps(list_local_groups(), indent=2))
```

#### b. Add a New Group (via `subprocess`)

```python
def add_group(groupname):
    system = platform.system()
    try:
        if system == "Linux":
            run_command(["sudo", "groupadd", groupname])
            return f"Group '{groupname}' added."
        elif system == "Windows":
            run_command(["net", "localgroup", groupname, "/add"])
            return f"Group '{groupname}' added."
        else:
            return "ERROR: Group creation not implemented for this OS."
    except Exception as e:
        return f"ERROR adding group '{groupname}': {e}"

# Example Usage:
# print(add_group("pydevs"))
```

#### c. Add/Remove User to/from Group (via `subprocess`)

```python
def add_user_to_group(username, groupname):
    system = platform.system()
    try:
        if system == "Linux":
            run_command(["sudo", "usermod", "-aG", groupname, username])
            return f"User '{username}' added to group '{groupname}'."
        elif system == "Windows":
            run_command(["net", "localgroup", groupname, username, "/add"])
            return f"User '{username}' added to group '{groupname}'."
        else:
            return "ERROR: Adding user to group not implemented for this OS."
    except Exception as e:
        return f"ERROR adding user '{username}' to group '{groupname}': {e}"

def remove_user_from_group(username, groupname):
    system = platform.system()
    try:
        if system == "Linux":
            run_command(["sudo", "gpasswd", "-d", username, groupname])
            return f"User '{username}' removed from group '{groupname}'."
        elif system == "Windows":
            run_command(["net", "localgroup", groupname, username, "/delete"])
            return f"User '{username}' removed from group '{groupname}'."
        else:
            return "ERROR: Removing user from group not implemented for this OS."
    except Exception as e:
        return f"ERROR removing user '{username}' from group '{groupname}': {e}"

# Example Usage:
# # Requires testuser_py and pydevs to exist
# print(add_user_to_group("testuser_py", "pydevs"))
# print(remove_user_from_group("testuser_py", "pydevs"))
```

#### d. Delete a Group (via `subprocess`)

```python
def delete_group(groupname):
    system = platform.system()
    try:
        if system == "Linux":
            run_command(["sudo", "groupdel", groupname])
            return f"Group '{groupname}' deleted."
        elif system == "Windows":
            run_command(["net", "localgroup", groupname, "/delete"])
            return f"Group '{groupname}' deleted."
        else:
            return "ERROR: Group deletion not implemented for this OS."
    except Exception as e:
        return f"ERROR deleting group '{groupname}': {e}"

# Example Usage:
# print(delete_group("pydevs"))
```

### 3. Auditing and Reporting

#### a. Identify Privileged Accounts (UID 0 on Unix, Administrators on Windows)

```python
def identify_privileged_users():
    privileged_users = []
    system = platform.system()

    if system == "Linux" or system == "Darwin":
        for p in pwd.getpwall():
            if p.pw_uid == 0:
                privileged_users.append({"username": p.pw_name, "uid": p.pw_uid})
    elif system == "Windows":
        admin_group_members_output = run_command(["net", "localgroup", "Administrators"])
        if not admin_group_members_output.startswith("ERROR"):
            parsing_members = False
            for line in admin_group_members_output.splitlines():
                if "-------" in line: parsing_members = True; continue
                if "The command completed successfully." in line: break
                if parsing_members:
                    if line.strip(): privileged_users.append({"username": line.strip(), "group": "Administrators"})
    return privileged_users

# Example Usage:
# print("--- Privileged Users ---")
# print(json.dumps(identify_privileged_users(), indent=2))
```

#### b. Identify Disabled/Locked Accounts (parsing `net user` on Windows, `shadow` on Linux)

```python
def identify_disabled_or_locked_users():
    disabled_users = []
    system = platform.system()

    if system == "Linux":
        # Requires parsing /etc/shadow, which needs root and careful handling
        # Example using subprocess to `passwd -S` for simple status
        for p in pwd.getpwall():
            status_output = run_command(["passwd", "-S", p.pw_name], check=False) # passwd -S returns non-zero for some users
            if "L" in status_output.split()[1]: # Check the status char, 'L' for locked
                disabled_users.append({"username": p.pw_name, "status": "Locked"})
    elif system == "Windows":
        # Parsing 'net user <username>' output for "Account active"
        all_users = list_local_users()
        for user_data in all_users:
            if user_data.get("account_active") == False:
                disabled_users.append({"username": user_data["username"], "status": "Disabled"})
    return disabled_users

# Example Usage:
# print("--- Disabled/Locked Users ---")
# print(json.dumps(identify_disabled_or_locked_users(), indent=2))
```

## 4. Full Script Structure (`user_group_manager.py`)

```python
#!/usr/bin/env python3

import os
import subprocess
import json
import re
import platform
import sys
import argparse
import pwd # Unix-like
import grp # Unix-like
from datetime import datetime

# --- Utility Function (run_command) ---
def run_command(command, shell=False, check=True, input_data=None):
    # ... (same as above) ...
    try:
        result = subprocess.run(command, shell=shell, capture_output=True, text=True, check=check, input=input_data)
        return result.stdout.strip()
    except subprocess.CalledProcessError as e: return f"ERROR: Command failed: {e.stderr.strip()}"
    except FileNotFoundError: return f"ERROR: Command not found."
    except Exception as e: return f"ERROR: An unexpected error occurred: {e}"

# --- User Management Functions ---
def list_local_users():
    # ... (same as above) ...
    users = []
    system = platform.system()
    if system == "Linux" or system == "Darwin":
        for p in pwd.getpwall(): users.append({"username": p.pw_name, "uid": p.pw_uid, "gid": p.pw_gid, "home_dir": p.pw_dir, "shell": p.pw_shell, "full_name": p.pw_gecos})
    elif system == "Windows":
        net_user_output = run_command(["net", "user"])
        if not net_user_output.startswith("ERROR"):
            lines = net_user_output.splitlines()
            if len(lines) > 2:
                user_list_line = lines[-1]
                usernames = user_list_line.split()
                for username in usernames:
                    if username:
                        user_details_output = run_command(["net", "user", username])
                        details = {}
                        for line in user_details_output.splitlines():
                            if ":" in line:
                                key, value = line.split(":", 1)
                                details[key.strip()] = value.strip()
                        users.append({"username": username, "full_name": details.get("Full Name"), "comments": details.get("Comment"), "account_active": True if details.get("Account active") == "Yes" else False, "password_expires": details.get("Password expires")})
    return users

def add_user(username, password, system_user=False):
    # ... (same as above) ...
    system = platform.system()
    try:
        if system == "Linux":
            command = ["sudo", "useradd"]
            if system_user: command.append("-r")
            command.append(username)
            run_command(command)
            run_command(["sudo", "passwd", username], input_data=f"{password}
{password}
")
            return f"User '{username}' added."
        elif system == "Windows":
            run_command(["net", "user", username, password, "/add"])
            return f"User '{username}' added."
        else: return "ERROR: User creation not implemented for this OS."
    except Exception as e: return f"ERROR adding user '{username}': {e}"

def modify_user_password(username, new_password):
    # ... (same as above) ...
    system = platform.system()
    try:
        if system == "Linux":
            run_command(["sudo", "passwd", username], input_data=f"{new_password}
{new_password}
")
            return f"Password for '{username}' changed."
        elif system == "Windows":
            run_command(["net", "user", username, new_password])
            return f"Password for '{username}' changed."
        else: return "ERROR: Password modification not implemented for this OS."
    except Exception as e: return f"ERROR modifying password for '{username}': {e}"

def lock_user_account(username):
    # ... (same as above) ...
    system = platform.system()
    try:
        if system == "Linux":
            run_command(["sudo", "usermod", "-L", username])
            return f"User '{username}' locked."
        elif system == "Windows":
            run_command(["net", "user", username, "/active:no"])
            return f"User '{username}' disabled."
        else: return "ERROR: Account locking not implemented for this OS."
    except Exception as e: return f"ERROR locking user '{username}': {e}"

def unlock_user_account(username):
    system = platform.system()
    try:
        if system == "Linux":
            run_command(["sudo", "usermod", "-U", username])
            return f"User '{username}' unlocked."
        elif system == "Windows":
            run_command(["net", "user", username, "/active:yes"])
            return f"User '{username}' enabled."
        else: return "ERROR: Account unlocking not implemented for this OS."
    except Exception as e: return f"ERROR unlocking user '{username}': {e}"

def delete_user(username, delete_home=False):
    # ... (same as above) ...
    system = platform.system()
    try:
        if system == "Linux":
            command = ["sudo", "userdel"]
            if delete_home: command.append("-r")
            command.append(username)
            run_command(command)
            return f"User '{username}' deleted."
        elif system == "Windows":
            run_command(["net", "user", username, "/delete"])
            return f"User '{username}' deleted."
        else: return "ERROR: User deletion not implemented for this OS."
    except Exception as e: return f"ERROR deleting user '{username}': {e}"

# --- Group Management Functions ---
def list_local_groups():
    # ... (same as above) ...
    groups = []
    system = platform.system()
    if system == "Linux" or system == "Darwin":
        for g in grp.getgrall(): groups.append({"groupname": g.gr_name, "gid": g.gr_gid, "members": g.gr_mem})
    elif system == "Windows":
        net_localgroup_output = run_command(["net", "localgroup"])
        if not net_localgroup_output.startswith("ERROR"):
            lines = net_localgroup_output.splitlines()
            group_names = []
            parsing_groups = False
            for line in lines:
                if "-------" in line: parsing_groups = True; continue
                if "The command completed successfully." in line: break
                if parsing_groups:
                    if line.strip(): group_names.append(line.strip())
            for groupname in group_names:
                if groupname:
                    group_details_output = run_command(["net", "localgroup", groupname])
                    members_found = False
                    members_list = []
                    for line in group_details_output.splitlines():
                        if "-------" in line: members_found = True; continue
                        if "The command completed successfully." in line: break
                        if members_found:
                            if line.strip(): members_list.append(line.strip())
                    groups.append({"groupname": groupname, "members": members_list})
    return groups

def add_group(groupname):
    # ... (same as above) ...
    system = platform.system()
    try:
        if system == "Linux":
            run_command(["sudo", "groupadd", groupname])
            return f"Group '{groupname}' added."
        elif system == "Windows":
            run_command(["net", "localgroup", groupname, "/add"])
            return f"Group '{groupname}' added."
        else: return "ERROR: Group creation not implemented for this OS."
    except Exception as e: return f"ERROR adding group '{groupname}': {e}"

def add_user_to_group(username, groupname):
    # ... (same as above) ...
    system = platform.system()
    try:
        if system == "Linux":
            run_command(["sudo", "usermod", "-aG", groupname, username])
            return f"User '{username}' added to group '{groupname}'."
        elif system == "Windows":
            run_command(["net", "localgroup", groupname, username, "/add"])
            return f"User '{username}' added to group '{groupname}'."
        else: return "ERROR: Adding user to group not implemented for this OS."
    except Exception as e: return f"ERROR adding user '{username}' to group '{groupname}': {e}"

def remove_user_from_group(username, groupname):
    # ... (same as above) ...
    system = platform.system()
    try:
        if system == "Linux":
            run_command(["sudo", "gpasswd", "-d", username, groupname])
            return f"User '{username}' removed from group '{groupname}'."
        elif system == "Windows":
            run_command(["net", "localgroup", groupname, username, "/delete"])
            return f"User '{username}' removed from group '{groupname}'."
        else: return "ERROR: Removing user from group not implemented for this OS."
    except Exception as e: return f"ERROR removing user '{username}' from group '{groupname}': {e}"

def delete_group(groupname):
    # ... (same as above) ...
    system = platform.system()
    try:
        if system == "Linux":
            run_command(["sudo", "groupdel", groupname])
            return f"Group '{groupname}' deleted."
        elif system == "Windows":
            run_command(["net", "localgroup", groupname, "/delete"])
            return f"Group '{groupname}' deleted."
        else: return "ERROR: Group deletion not implemented for this OS."
    except Exception as e: return f"ERROR deleting group '{groupname}': {e}"

# --- Auditing Functions ---
def identify_privileged_users():
    # ... (same as above) ...
    privileged_users = []
    system = platform.system()
    if system == "Linux" or system == "Darwin":
        for p in pwd.getpwall():
            if p.pw_uid == 0: privileged_users.append({"username": p.pw_name, "uid": p.pw_uid})
    elif system == "Windows":
        admin_group_members_output = run_command(["net", "localgroup", "Administrators"])
        if not admin_group_members_output.startswith("ERROR"):
            parsing_members = False
            for line in admin_group_members_output.splitlines():
                if "-------" in line: parsing_members = True; continue
                if "The command completed successfully." in line: break
                if parsing_members:
                    if line.strip(): privileged_users.append({"username": line.strip(), "group": "Administrators"})
    return privileged_users

def identify_disabled_or_locked_users():
    # ... (same as above) ...
    disabled_users = []
    system = platform.system()
    if system == "Linux":
        for p in pwd.getpwall():
            status_output = run_command(["passwd", "-S", p.pw_name], check=False)
            if "L" in status_output.split()[1]: disabled_users.append({"username": p.pw_name, "status": "Locked"})
    elif system == "Windows":
        all_users = list_local_users()
        for user_data in all_users:
            if user_data.get("account_active") == False: disabled_users.append({"username": user_data["username"], "status": "Disabled"})
    return disabled_users

# --- Main Logic ---
def main():
    parser = argparse.ArgumentParser(description="Basic User/Group Management and Audit Utility.")
    subparsers = parser.add_subparsers(dest='command', help='Available commands')

    # User commands
    user_parser = subparsers.add_parser('user', help='Manage user accounts.')
    user_subparsers = user_parser.add_subparsers(dest='action', help='User actions')

    list_user_parser = user_subparsers.add_parser('list', help='List all local users.')
    add_user_parser = user_subparsers.add_parser('add', help='Add a new user.')
    add_user_parser.add_argument('username', type=str)
    add_user_parser.add_argument('password', type=str)
    add_user_parser.add_argument('--system', action='store_true', help='Create as a system user (Linux only).')
    mod_user_passwd_parser = user_subparsers.add_parser('mod-passwd', help='Modify user password.')
    mod_user_passwd_parser.add_argument('username', type=str)
    mod_user_passwd_parser.add_argument('new_password', type=str)
    lock_user_parser = user_subparsers.add_parser('lock', help='Lock user account.')
    lock_user_parser.add_argument('username', type=str)
    unlock_user_parser = user_subparsers.add_parser('unlock', help='Unlock user account.')
    unlock_user_parser.add_argument('username', type=str)
    del_user_parser = user_subparsers.add_parser('delete', help='Delete a user.')
    del_user_parser.add_argument('username', type=str)
    del_user_parser.add_argument('--delete-home', action='store_true', help='Delete user home directory (Linux only).')

    # Group commands
    group_parser = subparsers.add_parser('group', help='Manage group accounts.')
    group_subparsers = group_parser.add_subparsers(dest='action', help='Group actions')

    list_group_parser = group_subparsers.add_parser('list', help='List all local groups.')
    add_group_parser = group_subparsers.add_parser('add', help='Add a new group.')
    add_group_parser.add_argument('groupname', type=str)
    add_user_to_group_parser = group_subparsers.add_parser('add-member', help='Add user to group.')
    add_user_to_group_parser.add_argument('username', type=str)
    add_user_to_group_parser.add_argument('groupname', type=str)
    rm_user_from_group_parser = group_subparsers.add_parser('rm-member', help='Remove user from group.')
    rm_user_from_group_parser.add_argument('username', type=str)
    rm_user_from_group_parser.add_argument('groupname', type=str)
    del_group_parser = group_subparsers.add_parser('delete', help='Delete a group.')
    del_group_parser.add_argument('groupname', type=str)

    # Audit commands
    audit_parser = subparsers.add_parser('audit', help='Perform security audits.')
    audit_subparsers = audit_parser.add_subparsers(dest='action', help='Audit actions')

    audit_priv_parser = audit_subparsers.add_parser('privileged', help='Identify privileged users.')
    audit_disabled_parser = audit_subparsers.add_parser('disabled', help='Identify disabled or locked users.')

    args = parser.parse_args()

    if args.command == 'user':
        if args.action == 'list': print(json.dumps(list_local_users(), indent=2))
        elif args.action == 'add': print(add_user(args.username, args.password, args.system))
        elif args.action == 'mod-passwd': print(modify_user_password(args.username, args.new_password))
        elif args.action == 'lock': print(lock_user_account(args.username))
        elif args.action == 'unlock': print(unlock_user_account(args.username))
        elif args.action == 'delete': print(delete_user(args.username, args.delete_home))
    elif args.command == 'group':
        if args.action == 'list': print(json.dumps(list_local_groups(), indent=2))
        elif args.action == 'add': print(add_group(args.groupname))
        elif args.action == 'add-member': print(add_user_to_group(args.username, args.groupname))
        elif args.action == 'rm-member': print(remove_user_from_group(args.username, args.groupname))
        elif args.action == 'delete': print(delete_group(args.groupname))
    elif args.command == 'audit':
        if args.action == 'privileged': print(json.dumps(identify_privileged_users(), indent=2))
        elif args.action == 'disabled': print(json.dumps(identify_disabled_or_locked_users(), indent=2))

if __name__ == "__main__":
    main()
```

## Guiding Principles in Python

*   **Portability:** Python's `platform` module allows for conditional execution of OS-specific `subprocess` calls (`useradd` vs `net user`). The `pwd` and `grp` modules are Unix-specific but standard.
*   **Efficiency:** Executing native commands via `subprocess` is efficient. Python's overhead for parsing is generally minimal.
*   **Minimal Dependencies:** This tutorial primarily uses Python's standard library. `os`, `subprocess`, `json`, `re`, `platform`, `sys`, `argparse`, `pwd`, and `grp` are all standard modules.
*   **CLI-centric:** The script uses `argparse` for robust command-line argument handling, making it a flexible and user-friendly CLI tool.
*   **Structured Data Handling:** Results are collected into Python dictionaries and lists, which are easily serialized to JSON, providing a clean, machine-readable output for further processing and integration.

## Conclusion

Python offers a flexible and powerful environment for basic user and group management and auditing. By carefully combining its standard library modules with platform-specific system commands, you can build portable and robust tools for maintaining system security and compliance. The ability to produce structured JSON output makes these tools invaluable for automated reporting and integration into larger security and administration frameworks. The next step is to apply this knowledge in practical exercises.
# Python Exercise: Basic User/Group Management and Audit

## Objective

This exercise challenges you to apply your Python scripting skills to perform fundamental user and group management tasks, as well as basic security auditing on a cross-platform system. You will use Python's standard library (`os`, `pwd`, `grp`, `subprocess`, `json`) to list, create, modify, and delete local user and group accounts, demonstrating proficiency in access control, system security best practices, and structured data handling.

## Framework Alignment

This exercise on "**User/Group Management and Audit**" using **Python** provides practical experience in building and utilizing tools for a "Cross-Platform Baseline Hardening & Auditing Framework." By completing these tasks, you will learn how to audit and manage user accounts and group memberships, ensuring compliance with security policies and identifying unauthorized privilege escalations—essential steps in maintaining a secure and auditable environment.


## Scenario

You are tasked with developing a portable Python script to manage user access and perform security audits on various operating systems (Linux, macOS, Windows). Your script needs to:
1.  List all local users and groups with essential attributes.
2.  Be able to create, modify (e.g., lock account), and delete user and group accounts.
3.  Audit for accounts with elevated privileges or security weaknesses (like disabled accounts).
4.  Output collected user/group data in a structured JSON format.

## Important Note

**This exercise involves creating, modifying, and deleting user and group accounts on your system.** It is highly recommended to perform these tasks in a **virtual machine or a non-production test environment** where changes will not impact a live system. All commands for management (add, modify, delete) will require elevated privileges (e.g., `sudo` on Unix-like systems, running the script as Administrator on Windows).

## Tasks

Write a Python script (`user_group_manager.py` or similar name) that, when executed, can perform the following tasks. Your script should be structured with functions for clarity and output results in a structured JSON format where appropriate.

### Part 1: User and Group Management Functions

1.  **List All Local Users:**
    *   Implement `list_local_users()` to return a list of dictionaries, each representing a user with `username`, `uid`, `gid`, `home_dir`, `shell` (Unix-like) or `username`, `account_active` (Windows).

2.  **List All Local Groups:**
    *   Implement `list_local_groups()` to return a list of dictionaries, each representing a group with `groupname`, `gid`, and `members`.

3.  **Add a New Group:**
    *   Implement `add_group(groupname)` that creates a new local group.

4.  **Add a New User:**
    *   Implement `add_user(username, password)` that creates a new local user with a home directory and sets the password.

5.  **Add User to Group:**
    *   Implement `add_user_to_group(username, groupname)` that adds a specified user to a specified group.

6.  **Lock User Account:**
    *   Implement `lock_user_account(username)` that disables or locks a user account.

7.  **Unlock User Account:**
    *   Implement `unlock_user_account(username)` that enables or unlocks a user account.

8.  **Delete User and Group:**
    *   Implement `delete_user(username, delete_home=False)` that removes a user account, optionally deleting its home directory.
    *   Implement `delete_group(groupname)` that removes a group.

### Part 2: Security Auditing Functions

1.  **Identify Privileged Accounts:**
    *   Implement `identify_privileged_users()` that lists all users with UID 0 (Unix-like) or who are members of the local "Administrators" group (Windows).

2.  **Identify Disabled/Locked Accounts:**
    *   Implement `identify_disabled_or_locked_users()` that lists all local user accounts that are currently disabled or locked.

3.  **Generate Comprehensive User/Group Report (JSON):**
    *   Combine the results from `list_local_users()` and `list_local_groups()` (including members) into a single structured dictionary and output it as a JSON string to `stdout`.

## Deliverables

Provide the complete Python script file (`user_group_manager.py`) that implements all the above tasks. Your script should be executable from the command line, possibly using `argparse` to select which function to run and what parameters to use (e.g., `python user_group_manager.py user add devuser Pa$$w0rd1`).

## Reflection Questions

1.  How did your script handle the platform-specific differences in user and group management commands (e.g., `useradd` vs `net user`)?
2.  Explain the role of `subprocess` module in performing user/group management actions. What are the security considerations when using `subprocess` with elevated privileges?
3.  What are the advantages of using Python's `pwd` and `grp` modules for Unix-like systems over parsing `cat /etc/passwd` or `getent` output directly?
4.  Describe how returning user and group information as Python dictionaries/objects and then serializing to JSON simplifies further programmatic analysis or integration with other systems.
5.  What are the advantages and disadvantages of using Python for basic user/group management and auditing compared to Bash, PowerShell, or a database like SQLite?

---
# Bash Tutorial: Basic User/Group Management and Audit (Unix-like Systems)

## Introduction

Bash, combined with standard Unix-like utilities, provides direct and powerful control over local user and group accounts. This tutorial will guide you through the essential Bash commands for listing, creating, modifying, and deleting users and groups, as well as performing basic security audits. These tools are fundamental for system administration, access control, and ensuring compliance, adhering to principles of minimal dependencies and CLI-centric operation.

## Framework Alignment

This tutorial on "**User/Group Management and Audit**" using **Bash** demonstrates the foundational skills necessary to implement components of a "Cross-Platform Baseline Hardening & Auditing Framework." The techniques learned here for managing user accounts and group memberships are directly applicable to auditing systems against defined security baselines and ensuring compliance.


## Core Bash Utilities for User/Group Management

These commands primarily interact with `/etc/passwd`, `/etc/shadow`, and `/etc/group` files on Unix-like systems (Linux, macOS).

*   **`getent`**: Get entries from Name Service Switch libraries. Useful for listing users/groups.
    *   `getent passwd`: Lists all users.
    *   `getent group`: Lists all groups.
*   **`id`**: Print user and group information for the specified USERNAME, or for the current user.
*   **`useradd`**: Create a new user account.
    *   `-m`: Create home directory.
    *   `-s <shell>`: Specify login shell.
    *   `-g <group>`: Specify primary group.
    *   `-G <group1,group2>`: Specify secondary groups.
*   **`userdel`**: Delete a user account.
    *   `-r`: Remove home directory and mail spool.
*   **`usermod`**: Modify a user account.
    *   `-l <new_name>`: Change login name.
    *   `-L`: Lock password.
    *   `-U`: Unlock password.
    *   `-e <YYYY-MM-DD>`: Set account expiration date.
    *   `-g <primary_group>`: Change primary group.
    *   `-aG <secondary_group>`: Add to secondary group.
*   **`passwd`**: Change user password.
    *   `-l`: Lock password.
    *   `-u`: Unlock password.
    *   `-e`: Force password expiry.
*   **`groupadd`**: Create a new group.
*   **`groupdel`**: Delete a group.
*   **`groupmod`**: Modify a group.
*   **`chage`**: Change user password expiry information.
    *   `-l`: List password age information.
    *   `-E <YYYY-MM-DD>`: Set account expiration date.

## Implementing Core Functionality with Bash

*Note: Most user/group management commands require root privileges (e.g., `sudo`).*

### 1. User Management

#### a. List All Local Users

```bash
echo "--- All Local Users (from /etc/passwd) ---"
getent passwd | cut -d: -f1,3,4,6,7 # Username, UID, GID, Home, Shell

# Optionally, more details per user (example for a specific user)
# USER="youruser"
# echo "--- Details for $USER ---"
# id "$USER"
# finger "$USER" # If installed
# chage -l "$USER" # Password expiry info
```

#### b. Add a New User

```bash
# Add user 'testuser' with home directory and bash shell
sudo useradd -m -s /bin/bash testuser

# Set a password for the new user
echo "testpassword" | sudo passwd --stdin testuser

# Verify creation
id testuser
```

#### c. Modify User Attributes

```bash
# Add 'testuser' to the 'sudo' group (or 'wheel' on some systems)
sudo usermod -aG sudo testuser

# Lock 'testuser' account
sudo usermod -L testuser # Also sudo passwd -l testuser

# Unlock 'testuser' account
sudo usermod -U testuser # Also sudo passwd -u testuser
```

#### d. Delete a User

```bash
# Delete 'testuser' and their home directory
sudo userdel -r testuser

# Verify deletion
getent passwd | grep testuser # Should show nothing
```

### 2. Group Management

#### a. List All Local Groups

```bash
echo "--- All Local Groups (from /etc/group) ---"
getent group | cut -d: -f1,3,4 # Group Name, GID, Members
```

#### b. Add a New Group

```bash
# Add group 'devteam'
sudo groupadd devteam

# Verify creation
getent group devteam
```

#### c. Add/Remove User to/from Group

```bash
# First, create a user for testing
sudo useradd -m testuser2
echo "testpass" | sudo passwd --stdin testuser2

# Add 'testuser2' to 'devteam' secondary group
sudo usermod -aG devteam testuser2

# Verify membership
id testuser2

# Remove 'testuser2' from 'devteam'
sudo gpasswd -d testuser2 devteam # Or sudo deluser testuser2 devteam
# Verify removal
id testuser2 # 'devteam' should not be listed as a secondary group
```

#### d. Delete a Group

```bash
# Delete 'devteam' group
sudo groupdel devteam

# Verify deletion
getent group devteam # Should show nothing
```

### 3. Auditing and Reporting

#### a. Identify Users with UID 0 (Root Equivalent)

```bash
echo "--- Users with UID 0 ---"
getent passwd | awk -F: '$3 == 0 {print $1}'
```

#### b. Identify Accounts with No Password or Locked Passwords

```bash
echo "--- Users with No/Locked Passwords (via /etc/shadow) ---"
# This requires reading /etc/shadow, which needs root privileges
sudo awk -F: '($2 == "" || $2 ~ /^\!/ || $2 ~ /^\*/) {print $1}' /etc/shadow
# "" for no password, "!" for locked, "*" for disabled
```

#### c. Identify Accounts with Expired Passwords or Locked Accounts

```bash
echo "--- Accounts with Password Expiry Info ---"
# Check all users
for user in $(getent passwd | cut -d: -f1); do
    # Requires root to get full chage info
    sudo chage -l "$user" 2>/dev/null | grep -E "Password expires|Account expires|Account locked" | sed "s/^/$user: /"
done
```

#### d. Generate Comprehensive User Report (Example)

```bash
#!/bin/bash

echo "--- Comprehensive User Report ---"
echo "Username,UID,GID,Home Directory,Shell,Account Status,Password Expiry"

getent passwd | while IFS=: read -r username _ uid gid _ home_dir shell; do
    account_status="Unknown"
    password_expiry="N/A"

    # Get account status and password expiry using chage (requires root)
    if chage_info=$(sudo chage -l "$username" 2>/dev/null); then
        if echo "$chage_info" | grep -q "Account expires\s*:\s*never"; then
            account_status="Active"
        elif echo "$chage_info" | grep -q "Account expires\s*:\s*(\S+)"; then
            account_status="Expired ($(echo "$chage_info" | grep "Account expires" | awk '{print $NF}'))"
        fi
        if echo "$chage_info" | grep -q "Account locked"; then
            account_status="Locked"
        fi
        if echo "$chage_info" | grep -q "Password expires\s*:\s*never"; then
            password_expiry="Never"
        elif echo "$chage_info" | grep -q "Password expires\s*:\s*(\S+)"; then
            password_expiry="$(echo "$chage_info" | grep "Password expires" | awk '{print $NF}')"
        fi
    else
        account_status="Error/No Privileges"
    fi

    # Check for disabled accounts in /etc/shadow (no password / locked)
    if shadow_entry=$(sudo awk -F: -v user="$username" '$1 == user {print $2}' /etc/shadow 2>/dev/null); then
        if [ "$shadow_entry" = "" ]; then
            account_status="No Password"
        elif [[ "$shadow_entry" == "!"* ]] || [[ "$shadow_entry" == "*"* ]]; then
            account_status="Locked/Disabled"
        fi
    fi

    echo "$username,$uid,$gid,$home_dir,$shell,$account_status,$password_expiry"
done
```

## Guiding Principles in Bash

*   **Portability:** The commands used (`useradd`, `usermod`, `userdel`, `groupadd`, `groupmod`, `groupdel`, `getent`, `id`, `chage`, `passwd`) are standard on most Unix-like systems. Windows requires different tools.
*   **Efficiency:** These commands are compiled binaries and are very efficient for managing local accounts.
*   **Minimal Dependencies:** Relies entirely on core system utilities.
*   **CLI-centric:** All operations are command-line based, ideal for scripting and integration into system startup/maintenance scripts.
*   **Security Focus:** Provides direct access to tools for implementing strong access control and auditing account configurations.

## Conclusion

Bash, through its suite of user and group management utilities, offers a powerful and direct way to administer access on Unix-like systems. While effective for scripting specific tasks, the manual parsing of output and conditional logic can become complex for comprehensive cross-platform reporting. These tools are invaluable for day-to-day administration and security auditing, particularly when managing local accounts. The next step is to apply this knowledge in practical exercises.
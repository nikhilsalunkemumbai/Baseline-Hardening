# Bash Exercise: Basic User/Group Management and Audit (Unix-like Systems)

## Objective

This exercise challenges you to apply your Bash scripting skills to perform fundamental user and group management tasks, as well as basic security auditing on a Unix-like system (e.g., Linux, macOS). You will use standard command-line utilities to list, create, modify, and delete local user and group accounts, demonstrating proficiency in access control and system security best practices.

## Framework Alignment

This exercise on "**User/Group Management and Audit**" using **Bash** provides practical experience in building and utilizing tools for a "Cross-Platform Baseline Hardening & Auditing Framework." By completing these tasks, you will learn how to audit and manage user accounts and group memberships, ensuring compliance with security policies and identifying unauthorized privilege escalations—essential steps in maintaining a secure and auditable environment.


## Scenario

You are a system administrator responsible for managing access to a critical server. You need to provision a new developer account, ensure proper group memberships, and regularly audit for any accounts with elevated privileges or security weaknesses (like disabled accounts).

## Important Note

**This exercise involves creating, modifying, and deleting user and group accounts on your system.** It is highly recommended to perform these tasks in a **virtual machine or a non-production test environment** where changes will not impact a live system. All commands will require `sudo` privileges.

## Tasks

Using only standard Bash commands and utilities (e.g., `useradd`, `userdel`, `usermod`, `groupadd`, `groupdel`, `getent`, `id`, `chage`), provide the command-line solution or a simple script for each of the following tasks.

### Part 1: User and Group Management

1.  **List All Local Users:**
    *   Display a list of all local user accounts, showing their username, UID, GID, home directory, and default shell.

2.  **List All Local Groups:**
    *   Display a list of all local groups, showing their group name, GID, and members.

3.  **Create a New Group:**
    *   Create a new group named `devteam`.

4.  **Create a New User:**
    *   Create a new user named `devuser` with a home directory and a default shell of `/bin/bash`. Set an initial password for this user (e.g., `Pa$$w0rd1`).

5.  **Add User to Group:**
    *   Add `devuser` to the `devteam` group as a secondary group.

6.  **Verify User and Group Membership:**
    *   Display `devuser`'s user and group information to confirm they are a member of `devteam`.

7.  **Lock User Account:**
    *   Lock the `devuser` account, preventing them from logging in.

8.  **Unlock User Account:**
    *   Unlock the `devuser` account.

9.  **Delete Test User and Group:**
    *   Delete the `devuser` account, including its home directory.
    *   Delete the `devteam` group.
    *   Verify that both have been removed.

### Part 2: Security Auditing

1.  **Identify Privileged Accounts (UID 0):**
    *   List all user accounts on the system that have a User ID (UID) of 0 (root equivalent).

2.  **Identify Disabled/Locked Accounts:**
    *   List all user accounts that are currently disabled or locked (i.e., cannot log in).

3.  **Identify Accounts with No Password (if any exist):**
    *   List any user accounts that do not have a password set (i.e., an empty password field in `/etc/shadow`).

## Deliverables

For each task, provide the single Bash command-line pipeline or script snippet that performs the action or produces the required output.

## Reflection Questions

1.  What are the security implications of finding multiple accounts with UID 0 on a system?
2.  Explain the difference between `usermod -L` and `passwd -l`. Are they interchangeable?
3.  Why is it crucial to use `sudo` for user and group management commands?
4.  Consider an automated script that regularly audits users. How would you handle the requirement for `sudo` and securely store credentials if necessary?
5.  What are the challenges of performing user/group management and auditing in Bash on a Windows system? How do these tools differ significantly from Windows' native utilities?

---
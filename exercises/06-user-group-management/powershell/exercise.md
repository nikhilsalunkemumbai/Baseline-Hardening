# PowerShell Exercise: Basic User/Group Management and Audit

## Objective

This exercise challenges you to apply your PowerShell scripting skills to perform fundamental user and group management tasks, as well as basic security auditing on a Windows (or PowerShell Core-enabled Linux/macOS) system. You will use standard PowerShell cmdlets to list, create, modify, and delete local user and group accounts, demonstrating proficiency in access control and system security best practices.

## Framework Alignment

This exercise on "**User/Group Management and Audit**" using **PowerShell** provides practical experience in building and utilizing tools for a "Cross-Platform Baseline Hardening & Auditing Framework." By completing these tasks, you will learn how to audit and manage user accounts and group memberships, ensuring compliance with security policies and identifying unauthorized privilege escalations—essential steps in maintaining a secure and auditable environment.


## Scenario

You are a system administrator responsible for managing access to a critical server. You need to provision a new developer account, ensure proper group memberships, and regularly audit for any accounts with elevated privileges or security weaknesses (like disabled accounts or accounts with old passwords).

## Important Note

**This exercise involves creating, modifying, and deleting user and group accounts on your system.** It is highly recommended to perform these tasks in a **virtual machine or a non-production test environment** where changes will not impact a live system. All commands will require elevated PowerShell privileges (Run as Administrator on Windows, `sudo` on Linux/macOS).

## Tasks

Using only standard PowerShell cmdlets (e.g., `Get-LocalUser`, `New-LocalUser`, `Set-LocalUser`, `Remove-LocalUser`, `Get-LocalGroup`, `New-LocalGroup`, `Add-LocalGroupMember`, `Remove-LocalGroupMember`), provide the command-line solution or a simple script for each of the following tasks.

### Part 1: User and Group Management

1.  **List All Local Users:**
    *   Display a list of all local user accounts, showing their `Name`, `Enabled` status, and `Description`.

2.  **List All Local Groups:**
    *   Display a list of all local groups, showing their `Name` and `Description`.

3.  **Create a New Group:**
    *   Create a new local group named `DevTeam`.

4.  **Create a New User:**
    *   Create a new local user named `DevUser`. Set a password for this user (e.g., `Pa$$w0rd1`). Ensure the account is initially enabled.

5.  **Add User to Group:**
    *   Add `DevUser` to the `DevTeam` group.

6.  **Verify User and Group Membership:**
    *   Display `DevUser`'s group memberships to confirm they are a member of `DevTeam`.

7.  **Disable User Account:**
    *   Disable the `DevUser` account, preventing them from logging in.

8.  **Enable User Account:**
    *   Enable the `DevUser` account.

9.  **Delete Test User and Group:**
    *   Remove the `DevUser` account.
    *   Remove the `DevTeam` group.
    *   Verify that both have been removed.

### Part 2: Security Auditing

1.  **Identify Privileged Accounts (Local Administrators):**
    *   List all members of the local "Administrators" group.

2.  **Identify Disabled Local User Accounts:**
    *   List all local user accounts that are currently disabled.

3.  **Identify Local User Accounts with Password Never Expires:**
    *   List all local user accounts where the password is set to "Never Expires". (Hint: Inspect properties of `Get-LocalUser`.)

4.  **Generate Comprehensive User Report (JSON):**
    *   Create a script that lists all local users and for each user, includes their `Name`, `Enabled` status, `LastLogon` time, and a list of groups they are a member of.
    *   Output this entire report as a single JSON array to the console.

## Deliverables

For each task, provide the single PowerShell command-line pipeline or script snippet that performs the action or produces the required output.

## Reflection Questions

1.  How does PowerShell's object-oriented nature simplify filtering and processing user/group information compared to parsing text output from command-line tools?
2.  Explain the cmdlets used for creating, modifying, and deleting local users and groups. What are the key parameters for each?
3.  What are the advantages of generating a comprehensive user report as JSON (Task 2.4) for automation and integration with other systems?
4.  Consider an automated script that regularly audits user accounts. How would you handle the requirement for elevated privileges in such a script?
5.  What are the challenges of performing user/group management and auditing in PowerShell on a Linux/macOS system compared to Windows? What differences did you observe (if any) in cmdlet behavior?

---
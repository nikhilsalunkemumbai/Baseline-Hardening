# PowerShell Tutorial: Basic User/Group Management and Audit

## Introduction

PowerShell offers a native, object-oriented, and often cross-platform way to manage local user and group accounts. Its cmdlets are designed to interact directly with the operating system's security principal database (SAM database on Windows, `/etc/passwd` and `/etc/group` on Linux/macOS via PowerShell Core), providing structured output that is easy to filter, inspect, and automate. This tutorial will guide you through using PowerShell for user and group management and auditing, emphasizing its strengths in producing actionable, structured data.

## Framework Alignment

This tutorial on "**User/Group Management and Audit**" using **PowerShell** demonstrates the foundational skills necessary to implement components of a "Cross-Platform Baseline Hardening & Auditing Framework." The techniques learned here for managing user accounts and group memberships are directly applicable to auditing systems against defined security baselines and ensuring compliance.


## Core PowerShell Cmdlets for User/Group Management

For managing *local* user and group accounts, PowerShell provides dedicated cmdlets:

*   **`Get-LocalUser`**: Gets local user accounts.
*   **`New-LocalUser`**: Creates a new local user account.
*   **`Set-LocalUser`**: Modifies properties of a local user account.
*   **`Remove-LocalUser`**: Deletes a local user account.
*   **`Enable-LocalUser`**: Enables a local user account.
*   **`Disable-LocalUser`**: Disables a local user account.
*   **`Rename-LocalUser`**: Renames a local user account.
*   **`Get-LocalGroup`**: Gets local security groups.
*   **`New-LocalGroup`**: Creates a new local security group.
*   **`Set-LocalGroup`**: Modifies a local security group.
*   **`Remove-LocalGroup`**: Deletes a local security group.
*   **`Add-LocalGroupMember`**: Adds members to a local group.
*   **`Remove-LocalGroupMember`**: Removes members from a local group.
*   **`Get-LocalGroupMember`**: Gets members from a local group.

*Note: These cmdlets are available in PowerShell Core (6.0+) on Windows, Linux, and macOS, and in Windows PowerShell (5.1) if the `Microsoft.PowerShell.LocalAccounts` module is imported.*

## Implementing Core Functionality with PowerShell

*Note: Most user/group management commands require elevated privileges (e.g., Administrator on Windows, `sudo` on Linux/macOS).*

### 1. User Management

#### a. List All Local Users

```powershell
Write-Host "--- All Local Users ---"
Get-LocalUser | Select-Object Name, SID, Enabled, LastLogon, Description | Format-Table -AutoSize
```

#### b. Add a New User

```powershell
# Create user 'testuser' with a password, but disabled initially
$Password = ConvertTo-SecureString "P@ssw0rd123" -AsPlainText -Force
New-LocalUser -Name "testuser" -Password $Password -AccountExpires (Get-Date).AddYears(1) -Description "Test account for tutorial" -Disabled
Write-Host "User 'testuser' created (disabled)."

# Enable the user
Enable-LocalUser -Name "testuser"
Write-Host "User 'testuser' enabled."
```

#### c. Modify User Attributes

```powershell
# Set a new password for 'testuser'
Set-LocalUser -Name "testuser" -Password $Password

# Set account to never expire
Set-LocalUser -Name "testuser" -AccountExpires (Get-Date).AddYears(100) # Effectively never expires

# Lock 'testuser' account
Disable-LocalUser -Name "testuser"
Write-Host "User 'testuser' disabled (locked)."

# Unlock 'testuser' account
Enable-LocalUser -Name "testuser"
Write-Host "User 'testuser' enabled (unlocked)."
```

#### d. Delete a User

```powershell
# Remove 'testuser'
Remove-LocalUser -Name "testuser" -Confirm:$false
Write-Host "User 'testuser' removed."
```

### 2. Group Management

#### a. List All Local Groups

```powershell
Write-Host "--- All Local Groups ---"
Get-LocalGroup | Select-Object Name, SID, Description | Format-Table -AutoSize
```

#### b. Add a New Group

```powershell
# Add group 'Developers'
New-LocalGroup -Name "Developers" -Description "Software Development Team"
Write-Host "Group 'Developers' created."
```

#### c. Add/Remove User to/from Group

```powershell
# Create a test user first
$Password = ConvertTo-SecureString "P@ssw0rd123" -AsPlainText -Force
New-LocalUser -Name "devuser" -Password $Password -Confirm:$false

# Add 'devuser' to 'Developers' group
Add-LocalGroupMember -Group "Developers" -Member "devuser"
Write-Host "User 'devuser' added to 'Developers' group."

# Verify membership
(Get-LocalGroup -Name "Developers").Members | Select-Object Name

# Remove 'devuser' from 'Developers' group
Remove-LocalGroupMember -Group "Developers" -Member "devuser"
Write-Host "User 'devuser' removed from 'Developers' group."

# Clean up test user
Remove-LocalUser -Name "devuser" -Confirm:$false
```

#### d. Delete a Group

```powershell
# Remove 'Developers' group
Remove-LocalGroup -Name "Developers" -Confirm:$false
Write-Host "Group 'Developers' removed."
```

### 3. Auditing and Reporting

#### a. Identify Privileged Accounts (Local Administrators)

```powershell
Write-Host "--- Local Administrator Accounts ---"
(Get-LocalGroup -Name "Administrators").Members | Select-Object Name, SID
```

#### b. Identify Disabled or Locked Accounts

```powershell
Write-Host "--- Disabled Local User Accounts ---"
Get-LocalUser | Where-Object {$_.Enabled -eq $false} | Select-Object Name, Description
```

#### c. Identify Accounts with Old Passwords (Windows-specific, via AD or WMI)

This is complex with local accounts only and often requires Active Directory integration or parsing security event logs. A direct cmdlet for local password age isn't readily available for general use. For auditing, you'd typically check password expiration policies.

#### d. Generate Comprehensive User Report (JSON)

```powershell
$UserReport = Get-LocalUser | Select-Object Name, SID, Enabled, LastLogon, PasswordChangeable, PasswordRequired, Description,
    @{Name="MemberOf";Expression={(Get-LocalGroup -Member $_.Name).Name -join ", "}}

$UserReport | ConvertTo-Json -Depth 3
```

## Guiding Principles in PowerShell

*   **Portability:** The `*-LocalUser` and `*-LocalGroup` cmdlets are designed for cross-platform compatibility with PowerShell Core, allowing consistent local account management on Windows, Linux, and macOS.
*   **Efficiency:** Cmdlets are optimized for interacting with the OS security features. Operations are generally fast and efficient.
*   **Minimal Dependencies:** PowerShell scripts rely only on the PowerShell runtime and its built-in modules. No external binaries or libraries are typically needed beyond what the OS provides.
*   **CLI-centric:** PowerShell is a command-line shell; scripts are executed directly from the console and leverage its robust argument parsing.
*   **Structured Data Handling:** Cmdlets return objects, not raw text. This simplifies filtering, sorting, and reporting, and allows for direct conversion to formats like JSON or CSV, making auditing and automation much easier.

## Conclusion

PowerShell provides an exceptionally powerful and intuitive environment for managing and auditing local user and group accounts. Its object-oriented pipeline simplifies the retrieval, filtering, and manipulation of security principal data, enabling administrators to enforce policies, audit for vulnerabilities, and automate user/group lifecycle tasks. The structured nature of its output makes it ideal for integrating security auditing into larger automation frameworks. The next step is to apply this knowledge in practical exercises.
# PowerShell Exercise: Configuration File Parsing and Validation

## Objective

This exercise challenges you to apply your PowerShell scripting skills to parse a JSON configuration file, extract specific settings, and validate them against security and operational policies. You will use standard PowerShell cmdlets to transform the configuration into structured objects, perform checks, and report on compliance, demonstrating proficiency in automated configuration auditing.

## Framework Alignment

This exercise on "**Configuration File Parsing and Validation**" using **PowerShell** provides practical experience in building and utilizing tools for a "Cross-Platform Baseline Hardening & Auditing Framework." By completing these tasks, you will learn how to audit and manage configuration settings, ensuring compliance with security policies and identifying unauthorized changes—essential steps in maintaining a secure and auditable environment.


## Scenario

You are a DevOps engineer responsible for deploying and maintaining a critical application. The application uses an `appsettings.json` file for its configuration. Before deploying, you need to automatically verify that certain security and performance-related settings are correctly configured.

## Setup

Before starting the tasks, create the following `appsettings.json` file in your current directory:

```powershell
Set-Content -Path "appsettings.json" -Value @"
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=MyAppDb;User Id=appuser;Password=securepassword;Port=5432"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Debug",
      "Microsoft": "Warning"
    }
  },
  "Application": {
    "FeatureToggle": {
      "EnableBetaFeatures": false,
      "AuditLogging": true
    },
    "MaxConcurrentUsers": 100,
    "Security": {
      "EnableHttpsRedirect": "false",
      "MinEncryptionStrength": "AES256"
    }
  }
}
"@

Write-Host "Configuration file 'appsettings.json' created."
```

## Tasks

Using only standard PowerShell cmdlets (`Get-Content`, `ConvertFrom-Json`, `Where-Object`, `Select-Object`), provide the command-line solution or a simple script for each of the following tasks.

### Part 1: Parsing Configuration Values

1.  **Parse the JSON File:**
    *   Read the `appsettings.json` file and convert its content into a PowerShell object. Store this object in a variable.

2.  **Extract Default Connection String:**
    *   From the parsed object, display the value of `ConnectionStrings.DefaultConnection`.

3.  **Extract Default Log Level:**
    *   Display the value of `Logging.LogLevel.Default`.

4.  **Extract `EnableBetaFeatures` Flag:**
    *   Display the boolean value of `Application.FeatureToggle.EnableBetaFeatures`.

5.  **Extract `MaxConcurrentUsers`:**
    *   Display the numeric value of `Application.MaxConcurrentUsers`.

### Part 2: Configuration Validation

Write a single PowerShell script (`validate_appsettings.ps1`) that performs the following checks and outputs a structured validation report as JSON.

1.  **Policy Check: Database Port:**
    *   Extract the `Port` number from `ConnectionStrings.DefaultConnection`.
    *   Verify that the extracted port is NOT `5432` (assuming `5432` is the default and a custom port is required for security/staging). Report `PASS` or `FAIL`.

2.  **Policy Check: Production Log Level:**
    *   Verify that `Logging.LogLevel.Default` is set to `Information` or `Warning` (i.e., NOT `Debug` for a production environment). Report `PASS` or `FAIL`.

3.  **Policy Check: `EnableHttpsRedirect`:**
    *   Verify that `Application.Security.EnableHttpsRedirect` is `true`. Report `PASS` or `FAIL`.

4.  **Policy Check: `AuditLogging`:**
    *   Verify that `Application.FeatureToggle.AuditLogging` is `true`. Report `PASS` or `FAIL`.

5.  **Policy Check: `MaxConcurrentUsers`:**
    *   Verify that `Application.MaxConcurrentUsers` is less than or equal to `200`. Report `PASS` or `FAIL`.

6.  **Structured Output:**
    *   The script should collect all validation results (e.g., `key_path`, `status`, `message`, `expected`, `actual`) into an array of PowerShell objects.
    *   Finally, convert this array of results into a JSON string and print it to the console.

## Deliverables

For Part 1, provide the exact PowerShell command-line solution for each task. For Part 2, provide the complete PowerShell script file (`validate_appsettings.ps1`).

## Reflection Questions

1.  How does `ConvertFrom-Json` simplify the process of accessing nested configuration values compared to text parsing in Bash?
2.  Explain how you extracted the database port from the `ConnectionStrings.DefaultConnection` string. Which PowerShell string manipulation methods or regular expressions were useful?
3.  Describe the advantages of generating a validation report as a structured JSON object. How can this be integrated into automated pipelines?
4.  If the configuration file format were YAML instead of JSON, how would your PowerShell parsing approach change?
5.  What are the challenges of performing configuration validation checks that depend on the data type (e.g., checking if a numeric value is within a range or if a boolean is true)?

---
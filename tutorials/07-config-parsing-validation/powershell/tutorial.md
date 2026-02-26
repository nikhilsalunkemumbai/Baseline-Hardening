# PowerShell Tutorial: Configuration File Parsing and Validation

## Introduction

PowerShell's robust cmdlets and object-oriented pipeline make it an excellent choice for parsing and validating various configuration file formats. Unlike text-based processing in Bash, PowerShell often converts configuration data directly into structured objects, simplifying extraction, manipulation, and validation. This tutorial will guide you through using PowerShell for common configuration file types (INI-style, JSON, YAML, simple key-value) to extract settings and enforce compliance policies.

## Framework Alignment

This tutorial on "**Configuration File Parsing and Validation**" using **PowerShell** demonstrates the foundational skills necessary to implement components of a "Cross-Platform Baseline Hardening & Auditing Framework." The techniques learned here for parsing and validating configuration files are directly applicable to auditing systems against defined security baselines and ensuring compliance.


## Core PowerShell Cmdlets for Configuration Parsing

*   **`Get-Content`**: Reads the content of a file.
*   **`Select-String`**: Searches for text and string patterns in input strings or files. Useful for initial filtering or parsing lines from INI/key-value files.
*   **`ConvertFrom-StringData`**: Converts a string containing one or more key-value pairs to a hashtable. Useful for simple key-value or INI sections.
*   **`ConvertFrom-Json`**: Converts a JSON formatted string to a PowerShell object.
*   **`ConvertFrom-Yaml`**: (Available in PowerShell Gallery, or natively in some newer PowerShell Core versions) Converts a YAML formatted string to a PowerShell object. For minimal dependency, parsing YAML via `ConvertFrom-Json` after converting YAML to JSON (e.g., via Python script or online tool) could be an alternative. For this tutorial, we'll assume `ConvertFrom-Yaml` is available or demonstrate basic regex.
*   **`Where-Object` (alias `where`)**: Filters objects based on property values.
*   **`Select-Object` (alias `select`)**: Selects specified properties of an object.

## Implementing Core Functionality with PowerShell

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

```powershell
function Get-IniValue {
    param (
        [string]$Path,
        [string]$Section,
        [string]$Key
    )

    $content = Get-Content -Path $Path | ForEach-Object { $_.Trim() } # Trim whitespace
    $inSection = $false
    $value = $null

    foreach ($line in $content) {
        if ($line -match "^\[(.*)\]$") {
            $currentSection = $Matches[1]
            $inSection = ($currentSection -eq $Section)
            continue
        }

        if ($inSection -and ($line -match "^$Key=(.*?)($|;|\#.*$)")) { # Key=Value or Key=Value;Comment or Key=Value#Comment
            $value = $Matches[1].Trim()
            break
        }
    }
    return $value
}

$configFile = "app.conf"
Set-Content -Path $configFile -Value @"
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
"@

Write-Host "Database Port: $(Get-IniValue -Path $configFile -Section "database" -Key "port")"
Write-Host "Permit Root Login: $(Get-IniValue -Path $configFile -Section "security" -Key "permit_root_login")"
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

```powershell
function Get-EnvValue {
    param (
        [string]$Path,
        [string]$Key
    )

    $content = Get-Content -Path $Path | ForEach-Object { $_.Trim() } # Trim whitespace
    $value = $null

    foreach ($line in $content) {
        # Ignore comments and empty lines
        if ($line.StartsWith("#") -or [string]::IsNullOrWhiteSpace($line)) {
            continue
        }
        # Match "KEY=VALUE"
        if ($line -match "^$Key=(.*)$") {
            $value = $Matches[1].Trim()
            break
        }
    }
    return $value
}

$envFile = ".env"
Set-Content -Path $envFile -Value @"
DB_HOST=127.0.0.1
DB_USER=root
# DB_PASSWORD=secret
APP_DEBUG=true
"@

Write-Host "DB Host: $(Get-EnvValue -Path $envFile -Key "DB_HOST")"
Write-Host "App Debug: $(Get-EnvValue -Path $envFile -Key "APP_DEBUG")"
```

### 3. Parsing JSON Files

Let's use a sample `config.json`:
```json
{
  "database": {
    "type": "postgres",
    "port": 5432,
    "credentials": {
      "user": "appuser",
      "password": "securepassword"
    }
  },
  "security": {
    "enable_https": true,
    "min_tls_version": "TLSv1.3"
  },
  "logging": {
    "level": "INFO",
    "file": "/var/log/app.log"
  }
}
```

#### a. Parse into object and extract values

```powershell
$jsonFile = "config.json"
Set-Content -Path $jsonFile -Value @"
{
  "database": {
    "type": "postgres",
    "port": 5432,
    "credentials": {
      "user": "appuser",
      "password": "securepassword"
    }
  },
  "security": {
    "enable_https": true,
    "min_tls_version": "TLSv1.3"
  },
  "logging": {
    "level": "INFO",
    "file": "/var/log/app.log"
  }
}
"@

$config = Get-Content -Path $jsonFile | ConvertFrom-Json

Write-Host "DB Type: $($config.database.type)"
Write-Host "DB User: $($config.database.credentials.user)"
Write-Host "Min TLS Version: $($config.security.min_tls_version)"
```

### 4. Parsing YAML Files

Let's use a sample `config.yaml`:
```yaml
database:
  type: mysql
  port: 3306
  credentials:
    user: webuser
    password: anothersecurepassword
security:
  enable_https: true
  min_tls_version: TLSv1.2
logging:
  level: DEBUG
  file: /var/log/web.log
```

#### a. Parse into object and extract values

```powershell
# Requires module installation or a workaround for older PS versions
# Install-Module -Name PowerShellGet -Force
# Install-Module -Name PSSYaml -Force

$yamlFile = "config.yaml"
Set-Content -Path $yamlFile -Value @"
database:
  type: mysql
  port: 3306
  credentials:
    user: webuser
    password: anothersecurepassword
security:
  enable_https: true
  min_tls_version: TLSv1.2
logging:
  level: DEBUG
  file: /var/log/web.log
"@

# If ConvertFrom-Yaml is available:
# $config = Get-Content -Path $yamlFile | ConvertFrom-Yaml

# Basic workaround if ConvertFrom-Yaml is not available (requires Python/yq or similar external tool)
# Or manually parse with regex for very simple cases.
# For simplicity in this tutorial, we will assume ConvertFrom-Yaml is available in modern PS Core
try {
    $config = Get-Content -Path $yamlFile | ConvertFrom-Yaml
    Write-Host "DB Type (YAML): $($config.database.type)"
    Write-Host "Min TLS Version (YAML): $($config.security.min_tls_version)"
}
catch {
    Write-Warning "ConvertFrom-Yaml cmdlet not found. Please install PSSYaml module or use PowerShell Core 7+."
    # Fallback to simple regex parsing if needed
    if (Get-Content -Path $yamlFile | Select-String -Pattern "^\s*type:\s*(.*)$" -CaseSensitive:$false) {
        $yamlDbType = (Get-Content -Path $yamlFile | Select-String -Pattern "^\s*type:\s*(.*)$").Matches[0].Groups[1].Value.Trim()
        Write-Host "DB Type (YAML via regex): $yamlDbType"
    }
}
```

### 5. Validation

#### a. Check if a key exists

```powershell
# Check if 'logging.level' exists in JSON config
if ($config.logging.level -ne $null) {
    Write-Host "Logging level setting found."
} else {
    Write-Host "Logging level setting NOT found."
}
```

#### b. Validate a value against an expected value or pattern

```powershell
$ExpectedMinTls = "TLSv1.3"
$ActualMinTls = $config.security.min_tls_version

if ($ActualMinTls -eq $ExpectedMinTls) {
    Write-Host "PASS: Min TLS version is set correctly to $ExpectedMinTls."
} else {
    Write-Host "FAIL: Min TLS version is '$ActualMinTls', expected '$ExpectedMinTls'."
}

# Validate DB port is within a range (e.g., > 1024)
$DbPort = $config.database.port
if ($DbPort -gt 1024) {
    Write-Host "PASS: DB port $DbPort is above 1024."
} else {
    Write-Host "FAIL: DB port $DbPort is not above 1024."
}
```

#### c. Policy Compliance Check (e.g., `enable_https` must be `true`)

```powershell
$EnableHttps = $config.security.enable_https

if ($EnableHttps -eq $true) {
    Write-Host "PASS: HTTPS is enabled."
} else {
    Write-Host "SECURITY ALERT: HTTPS is NOT enabled. It is set to '$EnableHttps'."
}
```

## Building a Configuration Validator Script

```powershell
# config_validator.ps1

param (
    [string]$ConfigFile,
    [string]$ConfigType # "ini", "json", "yaml", "env"
)

# Function to get INI values (as defined above)
function Get-IniValue { ... }
# Function to get ENV values (as defined above)
function Get-EnvValue { ... }

# --- Main Validation Logic ---
$validationReport = [System.Collections.ArrayList]::new()
$parsedConfig = $null

if (-not (Test-Path $ConfigFile)) {
    $validationReport.Add("FAIL: Configuration file '$ConfigFile' not found.") | Out-Null
} else {
    switch ($ConfigType) {
        "ini" {
            # In a real script, parse the whole INI file into a hashtable
            # For simplicity, we will call Get-IniValue directly for checks
        }
        "json" {
            try { $parsedConfig = Get-Content -Path $ConfigFile | ConvertFrom-Json }
            catch { $validationReport.Add("ERROR: Failed to parse JSON file: $_.Exception.Message") | Out-Null }
        }
        "yaml" {
            try { $parsedConfig = Get-Content -Path $ConfigFile | ConvertFrom-Yaml } # Assumes ConvertFrom-Yaml is available
            catch { $validationReport.Add("ERROR: Failed to parse YAML file: $_.Exception.Message. Fallback to regex might be needed.") | Out-Null }
        }
        "env" {
            # In a real script, parse the whole .env file into a hashtable
            # For simplicity, we will call Get-EnvValue directly for checks
        }
        default {
            $validationReport.Add("ERROR: Unsupported configuration type '$ConfigType'.") | Out-Null
        }
    }
}

Write-Host "--- Configuration Validation Report for $ConfigFile ($ConfigType) ---"

if ($parsedConfig -ne $null) {
    # JSON/YAML specific checks
    # Check 1: Min TLS Version must be TLSv1.3
    $ExpectedMinTls = "TLSv1.3"
    if ($parsedConfig.security.min_tls_version -eq $ExpectedMinTls) {
        $validationReport.Add("PASS: Min TLS version is set to $ExpectedMinTls.") | Out-Null
    } else {
        $validationReport.Add("FAIL: Min TLS version is '$($parsedConfig.security.min_tls_version)', expected '$ExpectedMinTls'.") | Out-Null
    }

    # Check 2: Database port must be > 1024
    $DbPort = $parsedConfig.database.port
    if ($DbPort -gt 1024) {
        $validationReport.Add("PASS: DB port $DbPort is above 1024.") | Out-Null
    } else {
        $validationReport.Add("FAIL: DB port $DbPort is not above 1024.") | Out-Null
    }

    # Check 3: HTTPS must be enabled
    $EnableHttps = $parsedConfig.security.enable_https
    if ($EnableHttps -eq $true) {
        $validationReport.Add("PASS: HTTPS is enabled.") | Out-Null
    } else {
        $validationReport.Add("SECURITY ALERT: HTTPS is NOT enabled. It is set to '$EnableHttps'.") | Out-Null
    }
}

# If INI or ENV, perform checks using Get-IniValue/Get-EnvValue functions
if ($ConfigType -eq "ini") {
    # Check PermitRootLogin (from app.conf example)
    $ActualPRL = Get-IniValue -Path $ConfigFile -Section "security" -Key "permit_root_login"
    if ($ActualPRL -eq "false") {
        $validationReport.Add("PASS: PermitRootLogin is 'false'.") | Out-Null
    } else {
        $validationReport.Add("SECURITY ALERT: PermitRootLogin is '$ActualPRL', expected 'false'.") | Out-Null
    }
}
if ($ConfigType -eq "env") {
    # Check APP_DEBUG (from .env example)
    $ActualAppDebug = Get-EnvValue -Path $ConfigFile -Key "APP_DEBUG"
    if ($ActualAppDebug -eq "true") {
        $validationReport.Add("SECURITY ALERT: APP_DEBUG is 'true', should be 'false' in production!") | Out-Null
    } else {
        $validationReport.Add("PASS: APP_DEBUG is not 'true'.") | Out-Null
    }
}

$validationReport | ForEach-Object { Write-Host $_ }
```
To run: `.\config_validator.ps1 -ConfigFile config.json -ConfigType json`

## Guiding Principles in PowerShell

*   **Portability:** PowerShell Core makes parsing `Get-Content`, `ConvertFrom-Json`, `ConvertFrom-Yaml` available cross-platform. Regular expressions (`-match`) are also cross-platform.
*   **Efficiency:** Native cmdlets for structured data parsing (`ConvertFrom-Json`, `ConvertFrom-Yaml`) are highly optimized.
*   **Minimal Dependencies:** Relies on PowerShell runtime and its built-in modules. For YAML, `ConvertFrom-Yaml` is available in PS Core 7+ or via a module installation.
*   **CLI-centric:** Scripts are executed from the command line, accepting parameters for configuration file path and type.
*   **Structured Data Handling:** PowerShell's object pipeline is invaluable. Once data is converted to objects, validating properties is straightforward and robust, without complex text parsing.

## Conclusion

PowerShell provides a powerful and effective environment for parsing and validating configuration files, particularly for structured formats like JSON and YAML. Its object-oriented nature simplifies data extraction and enables robust, policy-driven validation. The ability to produce clear, actionable reports makes it an invaluable tool for ensuring system hardening, compliance, and automated configuration management. The next step is to apply this knowledge in practical exercises.
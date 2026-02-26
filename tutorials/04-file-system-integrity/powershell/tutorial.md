# PowerShell Tutorial: File System Search and Integrity Check

## Introduction

PowerShell provides a robust and object-oriented approach to managing and inspecting file systems. With cmdlets designed for recursive directory traversal, content searching, and cryptographic hashing, it's an excellent tool for performing detailed file system searches and crucial integrity checks. This tutorial will guide you through using `Get-ChildItem`, `Get-FileHash`, `Select-String`, and `Compare-Object` to implement our file system search and integrity check design, emphasizing PowerShell's strengths in producing structured, actionable data.

## Framework Alignment

This tutorial on "**File System Search and Integrity Check**" using **PowerShell** demonstrates the foundational skills necessary to implement components of a "Cross-Platform Baseline Hardening & Auditing Framework." The techniques learned here for searching files and calculating cryptographic hashes are directly applicable to auditing systems against defined security baselines and ensuring compliance.


## Core PowerShell Cmdlets for File System Operations

*   **`Get-ChildItem` (alias `ls`, `dir`)**: Gets the items and child items in one or more specified locations. Essential for traversing directories.
    *   `-Recurse`: Recurses into subdirectories.
    *   `-Path`: Specifies the path to search.
    *   `-Include`, `-Exclude`: Specify patterns for items to include/exclude.
    *   `-File`, `-Directory`: Filters for files or directories only.
*   **`Where-Object` (alias `where`)**: Filters objects based on property values. Used for filtering `Get-ChildItem` results by size, date, etc.
*   **`Select-String`**: Searches for text and string patterns in input strings or files. The PowerShell equivalent of `grep`.
*   **`Get-FileHash`**: Computes the hash value for a file by using a specified hash algorithm. Supports MD5, SHA1, SHA256, etc.
*   **`Export-Csv` / `ConvertTo-Json`**: For exporting structured data (e.g., search results, baseline hashes).
*   **`Import-Csv` / `ConvertFrom-Json`**: For importing structured data (e.g., baseline hashes).
*   **`Compare-Object` (alias `diff`)**: Compares two sets of objects. Invaluable for comparing file lists or hash baselines.

## Implementing Core Functionality with PowerShell

### 1. File System Search

#### a. By Name (`Get-ChildItem` with `-Include` / `Where-Object`)

```powershell
# Find all files named 'report.log' in a directory and subdirectories
Get-ChildItem -Path ".\Logs" -Recurse -File -Include "report.log"

# Find all files with '.conf' extension
Get-ChildItem -Path ".\Config" -Recurse -File -Include "*.conf"
```

#### b. By Size (`Get-ChildItem` with `Where-Object`)

```powershell
# Find all files larger than 1MB
Get-ChildItem -Path ".\Data" -Recurse -File | Where-Object {$_.Length -gt 1MB}

# Find all files smaller than 10KB
Get-ChildItem -Path ".\Temp" -Recurse -File | Where-Object {$_.Length -lt 10KB}
```

#### c. By Modification Time (`Get-ChildItem` with `Where-Object`)

```powershell
# Find files modified in the last 24 hours
$24HoursAgo = (Get-Date).AddHours(-24)
Get-ChildItem -Path ".\Data" -Recurse -File | Where-Object {$_.LastWriteTime -gt $24HoursAgo}

# Find files modified before a specific date
$SpecificDate = [datetime]"2026-01-01"
Get-ChildItem -Path ".\Archives" -Recurse -File | Where-Object {$_.LastWriteTime -lt $SpecificDate}
```

#### d. By Content (`Select-String`)

```powershell
# Find lines containing "ERROR" recursively in files under .\Logs
Get-ChildItem -Path ".\Logs" -Recurse -File | Select-String -Pattern "ERROR" -CaseSensitive:$false

# List only the filenames containing "secret_key"
Get-ChildItem -Path ".\Confidential" -Recurse -File | Select-String -Pattern "secret_key" -List | Select-Object -ExpandProperty Path
```

### 2. Integrity Check (Hashing)

#### a. Generate Hash of a Single File

```powershell
# Generate SHA256 hash for a file
Get-FileHash -Path ".\Config\app.conf" -Algorithm SHA256

# Generate MD5 hash for a file
Get-FileHash -Path ".\Data\my_document.txt" -Algorithm MD5
```

#### b. Generate Hashes for Multiple Files (e.g., from `Get-ChildItem` output)

```powershell
# Find all .ps1 files and generate their SHA256 hashes
Get-ChildItem -Path ".\Scripts" -Recurse -File -Include "*.ps1" | Get-FileHash -Algorithm SHA256
```

#### c. Verify Hash of a Single File (against a known hash)

```powershell
$KnownHash = "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855" # SHA256 of empty file
$FileToCheck = ".\Temp\empty.txt"

$CurrentHash = (Get-FileHash -Path $FileToCheck -Algorithm SHA256).Hash

if ($CurrentHash -eq $KnownHash) {
    Write-Host "Integrity check PASSED for $FileToCheck."
} else {
    Write-Host "Integrity check FAILED for $FileToCheck. Hash mismatch!"
    Write-Host "  Known Hash: $KnownHash"
    Write-Host "  Current Hash: $CurrentHash"
}
```

### 3. Basic Baseline Management

#### a. Generate a Baseline File (as CSV or JSON)

```powershell
# Generate a SHA256 baseline for a directory and export to CSV
$BasePath = ".\SourceData"
Get-ChildItem -Path $BasePath -Recurse -File |
    ForEach-Object {
        $hash = (Get-FileHash -Path $_.FullName -Algorithm SHA256).Hash
        [PSCustomObject]@{
            Path = $_.FullName
            RelativePath = $_.FullName.Substring($BasePath.Length + 1)
            Hash = $hash
            LastWriteTime = $_.LastWriteTime
            Length = $_.Length
        }
    } | Export-Csv -Path ".\Baselines\App_Baseline.csv" -NoTypeInformation

# To JSON
# ... | ConvertTo-Json -Depth 3 | Set-Content -Path ".\Baselines\App_Baseline.json"
```

#### b. Compare Current State Against Baseline

```powershell
# Load the baseline
$Baseline = Import-Csv -Path ".\Baselines\App_Baseline.csv"

# Generate current hashes for comparison
$BasePath = ".\SourceData"
$CurrentState = Get-ChildItem -Path $BasePath -Recurse -File |
    ForEach-Object {
        $hash = (Get-FileHash -Path $_.FullName -Algorithm SHA256).Hash
        [PSCustomObject]@{
            Path = $_.FullName
            RelativePath = $_.FullName.Substring($BasePath.Length + 1)
            Hash = $hash
            LastWriteTime = $_.LastWriteTime
            Length = $_.Length
        }
    }

Write-Host "--- Integrity Check Report ---"
# Compare based on RelativePath and Hash
$Comparison = Compare-Object -ReferenceObject $Baseline -DifferenceObject $CurrentState -Property RelativePath, Hash -PassThru

foreach ($item in $Comparison) {
    if ($item.SideIndicator -eq "=>") {
        Write-Host "NEW: $($item.Path)" -ForegroundColor Green
    } elseif ($item.SideIndicator -eq "<=") {
        Write-Host "DELETED: $($item.Path)" -ForegroundColor Red
    }
}

# Find modified files after initial comparison
$ModifiedFiles = Compare-Object -ReferenceObject $Baseline -DifferenceObject $CurrentState -Property RelativePath |
    Where-Object {$_.SideIndicator -eq "=="} | 
    ForEach-Object {
        $baselineEntry = $Baseline | Where-Object {$_.RelativePath -eq $_.RelativePath}
        $currentEntry = $CurrentState | Where-Object {$_.RelativePath -eq $_.RelativePath}
        if ($baselineEntry.Hash -ne $currentEntry.Hash) {
            $currentEntry # This is a modified file
        }
    }

if ($ModifiedFiles) {
    Write-Host "`n--- Modified Files (Hash Mismatch) ---"
    $ModifiedFiles | Format-Table Path, Hash, LastWriteTime -AutoSize
}
```

## Guiding Principles in PowerShell

*   **Portability:** PowerShell Core enables cross-platform execution. Cmdlets like `Get-ChildItem`, `Get-FileHash`, `Select-String` are available across platforms.
*   **Efficiency:** Native cmdlets are optimized for performance. `Get-FileHash` uses efficient hashing algorithms.
*   **Minimal Dependencies:** Relies entirely on the PowerShell runtime and its built-in modules.
*   **CLI-centric:** PowerShell scripts are executed directly from the console and leverage its robust argument parsing.
*   **Structured Data Handling:** Objects are processed throughout the pipeline, simplifying filtering, sorting, and exporting to structured formats like CSV or JSON. `Compare-Object` is particularly powerful for comparing structured data.

## Conclusion

PowerShell provides a powerful and flexible environment for performing file system searches and integrity checks. Its object-oriented nature, combined with specialized cmdlets for file operations, hashing, and object comparison, allows for the creation of robust and highly accurate tools for security auditing and system administration. The ability to easily output structured data enhances its utility for automation and integration. The next step is to apply this knowledge in practical exercises.

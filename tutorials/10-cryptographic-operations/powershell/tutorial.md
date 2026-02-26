# PowerShell Tutorial: Basic Cryptographic Operations (Hashing/Encoding)

## Introduction

PowerShell, with its deep integration with the .NET framework, offers robust and flexible capabilities for performing basic cryptographic operations. This includes calculating file and string hashes for integrity verification, and encoding/decoding data using schemes like Base64, URL encoding, and hexadecimal representation. These operations are fundamental for data handling, security analysis, and automation across Windows and cross-platform PowerShell Core environments, all while adhering to our principles of minimal dependencies and CLI-centric operation.

## Framework Alignment

This tutorial on "**Basic Cryptographic Operations (Hashing/Encoding)**" using **PowerShell** demonstrates the foundational skills necessary to implement components of a "Cross-Platform Baseline Hardening & Auditing Framework." The techniques learned here for performing hashing and encoding are directly applicable to auditing systems against defined security baselines and ensuring compliance.


## Core PowerShell Cmdlets and .NET Classes for Cryptographic Operations

*   **`Get-FileHash`**: Computes the hash value for a file by using a specified hash algorithm. Supports MD5, SHA1, SHA256, SHA384, SHA512, MACTripleDES, RIPEMD160.
*   **`.NET Classes`**: Direct access to .NET classes provides comprehensive functionality.
    *   `[System.Security.Cryptography.HashAlgorithm]::Create()`: For hashing strings.
    *   `[System.Text.Encoding]::UTF8.GetBytes()`: To convert strings to byte arrays.
    *   `[System.Convert]::ToBase64String()` / `[System.Convert]::FromBase64String()`: For Base64 encoding/decoding.
    *   `[System.Uri]::EscapeDataString()` / `[System.Uri]::UnescapeDataString()`: For URL encoding/decoding.
    *   `[System.BitConverter]::ToString()`: For converting byte arrays to hexadecimal strings.

## Implementing Core Functionality with PowerShell

Let's create a sample file and string first:

```powershell
Set-Content -Path "sample.txt" -Value "Hello, World!"
Set-Content -Path "sample_no_newline.txt" -Value "Hello, World!" -NoNewline
```

### 1. Hashing (Data Integrity)

#### a. Hash a file

```powershell
$fileToHash = "sample.txt"

# Get SHA256 hash of a file
Write-Host "SHA256 of $fileToHash: $((Get-FileHash -Path $fileToHash -Algorithm SHA256).Hash)"

# Get MD5 hash of a file (use with caution for security-critical applications)
Write-Host "MD5 of $fileToHash: $((Get-FileHash -Path $fileToHash -Algorithm MD5).Hash)"
```

#### b. Hash a string

```powershell
$stringToHash = "MySecretPassword123"

# Hash string using SHA256
$hasher = [System.Security.Cryptography.SHA256]::Create()
$bytes = [System.Text.Encoding]::UTF8.GetBytes($stringToHash)
$hash = $hasher.ComputeHash($bytes)
$hashString = [System.BitConverter]::ToString($hash) -Replace '-' , ''
Write-Host "SHA256 of string: $hashString"

# Hash string using MD5
$hasherMd5 = [System.Security.Cryptography.MD5]::Create()
$bytesMd5 = [System.Text.Encoding]::UTF8.GetBytes($stringToHash)
$hashMd5 = $hasherMd5.ComputeHash($bytesMd5)
$hashStringMd5 = [System.BitConverter]::ToString($hashMd5) -Replace '-' , ''
Write-Host "MD5 of string: $hashStringMd5"
```

### 2. Base64 Encoding/Decoding

#### a. Encode a string to Base64

```powershell
$stringToEncode = "This is a test string for Base64 encoding."

$bytes = [System.Text.Encoding]::UTF8.GetBytes($stringToEncode)
$base64String = [System.Convert]::ToBase64String($bytes)
Write-Host "Base64 encoded string: $base64String"
```

#### b. Decode a Base64 string

```powershell
$base64String = "VGhpcyBpcyBhIHRlc3Qgc3RyaW5nIGZvciBCYXNlNjQgZW5jb2Rpbmcu"

$bytes = [System.Convert]::FromBase64String($base64String)
$decodedString = [System.Text.Encoding]::UTF8.GetString($bytes)
Write-Host "Base64 decoded string: $decodedString"
```

#### c. Encode a file to Base64

```powershell
$fileToEncode = "sample.txt"

$bytes = [System.IO.File]::ReadAllBytes($fileToEncode)
$base64FileContent = [System.Convert]::ToBase64String($bytes)
Write-Host "Base64 encoded file content (first 100 chars): $($base64FileContent.Substring(0, 100))..."
```

#### d. Decode a Base64 file (redirect output to a new file)

```powershell
$base64FileContent = "SGVsbG8sIFdvcmxkIQo=" # Base64 for "Hello, World!
"
$decodedBytes = [System.Convert]::FromBase64String($base64FileContent)
[System.IO.File]::WriteAllBytes("decoded_sample.txt", $decodedBytes)

Write-Host "Decoded content saved to decoded_sample.txt"
Get-Content -Path "decoded_sample.txt"
```

### 3. URL Encoding/Decoding

#### a. URL Encode a string

```powershell
$stringToUrlEncode = "https://example.com/search?query=hello world & test"

$urlEncodedString = [System.Uri]::EscapeDataString($stringToUrlEncode)
Write-Host "URL encoded string: $urlEncodedString"
```

#### b. URL Decode a string

```powershell
$urlEncodedString = "https%3A%2F%2Fexample.com%2Fsearch%3Fquery%3Dhello%20world%20%26%20test"

$urlDecodedString = [System.Uri]::UnescapeDataString($urlEncodedString)
Write-Host "URL decoded string: $urlDecodedString"
```

### 4. Hex Encoding/Decoding

#### a. Encode a string to Hex

```powershell
$stringToHexEncode = "Hello"

$bytes = [System.Text.Encoding]::UTF8.GetBytes($stringToHexEncode)
$hexString = [System.BitConverter]::ToString($bytes) -Replace '-' , ''
Write-Host "Hex encoded string: $hexString"
```

#### b. Decode a Hex string

```powershell
$hexString = "48656C6C6F" # "Hello" in hex

# Convert hex string to byte array
$bytes = for ($i=0; $i -lt $hexString.Length; $i+=2) {
    [byte]::Parse($hexString.Substring($i, 2), [System.Globalization.NumberStyles]::HexNumber)
}
$decodedString = [System.Text.Encoding]::UTF8.GetString($bytes)
Write-Host "Hex decoded string: $decodedString"
```

### 5. Consolidated Script Example (`crypto_operations.ps1`)

```powershell
# crypto_operations.ps1 - A simple PowerShell utility for basic cryptographic operations

[CmdletBinding()]
param (
    [Parameter(Mandatory=$true, HelpMessage="Operation to perform: Hash, Base64Encode, Base64Decode, UrlEncode, UrlDecode, HexEncode, HexDecode")]
    [ValidateSet("Hash", "Base64Encode", "Base64Decode", "UrlEncode", "UrlDecode", "HexEncode", "HexDecode")]
    [string]$Operation,

    [Parameter(Mandatory=$true, HelpMessage="Input string or file path")]
    [string]$Input,

    [Parameter(HelpMessage="Hashing algorithm (MD5, SHA256, SHA512) for 'Hash' operation")]
    [ValidateSet("MD5", "SHA1", "SHA256", "SHA384", "SHA512")]
    [string]$Algorithm = "SHA256"
)

function Get-StringHash {
    param (
        [string]$Value,
        [string]$Algorithm
    )
    $hasher = [System.Security.Cryptography.$Algorithm]::Create()
    $bytes = [System.Text.Encoding]::UTF8.GetBytes($Value)
    $hash = $hasher.ComputeHash($bytes)
    return ([System.BitConverter]::ToString($hash) -Replace '-' , '')
}

function Convert-StringToHex {
    param (
        [string]$Value
    )
    $bytes = [System.Text.Encoding]::UTF8.GetBytes($Value)
    return ([System.BitConverter]::ToString($bytes) -Replace '-' , '')
}

function Convert-HexToString {
    param (
        [string]$HexString
    )
    $bytes = for ($i=0; $i -lt $HexString.Length; $i+=2) {
        [byte]::Parse($HexString.Substring($i, 2), [System.Globalization.NumberStyles]::HexNumber)
    }
    return [System.Text.Encoding]::UTF8.GetString($bytes)
}

$outputObject = [PSCustomObject]@{
    Operation = $Operation
    Input = $Input
    Algorithm = $Algorithm
    Result = $null
    Error = $null
    Timestamp = (Get-Date).ToUniversalTime().ToString("yyyy-MM-ddTHH:mm:ssZ")
}

try {
    switch ($Operation) {
        "Hash" {
            if (Test-Path $Input -PathType Leaf) { # Input is a file
                $outputObject.Result = (Get-FileHash -Path $Input -Algorithm $Algorithm).Hash
            } else { # Input is a string
                $outputObject.Result = Get-StringHash -Value $Input -Algorithm $Algorithm
            }
        }
        "Base64Encode" {
            if (Test-Path $Input -PathType Leaf) { # Input is a file
                $bytes = [System.IO.File]::ReadAllBytes($Input)
                $outputObject.Result = [System.Convert]::ToBase64String($bytes)
            } else { # Input is a string
                $bytes = [System.Text.Encoding]::UTF8.GetBytes($Input)
                $outputObject.Result = [System.Convert]::ToBase64String($bytes)
            }
        }
        "Base64Decode" {
            if (Test-Path $Input -PathType Leaf) { # Input is a file (containing base64 string)
                $base64Content = Get-Content -Path $Input | Out-String
                $bytes = [System.Convert]::FromBase64String($base64Content)
                # Output as a string if it's text, otherwise to file or raw bytes
                $outputObject.Result = [System.Text.Encoding]::UTF8.GetString($bytes)
            } else { # Input is a string
                $bytes = [System.Convert]::FromBase64String($Input)
                $outputObject.Result = [System.Text.Encoding]::UTF8.GetString($bytes)
            }
        }
        "UrlEncode" {
            $outputObject.Result = [System.Uri]::EscapeDataString($Input)
        }
        "UrlDecode" {
            $outputObject.Result = [System.Uri]::UnescapeDataString($Input)
        }
        "HexEncode" {
            if (Test-Path $Input -PathType Leaf) { # Input is a file
                $bytes = [System.IO.File]::ReadAllBytes($Input)
                $outputObject.Result = ([System.BitConverter]::ToString($bytes) -Replace '-' , '')
            } else { # Input is a string
                $outputObject.Result = Convert-StringToHex -Value $Input
            }
        }
        "HexDecode" {
            $outputObject.Result = Convert-HexToString -HexString $Input
        }
    }
}
catch {
    $outputObject.Error = $_.Exception.Message
    $outputObject.Result = $null
}

$outputObject | ConvertTo-Json -Depth 5
```
Example Usage:
```powershell
.\crypto_operations.ps1 -Operation Hash -Input "sample.txt" -Algorithm SHA256 | ConvertFrom-Json
.\crypto_operations.ps1 -Operation Base64Encode -Input "secret_data" | ConvertFrom-Json
.\crypto_operations.ps1 -Operation UrlEncode -Input "https://example.com?q=test string" | ConvertFrom-Json
```

## Guiding Principles in PowerShell

*   **Portability:** PowerShell Core allows these operations to be performed consistently across Windows, Linux, and macOS environments due to its cross-platform nature and direct access to .NET Core functionality.
*   **Efficiency:** Direct access to the optimized .NET cryptography and conversion classes ensures high performance, especially for large files or complex operations.
*   **Minimal Dependencies:** Relies entirely on the PowerShell runtime and the .NET framework, requiring no external installations beyond PowerShell itself.
*   **CLI-centric:** Designed as a command-line script accepting parameters, ideal for integration into larger automation and security workflows.
*   **Structured Data Handling:** PowerShell's object pipeline facilitates handling inputs and outputs as objects, which can then be easily converted to JSON for machine-readable reports.

## Conclusion

PowerShell provides a powerful and versatile environment for performing basic cryptographic operations. Its seamless integration with the .NET framework allows access to highly optimized and reliable functions for hashing, encoding, and decoding. This makes it an invaluable tool for system administrators, security analysts, and developers seeking to automate data integrity checks, safely transfer data, and perform basic data transformations across diverse operating systems. The next step is to apply this knowledge in practical exercises.
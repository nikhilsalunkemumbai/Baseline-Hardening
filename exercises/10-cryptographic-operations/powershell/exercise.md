# PowerShell Exercise: Basic Cryptographic Operations (Hashing/Encoding)

## Objective

This exercise challenges you to apply your PowerShell scripting and cmdlet skills to perform fundamental cryptographic operations. You will learn to calculate hash checksums for file integrity verification, and encode/decode data using Base64, URL encoding, and hexadecimal representations, demonstrating proficiency in data handling and basic security practices.

## Framework Alignment

This exercise on "**Basic Cryptographic Operations (Hashing/Encoding)**" using **PowerShell** provides practical experience in building and utilizing tools for a "Cross-Platform Baseline Hardening & Auditing Framework." By completing these tasks, you will learn how to verify data integrity and detect unauthorized modifications—essential steps in maintaining a secure and auditable environment.


## Scenario

You are a system administrator or security analyst needing to perform quick data integrity checks, prepare data for web requests, or manipulate encoded strings. This involves tasks like verifying downloaded files, ensuring configuration file integrity, or handling text that has been Base64 or hex encoded.

## Setup

Before starting the tasks, create the following files in your current directory using PowerShell:

1.  **`test_document.txt`**:
    ```powershell
    Set-Content -Path "test_document.txt" -Value @"
    This is a test document.
    It contains some important information.
    Verify its integrity!
    "@
    Write-Host "Created test_document.txt"
    ```

2.  **`test_binary.bin`**:
    ```powershell
    [System.IO.File]::WriteAllBytes("test_binary.bin", [byte[]](0xDE,0xAD,0xBE,0xEF,0x00,0x01,0x02,0x03))
    Write-Host "Created test_binary.bin"
    ```

## Tasks

Using only standard PowerShell cmdlets and .NET classes, provide the command-line solution for each of the following tasks.

### Part 1: Hashing Operations

1.  **Calculate SHA256 Hash of `test_document.txt`:**
    *   Compute and display the SHA256 hash of the `test_document.txt` file.

2.  **Calculate MD5 Hash of `test_document.txt`:**
    *   Compute and display the MD5 hash of the `test_document.txt` file. (Use with caution for security-critical applications).

3.  **Calculate SHA512 Hash of a String:**
    *   Compute and display the SHA512 hash of the string `"MyVerySecurePassword"` using .NET classes.

### Part 2: Encoding and Decoding (Base64)

1.  **Base64 Encode a String:**
    *   Encode the string `"Secret message for transmission"` to Base64.

2.  **Base64 Decode a String:**
    *   Decode the Base64 string `"SGVsbG8gRnJvbSBCYXNlNjQh"` back to its original form.

3.  **Base64 Encode `test_binary.bin`:**
    *   Encode the content of `test_binary.bin` to Base64.

4.  **Base64 Decode to a File:**
    *   Take the Base64 content `Cg==` (which represents a single newline character) and decode it, saving the result to a new file named `decoded_newline.txt`.

### Part 3: Encoding and Decoding (URL Encoding)

1.  **URL Encode a String:**
    *   URL encode the string `"https://example.com/search?query=hello world & param=val"`

2.  **URL Decode a String:**
    *   URL decode the string `"https%3A%2F%2Fexample.com%2Fsearch%3Fquery%3Dhello+world+%26+param%3Dval"`

### Part 4: Encoding and Decoding (Hexadecimal)

1.  **Hex Encode a String:**
    *   Encode the string `"PowerShell"` to its hexadecimal representation.

2.  **Hex Decode a String:**
    *   Decode the hexadecimal string `"706f7765727368656c6c"` back to its original form.

3.  **Hex Encode `test_binary.bin`:**
    *   Encode the content of `test_binary.bin` to its hexadecimal representation.

### Part 5: Simple Script Utility

1.  **Create a Hashing Script (`file_string_hasher.ps1`):**
    *   Write a PowerShell script that takes two parameters: `Input` (string or file path) and `Algorithm` (e.g., `MD5`, `SHA256`, `SHA512`).
    *   The script should automatically detect if `Input` is a file or a string.
    *   It should compute and print the hash of the given input using the specified algorithm.
    *   Include basic error handling (e.g., if the file doesn't exist or algorithm is unsupported).
    *   Output the result as a custom object, potentially convertible to JSON.

## Deliverables

For Parts 1, 2, 3, and 4, provide the exact PowerShell command-line solution for each task. For Part 5, provide the complete PowerShell script file (`file_string_hasher.ps1`).

## Reflection Questions

1.  How does `Get-FileHash` simplify file hashing compared to manually using .NET `HashAlgorithm` classes? When would you still choose to use the .NET classes directly for hashing?
2.  Explain the purpose of `[System.Text.Encoding]::UTF8.GetBytes()` and `[System.BitConverter]::ToString()` when working with hashes and string conversions in PowerShell.
3.  Describe the difference in use cases between Base64 encoding and URL encoding.
4.  What are the advantages of PowerShell's direct access to the .NET framework for cryptographic operations?
5.  How would you modify `file_string_hasher.ps1` to also accept a parameter to encode a string to Base64?

---
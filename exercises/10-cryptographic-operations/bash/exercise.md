# Bash Exercise: Basic Cryptographic Operations (Hashing/Encoding)

## Objective

This exercise challenges you to apply your Bash scripting and command-line utility skills to perform fundamental cryptographic operations. You will learn to calculate hash checksums for file integrity verification, and encode/decode data using Base64 and hexadecimal representations, demonstrating proficiency in data handling and basic security practices.

## Framework Alignment

This exercise on "**Basic Cryptographic Operations (Hashing/Encoding)**" using **Bash** provides practical experience in building and utilizing tools for a "Cross-Platform Baseline Hardening & Auditing Framework." By completing these tasks, you will learn how to verify data integrity and detect unauthorized modifications—essential steps in maintaining a secure and auditable environment.


## Scenario

You are a system administrator or security analyst needing to perform quick data integrity checks and manipulate encoded data. This often involves verifying downloaded files, ensuring configuration file integrity, or handling text that has been Base64 or hex encoded for transmission.

## Setup

Before starting the tasks, create the following files in your current directory:

1.  **`test_document.txt`**:
    ```bash
    cat <<EOL > test_document.txt
    This is a test document.
    It contains some important information.
    Verify its integrity!
    EOL
    echo "Created test_document.txt"
    ```

2.  **`test_binary.bin`**:
    ```bash
    printf '\xDE\xAD\xBE\xEF\x00\x01\x02\x03' > test_binary.bin
    echo "Created test_binary.bin"
    ```

## Tasks

Using only standard Bash commands and utilities (`md5sum`, `sha256sum`, `sha512sum`, `base64`, `xxd`, `printf`, `echo`, `cat`), provide the command-line solution for each of the following tasks.

### Part 1: Hashing Operations

1.  **Calculate SHA256 Hash of `test_document.txt`:**
    *   Compute and display the SHA256 hash of the `test_document.txt` file.

2.  **Calculate MD5 Hash of `test_document.txt`:**
    *   Compute and display the MD5 hash of the `test_document.txt` file. (Remember MD5's security limitations, but use it here for practice).

3.  **Calculate SHA512 Hash of a String:**
    *   Compute and display the SHA512 hash of the string `"MyVerySecurePassword"`.

### Part 2: Encoding and Decoding (Base64)

1.  **Base64 Encode a String:**
    *   Encode the string `"Secret message for transmission"` to Base64.

2.  **Base64 Decode a String:**
    *   Decode the Base64 string `"SGVsbG8gRnJvbSBCYXNlNjQh"` back to its original form.

3.  **Base64 Encode `test_binary.bin`:**
    *   Encode the content of `test_binary.bin` to Base64.

4.  **Base64 Decode a File (from a variable):**
    *   Take the Base64 content `Cg==` (which represents a single newline character) and decode it, saving the result to a new file named `decoded_newline.txt`.

### Part 3: Encoding and Decoding (Hexadecimal)

1.  **Hex Encode a String:**
    *   Encode the string `"Bash"` to its hexadecimal representation.

2.  **Hex Decode a String:**
    *   Decode the hexadecimal string `"666f6f626172"` back to its original form.

3.  **Hex Encode `test_binary.bin`:**
    *   Encode the content of `test_binary.bin` to its hexadecimal representation.

### Part 4: Simple Script Utility

1.  **Create a Hashing Script (`file_hasher.sh`):**
    *   Write a Bash script that takes two arguments: `file_path` and `algorithm` (e.g., `md5`, `sha256`, `sha512`).
    *   The script should compute and print the hash of the given file using the specified algorithm.
    *   Include basic error handling (e.g., if the file doesn't exist or algorithm is unsupported).

## Deliverables

For Parts 1, 2, and 3, provide the exact Bash command-line solution for each task. For Part 4, provide the complete Bash script file (`file_hasher.sh`).

## Reflection Questions

1.  What are the primary use cases for hashing files in system administration and security?
2.  Explain why MD5 is considered cryptographically broken for security-critical applications, while SHA256/SHA512 are still recommended for integrity checks.
3.  Describe scenarios where Base64 encoding is useful, and contrast its purpose with encryption.
4.  What are the advantages of using `xxd` or `printf` for hexadecimal encoding/decoding compared to manual conversion?
5.  How would you modify `file_hasher.sh` to also compute hashes for strings passed as input, rather than just files?

---
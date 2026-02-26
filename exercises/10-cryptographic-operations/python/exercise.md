# Python Exercise: Basic Cryptographic Operations (Hashing/Encoding)

## Objective

This exercise challenges you to apply your Python scripting skills to perform fundamental cryptographic operations. You will use Python's standard library modules to calculate hash checksums for file integrity verification, and encode/decode data using Base64, URL encoding, and hexadecimal representations, demonstrating proficiency in data handling and basic security practices.

## Framework Alignment

This exercise on "**Basic Cryptographic Operations (Hashing/Encoding)**" provides practical experience in building and utilizing tools for a "Cross-Platform Baseline Hardening & Auditing Framework." By completing these tasks, you will learn how to verify data integrity and detect unauthorized modifications—essential steps in maintaining a secure and auditable environment.


## Scenario

You are a developer or security professional needing a versatile tool to perform various cryptographic tasks. This includes validating file integrity (e.g., after downloads), securely transmitting data over text-only channels (Base64), preparing data for web requests (URL encoding), or debugging binary data (hex encoding).

## Setup

Before starting the tasks, create the following files in your current directory using Python snippets:

1.  **`test_document.txt`**:
    ```python
    from pathlib import Path
    Path("test_document.txt").write_text("""This is a test document.
    It contains some important information.
    Verify its integrity!""")
    print("Created test_document.txt")
    ```

2.  **`test_binary.bin`**:
    ```python
    from pathlib import Path
    Path("test_binary.bin").write_bytes(b"\xDE\xAD\xBE\xEF\x00\x01\x02\x03")
    print("Created test_binary.bin")
    ```

## Tasks

Write a single Python script (`crypto_cli.py`) that implements the following functionalities. Your script should be executable from the command line, take appropriate arguments, and output results in JSON format.

### Part 1: Hashing Operations

1.  **Hash `test_document.txt` (SHA256 & MD5):**
    *   Implement a function to hash a file given its path and an algorithm (`md5`, `sha256`, `sha512`).
    *   Call this function to get SHA256 and MD5 hashes of `test_document.txt`.

2.  **Hash a String (SHA512):**
    *   Implement a function to hash a string given the string and an algorithm.
    *   Call this function to get the SHA512 hash of `"MyVerySecurePassword"`.

### Part 2: Encoding and Decoding (Base64)

1.  **Base64 Encode a String:**
    *   Implement a function to Base64 encode a string.
    *   Encode `"Secret message for transmission"`.

2.  **Base64 Decode a String:**
    *   Implement a function to Base64 decode a string.
    *   Decode `"SGVsbG8gRnJvbSBCYXNlNjQh"`.

3.  **Base64 Encode `test_binary.bin`:**
    *   Implement a function to Base64 encode the content of a file.
    *   Encode `test_binary.bin`.

4.  **Base64 Decode to a File:**
    *   Implement a function to Base64 decode a string and write the result to a specified file.
    *   Decode the Base64 content `Cg==` (newline character) and save it to `decoded_newline.txt`.

### Part 3: Encoding and Decoding (URL Encoding)

1.  **URL Encode a String:**
    *   Implement a function to URL encode a string.
    *   Encode `"https://example.com/search?query=hello world & param=val"`.

2.  **URL Decode a String:**
    *   Implement a function to URL decode a string.
    *   Decode `"https%3A%2F%2Fexample.com%2Fsearch%3Fquery%3Dhello+world+%26+param%3Dval"`.

### Part 4: Encoding and Decoding (Hexadecimal)

1.  **Hex Encode a String:**
    *   Implement a function to hex encode a string.
    *   Encode `"Python"`.

2.  **Hex Decode a String:**
    *   Implement a function to hex decode a string.
    *   Decode `"707974686f6e"`.

3.  **Hex Encode `test_binary.bin`:**
    *   Implement a function to hex encode the content of a file.
    *   Encode `test_binary.bin`.

### Part 5: Command-Line Interface and JSON Output

Integrate all the above functions into a single Python script (`crypto_cli.py`) that uses `argparse` to allow users to specify the operation, input, and optional parameters (like algorithm for hashing). The script should:
*   Take arguments for operation (`hash`, `b64encode`, `b64decode`, `urlencode`, `urldecode`, `hexencode`, `hexdecode`).
*   Take an `input` argument (string or file path, with a flag to indicate if it's a file).
*   For `hash` operation, take an optional `algorithm` argument.
*   Output the result of the operation in JSON format to `stdout`.

## Deliverables

Provide the complete Python script file (`crypto_cli.py`) that implements all the above tasks with a command-line interface and JSON output.

## Reflection Questions

1.  Compare Python's `hashlib` module with Bash's `sha*sum` utilities and PowerShell's `Get-FileHash` cmdlet. What are the key advantages of each?
2.  Explain why it's generally safer to work with byte strings (`b'...'`) rather than regular strings when performing cryptographic operations in Python.
3.  Describe scenarios where you would choose Base64 encoding over hexadecimal encoding, and vice-versa.
4.  How does Python's `urllib.parse` module simplify URL manipulation compared to manual string operations or regular expressions?
5.  What are the security implications of exposing cryptographic operations via a command-line utility? What precautions should be taken (e.g., handling sensitive input, not using weak algorithms for security-critical tasks)?

# Python Tutorial: Basic Cryptographic Operations (Hashing/Encoding)

## Introduction

Python's comprehensive standard library provides robust and platform-independent modules for performing basic cryptographic operations. These include calculating hash checksums for data integrity, and encoding/decoding data using schemes like Base64, URL encoding, and hexadecimal representation. These functionalities are critical for various tasks in system administration, security analysis, and data processing, all while adhering to our principles of minimal dependencies and CLI-centric operation.

## Framework Alignment

This tutorial on "**Basic Cryptographic Operations (Hashing/Encoding)**" using **Python** demonstrates the foundational skills necessary to implement components of a "Cross-Platform Baseline Hardening & Auditing Framework." The techniques learned here for performing hashing and encoding are directly applicable to auditing systems against defined security baselines and ensuring compliance.


## Core Python Modules for Cryptographic Operations

*   **`hashlib`**: Provides various hashing algorithms (MD5, SHA1, SHA256, SHA512, etc.).
*   **`base64`**: Functions for encoding and decoding using Base64.
*   **`urllib.parse`**: Functions for URL parsing and encoding/decoding.
*   **`binascii`**: Functions to convert between binary and various ASCII-encoded binary representations (like hex).
*   **`os`**: For file system operations.
*   **`argparse`**: For parsing command-line arguments, making scripts user-friendly.

## Implementing Core Functionality with Python

Let's create a sample file and string first:

```python
import hashlib
import base64
import urllib.parse
import binascii
import os
import sys
import argparse
import json
from pathlib import Path

# Create dummy files for demonstration
Path("sample.txt").write_text("Hello, World!")
Path("sample_binary.bin").write_bytes(b"\x00\x01\x02\x03\xff\xfe")
```

### 1. Hashing (Data Integrity)

#### a. Hash a file

```python
def hash_file(file_path, algorithm="sha256"):
    """Computes the hash of a file."""
    h = hashlib.new(algorithm)
    try:
        with open(file_path, 'rb') as f:
            while chunk := f.read(4096):
                h.update(chunk)
        return h.hexdigest()
    except FileNotFoundError:
        return None
    except Exception as e:
        print(f"Error hashing file {file_path}: {e}", file=sys.stderr)
        return None

# Example Usage:
# file_path = "sample.txt"
# print(f"SHA256 of '{file_path}': {hash_file(file_path, 'sha256')}")
# print(f"MD5 of '{file_path}': {hash_file(file_path, 'md5')}") # Use MD5 with caution!
```

#### b. Hash a string

```python
def hash_string(input_string, algorithm="sha256"):
    """Computes the hash of a string."""
    h = hashlib.new(algorithm)
    h.update(input_string.encode('utf-8')) # Ensure input is bytes
    return h.hexdigest()

# Example Usage:
# string_to_hash = "MySecretPassword123"
# print(f"SHA256 of string: {hash_string(string_to_hash, 'sha256')}")
```

### 2. Base64 Encoding/Decoding

#### a. Encode a string to Base64

```python
def base64_encode_string(input_string):
    """Encodes a string to Base64."""
    return base64.b64encode(input_string.encode('utf-8')).decode('utf-8')

# Example Usage:
# string_to_encode = "This is a test string for Base64 encoding."
# print(f"Base64 encoded string: {base64_encode_string(string_to_encode)}")
```

#### b. Decode a Base64 string

```python
def base64_decode_string(b64_string):
    """Decodes a Base64 string."""
    try:
        return base64.b64decode(b64_string).decode('utf-8')
    except binascii.Error:
        print("Invalid Base64 string.", file=sys.stderr)
        return None

# Example Usage:
# b64_string = "VGhpcyBpcyBhIHRlc3Qgc3RyaW5nIGZvciBCYXNlNjQgZW5jb2Rpbmcu"
# print(f"Base64 decoded string: {base64_decode_string(b64_string)}")
```

#### c. Encode a file to Base64

```python
def base64_encode_file(file_path):
    """Encodes a file's content to Base64."""
    try:
        with open(file_path, 'rb') as f:
            encoded_bytes = base64.b64encode(f.read())
            return encoded_bytes.decode('utf-8')
    except FileNotFoundError:
        return None

# Example Usage:
# file_path = "sample_binary.bin"
# print(f"Base64 encoded file (first 50 chars): {base64_encode_file(file_path)[:50]}...")
```

#### d. Decode a Base64 file (writes to new file)

```python
def base64_decode_to_file(b64_string, output_file_path):
    """Decodes a Base64 string and writes to a file."""
    try:
        decoded_bytes = base64.b64decode(b64_string)
        with open(output_file_path, 'wb') as f:
            f.write(decoded_bytes)
        return True
    except binascii.Error:
        print("Invalid Base64 string for decoding.", file=sys.stderr)
        return False
    except Exception as e:
        print(f"Error decoding to file: {e}", file=sys.stderr)
        return False

# Example Usage:
# b64_content = base64_encode_file("sample_binary.bin")
# if b64_content:
#     base64_decode_to_file(b64_content, "decoded_sample_binary.bin")
#     print("Decoded binary file created: decoded_sample_binary.bin")
```

### 3. URL Encoding/Decoding

#### a. URL Encode a string

```python
def url_encode_string(input_string):
    """URL-encodes a string."""
    return urllib.parse.quote_plus(input_string)

# Example Usage:
# url_string = "https://example.com/search?query=hello world & test"
# print(f"URL encoded string: {url_encode_string(url_string)}")
```

#### b. URL Decode a string

```python
def url_decode_string(url_encoded_string):
    """URL-decodes a string."""
    return urllib.parse.unquote_plus(url_encoded_string)

# Example Usage:
# encoded_url_string = "https%3A%2F%2Fexample.com%2Fsearch%3Fquery%3Dhello+world+%26+test"
# print(f"URL decoded string: {url_decode_string(encoded_url_string)}")
```

### 4. Hex Encoding/Decoding

#### a. Encode a string to Hex

```python
def hex_encode_string(input_string):
    """Encodes a string to hexadecimal."""
    return binascii.hexlify(input_string.encode('utf-8')).decode('utf-8')

# Example Usage:
# print(f"Hex encoded string 'Hello': {hex_encode_string('Hello')}")
```

#### b. Decode a Hex string

```python
def hex_decode_string(hex_string):
    """Decodes a hexadecimal string."""
    try:
        return binascii.unhexlify(hex_string).decode('utf-8')
    except binascii.Error:
        print("Invalid hexadecimal string.", file=sys.stderr)
        return None

# Example Usage:
# hex_val = "48656c6c6f" # "Hello"
# print(f"Hex decoded string '48656c6c6f': {hex_decode_string(hex_val)}")
```

### 5. Consolidated Python Script (`crypto_tool.py`)

```python
#!/usr/bin/env python3

import hashlib
import base64
import urllib.parse
import binascii
import os
import sys
import argparse
import json
from datetime import datetime
from pathlib import Path

# --- Hashing Functions ---
def hash_data(data, is_file, algorithm="sha256"):
    h = hashlib.new(algorithm)
    if is_file:
        try:
            with open(data, 'rb') as f:
                while chunk := f.read(4096):
                    h.update(chunk)
        except FileNotFoundError:
            return None, "File not found"
        except Exception as e:
            return None, f"Error reading file: {e}"
    else:
        h.update(data.encode('utf-8'))
    return h.hexdigest(), None

# --- Base64 Functions ---
def base64_encode_data(data, is_file):
    try:
        if is_file:
            with open(data, 'rb') as f:
                encoded_bytes = base64.b64encode(f.read())
        else:
            encoded_bytes = base64.b64encode(data.encode('utf-8'))
        return encoded_bytes.decode('utf-8'), None
    except FileNotFoundError:
        return None, "File not found"
    except Exception as e:
        return None, f"Error encoding: {e}"

def base64_decode_data(data, is_file):
    try:
        if is_file: # Assuming file contains base64 string
            with open(data, 'r') as f:
                b64_string = f.read().strip()
        else:
            b64_string = data
        
        decoded_bytes = base64.b64decode(b64_string)
        return decoded_bytes.decode('utf-8'), None # Try to decode as UTF-8
    except FileNotFoundError:
        return None, "File not found"
    except binascii.Error:
        return None, "Invalid Base64 string"
    except UnicodeDecodeError:
        return decoded_bytes.hex(), "Decoded to non-UTF-8 binary, returned as hex" # Return hex if not text
    except Exception as e:
        return None, f"Error decoding: {e}"

# --- URL Encoding/Decoding Functions ---
def url_encode_data(data):
    return urllib.parse.quote_plus(data), None

def url_decode_data(data):
    return urllib.parse.unquote_plus(data), None

# --- Hex Encoding/Decoding Functions ---
def hex_encode_data(data, is_file):
    try:
        if is_file:
            with open(data, 'rb') as f:
                encoded_bytes = binascii.hexlify(f.read())
        else:
            encoded_bytes = binascii.hexlify(data.encode('utf-8'))
        return encoded_bytes.decode('utf-8'), None
    except FileNotFoundError:
        return None, "File not found"
    except Exception as e:
        return None, f"Error encoding: {e}"

def hex_decode_data(data):
    try:
        decoded_bytes = binascii.unhexlify(data)
        return decoded_bytes.decode('utf-8'), None
    except binascii.Error:
        return None, "Invalid hexadecimal string"
    except UnicodeDecodeError:
        return decoded_bytes.hex(), "Decoded to non-UTF-8 binary, returned as hex" # Return hex if not text
    except Exception as e:
        return None, f"Error decoding: {e}"

def main():
    parser = argparse.ArgumentParser(description="Perform basic cryptographic operations (hashing, encoding/decoding).")
    parser.add_argument("operation", choices=['hash', 'b64encode', 'b64decode', 'urlencode', 'urldecode', 'hexencode', 'hexdecode'],
                        help="Operation to perform.")
    parser.add_argument("input", help="Input string or path to file. For decode operations, this is the encoded string/file.")
    parser.add_argument("-a", "--algorithm", default="sha256", choices=['md5', 'sha1', 'sha256', 'sha512'],
                        help="Hashing algorithm (for 'hash' operation). Default: sha256.")
    parser.add_argument("-f", "--is-file", action="store_true", help="Treat input as a file path.")
    parser.add_argument("-oJ", "--output-json", action="store_true", help="Output results in JSON format.")
    
    args = parser.parse_args()

    result = {
        "operation": args.operation,
        "input": args.input,
        "is_file": args.is_file,
        "algorithm": args.algorithm if args.operation == 'hash' else None,
        "output": None,
        "error": None,
        "timestamp": datetime.now().isoformat()
    }

    output_value, error_message = None, None

    if args.operation == 'hash':
        output_value, error_message = hash_data(args.input, args.is_file, args.algorithm)
    elif args.operation == 'b64encode':
        output_value, error_message = base64_encode_data(args.input, args.is_file)
    elif args.operation == 'b64decode':
        output_value, error_message = base64_decode_data(args.input, args.is_file)
    elif args.operation == 'urlencode':
        output_value, error_message = url_encode_data(args.input)
    elif args.operation == 'urldecode':
        output_value, error_message = url_decode_data(args.input)
    elif args.operation == 'hexencode':
        output_value, error_message = hex_encode_data(args.input, args.is_file)
    elif args.operation == 'hexdecode':
        output_value, error_message = hex_decode_data(args.input)
    
    result['output'] = output_value
    result['error'] = error_message

    if args.output_json:
        print(json.dumps(result, indent=2))
    else:
        if result['error']:
            print(f"Error: {result['error']}", file=sys.stderr)
        else:
            print(f"Operation: {result['operation']}")
            print(f"Input: {result['input']} (file: {result['is_file']})")
            if result['algorithm']:
                print(f"Algorithm: {result['algorithm']}")
            print(f"Output: {result['output']}")

if __name__ == "__main__":
    main()
```
Example Usage:
```bash
# Hash a file
python crypto_tool.py hash sample.txt -a sha256 -oJ

# Base64 encode a string
python crypto_tool.py b64encode "Hello World" -oJ

# URL encode a string
python crypto_tool.py urlencode "http://example.com?q=hello world" -oJ

# Hex encode a binary file
python crypto_tool.py hexencode sample_binary.bin -f -oJ
```

## Guiding Principles in Python

*   **Portability:** Python's standard library modules (`hashlib`, `base64`, `urllib.parse`, `binascii`) are inherently cross-platform, making the script highly portable.
*   **Efficiency:** These modules are often implemented in C for performance, providing efficient execution for cryptographic operations, even on large data. File hashing uses chunked reading to handle large files efficiently.
*   **Minimal Dependencies:** The solution relies exclusively on Python's standard library.
*   **CLI-centric:** The script uses `argparse` to create a flexible command-line interface, making it suitable for quick tasks and integration into automated workflows.
*   **Structured Data Handling:** Python easily handles inputs and outputs as strings or byte arrays, and can package results into dictionaries that are readily converted to JSON for machine-readable reports.

## Conclusion

Python, with its powerful and well-maintained standard cryptographic library, provides an excellent platform for building cross-platform utilities for hashing, encoding, and decoding. Its capabilities are essential for data integrity verification, secure data transmission, and various security and system administration tasks. The structured nature of Python allows for readable and maintainable code, making it an ideal choice for developing robust command-line tools. The next step is to apply this knowledge in practical exercises.
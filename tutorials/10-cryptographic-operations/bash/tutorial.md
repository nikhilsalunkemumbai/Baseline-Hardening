# Bash Tutorial: Basic Cryptographic Operations (Hashing/Encoding)

## Introduction

Bash, combined with a set of standard Linux/Unix command-line utilities, provides essential capabilities for performing basic cryptographic operations. These include calculating hash checksums for data integrity verification and encoding/decoding data using schemes like Base64, URL encoding, and hexadecimal representation. These operations are fundamental for system administration, security analysis, and data handling, all while adhering to our principles of minimal dependencies and CLI-centric operation.

## Framework Alignment

This tutorial on "**Basic Cryptographic Operations (Hashing/Encoding)**" using **Bash** demonstrates the foundational skills necessary to implement components of a "Cross-Platform Baseline Hardening & Auditing Framework." The techniques learned here for performing hashing and encoding are directly applicable to auditing systems against defined security baselines and ensuring compliance.


## Core Bash Utilities for Cryptographic Operations

*   **`md5sum`**: Calculate MD5 checksums.
*   **`sha1sum`**: Calculate SHA1 checksums.
*   **`sha256sum`**: Calculate SHA256 checksums.
*   **`sha512sum`**: Calculate SHA512 checksums.
*   **`base64`**: Base64 encode/decode data.
*   **`xxd`**: Create a hex dump of a given file or standard input. Can also convert hex dumps to binary.
*   **`hexdump`**: Another utility to display file contents in hexadecimal.
*   **`printf`**: Can be used for string manipulation and formatting, including hex conversions.
*   **`od`**: (Octal Dump) Similar to `hexdump`, can output in hex.
*   **`grep`** / **`awk`** / **`sed`**: For filtering and manipulating command output.
*   **`curl`** / **`python`**: Often used for URL encoding/decoding if native tools are not available.

## Implementing Core Functionality with Bash

### 1. Hashing (Data Integrity)

Let's create a sample file and string first:

```bash
echo "Hello, World!" > sample.txt
echo -n "Hello, World!" > sample_no_newline.txt # -n prevents trailing newline
```

#### a. Hash a file

```bash
#!/bin/bash

FILE_TO_HASH="sample.txt"

echo "MD5 of $FILE_TO_HASH: $(md5sum $FILE_TO_HASH | awk '{print $1}')"
echo "SHA256 of $FILE_TO_HASH: $(sha256sum $FILE_TO_HASH | awk '{print $1}')"
echo "SHA512 of $FILE_TO_HASH: $(sha512sum $FILE_TO_HASH | awk '{print $1}')"

# MD5 is cryptographically broken. Use SHA256/512 for security.
```

#### b. Hash a string

```bash
#!/bin/bash

STRING_TO_HASH="MySecretPassword123"

# Note: Passing strings to hashing tools typically involves piping 'echo -n' output
echo "MD5 of string: $(echo -n "$STRING_TO_HASH" | md5sum | awk '{print $1}')"
echo "SHA256 of string: $(echo -n "$STRING_TO_HASH" | sha256sum | awk '{print $1}')"
```

### 2. Base64 Encoding/Decoding

#### a. Encode a string to Base64

```bash
#!/bin/bash

STRING_TO_ENCODE="This is a test string for Base64 encoding."

echo "Base64 encoded string: $(echo -n "$STRING_TO_ENCODE" | base64)"
```

#### b. Decode a Base64 string

```bash
#!/bin/bash

BASE64_STRING="VGhpcyBpcyBhIHRlc3Qgc3RyaW5nIGZvciBCYXNlNjQgZW5jb2Rpbmcu"

echo "Base64 decoded string: $(echo -n "$BASE64_STRING" | base64 --decode)"
```

#### c. Encode a file to Base64

```bash
#!/bin/bash

FILE_TO_ENCODE="sample.txt"

echo "Base64 encoded file content:"
base64 "$FILE_TO_ENCODE"
```

#### d. Decode a Base64 file (redirect output to a new file)

```bash
#!/bin/bash

BASE64_FILE_CONTENT="$(echo -n "SGVsbG8sIFdvcmxkIQo=" | base64)" # Content of "Hello, World!
" base64 encoded
echo "$BASE64_FILE_CONTENT" | base64 --decode > decoded_sample.txt

echo "Decoded content saved to decoded_sample.txt"
cat decoded_sample.txt
```

### 3. URL Encoding/Decoding

Bash doesn't have a built-in `urlencode` or `urldecode` utility. Often, external tools (like `python` or `curl`) are used, or a custom function.

#### a. URL Encode a string (using Python in Bash)

```bash
#!/bin/bash

STRING_TO_URL_ENCODE="https://example.com/search?query=hello world & test"

# Using Python's urllib.parse.quote_plus module
echo "URL encoded string: $(python3 -c 'import urllib.parse; print(urllib.parse.quote_plus("'$STRING_TO_URL_ENCODE'"))')"
```

#### b. URL Decode a string (using Python in Bash)

```bash
#!/bin/bash

URL_ENCODED_STRING="https%3A%2F%2Fexample.com%2Fsearch%3Fquery%3Dhello+world+%26+test"

# Using Python's urllib.parse.unquote_plus module
echo "URL decoded string: $(python3 -c 'import urllib.parse; print(urllib.parse.unquote_plus("'$URL_ENCODED_STRING'"))')"
```

### 4. Hex Encoding/Decoding

#### a. Encode a string to Hex

```bash
#!/bin/bash

STRING_TO_HEX_ENCODE="Hello"

# Using xxd
echo "Hex encoded string (xxd): $(echo -n "$STRING_TO_HEX_ENCODE" | xxd -p | tr -d '
')"

# Using printf
printf "Hex encoded string (printf): "
printf '%s' "$STRING_TO_HEX_ENCODE" | od -An -tx1 | tr -d ' 
'
echo ""
```

#### b. Decode a Hex string

```bash
#!/bin/bash

HEX_STRING="48656c6c6f" # "Hello" in hex

# Using xxd
echo "Hex decoded string (xxd): $(echo -n "$HEX_STRING" | xxd -r -p)"
```

#### c. Encode a file to Hex

```bash
#!/bin/bash

FILE_TO_HEX_ENCODE="sample.txt"

echo "Hex encoded file content (xxd):"
xxd -p "$FILE_TO_HEX_ENCODE" | tr -d '
'
echo ""
```

## Consolidated Script Example (`crypto_utils.sh`)

```bash
#!/bin/bash

# crypto_utils.sh - A simple Bash utility for basic cryptographic operations

operation="$1"
input="$2" # Can be a file path or a string
algorithm="$3" # Optional, for hashing

case "$operation" in
    "hash")
        if [ -f "$input" ]; then # Input is a file
            case "$algorithm" in
                "md5") md5sum "$input" | awk '{print $1}' ;;
                "sha256") sha256sum "$input" | awk '{print $1}' ;;
                "sha512") sha512sum "$input" | awk '{print $1}' ;;
                *) echo "Error: Unsupported hashing algorithm or missing." >&2; exit 1 ;;
            esac
        else # Input is a string
            case "$algorithm" in
                "md5") echo -n "$input" | md5sum | awk '{print $1}' ;;
                "sha256") echo -n "$input" | sha256sum | awk '{print $1}' ;;
                "sha512") echo -n "$input" | sha512sum | awk '{print $1}' ;;
                *) echo "Error: Unsupported hashing algorithm or missing." >&2; exit 1 ;;
            esac
        fi
        ;;
    "base64_encode")
        if [ -f "$input" ]; then base64 "$input";
        else echo -n "$input" | base64; fi
        ;;
    "base64_decode")
        if [ -f "$input" ]; then base64 --decode "$input";
        else echo -n "$input" | base64 --decode; fi
        ;;
    "url_encode")
        python3 -c 'import urllib.parse; print(urllib.parse.quote_plus("'$input'"))'
        ;;
    "url_decode")
        python3 -c 'import urllib.parse; print(urllib.parse.unquote_plus("'$input'"))'
        ;;
    "hex_encode")
        if [ -f "$input" ]; then xxd -p "$input" | tr -d '
';
        else echo -n "$input" | xxd -p | tr -d '
'; fi
        echo ""
        ;;
    "hex_decode")
        if [ -f "$input" ]; then xxd -r -p "$input";
        else echo -n "$input" | xxd -r -p; fi
        ;;
    *)
        echo "Usage: $0 {hash|base64_encode|base64_decode|url_encode|url_decode|hex_encode|hex_decode} <input_string_or_file> [algorithm]" >&2
        exit 1
        ;;
esac

# Example Usage:
# ./crypto_utils.sh hash sample.txt sha256
# ./crypto_utils.sh base64_encode "secret_string"
# ./crypto_utils.sh url_encode "https://example.com/path?q=hello world"
```

## Guiding Principles in Bash

*   **Portability:** `md5sum`, `sha*sum`, `base64`, `xxd` (or `hexdump`) are standard on most modern Linux distributions. Relying on `python3` for URL encoding adds a dependency but is a common practice for complex string operations.
*   **Efficiency:** These native utilities are highly optimized, offering fast execution for cryptographic operations, even on large files.
*   **Minimal Dependencies:** The solution relies almost entirely on core system utilities, with a single, optional dependency on Python for URL encoding.
*   **CLI-centric:** All functions are directly exposed via command-line arguments, making them ideal for quick checks, piping, and scripting.
*   **Security Engineering Focus:** Provides tools to verify data integrity (hashing) and safely handle data for transmission (encoding), which are foundational for security.

## Conclusion

Bash, coupled with its powerful set of command-line utilities, provides a flexible and efficient environment for performing basic cryptographic operations. From hashing files for integrity checks to encoding/decoding data for various purposes, these tools are indispensable for system administrators, security practitioners, and developers working on Linux/Unix systems. The next step is to apply this knowledge in practical exercises.
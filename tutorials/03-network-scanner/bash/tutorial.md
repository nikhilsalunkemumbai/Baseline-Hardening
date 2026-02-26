# Bash Tutorial: Network Connectivity and Port Scanning

## Introduction

Bash, coupled with standard command-line network utilities, can be an effective and lightweight tool for performing basic network connectivity checks and port scanning. This tutorial will explore how to use common tools like `ping`, `nc` (netcat), and `telnet`, along with Bash's built-in `/dev/tcp` functionality, to implement our network scanner design while adhering to principles of minimal dependencies and cross-platform compatibility where possible.

## Framework Alignment

This tutorial on "**Network Connectivity and Port Scanner**" using **Bash** demonstrates the foundational skills necessary to implement components of a "Cross-Platform Baseline Hardening & Auditing Framework." The techniques learned here for auditing network connectivity and port accessibility are directly applicable to auditing systems against defined security baselines and ensuring compliance.


## Core Bash Utilities for Network Scanning

*   **`ping`**: The quintessential tool for checking host reachability using ICMP echo requests.
    *   `-c <count>`: Send count packets.
    *   `-W <timeout>`: Time to wait for a response, in seconds.
*   **`nc` (netcat)**: A versatile networking utility that reads and writes data across network connections. Often called the "TCP/IP Swiss army knife."
    *   `-z`: Zero-I/O mode (scan for listening daemons without sending any data).
    *   `-v`: Verbose output.
    *   `-w <timeout>`: Connection timeout in seconds.
    *   `-u`: UDP mode (default is TCP).
*   **`telnet`**: Primarily used for interactive remote logins, but can also be used to test connectivity to TCP ports.
    *   `telnet <host> <port>`: Attempts to connect.
*   **`/dev/tcp/<host>/<port>`**: A special Bash construct (available on Linux and some Unix-like systems) that allows making TCP connections directly from the shell. Provides a very lightweight way to check port status.

## Implementing Core Functionality with Bash

### 1. Host Reachability Check (`ping`)

Using `ping` to determine if a host is alive before attempting port scans.

```bash
#!/bin/bash

# Function to check host reachability
check_host_reachability() {
    local host="$1"
    echo "Pinging $host..."
    # Send 1 packet, wait 1 second for response
    if ping -c 1 -W 1 "$host" > /dev/null 2>&1; then
        echo "Host $host is UP"
        return 0 # Host is up
    else
        echo "Host $host is DOWN or UNREACHABLE"
        return 1 # Host is down
    fi
}

# Example Usage:
# check_host_reachability "8.8.8.8"
# check_host_reachability "nonexistent.example.com"
```

### 2. Basic Port Scanning (`nc` - Netcat)

Netcat is excellent for checking TCP ports.

```bash
#!/bin/bash

# Function to scan a single port using netcat
scan_port_netcat() {
    local host="$1"
    local port="$2"
    local timeout=1 # Connection timeout in seconds

    # -z: zero-I/O mode (don't send data)
    # -v: verbose output
    # -w: timeout
    echo "Scanning $host:$port (Netcat)..."
    if nc -zv -w "$timeout" "$host" "$port" > /dev/null 2>&1; then
        echo "Port $port on $host: Open"
        return 0
    else
        # nc will return non-zero for closed/filtered/timeout
        # Differentiating is hard without parsing stderr, typically assume closed/filtered
        echo "Port $port on $host: Closed/Filtered"
        return 1
    fi
}

# Example Usage:
# scan_port_netcat "google.com" "80"
# scan_port_netcat "127.0.0.1" "22" # Assuming SSH is running locally
# scan_port_netcat "127.0.0.1" "65535"
```
*Note: The specific `nc` options might vary slightly between different versions (BSD `nc` vs. GNU `netcat`).*

### 3. Basic Port Scanning (`telnet`)

`telnet` can be used as a fallback or alternative if `nc` is not available.

```bash
#!/bin/bash

# Function to scan a single port using telnet
scan_port_telnet() {
    local host="$1"
    local port="$2"
    echo "Scanning $host:$port (Telnet)..."
    # Connect and then immediately send Ctrl+] to enter telnet prompt, then quit
    # This is a bit hacky, but avoids waiting for the telnet session to timeout
    (echo >/dev/tcp/"$host"/"$port") >/dev/null 2>&1 && echo "Port $port on $host: Open" || echo "Port $port on $host: Closed/Filtered"
}

# Example Usage:
# scan_port_telnet "google.com" "80"
# scan_port_telnet "127.0.0.1" "22"
```
*Note: `telnet` is often not installed by default on modern Linux distributions and is generally considered insecure for its primary purpose. Its use for port scanning is limited.*

### 4. Lightweight Port Scanning (`/dev/tcp`)

Bash's `/dev/tcp` (and `/dev/udp`) pseudo-devices provide a very fast and lightweight way to test TCP/UDP connectivity without external commands.

```bash
#!/bin/bash

# Function to scan a single port using /dev/tcp
scan_port_dev_tcp() {
    local host="$1"
    local port="$2"
    local timeout=1 # Bash's timeout is for the command, not specifically connection

    echo "Scanning $host:$port (/dev/tcp)..."
    # Redirect stdout and stderr to /dev/null to suppress error messages
    # Use timeout command to limit connection attempt duration
    if timeout "$timeout" bash -c "exec 3<>/dev/tcp/$host/$port" >/dev/null 2>&1; then
        echo "Port $port on $host: Open"
        return 0
    else
        echo "Port $port on $host: Closed/Filtered"
        return 1
    fi
}

# Example Usage:
# scan_port_dev_tcp "google.com" "443"
# scan_port_dev_tcp "127.0.0.1" "8080"
```
*Note: `/dev/tcp` is a Bash feature and might not be available in other shells or on all systems (e.g., very old Linux, or systems with restricted Bash).*

### 5. Combining Checks and Output Formatting

A simple script to combine these elements:

```bash
#!/bin/bash

TARGET_HOST="127.0.0.1" # Example target, change as needed
TARGET_PORTS=(22 80 443 8080) # Example ports

echo "--- Network Scan Report for $TARGET_HOST ---"

# Check host reachability
if check_host_reachability "$TARGET_HOST"; then
    echo "Host is reachable. Proceeding with port scan."
    for port in "${TARGET_PORTS[@]}"; do
        scan_port_netcat "$TARGET_HOST" "$port" # Or scan_port_dev_tcp
    done
else
    echo "Host is unreachable. Skipping port scan."
fi

# Function definitions (check_host_reachability, scan_port_netcat) would be placed here
# as shown in previous sections.
```

## Guiding Principles in Bash

*   **Portability:** Emphasized the use of `ping` and `nc` which are widely available. Acknowledged variations and specific availability of `/dev/tcp`.
*   **Efficiency:** Bash scripts leveraging compiled system utilities are generally fast for single checks. For multiple ports/hosts, `nc` with timeouts is relatively efficient for connection attempts.
*   **Minimal Dependencies:** Relied entirely on standard system binaries and Bash's built-in features, requiring no external programming languages or libraries.
*   **CLI-centric:** The solutions are command-line focused, easily integrated into larger shell scripts or executed directly.

## Conclusion

Bash, while not a dedicated network scanning language, offers robust capabilities for basic network connectivity testing and port scanning through its interaction with standard system utilities like `ping` and `nc`. The `/dev/tcp` pseudo-device provides a particularly lightweight method for TCP connection checks. These tools allow for quick, effective diagnostics with minimal overhead and dependencies, making them invaluable for system administrators and security professionals in resource-constrained environments. The next step is to apply this knowledge in practical exercises.
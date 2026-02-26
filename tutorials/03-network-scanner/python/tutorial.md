# Python Tutorial: Network Connectivity and Port Scanner

## Introduction

Python's standard library provides powerful and flexible modules for network programming, making it an excellent choice for developing network connectivity and port scanning utilities. This tutorial will guide you through building a basic network scanner using Python's `socket` module for TCP connections and `subprocess` for ICMP (ping) checks, adhering to our design principles of minimal dependencies, cross-platform compatibility, and structured output.

## Framework Alignment

This tutorial on "**Network Connectivity and Port Scanner**" using **Python** demonstrates the foundational skills necessary to implement components of a "Cross-Platform Baseline Hardening & Auditing Framework." The techniques learned here for auditing network connectivity and port accessibility are directly applicable to auditing systems against defined security baselines and ensuring compliance.


## Core Python Modules for Network Scanning

*   **`socket`**: This module provides access to the BSD socket interface, allowing you to create network connections (TCP, UDP). It's fundamental for port scanning.
*   **`subprocess`**: Used to spawn new processes, which is ideal for executing external commands like `ping` to check host reachability.
*   **`json`**: For serializing scan results into a structured JSON format.
*   **`ipaddress`**: (Optional, for CIDR expansion) Provides tools for creating and manipulating IPv4 and IPv6 addresses and networks.
*   **`concurrent.futures`**: (Optional, for basic parallelism) Allows for easy management of parallel tasks (e.g., scanning multiple ports simultaneously).

## Implementing Core Functionality with Python

### 1. Host Reachability Check (`ping` via `subprocess`)

```python
import socket
import subprocess
import json
import ipaddress
import concurrent.futures
import platform
import os
import sys

def check_host_reachability(host, timeout_seconds=1):
    """
    Checks if a host is reachable using ping.
    Returns "Up" if reachable, "Down" otherwise.
    """
    param = "-n" if platform.system().lower() == "windows" else "-c"
    command = ["ping", param, "1", host]
    
    try:
        # Use subprocess.run with a timeout
        result = subprocess.run(
            command,
            capture_output=True,
            text=True,
            timeout=timeout_seconds,
            check=False # Do not raise exception for non-zero exit codes
        )
        # Ping output varies, check for common success indicators
        if result.returncode == 0:
            return "Up"
        else:
            return "Down" # Ping failed
    except subprocess.TimeoutExpired:
        return "Down" # Ping timed out
    except FileNotFoundError:
        return "Error: Ping command not found."
    except Exception as e:
        return f"Error: An unexpected error occurred during ping: {e}"

# Example Usage:
# print(f"google.com status: {check_host_reachability('google.com')}")
# print(f"192.0.2.1 status: {check_host_reachability('192.0.2.1')}") # Example unreachable IP
```

### 2. Basic Port Scanning (`socket` module)

The `socket` module allows direct TCP connection attempts.

```python
def scan_port(host, port, timeout_seconds=1):
    """
    Attempts to connect to a TCP port on a host.
    Returns "Open", "Closed", or "Filtered" (on timeout).
    """
    try:
        # Create a TCP socket
        sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        sock.settimeout(timeout_seconds) # Set a timeout for the connection attempt

        # Attempt to connect
        result = sock.connect_ex((host, port)) # connect_ex returns an error indicator
        
        if result == 0:
            return "Open"
        elif result == 111 or result == 10061: # Connection refused (Linux/Windows)
            return "Closed"
        else:
            return "Filtered" # Other errors, often timeout or firewall blocking
    except socket.gaierror:
        return "Error: Hostname could not be resolved."
    except socket.error as e:
        return f"Error: Socket error during scan: {e}"
    except Exception as e:
        return f"Error: An unexpected error occurred: {e}"
    finally:
        sock.close() # Always close the socket

# Example Usage:
# print(f"Port 80 on google.com: {scan_port('google.com', 80)}")
# print(f"Port 22 on 127.0.0.1: {scan_port('127.0.0.1', 22)}")
# print(f"Port 65535 on 127.0.0.1: {scan_port('127.0.0.1', 65535)}")
```

### 3. Scanning Multiple Ports and Hosts (Sequential)

```python
def perform_sequential_scan(hosts, ports, ping_timeout=1, port_timeout=1):
    scan_results = []
    for host in hosts:
        host_status = check_host_reachability(host, ping_timeout)
        host_scan_data = {
            "host": host,
            "host_status": host_status,
            "ports": []
        }
        
        if host_status == "Up":
            for port in ports:
                port_status = scan_port(host, port, port_timeout)
                host_scan_data["ports"].append({"port": port, "status": port_status})
        
        scan_results.append(host_scan_data)
    return scan_results

# Example Usage:
# target_hosts = ["google.com", "127.0.0.1"]
# target_ports = [22, 80, 443]
# results = perform_sequential_scan(target_hosts, target_ports)
# print(json.dumps(results, indent=2))
```

### 4. Scanning Multiple Ports and Hosts (Basic Parallelism with `concurrent.futures`)

For faster scans, `concurrent.futures.ThreadPoolExecutor` can run `scan_port` concurrently.

```python
def perform_parallel_scan(hosts, ports, max_workers=10, ping_timeout=1, port_timeout=1):
    scan_results = []
    with concurrent.futures.ThreadPoolExecutor(max_workers=max_workers) as executor:
        for host in hosts:
            host_status = check_host_reachability(host, ping_timeout)
            host_scan_data = {
                "host": host,
                "host_status": host_status,
                "ports": []
            }

            if host_status == "Up":
                # Submit port scans for the current host to the thread pool
                future_to_port = {executor.submit(scan_port, host, port, port_timeout): port for port in ports}
                for future in concurrent.futures.as_completed(future_to_port):
                    port = future_to_port[future]
                    try:
                        port_status = future.result()
                        host_scan_data["ports"].append({"port": port, "status": port_status})
                    except Exception as exc:
                        host_scan_data["ports"].append({"port": port, "status": f"Error: {exc}"})
                # Sort ports for consistent output
                host_scan_data["ports"].sort(key=lambda x: x["port"])
            
            scan_results.append(host_scan_data)
    return scan_results

# Example Usage:
# target_hosts = ["google.com", "127.0.0.1"]
# target_ports = [22, 80, 443, 8080]
# results = perform_parallel_scan(target_hosts, target_ports, max_workers=20)
# print(json.dumps(results, indent=2))
```

### 5. Handling CIDR Ranges (`ipaddress` module)

```python
def expand_cidr_range(cidr):
    """Expands a CIDR range into a list of individual IP addresses."""
    try:
        network = ipaddress.ip_network(cidr, strict=False)
        return [str(ip) for ip in network.hosts()] # .hosts() excludes network/broadcast
    except ValueError:
        return [] # Invalid CIDR
    except Exception as e:
        print(f"Error expanding CIDR {cidr}: {e}", file=sys.stderr)
        return []

# Example Usage:
# print(expand_cidr_range("192.168.1.0/30"))
```

### 6. Full Script Structure (`network_scanner.py`)

```python
#!/usr/bin/env python3

import socket
import subprocess
import json
import ipaddress
import concurrent.futures
import platform
import os
import sys
import argparse

# --- Helper Functions (as defined above) ---

def check_host_reachability(host, timeout_seconds=1):
    # ... (same as above) ...
    param = "-n" if platform.system().lower() == "windows" else "-c"
    command = ["ping", param, "1", host]
    try:
        result = subprocess.run(
            command, capture_output=True, text=True, timeout=timeout_seconds, check=False
        )
        if result.returncode == 0: return "Up"
        else: return "Down"
    except subprocess.TimeoutExpired: return "Down"
    except FileNotFoundError: return "Error: Ping command not found."
    except Exception as e: return f"Error: An unexpected error occurred during ping: {e}"

def scan_port(host, port, timeout_seconds=1):
    # ... (same as above) ...
    try:
        sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        sock.settimeout(timeout_seconds)
        result = sock.connect_ex((host, port))
        if result == 0: return "Open"
        elif result == 111 or result == 10061: return "Closed"
        else: return "Filtered"
    except socket.gaierror: return "Error: Hostname could not be resolved."
    except socket.error as e: return f"Error: Socket error during scan: {e}"
    except Exception as e: return f"Error: An unexpected error occurred: {e}"
    finally: sock.close()

def perform_parallel_scan(hosts, ports, max_workers=20, ping_timeout=1, port_timeout=1):
    # ... (same as above) ...
    scan_results = []
    with concurrent.futures.ThreadPoolExecutor(max_workers=max_workers) as executor:
        host_futures = {executor.submit(check_host_reachability, host, ping_timeout): host for host in hosts}
        host_reachability = {}
        for future in concurrent.futures.as_completed(host_futures):
            host = host_futures[future]
            host_reachability[host] = future.result()

        for host in hosts:
            host_status = host_reachability.get(host, "Unknown")
            host_scan_data = { "host": host, "host_status": host_status, "ports": [] }
            
            if host_status == "Up":
                port_futures = {executor.submit(scan_port, host, port, port_timeout): port for port in ports}
                for future in concurrent.futures.as_completed(port_futures):
                    port = port_futures[future]
                    try:
                        port_status = future.result()
                        host_scan_data["ports"].append({"port": port, "status": port_status})
                    except Exception as exc:
                        host_scan_data["ports"].append({"port": port, "status": f"Error: {exc}"})
                host_scan_data["ports"].sort(key=lambda x: x["port"])
            
            scan_results.append(host_scan_data)
    return scan_results

def parse_args():
    parser = argparse.ArgumentParser(description="Basic Network Connectivity and Port Scanner.")
    parser.add_argument("-t", "--targets", required=True, help="Comma-separated list of hosts or CIDR ranges (e.g., 192.168.1.1,google.com,10.0.0.0/29)")
    parser.add_argument("-p", "--ports", required=True, help="Comma-separated list of ports or port ranges (e.g., 22,80,443-445)")
    parser.add_argument("--ping-timeout", type=int, default=1, help="Timeout for ping checks in seconds.")
    parser.add_argument("--port-timeout", type=int, default=1, help="Timeout for individual port connection attempts in seconds.")
    parser.add_argument("--workers", type=int, default=20, help="Number of concurrent workers for scanning.")
    parser.add_argument("-oJ", "--output-json", action="store_true", help="Output results in JSON format.")
    return parser.parse_args()

def main():
    args = parse_args()

    targets_raw = args.targets.split(',')
    expanded_hosts = []
    for target in targets_raw:
        try:
            if '/' in target: # Likely a CIDR range
                network = ipaddress.ip_network(target, strict=False)
                expanded_hosts.extend([str(ip) for ip in network.hosts()])
            else: # Single host or hostname
                expanded_hosts.append(target)
        except ValueError:
            print(f"Warning: Invalid target format or CIDR '{target}'. Skipping.", file=sys.stderr)
            continue
    
    # Process ports
    ports_raw = args.ports.split(',')
    target_ports = set()
    for p_spec in ports_raw:
        if '-' in p_spec:
            start, end = map(int, p_spec.split('-'))
            target_ports.update(range(start, end + 1))
        else:
            target_ports.add(int(p_spec))
    target_ports = sorted(list(target_ports))

    if not expanded_hosts or not target_ports:
        print("No valid targets or ports to scan.", file=sys.stderr)
        sys.exit(1)

    print(f"Scanning {len(expanded_hosts)} hosts and {len(target_ports)} ports...", file=sys.stderr)
    results = perform_parallel_scan(
        expanded_hosts, target_ports,
        max_workers=args.workers,
        ping_timeout=args.ping_timeout,
        port_timeout=args.port_timeout
    )

    if args.output_json:
        print(json.dumps(results, indent=2))
    else:
        # Simple text output
        for host_res in results:
            print(f"Host: {host_res['host']} ({host_res['host_status']})")
            for port_res in host_res['ports']:
                print(f"  Port {port_res['port']}: {port_res['status']}")
            if not host_res['ports'] and host_res['host_status'] == "Up":
                print("  No ports scanned or host was unreachable during port scan.")
            print("-" * 20)

if __name__ == "__main__":
    main()
```
## Guiding Principles in Python

*   **Portability:** Python's `socket`, `subprocess`, and `platform` modules provide native cross-platform capabilities. The script is designed to run on Windows, Linux, and macOS.
*   **Efficiency:** The `socket` module provides low-level, efficient TCP connections. `concurrent.futures` enables basic parallelism, significantly speeding up scans with multiple targets/ports.
*   **Minimal Dependencies:** This tutorial primarily uses Python's standard library. While `ipaddress` is used for CIDR expansion, it's also a standard module. No external `pip` installations are strictly required for the core functionality.
*   **CLI-centric:** The script uses `argparse` for robust command-line argument handling, making it a flexible and user-friendly CLI tool.
*   **Structured Data Handling:** Results are collected into Python dictionaries and lists, which are easily serialized to JSON, providing a clean, machine-readable output for further processing.

## Conclusion

Python offers a powerful and flexible environment for building network connectivity and port scanning tools. Leveraging the `socket` module for direct TCP interactions, `subprocess` for system commands like `ping`, and `concurrent.futures` for parallelism, you can create efficient and robust scanners. The ability to handle structured data and output to JSON makes Python ideal for integrating network diagnostics into automated workflows. The next step is to apply this knowledge in practical exercises.
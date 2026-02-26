# Python Tutorial: Service/Process Monitoring and Health Check

## Introduction

Python, with its robust standard library and a few commonly used, minimal third-party modules, provides an excellent platform for developing cross-platform service and process monitoring tools. It allows for detailed inspection of system resources, service states, and application health, outputting structured data suitable for automation and reporting. This tutorial will explore how to achieve this, balancing minimal dependencies with powerful functionality.

## Framework Alignment

This tutorial on "**Service/Process Monitoring and Health Check**" using **Python** demonstrates the foundational skills necessary to implement components of a "Cross-Platform Baseline Hardening & Auditing Framework." The techniques learned here for monitoring services and processes are directly applicable to auditing systems against defined security baselines and ensuring compliance.


## Core Python Modules for Monitoring

*   **`subprocess`**: Allows spawning new processes, connecting to their input/output/error pipes, and obtaining their return codes. Essential for interacting with OS-native tools (`ps`, `systemctl`, `sc`).
*   **`os`** / **`sys`**: Operating system specific functionalities and system parameters.
*   **`socket`**: Low-level networking interface, useful for checking port availability.
*   **`http.client`**: A low-level HTTP protocol client, part of the standard library, for making HTTP requests.
*   **`psutil`** (Third-party, `pip install psutil`): A cross-platform library for retrieving information on running processes and system utilization (CPU, memory, disks, network, sensors). *Highly recommended for robust cross-platform process details, but an external dependency.*
*   **`requests`** (Third-party, `pip install requests`): An elegant and simple HTTP library. *Very commonly used, but an external dependency. We will primarily use `http.client` for strict standard library adherence, but mention `requests` for convenience.*

## Implementing Core Functionality with Python

### 1. Process Monitoring

#### a. Check if a process is running by name (using `subprocess` for `pgrep` or `ps`)

```python
import subprocess
import os
import sys
import socket
import http.client
import json
import time

# Optional: pip install psutil
try:
    import psutil
except ImportError:
    psutil = None
    print("Warning: psutil not installed. Process details will be limited.", file=sys.stderr)

def is_process_running_by_name_subprocess(process_name):
    """Checks if a process is running using pgrep (Linux/macOS) or tasklist (Windows)."""
    if sys.platform.startswith('linux') or sys.platform == 'darwin':
        # pgrep -x for exact match, -c for count
        cmd = ["pgrep", "-x", process_name]
    elif sys.platform == 'win32':
        # tasklist /nh /fi "imagename eq process_name"
        cmd = ["tasklist", "/nh", "/fi", f"imagename eq {process_name}.exe"]
    else:
        print(f"Unsupported OS: {sys.platform}", file=sys.stderr)
        return False

    try:
        output = subprocess.check_output(cmd, stderr=subprocess.PIPE).decode().strip()
        return bool(output) # pgrep returns PIDs, tasklist returns process info
    except subprocess.CalledProcessError:
        return False
    except FileNotFoundError:
        print(f"Error: Command not found. Make sure '{cmd[0]}' is in PATH.", file=sys.stderr)
        return False

# Example Usage:
# print("Is nginx running (subprocess)?", is_process_running_by_name_subprocess("nginx")) # Linux/macOS
# print("Is notepad running (subprocess)?", is_process_running_by_name_subprocess("notepad")) # Windows
```

#### b. Get PID, CPU, Memory for a process (using `psutil` or `subprocess`)

```python
def get_process_details_psutil(process_name):
    """Retrieves process details using psutil."""
    if not psutil:
        print("psutil not available. Cannot get detailed process info.", file=sys.stderr)
        return []
    
    details = []
    for proc in psutil.process_iter(['pid', 'name', 'cpu_percent', 'memory_info', 'username', 'cmdline']):
        if proc.info['name'] == process_name:
            try:
                details.append({
                    "pid": proc.info['pid'],
                    "name": proc.info['name'],
                    "cpu_percent": proc.cpu_percent(interval=0.1), # Non-blocking for current CPU usage
                    "memory_mb": round(proc.memory_info().rss / (1024 * 1024), 2),
                    "username": proc.info['username'],
                    "cmdline": ' '.join(proc.info['cmdline'])
                })
            except (psutil.NoSuchProcess, psutil.AccessDenied, psutil.ZombieProcess):
                continue
    return details

# Example Usage:
# print("
Nginx process details (psutil):", get_process_details_psutil("nginx")) # Linux/macOS
# print("
Notepad process details (psutil):", get_process_details_psutil("notepad.exe")) # Windows
```

#### c. Start a process (`subprocess.Popen`)

```python
def start_process(command, shell=False, background=True):
    """Starts a process."""
    if background:
        # Use subprocess.Popen for non-blocking background processes
        # On Windows, creationflags=subprocess.DETACHED_PROCESS makes it truly detached
        # On Linux, just ' & ' in shell=True command or no parent waiting
        if sys.platform == 'win32':
            process = subprocess.Popen(command, shell=shell, creationflags=subprocess.DETACHED_PROCESS, close_fds=True, stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL)
        else:
            process = subprocess.Popen(command, shell=shell, stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL, preexec_fn=os.setpgrp if not shell else None)
        print(f"Started process in background. PID: {process.pid}")
        return process.pid
    else:
        # For foreground processes, or if you need to wait for it
        print(f"Starting process in foreground: {' '.join(command) if isinstance(command, list) else command}")
        subprocess.run(command, shell=shell)
        return None

# Example Usage (create a dummy HTTP server for testing):
# if sys.platform != 'win32':
#     print("Starting Python simple HTTP server...")
#     http_server_cmd = [sys.executable, "-m", "http.server", "8000"]
#     server_pid = start_process(http_server_cmd, background=True)
#     print(f"HTTP server PID: {server_pid}")
#     time.sleep(2) # Give it time to start
```

#### d. Kill a process (`psutil` or `subprocess` for `kill`)

```python
def kill_process(pid):
    """Kills a process by PID."""
    if psutil:
        try:
            proc = psutil.Process(pid)
            proc.terminate()
            print(f"Process {pid} terminated via psutil.")
            return True
        except psutil.NoSuchProcess:
            print(f"Process {pid} not found.", file=sys.stderr)
            return False
        except psutil.AccessDenied:
            print(f"Access denied to terminate process {pid}.", file=sys.stderr)
            return False
    else:
        try:
            os.kill(pid, 9) # SIGKILL
            print(f"Process {pid} killed via os.kill.")
            return True
        except ProcessLookupError:
            print(f"Process {pid} not found.", file=sys.stderr)
            return False
        except OSError as e:
            print(f"Error killing process {pid}: {e}", file=sys.stderr)
            return False

# Example Usage:
# if 'server_pid' in locals() and server_pid:
#     kill_process(server_pid)
#     time.sleep(1)
#     print("Is HTTP server running after kill?", is_process_running_by_name_subprocess("http.server"))
```

### 2. Service Monitoring (using `subprocess` for OS tools)

#### a. Linux (`systemctl`)

```python
def get_service_status_linux(service_name):
    """Gets service status on Linux using systemctl."""
    try:
        # systemctl is-active --quiet returns 0 if active, 1 if inactive, other for failed
        subprocess.check_call(["systemctl", "is-active", "--quiet", service_name], stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL)
        status = "running"
    except subprocess.CalledProcessError as e:
        if e.returncode == 3: # Inactive/dead
            status = "stopped"
        elif e.returncode == 4: # Failed
            status = "failed"
        else:
            status = "unknown"
    except FileNotFoundError:
        print("systemctl command not found.", file=sys.stderr)
        status = "error"
    
    return {"name": service_name, "status": status}

# Example Usage:
# print("
Nginx service status (Linux):", get_service_status_linux("nginx"))
# print("SSHD service status (Linux):", get_service_status_linux("sshd"))
```

#### b. Windows (`sc`)

```python
def get_service_status_windows(service_name):
    """Gets service status on Windows using 'sc query'."""
    try:
        # sc query service_name | find "STATE"
        output = subprocess.check_output(["sc", "query", service_name], stderr=subprocess.DEVNULL).decode(errors='ignore')
        state_line = next((line for line in output.splitlines() if "STATE" in line), None)
        if state_line:
            status_match = re.search(r"STATE\s*:\s*\d+\s*(.*?)\s*$", state_line)
            if status_match:
                raw_status = status_match.group(1).strip().lower()
                # Map Windows service states to common terms
                if "running" in raw_status:
                    status = "running"
                elif "stopped" in raw_status:
                    status = "stopped"
                elif "paused" in raw_status:
                    status = "paused"
                else:
                    status = raw_status
                return {"name": service_name, "status": status}
    except subprocess.CalledProcessError:
        pass # Service not found or other error
    except FileNotFoundError:
        print("sc command not found.", file=sys.stderr)
    
    return {"name": service_name, "status": "unknown/not found"}

# Example Usage:
# print("
Spooler service status (Windows):", get_service_status_windows("Spooler"))
# print("WinRM service status (Windows):", get_service_status_windows("WinRM"))
```

### 3. Health Checks

#### a. Check if a TCP port is listening (`socket`)

```python
def check_tcp_port(host, port, timeout=1):
    """Checks if a TCP port is open and listening."""
    try:
        with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as sock:
            sock.settimeout(timeout)
            result = sock.connect_ex((host, port))
            if result == 0:
                return {"target": f"{host}:{port}", "status": "PASS", "message": "Port is open"}
            else:
                return {"target": f"{host}:{port}", "status": "FAIL", "message": f"Port is closed (Error code: {result})"}
    except socket.gaierror:
        return {"target": f"{host}:{port}", "status": "ERROR", "message": "Hostname could not be resolved"}
    except socket.error as e:
        return {"target": f"{host}:{port}", "status": "ERROR", "message": f"Socket error: {e}"}

# Example Usage:
# print("
Localhost port 80 check:", check_tcp_port("localhost", 80))
# print("Localhost port 22 check:", check_tcp_port("localhost", 22))
```

#### b. Perform an HTTP GET request and check status code (`http.client`)

```python
def check_http_endpoint_standard_lib(url, expected_status=200, timeout=5, require_content=None):
    """Performs an HTTP GET request and checks status code/content using http.client."""
    parsed_url = http.client.urlsplit(url)
    conn = None
    try:
        if parsed_url.scheme == 'https':
            conn = http.client.HTTPSConnection(parsed_url.hostname, parsed_url.port, timeout=timeout)
        else:
            conn = http.client.HTTPConnection(parsed_url.hostname, parsed_url.port, timeout=timeout)
        
        conn.request("GET", parsed_url.path or '/')
        response = conn.getresponse()
        
        status_code = response.status
        message = f"HTTP Status: {status_code}"
        check_status = "PASS"

        if status_code != expected_status:
            check_status = "FAIL"
            message += f", expected {expected_status}"
        
        if require_content:
            content = response.read().decode('utf-8', errors='ignore')
            if require_content not in content:
                check_status = "FAIL"
                message += f", required content '{require_content}' not found"

        return {"target": url, "status": check_status, "message": message, "http_status_code": status_code}
    except http.client.HTTPException as e:
        return {"target": url, "status": "ERROR", "message": f"HTTP error: {e}"}
    except socket.timeout:
        return {"target": url, "status": "ERROR", "message": "Connection timed out"}
    except ConnectionRefusedError:
        return {"target": url, "status": "ERROR", "message": "Connection refused"}
    except Exception as e:
        return {"target": url, "status": "ERROR", "message": f"An unexpected error occurred: {e}"}
    finally:
        if conn:
            conn.close()

# Example Usage:
# print("
HTTP check (example.com):", check_http_endpoint_standard_lib("http://example.com"))
# print("HTTP check (localhost:8000, content):", check_http_endpoint_standard_lib("http://localhost:8000", require_content="Directory listing"))
```

### 4. Consolidated Monitoring Script Example (`monitor_health.py`)

```python
#!/usr/bin/env python3

# ... (Include all functions defined above, and necessary imports) ...

def get_platform_specific_details(report):
    if sys.platform.startswith('linux') or sys.platform == 'darwin':
        # Add Linux/macOS specific checks
        report['system_uptime'] = subprocess.getoutput("uptime -p").strip()
        report['load_average'] = subprocess.getoutput("cat /proc/loadavg").split()[:3]
    elif sys.platform == 'win32':
        # Add Windows specific checks
        report['system_uptime'] = subprocess.getoutput("systeminfo | findstr /B "System Boot Time"").strip()
        # Add more Windows specific data using 'wmic' or 'Get-WmiObject' via subprocess
    return report

def main():
    parser = argparse.ArgumentParser(description="Cross-platform service/process monitoring and health check.")
    parser.add_argument("--process", nargs='*', help="Names of processes to check (e.g., nginx httpd).")
    parser.add_argument("--service", nargs='*', help="Names of services to check (e.g., sshd apache2).")
    parser.add_argument("--tcp-port", type=str, nargs='*', help="TCP ports to check (host:port, e.g., localhost:80).")
    parser.add_argument("--http-url", type=str, nargs='*', help="HTTP/S URLs to check (e.g., https://example.com).")
    parser.add_argument("-oJ", "--output-json", action="store_true", help="Output results in JSON format.")
    
    args = parser.parse_args()

    health_report = {
        "timestamp": datetime.now().isoformat(),
        "hostname": socket.gethostname(),
        "checks": []
    }

    # Process Checks
    if args.process:
        for p_name in args.process:
            if psutil:
                proc_details = get_process_details_psutil(p_name)
                if proc_details:
                    health_report['checks'].append({
                        "type": "process",
                        "target": p_name,
                        "status": "PASS",
                        "message": f"Process '{p_name}' is running.",
                        "details": proc_details
                    })
                else:
                    health_report['checks'].append({
                        "type": "process",
                        "target": p_name,
                        "status": "FAIL",
                        "message": f"Process '{p_name}' is not running."
                    })
            else:
                running = is_process_running_by_name_subprocess(p_name)
                health_report['checks'].append({
                    "type": "process",
                    "target": p_name,
                    "status": "PASS" if running else "FAIL",
                    "message": f"Process '{p_name}' is {'running' if running else 'not running'} (via subprocess)."
                })

    # Service Checks
    if args.service:
        for s_name in args.service:
            if sys.platform.startswith('linux') or sys.platform == 'darwin':
                result = get_service_status_linux(s_name)
            elif sys.platform == 'win32':
                result = get_service_status_windows(s_name)
            else:
                result = {"name": s_name, "status": "unsupported_os", "message": f"OS {sys.platform} not supported for service check"}
            
            health_report['checks'].append({
                "type": "service",
                "target": s_name,
                "status": "PASS" if result['status'] == "running" else "FAIL",
                "message": f"Service '{s_name}' status: {result['status']}"
            })
            
    # TCP Port Checks
    if args.tcp_port:
        for tp in args.tcp_port:
            try:
                host, port_str = tp.split(':')
                port = int(port_str)
                result = check_tcp_port(host, port)
                health_report['checks'].append({
                    "type": "tcp_port",
                    "target": tp,
                    "status": result['status'],
                    "message": result['message']
                })
            except ValueError:
                health_report['checks'].append({
                    "type": "tcp_port",
                    "target": tp,
                    "status": "ERROR",
                    "message": "Invalid format. Use host:port (e.g., localhost:80)."
                })

    # HTTP/S URL Checks
    if args.http_url:
        for url in args.http_url:
            result = check_http_endpoint_standard_lib(url)
            health_report['checks'].append({
                "type": "http_url",
                "target": url,
                "status": result['status'],
                "message": result['message'],
                "http_status_code": result.get('http_status_code')
            })

    health_report = get_platform_specific_details(health_report)

    if args.output_json:
        print(json.dumps(health_report, indent=2))
    else:
        print(f"--- Health Report for {health_report['hostname']} ({health_report['timestamp']}) ---")
        for check in health_report['checks']:
            status_color = "\033[92mPASS\033[0m" if check['status'] == "PASS" else "\033[91mFAIL\033[0m"
            print(f"[{status_color}] {check['type'].capitalize()}: {check['target']} - {check['message']}")
        
        if 'system_uptime' in health_report:
            print(f"
System Uptime: {health_report['system_uptime']}")
        if 'load_average' in health_report:
            print(f"Load Average (1m, 5m, 15m): {', '.join(health_report['load_average'])}")

if __name__ == "__main__":
    main()
```
To run: `python monitor_health.py --process notepad --service sshd --tcp-port localhost:80 --http-url http://example.com`

## Guiding Principles in Python

*   **Portability:** Python's standard library (`subprocess`, `os`, `socket`, `http.client`) provides fundamental cross-platform capabilities. `psutil` further enhances this by abstracting OS differences for process and system details.
*   **Efficiency:** For most monitoring tasks, Python is efficient enough. `subprocess` calls to native commands are fast. `psutil` is optimized C/Python code.
*   **Minimal Dependencies:** The tutorial prioritizes standard library modules. `psutil` is highlighted as a highly recommended but explicit external dependency for more robust process monitoring. `requests` is mentioned as an alternative to `http.client`.
*   **CLI-centric:** The script uses `argparse` to create a flexible command-line interface, allowing users to specify what to monitor.
*   **Structured Data Handling:** Python excels at processing data into dictionaries and lists, which are easily converted to JSON for machine-readable reports, making integration into monitoring systems straightforward.

## Conclusion

Python offers a powerful and flexible environment for building cross-platform service/process monitoring and health check utilities. By leveraging its standard library and thoughtfully chosen minimal external modules like `psutil`, developers can create robust tools that provide detailed insights into system health, support automation, and facilitate proactive problem-solving. Its ability to generate structured reports makes it an ideal choice for integrating with modern monitoring and alerting pipelines. The next step is to apply this knowledge in practical exercises.
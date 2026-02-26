# Python Exercise: Service/Process Monitoring and Health Check

## Objective

This exercise challenges you to apply your Python scripting skills to monitor the operational status and health of critical application components. You will use Python's standard library (`subprocess`, `socket`, `http.client`, `os`) and the `psutil` library (a common and highly recommended external dependency) to check process existence, resource usage, network port listening, and perform HTTP health checks. You will generate a structured JSON report, demonstrating proficiency in cross-platform system observability and automation.

## Framework Alignment

This exercise on "**Service/Process Monitoring and Health Check**" using **Python** provides practical experience in building and utilizing tools for a "Cross-Platform Baseline Hardening & Auditing Framework." By completing these tasks, you will learn how to audit and manage service and process health, ensuring that critical security and operational services are running as expected—essential steps in maintaining a secure and auditable environment.


## Scenario

You are a DevOps engineer deploying a Python-based web application. The application consists of a web server (serving HTTP requests) and a separate background worker process. You need to create a Python script that can monitor the health of these two components and their network accessibility.

## Setup

For this exercise, you will create two dummy Python scripts that will simulate your web server and worker process. These will be run in the background by your main monitoring script.

### 1. `dummy_http_server.py`

Create a file named `dummy_http_server.py` with the following content:

```python
import http.server
import socketserver
import os
import sys
import time

PORT = 8000
DIRECTORY = "."

class Handler(http.server.SimpleHTTPRequestHandler):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, directory=DIRECTORY, **kwargs)

if __name__ == "__main__":
    pid_file = "dummy_http_server.pid"
    with open(pid_file, "w") as f:
        f.write(str(os.getpid()))
    
    print(f"HTTP server serving at port {PORT} with PID {os.getpid()}", file=sys.stderr)
    try:
        with socketserver.TCPServer(("", PORT), Handler) as httpd:
            httpd.serve_forever()
    except KeyboardInterrupt:
        print(f"HTTP server {os.getpid()} stopped.", file=sys.stderr)
    finally:
        if os.path.exists(pid_file):
            os.remove(pid_file)
```

### 2. `dummy_worker_process.py`

Create a file named `dummy_worker_process.py` with the following content:

```python
import time
import os
import sys

if __name__ == "__main__":
    pid_file = "dummy_worker_process.pid"
    with open(pid_file, "w") as f:
        f.write(str(os.getpid()))
    
    print(f"Worker process started with PID {os.getpid()}", file=sys.stderr)
    try:
        while True:
            time.sleep(10) # Simulate work
    except KeyboardInterrupt:
        print(f"Worker process {os.getpid()} stopped.", file=sys.stderr)
    finally:
        if os.path.exists(pid_file):
            os.remove(pid_file)
```

### 3. Install `psutil`

If you haven't already, install the `psutil` library:
```bash
pip install psutil
```

## Tasks

Write a single Python script (`app_monitor.py`) that performs the following tasks. Your script should be executable from the command line and output a structured JSON monitoring report to standard output.

### Part 1: Process Management (within `app_monitor.py`)

1.  **Start Dummy HTTP Server:**
    *   Use `subprocess.Popen` to start `dummy_http_server.py` in the background. Store its PID.
    *   Ensure the server's PID is written to `dummy_http_server.pid` by the server script itself.

2.  **Start Dummy Worker Process:**
    *   Use `subprocess.Popen` to start `dummy_worker_process.py` in the background. Store its PID.
    *   Ensure the worker's PID is written to `dummy_worker_process.pid` by the worker script itself.

3.  **Wait for Processes to Start:**
    *   Include a short `time.sleep()` to allow both processes to fully initialize.

### Part 2: Monitoring and Health Checks (within `app_monitor.py`)

1.  **Check HTTP Server Process Status:**
    *   Read the PID from `dummy_http_server.pid`.
    *   Use `psutil.Process(pid)` to check if the HTTP server process is running.
    *   Retrieve its current CPU utilization (`cpu_percent()`) and memory usage (`memory_info().rss` in MB).

2.  **Check Worker Process Status:**
    *   Read the PID from `dummy_worker_process.pid`.
    *   Use `psutil.Process(pid)` to check if the worker process is running.
    *   Retrieve its current CPU utilization and memory usage.

3.  **Check HTTP Server Port Listening:**
    *   Use Python's `socket` module to check if `localhost:8000` is open and listening.

4.  **Perform HTTP Health Check:**
    *   Use Python's `http.client` module to make an HTTP GET request to `http://localhost:8000`.
    *   Verify that the HTTP status code returned is `200 OK`.

### Part 3: Reporting and Cleanup (within `app_monitor.py`)

1.  **Generate Structured JSON Report:**
    *   Collect all monitoring results (process status, resource usage, port status, HTTP health) into a Python dictionary.
    *   Include `timestamp` and `hostname`.
    *   Output this dictionary as a JSON string to `stdout`.

2.  **Graceful Shutdown:**
    *   Use `psutil.Process(pid).terminate()` or `kill` if `psutil` is not available, to stop both the HTTP server and worker processes.
    *   Clean up the `.pid` files.

## Deliverables

Provide the complete Python script file (`app_monitor.py`) that implements all the above tasks.

## Reflection Questions

1.  Compare using `subprocess` to execute external OS commands (like `ps` or `systemctl`) versus using a library like `psutil` for process monitoring. What are the advantages of each approach?
2.  How did you ensure that the dummy processes run in the background and that your main script could still manage their lifecycle (e.g., kill them later)?
3.  Explain how Python's `socket` module is used for network port checks, and what advantages it offers over external command-line tools like `netstat` or `ss` for automation.
4.  Describe how outputting the monitoring results as JSON makes your Python script more versatile for integration with automated monitoring systems, logging platforms, or dashboards.
5.  What are the challenges of making a truly cross-platform monitoring script, especially when dealing with OS-specific services (like Windows Services vs. Linux Systemd services)? How might you abstract this further in Python?

---
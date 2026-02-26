# Bash Tutorial: Service/Process Monitoring and Health Check

## Introduction

Bash, combined with a suite of standard Linux/Unix command-line utilities, is highly effective for monitoring the status and health of system processes and services. This tutorial will demonstrate how to use these tools to check if applications are running, verify service states, perform basic network and application health checks, and even manage their lifecycle, all adhering to our principles of minimal dependencies and CLI-centric operation.

## Framework Alignment

This tutorial on "**Service/Process Monitoring and Health Check**" using **Bash** demonstrates the foundational skills necessary to implement components of a "Cross-Platform Baseline Hardening & Auditing Framework." The techniques learned here for monitoring services and processes are directly applicable to auditing systems against defined security baselines and ensuring compliance.


## Core Bash Utilities for Monitoring

*   **`ps`**: Reports a snapshot of the current processes.
*   **`pgrep`**: Searches for processes whose names or other attributes match a regular expression.
*   **`pkill`**: Sends signals to processes based on name or other attributes.
*   **`systemctl`**: (Linux, Systemd-based systems) Controls the systemd system and service manager.
*   **`service`**: (Older/SysVinit-based systems) Runs a System V init script.
*   **`netstat`** / **`ss`**: Print network connections, routing tables, interface statistics, masquerade connections, and multicast memberships. `ss` is newer and often preferred.
*   **`curl`** / **`wget`**: Transfer data from or to a server. Used for HTTP/S health checks.
*   **`grep`** / **`awk`** / **`sed`**: Powerful text processing for filtering and extracting information from command output.

## Implementing Core Functionality with Bash

### 1. Process Monitoring

#### a. Check if a process is running by name

```bash
#!/bin/bash

PROCESS_NAME="nginx" # Example process name

if pgrep -x "$PROCESS_NAME" > /dev/null; then
    echo "$PROCESS_NAME is running."
else
    echo "$PROCESS_NAME is not running."
fi

# Alternative using ps and grep
if ps aux | grep -v grep | grep "$PROCESS_NAME" > /dev/null; then
    echo "$PROCESS_NAME is running (via ps)."
else
    echo "$PROCESS_NAME is not running (via ps)."
fi
```

#### b. Get PID, CPU, Memory for a process

```bash
#!/bin/bash

PROCESS_NAME="nginx"

echo "Monitoring $PROCESS_NAME:"
ps aux | grep -v grep | grep "$PROCESS_NAME" | awk '{print "PID:", $2, "CPU:", $3"%", "MEM:", $4"%", "CMD:", $11}'
```

#### c. Start a background process (example: simple web server for testing)

```bash
#!/bin/bash

# Create a simple Python web server for demonstration
echo "Starting a simple Python web server on port 8000..."
python3 -m http.server 8000 > /dev/null 2>&1 &
PYTHON_SERVER_PID=$!
echo "Python web server started with PID: $PYTHON_SERVER_PID"
sleep 2 # Give it a moment to start

# Check its status
if pgrep -f "http.server 8000" > /dev/null; then
    echo "Python web server is running."
else
    echo "Python web server failed to start."
fi

# Clean up (kill it later)
# kill "$PYTHON_SERVER_PID"
```

#### d. Kill a process

```bash
#!/bin/bash

PROCESS_TO_KILL="http.server 8000" # Or a specific PID

PID=$(pgrep -f "$PROCESS_TO_KILL")

if [ -n "$PID" ]; then
    echo "Killing process: $PROCESS_TO_KILL (PID: $PID)"
    kill "$PID"
    sleep 1
    if ! pgrep -f "$PROCESS_TO_KILL" > /dev/null; then
        echo "$PROCESS_TO_KILL killed successfully."
    else
        echo "Failed to kill $PROCESS_TO_KILL."
    fi
else
    echo "Process $PROCESS_TO_KILL not found."
fi
```

### 2. Service Monitoring (Linux - Systemd)

#### a. Check service status

```bash
#!/bin/bash

SERVICE_NAME="nginx" # Example service name

if systemctl is-active --quiet "$SERVICE_NAME"; then
    echo "$SERVICE_NAME is running."
elif systemctl is-failed --quiet "$SERVICE_NAME"; then
    echo "$SERVICE_NAME is in a failed state."
else
    echo "$SERVICE_NAME is not running or not active."
fi

# Get full status
systemctl status "$SERVICE_NAME" --no-pager
```

#### b. Start/Stop/Restart a service

```bash
#!/bin/bash

SERVICE_NAME="nginx"

echo "Attempting to restart $SERVICE_NAME..."
# sudo systemctl restart "$SERVICE_NAME"
# sleep 5
# systemctl status "$SERVICE_NAME" --no-pager
echo "Commands commented out for safety; uncomment and use 'sudo' for actual control."
```

#### c. Check if a service is enabled at boot

```bash
#!/bin/bash

SERVICE_NAME="nginx"

if systemctl is-enabled --quiet "$SERVICE_NAME"; then
    echo "$SERVICE_NAME is enabled to start on boot."
else
    echo "$SERVICE_NAME is not enabled to start on boot."
fi
```

### 3. Health Checks

#### a. Check if a TCP port is listening

```bash
#!/bin/bash

HOST="127.0.0.1"
PORT="80"

if netstat -tuln | grep ":$PORT\b" > /dev/null; then
    echo "Port $PORT is listening on $HOST (via netstat)."
else
    echo "Port $PORT is not listening on $HOST (via netstat)."
fi

# Alternative using 'ss'
if ss -tuln | grep ":$PORT\b" > /dev/null; then
    echo "Port $PORT is listening on $HOST (via ss)."
else
    echo "Port $PORT is not listening on $HOST (via ss)."
fi

# Check remote port (requires 'nc' or similar)
# nc -zv $HOST $PORT &> /dev/null
# if [ $? -eq 0 ]; then
#     echo "Remote port $PORT on $HOST is open."
# else
#     echo "Remote port $PORT on $HOST is closed."
# fi
```

#### b. Perform an HTTP GET request and check status code

```bash
#!/bin/bash

URL="http://localhost:8000" # Assuming Python web server from earlier example
EXPECTED_STATUS=200

HTTP_STATUS=$(curl -o /dev/null -s -w "%{http_code}
" "$URL")

if [ "$HTTP_STATUS" -eq "$EXPECTED_STATUS" ]; then
    echo "HTTP Health Check PASSED for $URL (Status: $HTTP_STATUS)."
else
    echo "HTTP Health Check FAILED for $URL (Status: $HTTP_STATUS), expected $EXPECTED_STATUS."
fi

# Check for specific content
if curl -s "$URL" | grep -q "Directory listing for"; then
    echo "Content check PASSED: 'Directory listing for' found."
else
    echo "Content check FAILED: 'Directory listing for' not found."
fi
```

### 4. Consolidated Monitoring Script Example

```bash
#!/bin/bash

# Configuration
SERVICE_TO_MONITOR="nginx"
PROCESS_TO_MONITOR="http.server 8000" # Adjust if not running
WEB_APP_URL="http://localhost:8000"
WEB_APP_PORT="8000"

echo "--- System Health Check Report ($(date)) ---"
echo ""

# 1. Check Service Status
echo "Checking service: $SERVICE_TO_MONITOR"
if systemctl is-active --quiet "$SERVICE_TO_MONITOR"; then
    echo "  [PASS] $SERVICE_TO_MONITOR service is running."
else
    echo "  [FAIL] $SERVICE_TO_MONITOR service is NOT running."
fi
echo ""

# 2. Check Process Status (if any specific process)
echo "Checking process: $PROCESS_TO_MONITOR"
if pgrep -f "$PROCESS_TO_MONITOR" > /dev/null; then
    echo "  [PASS] $PROCESS_TO_MONITOR process is running."
    PID=$(pgrep -f "$PROCESS_TO_MONITOR")
    CPU_MEM=$(ps -p "$PID" -o %cpu,%mem --no-headers)
    echo "    PID: $PID, CPU/MEM: $CPU_MEM"
else
    echo "  [FAIL] $PROCESS_TO_MONITOR process is NOT running."
fi
echo ""

# 3. Check Web App Port Listening
echo "Checking web application port: $WEB_APP_PORT"
if ss -tuln | grep ":$WEB_APP_PORT\b" > /dev/null; then
    echo "  [PASS] Port $WEB_APP_PORT is listening."
else
    echo "  [FAIL] Port $WEB_APP_PORT is NOT listening."
fi
echo ""

# 4. Check Web App HTTP Endpoint
echo "Checking web application URL: $WEB_APP_URL"
HTTP_STATUS=$(curl -o /dev/null -s -w "%{http_code}
" "$WEB_APP_URL")
if [ "$HTTP_STATUS" -eq 200 ]; then
    echo "  [PASS] HTTP endpoint returns 200 OK."
else
    echo "  [FAIL] HTTP endpoint returns status $HTTP_STATUS (expected 200)."
fi
echo ""

echo "--- Report End ---"

# Clean up Python server if started for demo
# kill $(pgrep -f "http.server 8000") > /dev/null 2>&1
```

## Guiding Principles in Bash

*   **Portability:** The tools `ps`, `pgrep`, `kill`, `netstat`, `ss`, `curl`, `grep`, `awk`, `sed` are standard on virtually all modern Linux distributions. `systemctl` is specific to Systemd-based systems but widely adopted.
*   **Efficiency:** These command-line utilities are highly optimized, making them efficient for quick and frequent checks. Piping allows for efficient data flow.
*   **Minimal Dependencies:** Relies entirely on core system utilities.
*   **CLI-centric:** All operations are command-line based, ideal for scripting cron jobs and quick manual checks.
*   **Resilience Focus:** Bash scripts can be easily integrated into watchdog systems or alert pipelines to ensure continuous operation of critical services.

## Conclusion

Bash, leveraging its powerful ecosystem of command-line tools, provides a flexible and efficient way to monitor processes, check service statuses, and perform health assessments on Linux/Unix systems. While it involves text processing and careful parsing of command output, its native availability and speed make it an indispensable tool for system administrators and automation engineers. The next step is to apply this knowledge in practical exercises.
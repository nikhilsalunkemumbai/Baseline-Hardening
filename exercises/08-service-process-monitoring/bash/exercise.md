# Bash Exercise: Service/Process Monitoring and Health Check

## Objective

This exercise challenges you to apply your Bash scripting and command-line utility skills to monitor the operational status and health of critical system components. You will use standard Linux/Unix tools to check process existence, service states, network port listening, and perform basic application health checks, demonstrating proficiency in system observability and automation.

## Framework Alignment

This exercise on "**Service/Process Monitoring and Health Check**" using **Bash** provides practical experience in building and utilizing tools for a "Cross-Platform Baseline Hardening & Auditing Framework." By completing these tasks, you will learn how to audit and manage service and process health, ensuring that critical security and operational services are running as expected—essential steps in maintaining a secure and auditable environment.


## Scenario

You are a system administrator responsible for a Linux web server running Nginx (or Apache), PHP-FPM, and a Python web application. Your task is to create a set of Bash commands and a script to periodically check the health of these components.

## Setup

For this exercise, you will need access to a Linux environment. If Nginx, PHP-FPM, or a web server on port 80/443 are not readily available, you can simulate some components:

### A. Simulating a Web Server (if Nginx/Apache not installed)

You can run a simple Python HTTP server for testing HTTP health checks and port listening.
Create a file named `dummy_webserver.py` with the following content:

```python
import http.server
import socketserver

PORT = 8000

Handler = http.server.SimpleHTTPRequestHandler

with socketserver.TCPServer(("", PORT), Handler) as httpd:
    print(f"serving at port {PORT}")
    httpd.serve_forever()
```

Run this in the background (you might need `nohup` or `&`):
```bash
python3 dummy_webserver.py > /dev/null 2>&1 &
DUMMY_WEBSERVER_PID=$!
echo "Dummy web server started with PID: $DUMMY_WEBSERVER_PID on port 8000"
sleep 2 # Give it a moment to start
```
**Remember to kill this process (`kill $DUMMY_WEBSERVER_PID`) when you are done with the exercise.**

### B. Simulating a PHP-FPM Process (if not installed)

You can use a simple `sleep` command in the background to simulate a long-running process for testing process monitoring.
```bash
sleep 3600 > /dev/null 2>&1 &
DUMMY_PHP_FPM_PID=$!
echo "Dummy PHP-FPM process simulated with PID: $DUMMY_PHP_FPM_PID"
```
**Remember to kill this process (`kill $DUMMY_PHP_FPM_PID`) when you are done with the exercise.**

## Tasks

Using only standard Bash commands and utilities (`ps`, `pgrep`, `systemctl`, `ss` or `netstat`, `curl`, `grep`, `awk`), provide the command-line solution or a simple script for each of the following tasks.

### Part 1: Individual Checks

1.  **Check Nginx Service Status:**
    *   Determine if the `nginx` service is currently running. If you don't have Nginx, you can check `sshd` or `cron`.

2.  **Check `php-fpm` Process Status:**
    *   Determine if any process named `php-fpm` (or the `sleep` dummy process you started) is running.

3.  **Get CPU and Memory for `php-fpm` Process:**
    *   If `php-fpm` (or your dummy process) is running, retrieve its PID and then display its current CPU (`%CPU`) and Memory (`%MEM`) usage.

4.  **Check Web Server Port Listening:**
    *   Verify if port `80` (or `8000` if using the dummy web server) is currently listening on `localhost`.

5.  **Perform HTTP Health Check:**
    *   Make an HTTP GET request to `http://localhost` (or `http://localhost:8000` if using the dummy web server).
    *   Check if the HTTP status code returned is `200 OK`.

### Part 2: Consolidated Health Check Script

Create a single Bash script (`web_server_health.sh`) that performs the following:

1.  **Check Nginx Service:** Report if `nginx` (or chosen service) is running or not.
2.  **Check PHP-FPM Process:** Report if `php-fpm` (or dummy process) is running or not, and if running, display its PID, CPU, and memory usage.
3.  **Check Port 80/8000:** Report if the web server port is listening.
4.  **Check HTTP Endpoint:** Report if the `http://localhost` (or `http://localhost:8000`) endpoint returns `200 OK`.
5.  **Summarize Status:** At the end, output a summary (e.g., "All checks passed" or "Some checks failed").

## Deliverables

For Part 1, provide the exact Bash command-line solution for each task. For Part 2, provide the complete Bash script file (`web_server_health.sh`).

## Reflection Questions

1.  Which Bash utilities did you find most effective for quickly checking the existence of a process versus a managed service? Why?
2.  How did you handle extracting specific data points (like CPU/MEM usage) from the output of `ps`?
3.  Describe any challenges you faced in writing the `web_server_health.sh` script to be robust and handle cases where services/processes might not be running.
4.  If you needed to restart a failed service automatically, how would you integrate `systemctl restart` into your script, and what precautions would you take?
5.  What are the limitations of using Bash for comprehensive monitoring compared to more programmatic approaches (e.g., Python `psutil`) for gathering detailed metrics or interacting with complex APIs?

---
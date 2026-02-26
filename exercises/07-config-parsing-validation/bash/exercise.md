# Bash Exercise: Configuration File Parsing and Validation

## Objective

This exercise challenges you to apply your Bash text processing skills to parse configuration files and validate their settings against security policies. You will use standard command-line utilities (`grep`, `sed`, `awk`, `cut`) to extract specific values, handle comments, and report on compliance, demonstrating proficiency in system hardening and auditing.

## Framework Alignment

This exercise on "**Configuration File Parsing and Validation**" using **Bash** provides practical experience in building and utilizing tools for a "Cross-Platform Baseline Hardening & Auditing Framework." By completing these tasks, you will learn how to audit and manage configuration settings, ensuring compliance with security policies and identifying unauthorized changes—essential steps in maintaining a secure and auditable environment.


## Scenario

You are a security auditor tasked with reviewing the configuration of a web server. Your goal is to ensure that critical security settings in the `webserver.conf` file are correctly configured according to your organization's policy. You need to automate this check using Bash.

## Setup

Before starting the tasks, create the following `webserver.conf` file in your current directory:

```bash
cat <<EOL > webserver.conf
# Web Server Configuration for Prod Environment

[Server]
Port=80
DocumentRoot=/var/www/html
ServerAdmin=webmaster@example.com

[Security]
EnableHTTPS=false
MinTLSVersion=TLSv1.1
PermitRootLogin=yes ; Should be 'no' for security!

[Logging]
LogLevel=INFO
LogFile=/var/log/webserver/access.log
EOL

echo "Configuration file 'webserver.conf' created."
```

## Tasks

Using only standard Bash commands and utilities (`grep`, `sed`, `awk`, `cut`, `test`, `if`), provide the command-line solution or a simple script for each of the following tasks.

### Part 1: Parsing Configuration Values

1.  **Extract Server Port:**
    *   Extract the value of `Port` from the `[Server]` section.

2.  **Extract Document Root:**
    *   Extract the value of `DocumentRoot` from the `[Server]` section.

3.  **Extract `EnableHTTPS` Setting:**
    *   Extract the value of `EnableHTTPS` from the `[Security]` section, handling any trailing comments (e.g., `; Should be 'no' for security!`).

4.  **Extract `LogLevel` Setting:**
    *   Extract the value of `LogLevel` from the `[Logging]` section.

### Part 2: Configuration Validation

Write a single Bash script (`validate_webserver_conf.sh`) that performs the following checks and outputs a validation report.

1.  **Policy Check: Server Port:**
    *   Verify that `Port` in the `[Server]` section is NOT `80`. If it is `80`, report a `FAIL` (as it should likely be HTTPS/443 in production or a high port). Expected: Any value other than `80`.

2.  **Policy Check: `EnableHTTPS`:**
    *   Verify that `EnableHTTPS` in the `[Security]` section is `true`. Report `PASS` or `FAIL`.

3.  **Policy Check: `MinTLSVersion`:**
    *   Verify that `MinTLSVersion` in the `[Security]` section is `TLSv1.2` or higher (i.e., not `TLSv1.1`). Expected: `TLSv1.2` or `TLSv1.3`.

4.  **Policy Check: `PermitRootLogin`:**
    *   Verify that `PermitRootLogin` in the `[Security]` section is `false`. Report `SECURITY ALERT` if it's `yes`.

5.  **Policy Check: `LogLevel`:**
    *   Verify that `LogLevel` in the `[Logging]` section is `WARN` or `ERROR` (i.e., not `INFO` for production to avoid excessive logging). Expected: `WARN` or `ERROR`.

## Deliverables

For Part 1, provide the exact Bash command-line solution for each task. For Part 2, provide the complete Bash script file (`validate_webserver_conf.sh`).

## Reflection Questions

1.  Which text processing utilities (`grep`, `sed`, `awk`) did you find most flexible for extracting values from different parts of the configuration file?
2.  How did you handle comments and whitespace when extracting values from the INI-style file?
3.  Describe how conditional logic (`if`, `test`) in Bash helps in performing policy-based validation checks.
4.  What are the limitations of using purely Bash text processing for parsing complex configuration formats like deeply nested JSON or YAML?
5.  How would you extend your validation script to generate an exit code (e.g., 0 for all pass, 1 for any fail) for use in automated CI/CD pipelines?

---
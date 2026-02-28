# Baseline-Hardening (IT-Audit-Snippets)

A portable library of IT utility designs, tutorials, and exercises for infrastructure resilience and security automation.

---

### **⚠️ Disclaimer**
**This toolkit is provided for educational and administrative purposes only.** Automating security configurations or hardware management carries inherent risks. Always test scripts in a non-production environment before deployment. The authors are not responsible for any system downtime, data loss, or hardware issues resulting from the use of these tools.

---

## 🚀 Quick Links
*   **[Fundamentals](./fundamentals.md)**: Core pillars and the 11-design strategy.
*   **[Setup Guide](./setup.md)**: Preparing Python, PowerShell, and Bash.
*   **[Troubleshooting](./troubleshoot.md)**: Solving cross-platform runtime issues.

## 📚 11 Pillars of Baseline Hardening
The library provides step-by-step guides for:
1.  **Log Analysis**: Extracting critical events.
2.  **System Snapshot**: OS, CPU, and Memory state.
3.  **Network Scanner**: Connectivity and port checks.
4.  **File Integrity**: Hashing and change detection.
5.  **Process Management**: Monitoring active tasks.
6.  **User/Group Audit**: Privileged account verification.
7.  **Config Validation**: INI, JSON, and YAML auditing.
8.  **Service Monitoring**: Health checks for daemons.
9.  **Scheduled Tasks**: Cron and Task Scheduler audits.
10. **Cryptography**: Safe encoding and hashing.
11. **Drift Detection**: Differential state analysis.

## 🏗️ Technical Philosophy
*   **Minimal Dependencies**: Built on standard libraries and native OS tools.
*   **Portability**: Consistent logic across Windows, Linux, and macOS.
*   **Actionable Data**: Every snippet outputs structured **JSON** for easy integration.

## 📁 Repository Structure
*   `/designs`: Technology-agnostic logic (Pseudo-code).
*   `/tutorials`: Implementation guides for Python, PowerShell, Bash, and SQLite.
*   `/exercises`: Hands-on challenges to verify your skills.
*   `/framework-integration`: Unified auditor logic (`master_audit.py`).
*   `/policies`: Sample baseline definitions (YAML).

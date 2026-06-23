# OWASP Juice Shop: DevSecOps CI/CD Pipeline

[![DevSecOps Pipeline](https://img.shields.io/badge/Security-Shift--Left-blueviolet?style=for-the-badge)](https://github.com/)
[![AQUA Trivy](https://img.shields.io/badge/SCA%20%2F%20Container-Trivy-blue?style=flat-square)](https://aquasecurity.github.io/trivy/)
[![Semgrep](https://img.shields.io/badge/SAST-Semgrep-darkgreen?style=flat-square)](https://semgrep.dev/)

This repository demonstrates a production-ready **DevSecOps CI/CD Pipeline** built with GitHub Actions. It implements the **Shift-Left Security** philosophy by embedding automated security gates directly into the software development lifecycle (SDLC).

As a testbed, this project uses a simulated environment of **OWASP Juice Shop** (intentionally vulnerable Node.js application) to showcase how security scanners detect, log, and block vulnerabilities before they reach production.

---

## Pipeline Architecture

The pipeline is structured into logical stages to ensure fast feedback loops for developers:

1. **Static Analysis (SAST):** Code scanning using **Semgrep** to find patterns leading to injection, broken authentication, or insecure configurations.
2. **Software Composition Analysis (SCA):** Dependency vulnerability scanning via **Trivy** to intercept known CVEs in open-source libraries.
3. **Container Security:** *(In Progress)* Automated scanning of Docker image layers for OS-level vulnerabilities.
4. **Dynamic Application Testing (DAST):** *(In Progress)* Testing the running application against web attacks using OWASP ZAP.

---

## Security Tools Integrated

| Tool | Type | Target | Gate Condition (Fail-on) |
| :--- | :--- | :--- | :--- |
| **Semgrep** | SAST | Source Code (`.js`, `.ts`) | Custom Rules / Errors |
| **Trivy (FS)** | SCA | `package.json` dependencies | `CRITICAL` & `HIGH` vulnerabilities |
| **Trivy (Image)**| Container | Docker Layers | `CRITICAL` vulnerabilities |

---

## Vulnerability Management & Risk Acceptance

In a real-world DevSecOps culture, security should not blindly block development. This project demonstrates advanced scanner configuration:
* **False Positives Handling:** Utilizing ignore configs (e.g., `.trivyignore`) to whitelist acceptable business risks.
* **Alert Thresholds:** The pipeline is configured to fail (`exit code 1`) *only* on actionable, high-severity risks, preventing pipeline fatigue.

---

## How to Explore

1. **Check the GitHub Actions Tab:** Navigate to the "Actions" tab in this repository to see the security checks running in real-time.
2. **Review Configuration:** * Check `.github/workflows/devsecops-pipeline.yml` to see how scanners are declared as code.

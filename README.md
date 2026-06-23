# 🛡️ OWASP Juice Shop DevSecOps CI/CD pipeline

This repository demonstrates a production-ready **DevSecOps Security Architecture** implemented for the **OWASP Juice Shop** application. The project showcases how to bridge the gap between automated shift-left security gates and dynamic, real-world exploitation testing.

---

## 📊 Security Executive Summary

| Metrics & Gates | Status / Result | Target Objective |
| :--- | :--- | :--- |
| **CI/CD Quality Gate** |  **FAILED (Expected)** | Block deployment if any `CRITICAL` vulnerability is found |
| **SCA / Vulnerability Count** | **63 CVEs Found** (5 Critical, 35 High) | Base OS & Dependency vetting |
| **Hardcoded Secrets** |  **2 Active Leaks Trapped** | Eliminate credential leakage in container layers |
| **DAST Flaws Validated** |  **3 High/Medium Exploits** | Verify runtime risks (SQLi, CORS, Missing CSP) |

---

## Phase 1: Automated DevSecOps CI/CD Pipeline

The first defense line is built using **GitHub Actions**, acting as a strict **Quality Gate**. Instead of scanning static application code, the pipeline audits the finalized infrastructure: the container images and orchestration configurations.

### Key Pipeline Mechanisms:
1. **SCA (Software Composition Analysis):** Scans the `bkimminich/juice-shop` container layers for public CVEs using **Trivy**.
2. **Secret Detection:** Inspects embedded files for asymmetric keys and authorization tokens.
3. **Automated Enforcement:** Configured with `--exit-code 1` to intentionally break the build when risk thresholds are exceeded.

<details>
<summary> Click to view Node.js & Debian OS Vulnerabilities Proof</summary>

Here is the automated Trivy report highlighting out-of-bounds reads and memory safety flaws within the base image dependencies:

![Node.js Scan](https://github.com/cbrkrtek/owasp-juice-shop-devsecops-ci-cd/blob/main/pictures%20for%20README/node%20js%20scan.PNG)
![Debian Scan](https://github.com/cbrkrtek/owasp-juice-shop-devsecops-ci-cd/blob/main/pictures%20for%20README/result%20of%20scanning%20juiceshop%20via%20trivy.PNG)
</details>

<details>
<summary> Click to view Trapped Hardcoded Secrets Proof</summary>

The pipeline successfully prevented credential leakage by capturing pre-baked secrets inside the image:
* **HIGH:** RSA Asymmetric Private Key (`insecurity.js`)
* **MEDIUM:** Hardcoded JWT Token (`app.guard.spec.ts`)

![Secrets Scan 1](https://github.com/cbrkrtek/owasp-juice-shop-devsecops-ci-cd/blob/main/pictures%20for%20README/secrets%20scanning%20of%20juice%20shop.PNG)
![Secrets Scan 2](https://github.com/cbrkrtek/owasp-juice-shop-devsecops-ci-cd/blob/main/pictures%20for%20README/scanning%20token%20secrets%20of%20juice%20shop.PNG)
</details>

---

## Phase 2: Dynamic Testing & Proof-of-Concept (PoC) Exploitation

Moving beyond automated static scans, we performed **DAST (Dynamic Application Security Testing)** via **OWASP ZAP** combined with manual exploitation to prove real-world business impact.

### Case Study 1: SQL Injection (SQLi) Auth Bypass
* **Vulnerability:** Unescaped input concatenation in the authentication endpoint (CWE-89).
* **Impact:** **CRITICAL** (Full account takeover).

During ZAP probing on `POST /rest/user/login`, the application leaked raw SQLite query failures upon receiving a single quote (`'`). 

#### Exploitation Steps:
By injecting the comment vector `' OR 1=1 --` into the Email input field, the database query syntax was altered to bypass the password check completely, logging the attacker into the first database record (the **Administrator** account).

<details>
<summary>📸 View ZAP SQLi Alert Evidence</summary>

![SQLi Proof](https://github.com/cbrkrtek/owasp-juice-shop-devsecops-ci-cd/blob/main/pictures%20for%20README/SQLi%20in%20juice-shop.PNG)
</details>

### Case Study 2: Permissive CORS & Missing CSP Headers
* **Vulnerability:** Wildcard origin reflection (`Access-Control-Allow-Origin: *`) paired with a complete absence of a `Content-Security-Policy` header.
* **Impact:** **HIGH** (Session Hijacking via Stored XSS).

#### Exploitation Steps:
1. **CORS Bypass:** The `*` configuration enables unauthorized scripts from third-party malicious sites (e.g., `evil.com`) to extract authenticated data from the victim's session.
2. **Stored XSS via CSP Bypass:** By bypassing frontend validation, an attacker can register an account with a malicious payload embedded as their email: `<iframe src="javascript:alert('XSS')">@gmail.com`. Because there is no CSP header to enforce boundaries, the browser executes this inline script whenever the user profile is viewed, allowing silent exfiltration of session tokens via `document.cookie`.

<details>
<summary>📸 View CORS & Missing CSP Evidence</summary>

![CORS and CSP Proof](https://github.com/cbrkrtek/owasp-juice-shop-devsecops-ci-cd/blob/main/pictures%20for%20README/Cross-Domain%20Misconfiguration%20in%20juice-shop.PNG)
</details>

---

## Phase 3: Infrastructure Hardening & Remediation

As DevSecOps engineers, we mitigated these runtime application flaws **at the infrastructure layer** without rewriting the application source code.

### 1. Reverse Proxy Hardening (`nginx.conf`)
We engineered an NGINX layer to intercept traffic and append robust security controls:

```nginx
add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline';" always;
add_header Access-Control-Allow-Origin "[https://trusted-domain.com](https://trusted-domain.com)" always;
```

### 2. CI/CD 
The pipeline now actively rejects commits introducing new container-level vulnerabilities, enforcing security hygiene before code ever reaches deployment stages.

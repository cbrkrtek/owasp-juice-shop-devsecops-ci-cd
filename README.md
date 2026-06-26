# 🛡️ DevSecOps CI/CD Pipeline — OWASP Juice Shop

[![Pipeline Status](https://github.com/cbrkrtek/owasp-juice-shop-devsecops-ci-cd/actions/workflows/devsecops-pipeline.yml/badge.svg)](https://github.com/cbrkrtek/owasp-juice-shop-devsecops-ci-cd/actions)
[![Security Scan](https://img.shields.io/badge/trivy-CRITICAL%20gate-red)](https://github.com/cbrkrtek/owasp-juice-shop-devsecops-ci-cd/security/code-scanning)
[![SBOM](https://img.shields.io/badge/SBOM-CycloneDX-blue)](https://github.com/cbrkrtek/owasp-juice-shop-devsecops-ci-cd/actions)
[![License](https://img.shields.io/badge/license-MIT-green)](LICENSE)

A **production-ready DevSecOps security architecture** built around OWASP Juice Shop — an intentionally vulnerable application used as a real-world target. The project demonstrates how automated shift-left security gates and dynamic runtime exploitation testing work together in a single pipeline.

> **Goal:** Prove that security belongs in the pipeline — not bolted on after deployment.

---

## 📊 Security Executive Summary

| Gate | Result | Objective |
| :--- | :--- | :--- |
| **CI/CD Quality Gate** | ❌ FAILED *(expected)* | Block deploy on any `CRITICAL` CVE |
| **SCA — Vulnerability Count** | 63 CVEs (5 Critical, 35 High) | Base OS & dependency vetting |
| **Secret Detection** | ⚠️ 2 Active Leaks Trapped | Eliminate credential leakage in container layers |
| **SBOM Generated** | ✅ CycloneDX artifact | Supply chain transparency |
| **DAST Findings** | ⚠️ 3 High/Medium Exploits | Verify runtime risks: SQLi, CORS, Missing CSP |
| **Infrastructure Hardening** | ✅ NGINX proxy applied | Runtime remediation without source code changes |

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Developer Workstation                     │
│  git commit → [pre-commit: detect-secrets] → git push       │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                  GitHub Actions CI/CD                        │
│                                                             │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌───────────┐  │
│  │  Trivy   │  │  Trivy   │  │   ZAP    │  │  Quality  │  │
│  │  (Table) │→ │ (SARIF)  │→ │  (DAST)  │→ │   Gate    │  │
│  │  + SBOM  │  │→ Security│  │→ HTML    │  │ CRITICAL  │  │
│  │ CycloneDX│  │   Tab    │  │  Report  │  │ exit-code │  │
│  └──────────┘  └──────────┘  └──────────┘  └───────────┘  │
│       ↓               ↓            ↓                        │
│  [Artifact]    [Security Tab]  [Artifact]   ❌ Blocks Deploy │
└─────────────────────────────────────────────────────────────┘
                           │
                           ▼ (if gate passes)
┌─────────────────────────────────────────────────────────────┐
│                    Runtime Environment                       │
│                                                             │
│  [Client] → NGINX Reverse Proxy → Juice Shop :3000         │
│              (CSP, CORS, X-Frame, X-Content-Type)           │
└─────────────────────────────────────────────────────────────┘
```

---

## 🚀 Quick Start

**Prerequisites:** Docker, Docker Compose

```bash
# 1. Clone the repository
git clone https://github.com/cbrkrtek/owasp-juice-shop-devsecops-ci-cd.git
cd owasp-juice-shop-devsecops-ci-cd

# 2. Start the hardened stack
docker compose up -d

# 3. Access Juice Shop through the NGINX security proxy
open http://localhost
```

Juice Shop runs on port 3000 internally but is **only accessible through the NGINX hardening layer on port 80**.

---

## 📋 Project Structure

```
.
├── .github/
│   └── workflows/
│       └── devsecops-pipeline.yml   # Main CI/CD security pipeline
├── .pre-commit-config.yaml          # Local secret detection hooks
├── .trivyignore                     # Accepted risk register with justifications
├── docker-compose.yml               # Orchestration: Juice Shop + NGINX
├── package.json                     # Json-versions of Juice Shop
└── nginx.conf                       # Hardened reverse proxy configuration
```

---

## Phase 1 — Automated CI/CD Security Gates

The pipeline runs on every `push` and `pull_request` to `main`, auditing the finalized infrastructure — the container image and its layers — rather than raw source code.

### Pipeline Steps

| Step | Tool | Output | Purpose |
| :--- | :--- | :--- | :--- |
| Full Audit Table | Trivy | Console log | Human-readable CVE overview |
| SARIF Report | Trivy | GitHub Security Tab | Interactive CVE dashboard in repo |
| SBOM | Trivy (CycloneDX) | Pipeline artifact | Supply chain transparency |
| DAST | OWASP ZAP | HTML artifact | Runtime vulnerability validation |
| Quality Gate | Trivy | Exit code 1 | Blocks deploy on CRITICAL findings |

### SCA Results — 63 CVEs Found

Trivy scanned all layers of `bkimminich/juice-shop:latest` and found vulnerabilities across the Node.js runtime and Debian base OS.

**Top Critical CVEs:**

| CVE | CVSS | Component | Description | Status |
| :--- | :--- | :--- | :--- | :--- |
| CVE-2023-44487 | 7.5 | Node.js | HTTP/2 Rapid Reset DoS | Accepted (no public exposure) |
| CVE-2024-21626 | 8.6 | runc | Container escape via /proc | Accepted (no privileged mode) |
| CVE-2022-4304 | 7.4 | OpenSSL | RSA timing oracle | Accepted (TLS handled by NGINX) |

> Full accepted risk register: [`.trivyignore`](.trivyignore)

<details>
<summary>📸 View Trivy Scan Evidence</summary>

![Node.js Scan](pictures%20for%20README/node%20js%20scan.PNG)
![Debian OS Scan](pictures%20for%20README/result%20of%20scanning%20juiceshop%20via%20trivy.PNG)

</details>

### Secret Detection — 2 Leaks Trapped

| Severity | Secret Type | File | Action |
| :--- | :--- | :--- | :--- |
| HIGH | RSA Asymmetric Private Key | `insecurity.js` | Flagged in pipeline |
| MEDIUM | Hardcoded JWT Token | `app.guard.spec.ts` | Flagged in pipeline |

Both secrets are **pre-baked into the container image layers** — demonstrating why scanning the final artifact (not just source code) is critical.

<details>
<summary>📸 View Secret Detection Evidence</summary>

![Secrets Scan](pictures%20for%20README/secrets%20scanning%20of%20juice%20shop.PNG)
![Token Scan](pictures%20for%20README/scanning%20token%20secrets%20of%20juice%20shop.PNG)

</details>

---

## Phase 2 — DAST & Manual Exploitation (Proof of Concept)

Moving beyond static scans, OWASP ZAP performed active probing of the running application. Three vulnerabilities were confirmed with end-to-end exploitation.

### Case Study 1 — SQL Injection Auth Bypass

- **CWE-89** | Impact: **CRITICAL** — Full administrator account takeover
- **Vector:** `POST /rest/user/login` — unescaped input concatenation in SQLite query

The application returned raw SQLite error messages when receiving a single quote (`'`), leaking the underlying query structure. Injecting `' OR 1=1 --` into the email field rewrote the query logic to bypass password verification entirely, granting access to the first database record — the Administrator account.

<details>
<summary>📸 View SQLi Exploitation Evidence</summary>

![SQLi Proof](pictures%20for%20README/SQLi%20in%20juice-shop.PNG)

</details>

### Case Study 2 — Permissive CORS + Missing CSP

- **CWE-942 / CWE-693** | Impact: **HIGH** — Session hijacking via Stored XSS

Two misconfigurations combined into a critical chain:

1. **CORS Wildcard (`Access-Control-Allow-Origin: *`)** — any third-party origin (e.g. `evil.com`) can read authenticated API responses from a victim's session.
2. **No Content-Security-Policy header** — registering an account with email `<iframe src="javascript:alert('XSS')">@gmail.com` bypasses frontend validation. Without CSP, the browser executes the inline script whenever the profile is rendered, enabling silent `document.cookie` exfiltration.

<details>
<summary>📸 View CORS & CSP Evidence</summary>

![CORS and CSP Proof](pictures%20for%20README/Cross-Domain%20Misconfiguration%20in%20juice-shop.PNG)

</details>

---

## Phase 3 — Infrastructure Hardening (Remediation Without Source Changes)

Both runtime vulnerabilities were mitigated **at the infrastructure layer** via NGINX — no application source code was modified.

### NGINX Security Headers Applied

```nginx
# Content-Security-Policy — eliminates inline script execution
add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline'; connect-src 'self'; img-src 'self' data:;" always;

# CORS — restricts cross-origin requests to trusted origin only
add_header Access-Control-Allow-Origin "http://localhost" always;

# Clickjacking protection
add_header X-Frame-Options "SAMEORIGIN" always;

# MIME-sniffing protection
add_header X-Content-Type-Options "nosniff" always;
```

**Before (application layer):** `Access-Control-Allow-Origin: *` — any origin allowed
**After (NGINX proxy):** `Access-Control-Allow-Origin: http://localhost` — allowlist enforced

---

## Phase 4 — Local Security Barrier (Pre-commit Hooks)

`detect-secrets` runs locally **before** code ever reaches the pipeline, creating a shift-left barrier at the developer workstation level.

### Setup

```bash
# Install dependencies
pip install pre-commit detect-secrets

# Install hooks into local .git/hooks/
pre-commit install

# Create baseline (tells detect-secrets what's already known/accepted)
detect-secrets scan > .secrets.baseline

# Test it — every commit now triggers secret scanning automatically
git add .
git commit -m "test"
# Secret Detection hook runs on staged files
```

After setup, `git commit` automatically runs all hooks in `.pre-commit-config.yaml`:
- `detect-secrets` — catches API keys, tokens, private keys in staged files
- `detect-private-key` — dedicated private key check (defense in depth)
- `check-yaml` — validates pipeline YAML syntax before push

---

##  GitHub Advanced Security Setup

> Required for the SARIF upload step to populate the **Security → Code Scanning** tab.

**For public repositories (free):**

No action needed — Advanced Security is enabled by default for all public repos. After the first pipeline run, navigate to:
`Your Repo → Security tab → Code scanning`

**For private repositories:**

1. Go to `Settings → Security & analysis`
2. Enable **GitHub Advanced Security** (requires GitHub Team or Enterprise plan)
3. Enable **Code scanning** and **Secret scanning**

After enabling, the pipeline's `upload-sarif` step will populate the Security tab with interactive CVE details on every run.

---

## 🛠️ Tech Stack

| Layer | Tool | Purpose |
| :--- | :--- | :--- |
| CI/CD | GitHub Actions | Pipeline orchestration |
| SCA | Trivy | CVE scanning, secret detection, SBOM |
| DAST | OWASP ZAP | Runtime vulnerability validation |
| Hardening | NGINX | Reverse proxy, security headers |
| Containerization | Docker / Docker Compose | Environment orchestration |
| Pre-commit | detect-secrets | Local secret scanning |
| Supply Chain | CycloneDX SBOM | Software Bill of Materials |
| Target App | OWASP Juice Shop | Intentionally vulnerable application |

---

## 📖 Key Takeaways

**1. Scan the artifact, not just the source.**
Secrets baked into Docker image layers are invisible to source-code-only scanners. Trivy scanning the final image caught credentials that a standard SAST tool would have missed entirely.

**2. DAST validates what SAST cannot prove.**
Static analysis flagged the SQLi risk as theoretical. ZAP confirmed it as exploitable in under 30 seconds, logging in as Administrator with a single payload.

**3. Infrastructure can remediate application flaws.**
Adding CSP and CORS headers at the NGINX layer neutralized two High severity findings without touching Juice Shop's source code — a realistic approach when you don't own the application code.

**4. Quality gates only work when they actually block.**
`--exit-code 1` on CRITICAL findings means the pipeline fails loudly. Security findings that don't break the build get ignored.

---

## 📄 License

MIT — see [LICENSE](LICENSE)

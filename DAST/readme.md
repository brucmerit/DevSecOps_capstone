# Dynamic Application Security Testing (DAST)

## 1. Overview

This module implements Dynamic Application Security Testing (DAST) as part of the DevSecOps Capstone Project.

DAST is performed using **OWASP ZAP (Zed Attack Proxy)** to identify potential security vulnerabilities in the running web application.

The target application for testing is **OWASP Juice Shop**.

The DAST scanning process is integrated into the main Jenkins pipeline (`DevSecOps-capstone`) to automate vulnerability scanning, report generation, security evaluation and cleanup.

## 2. Tools and Technologies

| Tool | Purpose |
|---|---|
| OWASP ZAP | Dynamic application security scanning |
| Jenkins | Pipeline automation and execution |
| Docker | Deployment of the temporary testing environment |
| OWASP Juice Shop | Target web application |
| HTML Publisher Plugin | Display security reports in Jenkins |
| Pipeline Utility Steps Plugin | Process ZAP JSON scan results |

## 3. DAST Implementation

The DAST process consists of the following stages:

### 3.1 Deploy DAST Test Environment

Jenkins creates an isolated Docker network and volume for the DAST scan.

A temporary OWASP Juice Shop container is deployed as the target application.

This allows ZAP to scan the application without relying on an existing manually deployed testing environment.

### 3.2 Perform Security Scanning

OWASP ZAP is executed using its Baseline Scan:

`zap-baseline.py`

The scanner performs automated security checks against the running Juice Shop application.

The scan generates two reports:

- `zap-juice-shop-report.html` – Human-readable security assessment report.
- `zap-results.json` – Structured scan results for automated processing.

### 3.3 Process DAST Results

Jenkins extracts the generated reports from the Docker volume.

The reports are archived as Jenkins build artifacts, while the HTML report is published using the HTML Publisher Plugin.

The JSON report is processed to identify High and Medium risk findings.

### 3.4 DAST Security Gate

The pipeline evaluates the scan results using the following conditions:

| Risk level | Security gate condition |
|---|---|
| High | Build becomes UNSTABLE if one or more High-risk findings are detected |
| Medium | Build becomes UNSTABLE if more than five Medium-risk findings are detected |
| Low | Does not affect the current security gate |

If neither condition is exceeded, the security gate passes.

**Note:** A successful security gate does not mean that the application is free from vulnerabilities. It means that the findings did not exceed the configured thresholds.

### 3.5 Cleanup

After execution, Jenkins automatically removes the temporary Juice Shop container, Docker network and Docker volume.

This prevents unnecessary testing resources from remaining after the pipeline completes.

## 4. DAST Scan Findings

The OWASP ZAP assessment identified the following findings:

| Finding | Risk | OWASP Top 10 Mapping |
|---|---|---|
| Dangerous JavaScript Functions | Low | A03:2021 – Injection |
| Content Security Policy (CSP) Header Not Set | Medium | A05:2021 – Security Misconfiguration |

### 4.1 Dangerous JavaScript Functions

The scan identified the use of potentially dangerous JavaScript functions.

The use of functions such as `bypassSecurityTrustHtml()` with untrusted user input may introduce Cross-Site Scripting (XSS) risks.

**Recommendation:**
- Avoid bypassing built-in HTML sanitisation for untrusted content.
- Validate and sanitise user-supplied input.
- Apply appropriate output encoding.

### 4.2 Content Security Policy (CSP) Header Not Set

The application does not provide a Content Security Policy (CSP) response header.

Without an appropriate CSP, the application has reduced protection against certain Cross-Site Scripting (XSS) and content injection attacks.

**Recommendation:**
- Configure an appropriate Content Security Policy response header.
- Restrict permitted script sources and other content sources.
- Review and test the policy to ensure legitimate application functionality is maintained.

## 5. Reports and Evidence

The DAST reports and execution screenshots are maintained in this directory.

### Reports

- `reports/zap-juice-shop-report.html` – OWASP ZAP HTML security assessment report.
- `reports/zap-results.json` – OWASP ZAP JSON scan results.

### Screenshots

- `screenshots/jenkins-dast-pipeline.png` – Evidence of the integrated DAST pipeline execution in Jenkins.
- `screenshots/zap-security-report.png` – Evidence of the OWASP ZAP security findings.

## 6. Conclusion

OWASP ZAP has been integrated into the main Jenkins pipeline to automate Dynamic Application Security Testing against OWASP Juice Shop.

The implementation demonstrates automated deployment of a testing environment, vulnerability scanning, report generation, security gate evaluation and resource cleanup.

The generated reports provide visibility into potential security weaknesses and support further remediation and security improvements.

DAST is one of the three security testing components in this DevSecOps project, alongside Static Application Security Testing (SAST) and Software Composition Analysis (SCA).

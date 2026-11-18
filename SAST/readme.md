# OWASP Juice Shop - Static Application Security Testing (SAST)

## 1. SonarQube Project Details
* **Project Name:** OWASP-Juice-Shop-SAST
* **Project Key:** owasp-juice-shop-sast
* **Target Application:** OWASP Juice Shop (Node.js / TypeScript / Angular)
* **SAST Tool:** SonarQube Community Build (Docker)

---

## 2. SAST Scanning Procedure
1. **SonarQube Initialization:** Deployed SonarQube Community Build locally via Docker (http://localhost:9000) and created the manual project OWASP-Juice-Shop-SAST with project key owasp-juice-shop-sast.
2. **Authentication Setup:** Generated a project-scoped analysis token (sqp_...) for scanner authentication.
3. **Scanner Configuration:** Configured sonar-project.properties to analyze core backend (outes, lib, models) and frontend (rontend/src/app) source directories while excluding 
ode_modules, test suites, and static data folders to prevent scan timeouts.
4. **Quality Gate Configuration:** Created and bound the custom Quality Gate (Juice-Shop-Lab-QG) to evaluate the codebase against security and reliability thresholds.
5. **Scan Execution:** Executed sonarsource/sonar-scanner-cli via Docker against the OWASP Juice Shop source code and verified completion (EXECUTION SUCCESS) on the SonarQube Summary dashboard.

---

## 3. Quality Gate Conditions (Juice-Shop-Lab-QG)
| Metric | Scope | Operator | Threshold |
| :--- | :--- | :--- | :--- |
| Security Rating | Overall Code | is worse than | A |
| Reliability Rating | Overall Code | is worse than | A |
| Maintainability Rating | Overall Code | is worse than | A |
| Security Issues | Overall Code | is greater than | 0 |
| Security Hotspots Reviewed | Overall Code | is less than | 100% |

---

## 4. Scan Results & Findings
* **Scan Status:** Completed (EXECUTION SUCCESS)
* **Lines of Code Analyzed:** 27k
* **Total Issues Detected:** 107
* **Summary Metrics (Overall Code):**
  * **Security:** 21 Open Issues (Rating: **E**)
  * **Reliability:** 81 Open Issues (Rating: **D**)
  * **Maintainability:** 32 Open Issues (Rating: **A**)
  * **Security Hotspots:** 0 (Rating: **A**)
  * **Code Coverage:** 0.0%
  * **Duplications:** 0.7%

### Key OWASP Top 10 Findings
1. **[Vulnerability 1 Title — copy from SonarQube Security Issues tab]**
   * **OWASP Top 10 Category:** [e.g., A03:2021 – Injection / A02:2021 – Cryptographic Failures]
   * **Affected Location:** [e.g., routes/fileName.ts, Line XX]
   * **Description:** [Brief description of the vulnerable code pattern flagged by SonarQube]
   * **Proposed Remediation:** [Specific code fix to eliminate the vulnerability]

2. **[Vulnerability 2 Title — copy from SonarQube Security Issues tab]**
   * **OWASP Top 10 Category:** [e.g., A07:2021 – Identification and Authentication Failures]
   * **Affected Location:** [e.g., lib/insecurity.ts, Line XX]
   * **Description:** [Brief description of the vulnerable code pattern flagged by SonarQube]
   * **Proposed Remediation:** [Specific code fix to eliminate the vulnerability]

# DevSecOps_capstone Project

## 1. Project Overview

This project implements an automated DevSecOps pipeline to integrate security testing into the Software Development Life Cycle (SDLC).

The project uses Jenkins to automate the execution of security testing tools against OWASP Juice Shop, a deliberately vulnerable web application.

The pipeline incorporates three main security testing practices:

- **Static Application Security Testing (SAST):** Identifies potential vulnerabilities in application source code.
- **Software Composition Analysis (SCA):** Identifies vulnerable third-party dependencies and software packages.
- **Dynamic Application Security Testing (DAST):** Identifies potential vulnerabilities in a running web application.

The objective is to demonstrate how automated security testing can be incorporated into a CI/CD pipeline to improve application security and provide continuous visibility of security risks.

## 2. Team Structure and Responsibilities

The project is carried out by a four-member team, with each member assigned specific responsibilities.

| Team Member | Role | Responsibility | Security Tool |
|---|---|---|---|
| JJ | Team Leader | Jenkins environment setup, pipeline development, integration, team coordination and overall verification | Jenkins |
| Jolin | Team Member 1 | Static Application Security Testing (SAST) | SonarQube |
| Raziq | Team Member 2 | Software Composition Analysis (SCA) | Snyk |
| Amos | Team Member 3 | Dynamic Application Security Testing (DAST) | OWASP ZAP |

### 2.1 Team Leader – JJ

Responsible for coordinating the overall project, establishing the Jenkins environment, developing the main Jenkins pipeline, integrating the individual security testing components and verifying the complete DevSecOps workflow.

Key responsibilities include:

- Setting up and configuring the Jenkins environment.
- Establishing GitHub repository integration.
- Developing and maintaining the main Jenkinsfile.
- Integrating SAST, SCA and DAST into the main pipeline.
- Coordinating team members and consolidating their contributions.
- Verifying pipeline execution and documenting project evidence.

### 2.2 Team Member 1 – Jolin

Responsible for implementing Static Application Security Testing (SAST) using SonarQube.

Key responsibilities include:

- Configuring the SonarQube project.
- Performing source code security analysis.
- Identifying and documenting potential vulnerabilities.
- Configuring and evaluating the Quality Gate.
- Providing supporting reports and screenshots.

### 2.3 Team Member 2 – Raziq

Responsible for implementing Software Composition Analysis (SCA) using Snyk.

Key responsibilities include:

- Configuring Snyk for dependency analysis.
- Scanning application dependencies for known vulnerabilities.
- Identifying vulnerable third-party packages.
- Documenting vulnerability findings and recommendations.
- Providing supporting reports and screenshots.

### 2.4 Team Member 3 – Amos

Responsible for implementing Dynamic Application Security Testing (DAST) using OWASP ZAP.

Key responsibilities include:

- Configuring OWASP ZAP for dynamic security testing.
- Performing automated scans against OWASP Juice Shop.
- Generating HTML and JSON security reports.
- Analysing and documenting identified vulnerabilities.
- Providing supporting reports and screenshots.

## 3. Tools and Technologies

| Tool / Technology | Purpose |
|---|---|
| Jenkins | CI/CD automation and pipeline orchestration |
| Docker | Containerisation and deployment of testing environments |
| GitHub | Source code repository and version control |
| OWASP Juice Shop | Target application for security testing |
| SonarQube | Static Application Security Testing |
| Snyk | Software Composition Analysis |
| OWASP ZAP | Dynamic Application Security Testing |

## 4. DevSecOps Pipeline Architecture

The main Jenkins pipeline (`DevSecOps-capstone`) serves as the central automation and integration point for the project.

The pipeline is maintained in the root directory of the GitHub repository through the `Jenkinsfile`.

The intended integrated workflow consists of the following stages:

1. Initialise the Jenkins pipeline.
2. Verify the execution environment and project repository.
3. Perform Static Application Security Testing (SAST) using SonarQube.
4. Perform Software Composition Analysis (SCA) using Snyk.
5. Deploy the DAST testing environment.
6. Perform Dynamic Application Security Testing (DAST) using OWASP ZAP.
7. Generate and archive security reports.
8. Evaluate security findings through the configured security gates.
9. Clean up temporary testing resources.

The SAST, SCA and DAST components are maintained in their respective repository directories.

## 5. Jenkins Environment and Configuration

The Jenkins environment is deployed using Docker.

The main configuration includes:

- Jenkins server running in a Docker container.
- GitHub repository integration for pipeline source retrieval.
- Jenkins Pipeline job configured to retrieve the `Jenkinsfile` from the `main` branch.
- Docker access for managing testing containers and networks.
- SonarQube server configuration for SAST integration.
- Jenkins plugins to support report processing and HTML report publishing.

The main Jenkins job is named:

`DevSecOps-capstone`

The pipeline configuration is maintained in the repository root.

## 6. Pipeline Integration and Responsibilities

As the team leader, JJ is responsible for consolidating the security testing contributions from the respective team members into the main Jenkins pipeline.

### 6.1 Static Application Security Testing (SAST)

**Tool: SonarQube**

SonarQube is used to analyse application source code for potential security vulnerabilities and code quality issues.

The SAST implementation is maintained under:

`SAST/`

### 6.2 Software Composition Analysis (SCA)

**Tool: Snyk**

Snyk is used to analyse third-party dependencies and identify known vulnerabilities in application packages.

The SCA implementation is maintained under:

`SCA/`

### 6.3 Dynamic Application Security Testing (DAST)

**Tool: OWASP ZAP**

OWASP ZAP is integrated into the main Jenkins pipeline to perform automated security testing against the running OWASP Juice Shop application.

The DAST implementation includes:

- Temporary Docker-based testing environment.
- Automated ZAP Baseline Scan.
- HTML and JSON report generation.
- Jenkins report archiving and HTML publishing.
- Automated security gate evaluation.
- Docker resource cleanup after execution.

The DAST implementation and supporting evidence are maintained under:

`DAST/`

## 7. Security Gate and Pipeline Evaluation

Security gates are used to evaluate detected vulnerabilities and determine whether the configured security thresholds have been exceeded.

For the current DAST implementation, the following thresholds are configured:

| Severity | Configured Condition | Pipeline Result |
|---|---|---|
| High | More than 0 findings | UNSTABLE |
| Medium | More than 5 findings | UNSTABLE |
| Low | No threshold configured | Does not affect the gate |

The security gate provides an automated mechanism to identify findings that exceed the configured thresholds.

A successful pipeline execution indicates that the configured conditions were satisfied. It does not necessarily indicate that the application is free from vulnerabilities.

The SAST and SCA security gate configurations will be documented alongside their respective implementations once integrated and verified.

## 8. Repository Structure

The project repository is organised as follows:

```text
DevSecOps_capstone/
│
├── Jenkinsfile
├── README.md
│
├── architecture/
│   └── README.md
│
├── SAST/
│   └── readme.md
│
├── SCA/
│   └── readme.md
│
└── DAST/
    ├── readme.md
    ├── reports/
    │   ├── zap-juice-shop-report.html
    │   └── zap-results.json
    │
    └── screenshots/
        ├── jenkins-dast-pipeline.png
        └── zap-security-report.png
```

## 9. Testing and Verification

The main Jenkins pipeline is used to verify the integration and execution of the security testing components.

Verification activities include:

- Successful retrieval of the Jenkinsfile from GitHub.
- Jenkins pipeline execution.
- Deployment of the required testing environment.
- Security scanning and report generation.
- Processing of scan results.
- Evaluation of configured security gates.
- Availability of reports and supporting evidence.
- Cleanup of temporary Docker resources.

Pipeline execution logs, security reports and screenshots provide supporting evidence of the implementation and testing activities.

## 10. Conclusion

This project demonstrates the implementation of a DevSecOps pipeline that incorporates automated security testing into the application development workflow.

As the team leader, JJ coordinated the development activities, established the Jenkins automation environment and integrated the security testing contributions from Jolin, Raziq and Amos.

The use of SonarQube, Snyk and OWASP ZAP provides a foundation for identifying security vulnerabilities across application source code, third-party dependencies and the running application.

The project demonstrates the importance of integrating security practices into the SDLC rather than treating security testing as a separate activity at the end of development.

# QAD_ARYANPATIL



 Continuous Security Testing Pipeline (DevSecOps QA)

## 1. Overview
This project demonstrates how to implement a Continuous Security Testing pipeline within a DevSecOps workflow. The goal is to embed security checks into the QA and CI/CD process so vulnerabilities are detected early, compliance is enforced, and only secure builds are deployed.

---

## 2. Problem Statement
The task is to build a QA pipeline that integrates security checks across the development lifecycle. The pipeline must perform:

- Static Application Security Testing (SAST)
- Dynamic Application Security Testing (DAST)
- Dependency and container vulnerability scanning
- Infrastructure-as-Code (IaC) security checks
- Secrets detection
- Compliance verification based on OWASP standards
- Aggregated and prioritized reporting with remediation guidance

---

## 3. Solution Design (Optimal Approach)
The implemented pipeline ensures that security checks run automatically on every pull request and push to the main branch. It uses ephemeral environments for dynamic testing, aggregates results from all tools, and enforces quality gates to prevent insecure code from merging.

**## 4. Architecture Diagram

Developer → Pushes code or creates PR → GitHub Repository
GitHub Repository → Triggers GitHub Actions CI Pipeline

Inside the CI Pipeline:
Semgrep SAST → Runs static application security testing

Secrets Scan → Detects hardcoded secrets

Checkov IaC Scan → Scans Infrastructure-as-Code for misconfigurations

Build Container Image → Builds Docker image

Image is scanned by Trivy for vulnerabilities


Then:
Ephemeral Review App is created → OWASP ZAP DAST performs dynamic security testing

Reports:

All results (SAST, IaC, Image Scan, DAST) are sent to a Report Aggregator

Aggregator posts a PR Comment + Summary

Quality Gate / Policy-as-Code checks if build passes or fails

Based on result → Block or Allow merge in GitHub**flowchart TD
    Dev[Developer] -->|Push / PR| GitHub[GitHub Repository]
    GitHub --> Actions[GitHub Actions CI Pipeline]

    Actions --> SAST[Semgrep SAST]
    Actions --> Secrets[Secrets Scan]
    Actions --> IaC[Checkov IaC Scan]
    Actions --> Build[Build Container Image]

    Build --> ImageScan[Trivy Image Scan]

    Actions --> DeployEP[Ephemeral Review App]
    DeployEP --> DAST[OWASP ZAP DAST]

    SAST -->|SARIF| Aggregator[Report Aggregator]
    IaC -->|SARIF| Aggregator
    ImageScan -->|SARIF| Aggregator
    DAST -->|JSON| Aggregator

    Aggregator --> PRComment[PR Comment + Summary]
    Aggregator --> Policy[Quality Gate / Policy-as-Code]
---

## 5. Tools and Their Purpose

-Semgrep (SAST)
Detects insecure coding patterns and vulnerabilities by analyzing source code without executing it.

-detect-secrets (Secrets Scan)
Identifies sensitive information such as API keys, passwords, or tokens accidentally committed to the repository.

-Checkov (IaC Scan)
Scans infrastructure-as-code files (Terraform, Kubernetes, CloudFormation, etc.) for misconfigurations that could expose security risks.

-Trivy (Dependency and Container Scan)
Scans dependencies, operating system packages, and container images for known CVEs. Ensures that images used in deployment are free from critical vulnerabilities.

-OWASP ZAP (DAST)
Performs automated penetration testing by simulating runtime attacks on a deployed application. Validates how the system behaves against real-world threats.

-GitHub SARIF Upload
Provides a standardized format for security findings, allowing results from tools to be viewed directly in GitHub’s Security tab.


---

## 6. Pipeline Workflow

The GitHub Actions pipeline executes the following stages:

Checkout – fetches source code.

SAST (Semgrep) – scans for insecure code patterns.

Secrets Scan (detect-secrets) – checks for exposed credentials.

IaC Scan (Checkov) – validates security of infrastructure-as-code.

Build & Image Scan (Trivy) – builds Docker image and scans for vulnerabilities.

Deploy Ephemeral App – launches a temporary environment for testing.

DAST (OWASP ZAP) – runs penetration tests on the deployed app.

Aggregator & Reporting – consolidates findings, prioritizes issues, and posts results to the pull request.


---

## 7. Running the Pipeline
Locally

Install the tools and run scans manually:

pip install semgrep checkov detect-secrets

brew install aquasecurity/trivy

### Run SAST
semgrep ci --sarif > semgrep.sarif

### Run IaC scan
checkov -d . -o sarif > checkov.sarif

### Run container scan
trivy fs --format sarif --output trivy.sarif .

### Run secrets detection
detect-secrets scan > secrets.baseline

In GitHub Actions CI/CD

The pipeline runs automatically on pull requests and pushes to the main branch.
To trigger manually:

gh workflow run ci-security.yml


Results appear in the GitHub Security tab and as summarized comments on the pull request.

---

## 8. Trade-offs and Limitations

Static and dynamic tools may produce false positives and require tuning.

DAST is limited in handling authenticated or complex user flows.

Running ephemeral environments increases CI execution time.

Business-logic vulnerabilities are not covered by automated tools and still require manual testing.


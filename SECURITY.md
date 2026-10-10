# Security Policy

## Supported Versions

Git Security Sentinel follows semantic versioning. Security updates are applied to the active release line:

| Version | Supported          |
| ------- | ------------------ |
| 1.0.x   | :white_check_mark: |
| < 1.0   | :x:                |

---

## Reporting a Vulnerability

The maintainers of Git Security Sentinel take software security and confidentiality seriously. If you discover a security vulnerability, flaw in secret redaction, or potential bypass, please report it responsibly.

### Recommended Reporting Mechanism

1. **GitHub Private Vulnerability Reporting (Preferred)**:
   Please submit a private advisory through the repository's **Security** tab:
   - Navigate to the GitHub repository.
   - Click **Security** > **Advisories**.
   - Click **Report a vulnerability** to open a confidential report directly to repository maintainers.

2. **Alternative Safe Disclosure**:
   If private vulnerability reporting is not accessible on the repository, please open an issue with the label `security-inquiry` requesting a secure communication channel **without** posting exploit details, proof-of-concept payloads, or sensitive tokens publicly in open issues.

---

## What to Include in Your Report

To help us triage and resolve the issue quickly, please provide:
- A clear description of the vulnerability (e.g., regex bypass, regex denial of service, information leakage in redaction).
- Steps to reproduce using **synthetic, non-sensitive test strings only**.
- Affected component (`scripts/scan_diff.py`, `scripts/ai_review.py`, or `SKILL.md`).
- Suggested remediation or patch if available.

---

## Disclosure Guidelines

- Please allow maintainers a reasonable remediation window (typically 30–90 days) before publicly disclosing any vulnerability.
- Maintainers will acknowledge receipt within 48 hours and provide progress updates during remediation.

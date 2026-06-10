# Workstream B Report: Web Application Security Assessment

## Scope

This report covers the Project KAVACH Workstream B web application assessment. The lab uses two intentionally vulnerable applications:

- DVWA at `http://localhost:8080`
- OWASP Juice Shop at `http://localhost:3000`

The assessment uses only local Docker-hosted targets and free/open-source tooling, matching the engagement brief constraints.

## Reproducible Environment

Start the lab:

```bash
cd Webapp2
docker compose -f env/docker-compose.yml up -d
```

Verify:

```bash
curl -I http://localhost:8080
curl -I http://localhost:3000
```

Stop the lab:

```bash
docker compose -f env/docker-compose.yml down
```

Expected runtime is under 15 minutes on a laptop with Docker Desktop.

## Coverage Summary

| ID | Finding | OWASP Top 10 2021 | App | Severity | Evidence |
| --- | --- | --- | --- | --- | --- |
| F-01 | SQL Injection | A03 Injection | DVWA | High | `findings/F-01-sqli/` |
| F-02 | Cross-Site Scripting | A03 Injection | DVWA | High | `findings/F-02-xss/` |
| F-03 | IDOR / Broken Access Control | A01 Broken Access Control | Juice Shop | High | `findings/F-03-idor/` |
| F-04 | Command Injection | A03 Injection | DVWA | Critical | `findings/F-04-command-injection/` |
| F-05 | Unrestricted File Upload | A05 Security Misconfiguration | DVWA | High | `findings/F-05-file-upload/` |
| F-06 | Authentication Weaknesses | A07 Identification and Authentication Failures | Juice Shop | Medium | `findings/F-06-authentication-failures/` |
| F-07 | Version and Dependency Risk | A06 Vulnerable and Outdated Components | Juice Shop | Medium | `findings/F-07-vulnerable-components/` |

The normalized finding set covers five OWASP Top 10 2021 categories: A01, A03, A05, A06, and A07. It also includes the mandatory SQL injection, XSS, IDOR/broken access control, and authentication-failure tracks.

## Key Findings

### F-01: SQL Injection

DVWA accepted SQL syntax in a user-controlled field, allowing query manipulation. In a financial portal, the equivalent issue could expose borrower, merchant, or account-statement records.

Remediation: use parameterized queries, validate identifiers, and reduce database account privileges.

### F-02: Cross-Site Scripting

DVWA executed attacker-controlled JavaScript in the application origin. In a customer portal, this could lead to session theft, user deception, or unauthorized browser-side actions.

Remediation: apply context-aware output encoding and a restrictive Content Security Policy.

### F-03: IDOR / Broken Access Control

Juice Shop API traffic showed object identifiers and authentication tokens suitable for object-level authorization testing. The documented vulnerable pattern is identifier tampering to access another user's object.

Remediation: enforce ownership checks on every object fetch and log authorization failures.

### F-04: Command Injection

DVWA executed additional operating-system commands appended to a normal input value. This is the highest-risk issue because it crosses from application input into host command execution.

Remediation: avoid shell execution, use safe APIs, and validate input with strict allowlists.

### F-05: Unrestricted File Upload

DVWA accepted uploaded files without sufficient type and storage controls. In a production portal, this could enable malware delivery, stored content attacks, or server-side execution.

Remediation: allowlist MIME types, rename files, store outside the web root, and serve downloads through an authorization-checked handler.

### F-06: Authentication Weaknesses

Juice Shop login/API testing provides the track for A07 authentication failures. The finding documents repeated-login and token-handling validation steps that should be captured with raw HTTP evidence.

Remediation: rate-limit login attempts, return generic errors, log failed attempts, and use short-lived tokens.

### F-07: Version and Dependency Risk

Juice Shop exposed version/API metadata during developer-tools inspection. This supports dependency fingerprinting and should be paired with dependency scanning.

Remediation: remove public version disclosure and add dependency scan gates to CI.

## Remediation Status

Because the repository contains screenshots and markdown evidence rather than DVWA/Juice Shop source code, remediation is provided as proposed patch artifacts in each finding folder. The patches show the intended secure implementation pattern but were not applied to upstream application source.

At least three code-level remediation patterns are included:

- `findings/F-01-sqli/remediation.patch`
- `findings/F-02-xss/remediation.patch`
- `findings/F-03-idor/remediation.patch`
- `findings/F-04-command-injection/remediation.patch`
- `findings/F-05-file-upload/remediation.patch`
- `findings/F-06-authentication-failures/remediation.patch`
- `findings/F-07-vulnerable-components/remediation.patch`

## SAST Status

SAST placeholders are included under `sast/`:

- `sast/before.json`
- `sast/after.json`

These are structured and reviewable, but not scanner-generated. A real SAST diff requires adding the target source code and running Semgrep CE or ESLint security rules before and after applying fixes.

## Evidence Index

The cleaned deliverable artifacts are:

- `env/docker-compose.yml`
- `findings/`
- `sast/`
- `report.md`

Each finding folder contains its own `evidence/` screenshots, `request.http` examples, and `remediation.patch`.

## Remaining Gaps

1. Capture raw HTTP requests and responses using curl, Burp Community, or ZAP and store them in each finding folder.
2. Add actual source code or cloned upstream vulnerable app source to generate real before/after SAST reports.
3. Convert proposed remediation patches into real commits if source code is added.

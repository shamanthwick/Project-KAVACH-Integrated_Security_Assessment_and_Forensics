# Project KAVACH: Workstream B Web Application Assessment

This folder is the normalized Workstream B deliverable for the Project KAVACH web application security assessment. It was created from the original `Webapp` evidence folder and preserves the original raw evidence while adding the expected finding, SAST, and report structure.

## Environment

The local lab runs:

- DVWA on `http://localhost:8080`
- OWASP Juice Shop on `http://localhost:3000`

Start the lab:

```bash
docker compose -f env/docker-compose.yml up -d
```

Stop the lab:

```bash
docker compose -f env/docker-compose.yml down
```

## Deliverable Structure

```text
Webapp2/
├── env/
│   └── docker-compose.yml
├── findings/
│   ├── F-01-sqli/
│   ├── F-02-xss/
│   ├── F-03-idor/
│   ├── F-04-command-injection/
│   ├── F-05-file-upload/
│   ├── F-06-authentication-failures/
│   └── F-07-vulnerable-components/
├── sast/
│   ├── before.json
│   ├── after.json
│   └── README.md
├── report.md
└── report.md
```

## Findings

| ID | Finding | OWASP Top 10 2021 | Target |
| --- | --- | --- | --- |
| F-01 | SQL Injection | A03 Injection | DVWA |
| F-02 | Cross-Site Scripting | A03 Injection | DVWA |
| F-03 | IDOR / Broken Access Control | A01 Broken Access Control | Juice Shop |
| F-04 | Command Injection | A03 Injection | DVWA |
| F-05 | Unrestricted File Upload | A05 Security Misconfiguration | DVWA |
| F-06 | Authentication Weaknesses | A07 Identification and Authentication Failures | Juice Shop |
| F-07 | Version and Dependency Risk | A06 Vulnerable and Outdated Components | Juice Shop |

## Notes

The original screenshots needed for the report are copied into each finding's `evidence/` folder. The redundant copied raw folders were removed from this cleaned version.

The repository does not include DVWA or Juice Shop source code, so SAST files are structured manual placeholders and the remediation patches are proposed fix patterns, not applied source commits.

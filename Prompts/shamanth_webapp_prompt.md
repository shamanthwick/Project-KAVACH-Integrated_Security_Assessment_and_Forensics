# Prompts used for Webapp Assessment – Project KAVACH
**Author: Shamanth**

These prompts focus on vulnerability research, payload crafting, and remediation planning.

## Prompt 1: Setting up the Vulnerable Lab
> "I need to set up a local security testing environment for Project KAVACH. Can you help me create a `docker-compose.yml` file that spins up both the Damn Vulnerable Web Application (DVWA) and OWASP Juice Shop? I need them accessible on ports 8080 and 3000 respectively for manual penetration testing."

## Prompt 2: Crafting SQL Injection Payloads
> "I'm testing the User ID field in DVWA for SQL Injection. Can you explain the difference between a simple `' OR '1'='1` payload and a more advanced `UNION` based attack? I want to demonstrate how an attacker could extract the database version or table names. Please provide 2-3 sample payloads I can use for my evidence screenshots."

## Prompt 3: Explaining Command Injection Risk
> "During my assessment of DVWA's DNS lookup tool, I successfully executed a command injection using `; whoami`. Can you help me write a technical finding for this? I need to explain the risk of moving from a simple application input to full OS-level command execution and why this is considered a 'Critical' severity finding."

## Prompt 4: API Assessment & IDOR
> "I'm analyzing the network traffic of OWASP Juice Shop using browser developer tools. I've noticed that the `/api/BasketItems/` endpoint uses numeric IDs. How can I test this for Insecure Direct Object Reference (IDOR)? Guide me through the process of tampering with the basket ID to see if I can view or modify another user's cart."

## Prompt 5: Generating Remediation Patches
> "For my Webapp report, I need to provide remediation advice for the SQLi and XSS vulnerabilities I found. Can you generate 'Remediation Patches' (diff format) for a generic PHP application? Show me how to replace unsafe variable concatenation with PDO parameterized queries for the SQLi fix, and how to use `htmlspecialchars()` for the XSS fix."

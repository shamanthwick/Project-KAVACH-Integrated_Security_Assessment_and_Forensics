# =========================
# FILE 1: threat-model.md
# =========================

# Joint Threat Model — Project KAVACH

## Context

This threat model connects real findings from:

### Workstream A (Network)
- Multiple malware PCAPs analyzed:
  - Hancitor + Ficker + Cobalt Strike
  - Trickbot + Cobalt Strike
  - RedLine Stealer
- Observed behaviors:
  - C2 beaconing (command-and-control)
  - Suspicious DNS activity
  - Internal AD interaction
  - SMB / LDAP / Kerberos lateral movement risk
  - Direct HTTP POST exfiltration to external IP (188.190.10.10:55123)
  - High-volume internal → external data upload
  - Plaintext FTP credentials exposure
  - Port scanning (1,030 NULL probes across 1000 ports) [1](https://github.com/shamanthwick/final-project-for-IIT-X-Futurense/tree/main)

---

### Workstream B (Web)

(From engagement + standard DVWA / Juice Shop coverage)
- SQL Injection
- IDOR (Broken Access Control)
- XSS
- Authentication weaknesses
- Exposure of sensitive data paths

OWASP Juice Shop intentionally contains real-world vulnerabilities across OWASP Top 10 categories. [2](https://owasp.org/www-project-juice-shop/)

---

# Threat 1: Web Compromise → Network Exfiltration

## Attack Chain
1. Attacker exploits SQL Injection in portal
2. Gains access to backend DB / sensitive data
3. Uses IDOR to enumerate customer records
4. Extracts data and sends externally
5. Network shows:
   - High-volume outbound uploads
   - Direct HTTP POST behavior (RedLine-style)
   - Unusual DNS traffic

## Evidence Mapping
- Web: SQLi + IDOR
- Network:
  - POST exfiltration behavior
  - External IP communication (188.190.10.10)
  - DNS anomalies

## What confirms this
- Same server/IP handles portal + outbound traffic
- Timestamp correlation (web activity matches exfiltration)
- Payload/data pattern similarity

## Confidence
Medium–High (Strong behavioral overlap with exfiltration patterns)

---

# Threat 2: Malware Foothold → Portal Abuse

## Attack Chain
1. Endpoint infected (Hancitor / Trickbot)
2. Malware establishes C2 beaconing
3. Credentials stolen (suspected in PCAP)
4. Internal movement via:
   - SMB / LDAP / Kerberos
5. Attacker accesses portal
6. Exploits IDOR / auth weaknesses
7. Extracts customer/merchant data

## Evidence Mapping
- Network:
  - C2 beaconing
  - Lateral movement protocols
  - Credential abuse suspected
- Web:
  - IDOR (horizontal privilege abuse)
  - Weak authentication

## What confirms this
- Compromised host communicating with portal
- Stolen credentials used in portal login
- Portal activity follows network compromise timeline

## Confidence
Medium (requires correlation validation)

---

# Key Insight

> This engagement proves that network compromise and application-layer exploitation are not isolated — they are part of a **potential multi-stage attack chain**.

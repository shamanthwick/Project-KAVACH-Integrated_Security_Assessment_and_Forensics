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

---

# =========================
# FILE 2: defense-in-depth.md
# =========================

# Defense-in-Depth Strategy — Project KAVACH

## Objective

Mitigate:
- Malware foothold
- Lateral movement
- Data exfiltration
- Application exploitation

---

## Layered Controls

| Layer | Control | Mapped Finding | Effort | Trade-off |
|------|--------|---------------|--------|----------|
| Identity | Enforce MFA + session hardening | Credential theft risk from malware | Medium | User friction |
| Perimeter | WAF + outbound filtering | SQLi + data exfiltration traffic | Medium | Tuning required |
| Segmentation | Restrict SMB / LDAP / Kerberos flows | Lateral movement observed | High | Complex rollout |
| Application | Parameterized queries + authZ validation | SQLi + IDOR vulnerabilities | Medium | Dev effort |
| Data | Encrypt + restrict statement access | Sensitive data exposure via IDOR | Medium | Key mgmt overhead |
| Observability | Correlate DNS + HTTP + app logs | DNS anomaly + C2 beaconing | Medium | Alert fatigue |
| Response | Unified IR playbook (network + web) | Multi-stage attack chain | Low | Needs testing |

---

## Strategy Focus

1. **Stop initial compromise**
   - Fix SQLi
   - Improve authentication

2. **Contain spread**
   - Block lateral movement

3. **Detect early**
   - Monitor DNS + traffic baseline

---

## Key Insight

> Security is not one control — it is a **chain of controls breaking attacker progression at each step**.

---

# =========================
# FILE 3: exec-readout.md
# =========================

# Executive Readout — Project KAVACH

## Summary

Two critical signals triggered this assessment:
- Malware-like activity and anomalous network traffic
- Confirmed vulnerabilities in the customer portal

The key question:
> Are these independent issues — or part of one attack?

---

## Key Findings

### 1. Malware-style activity in network traffic
Analysis shows:
- Command-and-control communication
- Potential credential abuse
- Data exfiltration patterns

This indicates possible compromise inside the environment.

---

### 2. Customer portal has exploitable vulnerabilities
The portal allows:
- Unsafe input handling (SQL Injection)
- Unauthorized data access (IDOR)

This creates a direct risk to customer data.

---

### 3. Strong possibility of multi-stage attack
The combination suggests:
- Initial foothold via malware OR portal
- Followed by lateral movement or data extraction

---

## Recommendations

### 1. Fix application vulnerabilities immediately
Eliminate SQL injection and access control weaknesses.

---

### 2. Restrict and monitor internal traffic
Stop unnecessary system-to-system communication and detect anomalies.

---

### 3. Unify security visibility
Combine application and network monitoring for early detection of attack chains.

---

## One Ask

Approve a focused **security hardening initiative** that:
- Secures the application
- Controls internal movement
- Improves detection across the environment

---

## Closing Note

> The risk is not just individual vulnerabilities —  
> it is how attackers combine them into a full attack path.

A layered defense will break that chain early.
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

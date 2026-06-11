# Reflection — Madhurjya Deka

## Q1 — Specific packet that changed understanding

While analyzing the RedLine Stealer PCAP, a HTTP POST request to external IP (188.190.10.10:55123) revealed high-volume outbound data.

Initially, I believed it was normal browsing traffic. However, packet inspection showed structured data transfer consistent with exfiltration behavior.

This shifted my interpretation from benign traffic to active data theft.

---

## Q2 — Wrong hypothesis

We initially assumed DNS-heavy traffic indicated data exfiltration.

However, further analysis showed some DNS activity was legitimate system behavior.

This was falsified by absence of structured payload transfer.

---

## Q3 — Vulnerability exploitation journey

For SQL Injection:
- First attempt failed using basic payload
- Discovered input filtering
- Adjusted with encoding and syntax
- Final payload successfully bypassed login

Key learning: payload tuning matters more than complexity.

---

## Q4 — Remediation decision

We implemented parameterized queries instead of input filtering.

Alternative considered: blacklist filtering  
Rejected because it is bypassable.

Trade-off: increased development effort but stronger security.

---

## Q5 — LLM mistake

LLM suggested some DNS traffic patterns were malicious.

However, manual analysis showed normal system behavior.

This reinforced need for validation.

---

## Q6 — Cross-surface attack chain

Network finding:
C2 beaconing + credential theft

Web implication:
Stolen credentials used to exploit IDOR vulnerability

This creates a chain:
Network compromise → web abuse → data extraction

---

## Q7 — Risky defense decision

Network segmentation was high-impact but costly.

Trade-off:
- Improves security significantly
- But increases operational complexity

Still recommended due to high lateral movement risk.

---

## Q8 — Commit reflection

Most proud:
Created joint threat model linking network + web findings

Redo:
Initial IOC extraction lacked context and needed refinement

# Project KAVACH: Integrated Security Assessment & Forensics

## 🛡️ Project Overview
**Project KAVACH** is a comprehensive cybersecurity investigation that simulates a realistic, multi-stage breach against a fictional financial enterprise, *Meridian FinServe*. 

Unlike traditional isolated audits, Project KAVACH integrates **Network Forensics** (Workstream A) and **Web Application Security** (Workstream B) into a unified **Joint Threat Model** (Workstream C). This approach demonstrates the critical link between application vulnerabilities and network-wide compromise.

---

## 👥 The Team
| Name | Role | Core Activity |
| :--- | :--- | :--- |
| **Shamanth R** | **Network Forensics Lead** | Deep-dive PCAP analysis, C2 beaconing identification, and IOC extraction. |
| **Saiteja Kacham** | **Web Application Lead** | Penetration testing of DVWA/Juice Shop and SAST analysis. |
| **Madhurjya Deka** | **Strategic Synthesis Lead** | Threat modeling, attack narrative construction, and executive reporting. |
| **Ajith Mohan** | **Project Maintainer** | Document finalization, GitHub management, and Quality Assurance. |

---

## 🔗 The Unified Attack Narrative
The core of Project KAVACH is the discovery of a **Multi-Stage Attack Chain**:
1.  **Initial Foothold:** Attacker exploits a **SQL Injection** in the customer portal (Workstream B).
2.  **Data Discovery:** Using **IDOR**, the attacker enumerates sensitive borrower and merchant records.
3.  **Exfiltration & C2:** The attacker uses a **Malware Staging** tactic (Workstream A) to establish a Command & Control channel and exfiltrates the stolen data via high-volume outbound POSTs to a direct external IP.

---

## 📂 Directory Map
*   [**1 Network/**](./1%20Network/) - PCAP triage notes, Hancitor/Trickbot forensics, and IOC lists.
*   [**2 Webapp/**](./2%20Webapp/) - Web findings (SQLi, XSS, Command Injection) and remediation patches.
*   [**3 Synthesis/**](./3%20Synthesis/) - The master report connecting the dots into a Joint Threat Model.
*   [**Prompts/**](./Prompts/) - Reconstructed AI instructions used to drive the investigation.
*   [**Reflections/**](./Reflections/) - Personal technical insights and retrospective learnings from each lead.

---

## 🚀 Technical Highlights

### Defense-in-Depth Strategy
We proposed a layered security architecture to break the attack chain:
*   **Perimeter:** WAF for SQLi prevention and Egress Proxies for C2 blocking.
*   **Identity:** MFA enforcement to mitigate credential theft observed in PCAPs.
*   **Segmentation:** Micro-segmentation around Domain Controllers to stop lateral movement.

### Reproducibility
The webapp findings can be reproduced in under 15 minutes using the included Docker environment:
```bash
cd "2 Webapp"
docker compose -f env/docker-compose.yml up -d
```

---

## 🔑 Key Takeaway
> "Security analysis is not about finding issues—it is about proving them with evidence and connecting them into a coherent story." — *Project KAVACH Retrospective*

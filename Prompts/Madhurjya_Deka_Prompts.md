# LLM Prompt Log — Madhurjya Deka

## Prompt 1 — Understanding Network Anomaly Patterns

**Prompt:**
Explain how to identify C2 beaconing in PCAP traffic.

**Tool:**
ChatGPT

**Response Summary:**
Model explained beaconing patterns such as periodic intervals and DNS anomalies.

**What I did with it:**
Validated against PCAP evidence (Hancitor traffic showing periodic DNS queries).

**Where it was wrong / limitation:**
Did not consider encryption (TLS-based C2), which required manual analysis.

---

## Prompt 2 — SQL Injection Payloads

**Prompt:**
Generate SQL injection payloads for login bypass.

**Response Summary:**
Generated common payloads like `' OR 1=1--`

**What I did:**
Tested in DVWA and confirmed effectiveness.

**Limitation:**
Had to adjust encoding and syntax manually.

---

## Prompt 3 — IDOR Explanation

**Prompt:**
Explain IDOR vulnerability in simple terms.

**What I did:**
Used explanation to map vulnerability to business impact.

---

## Prompt 4 — Threat Modeling Approach

**Prompt:**
How to connect network and application findings in threat modeling?

**What I did:**
Used output as initial structure but built my own chaining logic.

**Limitation:**
Model did not reference actual PCAP artifacts.

---

## Prompt 5 — Defense-in-Depth Layers

**Prompt:**
List layers in defense-in-depth model.

**What I did:**
Mapped each layer to actual findings in project.

---

## Prompt 6 — IOC Identification

**Prompt:**
What qualifies as an IOC in network traffic?

**What I did:**
Compared with repository IOC list (IPs, DNS patterns).

---

## Prompt 7 — Executive Summary Drafting

**Prompt:**
Write executive summary for cybersecurity incident.

**What I did:**
Rewrote completely to remove jargon and align with board audience.

---

## Prompt 8 — Validation Prompt (Critical Thinking)

**Prompt:**
Are these two attack paths logically valid?

**What I did:**
Used this to challenge assumptions and refine threat model.

**Key Learning:**
LLM agreement does NOT equal correctness — validation required.

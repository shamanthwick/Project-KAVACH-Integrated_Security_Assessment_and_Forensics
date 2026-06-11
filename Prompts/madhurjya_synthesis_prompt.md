# Prompts used for Strategic Synthesis – Project KAVACH
**Author: Madhurjya Deka**

These prompts focus on correlating multi-layer evidence into a unified threat model.

## Prompt 1: Building the Joint Threat Model
> "I have two sets of findings: Network Forensics (Workstream A) showing C2/Exfiltration, and Web Security (Workstream B) showing SQLi and IDOR. Can you help me build a **Joint Threat Model**? I need to map a plausible attack path that starts with a portal compromise and ends with the large-scale data upload we saw in the RedLine Stealer capture. Please focus on evidence correlation."

## Prompt 2: Defense-in-Depth Strategy
> "Based on our findings, create a **Defense-in-Depth** table. I need to map layered controls (Identity, Perimeter, Segmentation, Application, Data) directly to our specific findings. For each control, include the estimated effort and the potential trade-off for the business."

## Prompt 3: Executive Readout Generation
> "Draft an **Executive Readout** for Project KAVACH. The goal is to answer one key question for leadership: 'Are these independent issues, or part of one attack?' Summarize the risk of a multi-stage attack and provide three clear, high-impact recommendations for security hardening."

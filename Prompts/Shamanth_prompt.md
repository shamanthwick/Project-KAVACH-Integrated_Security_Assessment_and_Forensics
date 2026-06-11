# Prompts used for Network Forensics – Project KAVACH
**Author: Shamanth**

These prompts reflect a "Forensics First" mindset, focusing on evidence extraction and architectural reasoning.

## Prompt 1: Initial PCAP Triage
> "I have several PCAP files related to a potential malware intrusion (Hancitor, Trickbot, and RedLine Stealer). Can you perform a triage pass on `2021-06-15-Hancitor-with-Ficker-Stealer-and-Cobalt-Strike.pcap`? I need to identify the infected internal host, the top external talkers, any suspicious DNS resolutions, and potential payload staging URLs. Please follow a baseline-vs-anomaly methodology."

## Prompt 2: Deep Dive into C2 Beaconing
> "In the Hancitor capture, I've noticed repeated requests from `10.0.0.134` to `5.252.177.17`. Can you analyze the frequency and URI patterns? I'm specifically looking for evidence of beaconing or loader activity. Also, check if there's any unusual protocol use, like plaintext HTTP over port 443."

## Prompt 3: Extracting Indicators of Compromise (IOCs)
> "Based on our analysis of the Hancitor, Trickbot, and RedLine captures, consolidate all malicious IPs, suspicious domains, and file hashes into a structured CSV format. Label them by type and source. Include the reconstructable object hashes for `15.bin` and `8mJm` that we found during triage."

## Prompt 4: Architecture Reasoning (Before vs. After)
> "I need to design a defense-in-depth architecture for 'Meridian FinServe' based on our findings. Currently, they have a flat network with direct outbound access. Create a Mermaid diagram showing the 'Before' state. Then, propose a 'After' architecture that includes an egress proxy, DNS sinkholing, protocol-aware inspection, and micro-segmentation to block the specific lateral movement we saw in the Trickbot case."

## Prompt 5: Final Forensics Report Generation
> "Draft a comprehensive Network Forensics Report for Project KAVACH. Structure it as a professional SOC/DFIR investigation. Include an executive summary, methodology, detailed findings for each malware family, a consolidated IOC table, and actionable containment recommendations. Ensure the tone is objective and distinguish between 'confirmed' and 'suspected' behaviors."

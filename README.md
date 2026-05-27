# Network Forensics and Architecture

Workstream A follows the investigation pipeline:

`Artifacts -> Reasoning -> Recommendation`

## Scope
This project analyzes all PCAPs currently present in this workstream directory:

- `2021-06-15-Hancitor-with-Ficker-Stealer-and-Cobalt-Strike.pcap`
- `2021-05-26-Trickbot-infection-with-Cobalt-Strike.pcap`
- `2024-10-23-Redline-Stealer-infection-traffic.pcap`
- `exfiltration.pcap`
- `ftp2.pcap`
- `nmap_null.pcap`

The Hancitor capture remains the primary malware-chain source. The other captures are used as supporting analogues to cover the full assignment behavior set: C2, scanning, exfiltration, credential abuse, lateral movement risk, and malware activity.

## Deliverables
| Workstream Step | File |
|---|---|
| A.1 Source PCAP | `Source-selection.md` |
| A.2 Triage Pass | `triage-notes.md` |
| A.3 Hypotheses | `hypotheses.md` |
| A.4 IOC List | `iocs.csv` |
| A.5 Architecture | `architecture/architecture-notes.md` |
| Figure 02 Pipeline | `architecture/figure-02-workstream-a-pipeline.mmd` |
| Before Diagram Source | `architecture/before.mmd` |
| After Diagram Source | `architecture/after.mmd` |

## Analysis Standard
The report is written as a SOC/DFIR investigation, not a packet-hunting checklist.

Each conclusion should show:

- the artifact observed in the PCAP
- the reasoning that connects it to attacker behavior
- the confidence level
- the recommendation or architecture control that follows

## Evidence Summary
| Capture | Confirmed Behaviors |
|---|---|
| Hancitor/Ficker/Cobalt Strike | suspicious DNS, payload downloads, C2 beaconing, suspected stealer POSTs, internal AD interaction |
| Trickbot/Cobalt Strike | Trickbot-style bot URIs, C2 beaconing, suspicious typo-squatted domain, SMB/LDAP/Kerberos lateral movement risk |
| RedLine Stealer | direct HTTP POSTs to `188.190.10.10:55123`, high-volume internal-to-external upload, public IP discovery |
| Exfiltration | DNS-heavy and TLS-heavy browsing/exfiltration analogue, useful for baseline-vs-anomaly reasoning |
| FTP transfer | plaintext FTP login, exposed password, file upload/download over passive data channels |
| Nmap NULL scan | 1,030 NULL probes across 1,000 destination ports from one host to another |

## Important Evidence Limits
Credential theft and exfiltration are strongly suspected in the Hancitor/Ficker capture and suspected in the Trickbot capture. RedLine shows direct upload behavior consistent with stealer exfiltration. FTP plaintext credentials are directly visible in `ftp2.pcap`.

Use phrasing such as:

- `suspected stealer upload`
- `credential theft capability`
- `exfiltration-like POST activity`

Avoid phrasing such as:

- `confirmed stolen credentials`
- `confirmed exfiltrated files`

unless additional host artifacts or decoded payload contents are added.

# Source PCAP Selection (A.1)

## Selection Decision
This workstream now analyzes six public captures. Two are full malware-chain captures and four are focused behavior captures used to prove the required categories in the assignment brief.

| Role | PCAP | Malware / Activity | Source |
|---|---|---|---|
| Primary updated capture | `2021-06-15-Hancitor-with-Ficker-Stealer-and-Cobalt-Strike.pcap` | Hancitor, Ficker Stealer, Cobalt Strike | `https://www.malware-traffic-analysis.net/2021/06/15/index.html` |
| Comparative capture | `2021-05-26-Trickbot-infection-with-Cobalt-Strike.pcap` | Trickbot, Cobalt Strike | `https://www.malware-traffic-analysis.net/2021/05/26/index.html` |
| Stealer/exfiltration capture | `2024-10-23-Redline-Stealer-infection-traffic.pcap` | RedLine Stealer infection traffic | Public malware PCAP corpus |
| Exfiltration baseline capture | `exfiltration.pcap` | DNS/TLS-heavy web exfiltration analogue | Public/sample PCAP corpus |
| Credential/file-transfer capture | `ftp2.pcap` | Plaintext FTP authentication and file transfer | Public/sample PCAP corpus |
| Scanning capture | `nmap_null.pcap` | Nmap NULL scan | Public/sample PCAP corpus |

The assignment asks for one defensible public capture. The Hancitor capture is treated as the primary source because it contains a compact intrusion chain: suspicious DNS, malware payload retrieval, credential-stealer activity, C2 beaconing, and internal Active Directory interaction. The additional captures strengthen the final report by giving explicit packet evidence for scanning, plaintext credential exposure, and high-volume upload behavior.

## Analogue Choice
Both captures are good analogues for the Meridian FinServe scenario because they show what a financial-sector incident responder would need to explain: externally staged malware, internal Windows domain activity, beaconing to command infrastructure, and weak egress controls.

The Hancitor capture is especially useful because the traffic moves quickly from initial HTTP activity into a stealer/C2 pattern:

- `10.0.0.134` performs public IP discovery through `api.ipify.org`.
- The host resolves suspicious domains such as `sciandwourgy.com`, `pariamarraire.ru`, `larn9kany.ru`, and `pospvisis.com`.
- It downloads payload-like files from `larn9kany.ru` and `162.244.83.95`.
- It sends repeated C2-style traffic to `5.252.177.17`, including `/activity` and `/submit.php?id=159302031`.
- It communicates with `10.0.0.7`, the likely domain controller for `kindness-trap.com`.

The Trickbot capture contributes a second family pattern:

- `10.5.26.132` uses Trickbot-style bot URI paths such as `/rob87/DESKTOP-OG16DGY.../90`.
- `10.5.26.4` later uses `/tot108/VICTORYPUNK-DC.../90`.
- `antivirusupdaty.com` and `5.199.162.3` show repeated beacon-like traffic.
- SMB, LDAP, and Kerberos activity support internal enumeration and possible lateral movement.

The newly added captures fill specific investigation gaps:

- RedLine shows a short, high-volume upload from `10.10.23.101` to `188.190.10.10:55123` with repeated HTTP `POST /` requests and public IP discovery through `api.ip.sb`.
- `exfiltration.pcap` shows how noisy DNS and TLS browsing can hide suspicious activity, making it useful for the assignment clue: baseline first, anomaly second.
- `ftp2.pcap` gives plaintext credential abuse evidence: `USER Administrator` followed by `PASS napier`, then `STOR 111.png` and `RETR manual.txt`.
- `nmap_null.pcap` gives clean reconnaissance evidence: `192.168.47.171` sends NULL TCP probes to 1,000 destination ports on `192.168.47.134`.

## Suitability Matrix
| Required Behavior | Best Evidence | Verdict |
|---|---|---|
| Command-and-control beaconing | Hancitor `/activity` to `5.252.177.17`; Trickbot `/rob87/`, `/tot108/`, and `antivirusupdaty.com`; RedLine POSTs to `188.190.10.10:55123`. | Present |
| Scanning / enumeration | `nmap_null.pcap`: `192.168.47.171` sends 1,030 NULL probes to 1,000 ports on `192.168.47.134`; Hancitor/Trickbot also show AD enumeration risk. | Present |
| Exfiltration | RedLine upload volume from `10.10.23.101` to `188.190.10.10`; Hancitor `/submit.php`; Trickbot `/as`; FTP `STOR 111.png`. | Present or strongly suspected depending on capture |
| Credential abuse | `ftp2.pcap` exposes `USER Administrator` and `PASS napier`; Hancitor/Ficker and Trickbot imply stealer capability. | Present |
| Lateral movement | Trickbot workstation-to-DC SMB and Hancitor internal DCERPC/LDAP/SMB to likely DCs. | Present |
| Malware activity | Hancitor payloads, Trickbot bot URIs, RedLine direct stealer POSTs. | Present |

## Selection Rationale
The Hancitor capture is the stronger final-report primary source because it has a clear source-to-impact chain in about 25 minutes:

1. Public IP check.
2. Suspicious DNS.
3. Malware staging and binary retrieval.
4. Repeated C2-style activity.
5. POST-based suspected stealer upload.
6. Internal Windows domain interactions.

The Trickbot capture should be used as supporting comparative evidence showing the same architectural weakness: flat internal access and direct outbound C2.

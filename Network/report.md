# Network Forensics Report

## Executive Summary

This report analyzes the packet-capture evidence in the `Network` workstream as a SOC/DFIR investigation. The primary case is the Hancitor/Ficker Stealer/Cobalt Strike capture, supported by Trickbot, RedLine Stealer, plaintext FTP, exfiltration-baseline, and Nmap NULL-scan captures.

The evidence shows a realistic intrusion pattern: public IP discovery, suspicious DNS, payload staging, repeated command-and-control traffic, suspected stealer upload behavior, internal Windows domain interaction, plaintext credential exposure, and reconnaissance. The strongest conclusion is that several internal hosts would require containment and investigation in an enterprise environment. The strongest architecture lesson is that Meridian FinServe needs deny-by-default egress, filtered DNS, protocol-aware inspection, segmentation around domain controllers, secure file transfer, scan detection, DLP, and centralized telemetry.

## Scope and Evidence Sources

| Evidence Source | Role in Report | Main Behavior |
|---|---|---|
| `2021-06-15-Hancitor-with-Ficker-Stealer-and-Cobalt-Strike.pcap` | Primary malware-chain capture | Hancitor-style infection, payload downloads, C2, suspected stealer POSTs, internal AD interaction |
| `2021-05-26-Trickbot-infection-with-Cobalt-Strike.pcap` | Comparative malware-chain capture | Trickbot-style bot URI paths, C2, suspicious TLS/SNI, SMB/LDAP/Kerberos risk |
| `2024-10-23-Redline-Stealer-infection-traffic.pcap` | Stealer/exfiltration evidence | Large outbound upload to a direct external IP on non-standard port `55123` |
| `exfiltration.pcap` | Baseline-vs-anomaly example | DNS/TLS-heavy browsing, analytics, CDN, and OCSP noise that can hide egress risk |
| `ftp2.pcap` | Credential and file-transfer evidence | Plaintext FTP login and upload/download activity |
| `nmap_null.pcap` | Reconnaissance evidence | Nmap NULL scan across 1,000 destination ports |

## Methodology

The analysis followed a baseline-first workflow:

1. Identify capture time bounds, internal hosts, external endpoints, and dominant protocols.
2. Separate normal Windows/domain, Microsoft update, web, DNS, TLS, OCSP, CDN, and browsing traffic from anomalous activity.
3. Review DNS, HTTP, TLS SNI, TCP conversations, endpoint statistics, file-transfer commands, and TCP flag patterns.
4. Convert packet artifacts into hypotheses, confidence levels, IOCs, and architecture recommendations.

This report avoids overclaiming. Packet metadata strongly supports malware and exfiltration-like behavior, but exact stolen contents are not proven unless the capture directly exposes them, as in the FTP credential case.

## Key Findings

### Finding 1: Hancitor-Style Infection and C2 on `10.0.0.134`

**Severity:** High  
**Confidence:** High for infection and C2; medium-to-high for Ficker/Cobalt Strike follow-on based on traffic pattern and dataset context.

The host `10.0.0.134` is the central infected workstation in the Hancitor capture. It performs public IP discovery, resolves suspicious domains, downloads payload-like files, repeatedly contacts C2-like endpoints, and interacts with internal domain services.

| Artifact | Evidence | Interpretation |
|---|---|---|
| Public IP discovery | `api.ipify.org` at frames `1`, `6`, and `750` | Common malware pre-flight behavior when followed by suspicious downloads and C2 |
| Suspicious DNS | `sciandwourgy.com`, `pariamarraire.ru`, `larn9kany.ru`, `pospvisis.com` | Domains do not match normal enterprise update or business activity |
| Payload staging | `/15.bin`, `/15s.bin`, `/f7h7jhhjbch.exe`, `/ZsDK`, `/8mJm` | Binary/object retrieval from suspicious external infrastructure |
| C2 polling | `25` `GET /load` requests and `1,689` `GET /activity` requests to `5.252.177.17` | Repeated simple URI pattern consistent with beaconing |
| Suspected stealer submit | Six `POST /submit.php?id=159302031` requests | Exfiltration-like POST behavior, but exact stolen content is not proven |
| Internal domain contact | SMB, LDAP, and DCERPC traffic with likely DC `10.0.0.7` | Creates lateral movement and credential exposure risk |

Notable reconstructed object hashes:

| Object | SHA256 |
|---|---|
| `15.bin` | `F9DD938DEF42C00911A7D3DA7A823BF3A9BA24998933763988492C711960B6FE` |
| `15s.bin` | `8673EC9943347CDD6C0AEB5B3A8DA3E20110C0E6C500AC7DE39994417440BE75` |
| `ZsDK` | `416639BFCF73C0CB7D06E7C70DA46BBCAF06B707114E2DDBD034AB40E9CCE293` |
| `8mJm` | `8B8F1CC2048E4C5C30E39D5E1C74B7E0AC67964AD029797DBEC609102CC57C9A` |

The `/8mJm` request is especially important because it is HTTP decoded on destination port `443`. A control stack that assumes port `443` always means encrypted HTTPS would miss this.

### Finding 2: Trickbot-Style C2 and Internal Expansion Risk

**Severity:** High  
**Confidence:** High for Trickbot-style C2; medium-to-high for Cobalt Strike follow-on.

The Trickbot capture shows `10.5.26.132` behaving as an infected workstation and `10.5.26.4` appearing as a domain-controller candidate later involved in suspicious C2-like traffic.

| Artifact | Evidence | Interpretation |
|---|---|---|
| Infected workstation | `10.5.26.132` / `DESKTOP-OG16DGY` | Primary suspicious internal host |
| Domain-controller candidate | `10.5.26.4` / `VictoryPunk-DC` | Critical host involved in internal and suspicious external communication |
| Trickbot bot URI | `/rob87/DESKTOP-OG16DGY_W617601.../90` | Bot identifier embedded in URI |
| Later suspicious host URI | `/tot108/VICTORYPUNK-DC_W617601.../90` | Suggests expansion or DC involvement |
| Suspicious domain | `antivirusupdaty.com` resolving/contacted through `5.199.162.3` | Typo-squatted security/update-like name |
| Beacon/upload paths | Repeated `GET /logo?hour=true` and `POST /as` | C2-like beacon and suspected upload channel |
| Lateral movement path | SMB conversation `10.5.26.132:49539 <-> 10.5.26.4:445` | Workstation-to-DC reachability creates domain compromise risk |

This capture supports the same architecture conclusion as the Hancitor case: compromised workstations should not have broad direct access to domain-controller services, and C2 should not be allowed to exit directly from endpoints.

### Finding 3: RedLine Stealer-Like High-Volume Upload

**Severity:** High  
**Confidence:** High for stealer-like upload behavior.

The RedLine capture is short, but the traffic pattern is clear. Host `10.10.23.101` sends nearly all significant traffic to `188.190.10.10:55123`.

| Artifact | Evidence | Interpretation |
|---|---|---|
| Dominant conversation | `10.10.23.101:49697 <-> 188.190.10.10:55123` | Central event in the capture |
| Upload volume | About `5,783 kB` sent from internal host to external IP | Strong internal-to-external exfiltration pattern |
| HTTP POSTs | Frames `7`, `14`, `4387`, and `8786` show `POST /` | Repeated direct-IP POSTs |
| Non-standard port | Destination port `55123` | Bypasses controls focused only on ports `80` and `443` |
| Public IP discovery | `api.ip.sb` DNS and TLS SNI | Common stealer staging behavior |

The exact uploaded content is not decoded in this report. The traffic still warrants containment because it is not consistent with normal browsing or update activity.

### Finding 4: Plaintext FTP Exposes Credentials and Files

**Severity:** High  
**Confidence:** High.

The `ftp2.pcap` capture directly exposes credentials and file movement over FTP.

| Packet(s) | Evidence | Risk |
|---:|---|---|
| `5`, `38` | `USER Administrator` | Privileged username exposed in cleartext |
| `7`, `40` | `PASS napier` | Password exposed in cleartext |
| `8`, `41` | Successful login response | Credentials are valid in the session |
| `50-123` | `STOR 111.png` | File upload |
| `166-177` | `RETR manual.txt` | File download |

This is direct evidence that legacy FTP must be removed from sensitive enterprise workflows. The issue is not only encryption; it is also privileged credential use, unmanaged file movement, and weak auditability.

### Finding 5: Nmap NULL Scan Indicates Reconnaissance

**Severity:** Medium to High  
**Confidence:** High for scanning behavior; authorization unknown.

The `nmap_null.pcap` capture shows `192.168.47.171` scanning `192.168.47.134`.

| Artifact | Evidence | Interpretation |
|---|---|---|
| Scanner | `192.168.47.171` | Source of probe burst |
| Target | `192.168.47.134` | Host being mapped |
| NULL probes | `1,030` packets with `tcp.flags == 0x000` | Strong signature of NULL scanning |
| Port spread | `1,000` unique destination ports | Broad reconnaissance |
| Duration | About `2.6` seconds | Dense automated scan |

This should be validated against an approved vulnerability-scan calendar. If unauthorized, it is an internal reconnaissance event.

### Finding 6: DNS/TLS Noise Can Hide Egress Risk

**Severity:** Medium  
**Confidence:** Medium.

The `exfiltration.pcap` capture is valuable because much of it resembles normal web noise: DNS, TLS, OCSP, CDN, advertising, and analytics traffic. The main host is `10.0.2.15`, using `1.1.1.1` for DNS and contacting domains such as `dt.adsafeprotected.com`, `www.redditstatic.com`, `ophan.theguardian.com`, `assets.guim.co.uk`, and analytics/ad infrastructure.

The lesson is that high-volume DNS/TLS traffic is not automatically malicious. Detection should compare destination reputation, query frequency, upload direction, new destination history, SNI/certificate metadata, and user/device baseline. This supports deploying DNS analytics, TLS metadata logging, and DLP rather than relying on domain lists alone.

## Consolidated IOC Summary

| Type | IOC | Source / Meaning |
|---|---|---|
| Internal host | `10.0.0.134` | Hancitor infected workstation |
| Internal host | `10.0.0.7` | Likely domain controller contacted by infected Hancitor host |
| Internal host | `10.5.26.132` | Trickbot infected workstation |
| Internal host | `10.5.26.4` | Domain-controller candidate involved in suspicious traffic |
| Internal host | `10.10.23.101` | RedLine infected workstation candidate |
| External IP | `5.252.177.17` | Hancitor/Ficker C2-like endpoint |
| External IP | `194.226.60.15` | `pariamarraire.ru` POST endpoint |
| External IP | `8.209.119.208` | `larn9kany.ru` payload host |
| External IP | `162.244.83.95` | Payload host serving `/ZsDK` and `/8mJm` |
| External IP | `188.190.10.10` | RedLine direct-IP upload endpoint |
| External IP | `5.199.162.3` | `antivirusupdaty.com` infrastructure |
| Domain | `sciandwourgy.com` | Suspicious Hancitor sequence domain |
| Domain | `pariamarraire.ru` | Hancitor-style POST/check-in domain |
| Domain | `larn9kany.ru` | Payload domain |
| Domain | `pospvisis.com` | Suspicious Hancitor sequence domain |
| Domain | `antivirusupdaty.com` | Suspicious typo-squatted security/update-like domain |
| URI | `/8/forum.php` | Hancitor repeated POST endpoint |
| URI | `/load` | Hancitor loader/C2 polling |
| URI | `/activity` | Hancitor high-frequency beacon path |
| URI | `/submit.php?id=159302031` | Suspected Hancitor/Ficker stealer submit path |
| URI | `/rob87/.../90` | Trickbot-style bot path |
| URI | `/tot108/.../90` | Trickbot-style bot path from DC candidate |
| URI | `/logo?hour=true` | Trickbot/C2-like repeated GET |
| URI | `/as` | Trickbot/C2-like POST |
| Credential | `Administrator` / `napier` | Plaintext FTP credentials observed in `ftp2.pcap` |

## Attack Narrative

The Hancitor capture begins with public IP discovery and suspicious DNS from `10.0.0.134`. The host then downloads payload-like files from `larn9kany.ru` and `162.244.83.95`, including an executable path and HTTP-over-443 traffic. After staging, the host repeatedly contacts `5.252.177.17` using simple loader and activity paths, then sends suspected stealer POSTs to `/submit.php?id=159302031`. Internal SMB, LDAP, and DCERPC traffic to a likely domain controller raises the risk that the compromise could move from workstation infection to domain exposure.

The Trickbot capture provides a second example of the same enterprise weakness. Bot-style URI paths include workstation and domain-controller hostnames, and later suspicious communication appears from a domain-controller candidate. SMB, LDAP, Kerberos, and external C2-like traffic exist close together, which means endpoint compromise cannot be treated as isolated if the network allows broad domain access.

The RedLine capture compresses the exfiltration problem into a short window: one host sends a high volume of data to a direct external IP on an unusual port. The FTP and Nmap captures provide clear examples of supporting risks: plaintext credential/file movement and rapid internal reconnaissance.

## Impact Assessment

| Impact Area | Assessment |
|---|---|
| Endpoint compromise | High. Multiple captures show infected-workstation behavior, payload staging, and C2. |
| Credential risk | High. FTP exposes credentials directly; stealer families imply credential theft capability. |
| Data exfiltration risk | High. RedLine shows high-volume upload; Hancitor and Trickbot show suspected POST-based upload paths. |
| Domain compromise risk | High. Infected hosts communicate with domain-controller services over SMB, LDAP, Kerberos, and DCERPC. |
| Reconnaissance risk | Medium to High. NULL scan shows fast service enumeration across 1,000 ports. |
| Detection maturity gap | High. Several behaviors would bypass controls that rely only on destination port or manual packet review. |

## Recommended Response Actions

### Immediate Containment

1. Isolate suspected infected hosts: `10.0.0.134`, `10.5.26.132`, `10.5.26.4`, and `10.10.23.101`.
2. Block or sinkhole malicious and suspicious destinations listed in the IOC summary.
3. Disable FTP access and rotate the exposed `Administrator` credential from `ftp2.pcap`.
4. Search proxy, DNS, EDR, firewall, and authentication logs for the listed domains, IPs, URIs, and file hashes.
5. Validate whether the Nmap NULL scan source `192.168.47.171` was authorized.

### Investigation and Eradication

1. Acquire memory, disk, EDR timeline, scheduled tasks, autoruns, browser artifacts, and PowerShell history from affected hosts.
2. Review domain-controller logs for authentication anomalies, service creation, SMB admin shares, LDAP enumeration, and Kerberos events.
3. Hunt for repeated URI paths: `/activity`, `/load`, `/submit.php`, `/rob87/`, `/tot108/`, `/logo?hour=true`, and `/as`.
4. Identify any additional hosts contacting direct external IPs over non-standard ports.
5. Use recovered payload hashes to search endpoint and file-scanning telemetry.

### Recovery

1. Reimage confirmed infected endpoints rather than relying only on file deletion.
2. Reset credentials used on affected hosts, prioritizing privileged and domain accounts.
3. Review firewall and proxy rules before restoring normal network access.
4. Confirm no recurring C2 traffic after containment and credential rotation.

## Architecture Recommendations

The PCAP evidence points to a weak segment with direct outbound access, limited protocol validation, broad workstation-to-domain-controller reachability, plaintext FTP, and insufficient scan/exfiltration detection.

Recommended target controls:

| Control | Why It Matters |
|---|---|
| Explicit egress proxy with deny-by-default outbound rules | Blocks direct C2 to external IPs and suspicious domains |
| Default deny for non-standard outbound ports | Prevents RedLine-style upload to `188.190.10.10:55123` |
| Protocol-aware inspection | Detects HTTP-over-443 rather than trusting port numbers |
| Filtered DNS with sinkhole and threat intelligence | Blocks suspicious domains before payload retrieval |
| DNS/TLS metadata analytics | Separates normal CDN/OCSP/ad traffic from suspicious egress patterns |
| Egress DLP and upload thresholds | Detects large outbound uploads from a single internal host |
| Segmentation around domain controllers | Reduces workstation-to-DC lateral movement paths |
| Managed SFTP/MFT replacement for FTP | Prevents plaintext credential exposure and unmanaged file movement |
| IDS/NDR scan detection | Detects NULL, FIN, Xmas, SYN sweeps, and high unique-port counts |
| Proxy file scanning and sandboxing | Blocks staged payloads such as `.bin` objects and EXE downloads |
| SIEM correlation and case workflow | Connects DNS, proxy, firewall, EDR, DLP, and packet evidence |

## Suggested Detection Logic

| Detection | Trigger |
|---|---|
| HTTP on port `443` | Protocol decoder identifies plaintext HTTP on destination port `443` |
| Repeated beacon URI | One host requests the same URI path repeatedly within a short window |
| Public IP discovery plus payload download | `api.ipify.org` or `api.ip.sb` followed by suspicious binary retrieval |
| Direct-IP POST to unusual port | HTTP POST to an external IP on a non-standard service port |
| Large outbound upload | One internal host sends unusually high bytes to a new external destination |
| FTP credential exposure | FTP `USER` and `PASS`, especially privileged accounts |
| NULL scan | Many TCP packets with no flags set across many destination ports |
| Workstation-to-DC plus C2 | SMB/LDAP/Kerberos/DCERPC to DC plus external suspicious traffic from the same host |
| Suspicious update/security typo domain | Domains resembling antivirus or update services but not belonging to approved vendors |

## Evidence Limits

This report distinguishes between confirmed packet artifacts and inferred attacker objectives. Plaintext FTP credentials are directly confirmed. The NULL scan is directly confirmed. RedLine upload behavior is strongly supported by volume, direction, direct-IP POSTs, and non-standard port usage, but exact uploaded content is not decoded. Hancitor and Trickbot C2 behavior is strongly supported by domains, URI paths, repetition, endpoint concentration, and payload downloads, but exact stolen data and full Cobalt Strike configuration are not extracted here.

## Final Conclusion

The `Network` evidence supports a high-confidence conclusion that Meridian FinServe's modeled environment is vulnerable to common intrusion chains: malware staging, C2 beaconing, credential theft, exfiltration-like uploads, lateral movement risk, plaintext credential exposure, and reconnaissance. The current architecture should be treated as insufficient because it allows direct outbound communication, trusts ports too heavily, permits legacy cleartext transfer, and gives compromised workstations too much access to domain services.

The recommended hardened design is an egress-controlled, DNS-filtered, protocol-aware, segmented environment with secure file transfer, scan detection, DLP, EDR, IDS/NDR, and SIEM correlation. These controls directly map to the packet evidence and would reduce both the likelihood of successful compromise and the time required to detect and contain it.

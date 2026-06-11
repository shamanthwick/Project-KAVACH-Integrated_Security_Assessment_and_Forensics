# Triage Pass Notes (A.2)

## Method
Triage followed the `wireshark-analysis` workflow: protocol hierarchy, endpoints, TCP conversations, DNS queries, HTTP requests, TLS SNI, and reconstructable HTTP objects. The goal is baseline first, anomaly second.

## Capture 1: Hancitor with Ficker Stealer and Cobalt Strike

### Time Bounds
| Field | Value |
|---|---|
| File | `2021-06-15-Hancitor-with-Ficker-Stealer-and-Cobalt-Strike.pcap` |
| Frames | `20,715` |
| Bytes | `3,993,853` |
| Start | `2021-06-15 20:38:40 +0530` |
| End | `2021-06-15 21:03:56 +0530` |
| Duration | About `25 minutes 16 seconds` |

### Protocol Summary
| Protocol | Frames | Bytes | Triage Meaning |
|---|---:|---:|---|
| TCP | `19,815` | `3,947,111` | Dominant transport for HTTP, TLS, LDAP, SMB, and DCERPC. |
| HTTP | `3,482` | `1,053,839` | Main malware staging and C2 channel. |
| TLS | `473` | `285,491` | Mostly Microsoft baseline traffic; no suspicious SNI was observed for `5.252.177.17`. |
| DNS | `85` | `11,638` | Small but high-value: suspicious domains appear immediately. |
| DCERPC | `136` | `44,210` | Internal Windows domain interaction. |
| LDAP | `72` | `24,016` | Active Directory baseline/discovery against `kindness-trap.com`. |
| SMB/SMB2 | `60` | `14,143` | Internal Windows file/service activity. |
| ARP | `807` | `33,894` | Local network baseline noise. |

### Top Endpoints
| IP | Packets | Bytes | Role |
|---|---:|---:|---|
| `10.0.0.134` | `19,908` | `3,959 kB` | Infected internal workstation. |
| `5.252.177.17` | `15,839` | `1,808 kB` | Dominant C2/beacon endpoint. |
| `185.66.15.228` | `1,362` | `732 kB` | External HTTP server with long transfer. |
| `10.0.0.7` | `594` | `156 kB` | Likely domain controller: `Kindness-DC`. |
| `162.244.83.95` | `431` | `443 kB` | Payload server for `/ZsDK` and `/8mJm`. |
| `8.209.119.208` | `284` | `290 kB` | Payload host for `larn9kany.ru`. |
| `194.226.60.15` | `139` | `15 kB` | `pariamarraire.ru` POST endpoint. |

### Top Conversations
| Conversation | Frames | Bytes | Interpretation |
|---|---:|---:|---|
| `10.0.0.134:52892 <-> 185.66.15.228:80` | `1,343` | `731 kB` | Long HTTP transfer. |
| `10.0.0.134:52882 <-> 8.209.119.208:80` | `284` | `290 kB` | Payload retrieval from `larn9kany.ru`. |
| `10.0.0.134:52884 <-> 162.244.83.95:443` | `217` | `222 kB` | HTTP decoded on port `443`; suspicious payload-like path. |
| `10.0.0.134:52883 <-> 162.244.83.95:80` | `214` | `221 kB` | Suspicious payload-like path. |
| Many `10.0.0.134:* <-> 5.252.177.17:443` | small repeated sessions | high frequency | Beaconing/C2 pattern. |

### DNS Baseline and Anomalies
Normal baseline:

- Microsoft update and telemetry domains: `fe2cr.update.microsoft.com`, `v10.events.data.microsoft.com`, `login.microsoftonline.com`.
- Internal AD names: `_ldap._tcp...kindness-trap.com`, `Kindness-DC.kindness-trap.com`, `wpad.kindness-trap.com`.

Suspicious early DNS:

| Frame | Source | Query | Assessment |
|---:|---|---|---|
| `1` | `10.0.0.134` | `api.ipify.org` | Public IP discovery at capture start. |
| `10-14` | `10.0.0.134` | `sciandwourgy.com` | Repeated suspicious lookup. |
| `16` | `10.0.0.134` | `pariamarraire.ru` | Hancitor-style POST endpoint. |
| `25`, `838` | `10.0.0.134` | `larn9kany.ru` | Payload-hosting domain. |
| `840` | `10.0.0.134` | `pospvisis.com` | Suspicious domain in same early burst. |

### HTTP Findings
| Frame / Range | Flow | Request | User-Agent | Assessment |
|---|---|---|---|---|
| `6`, `750` | `10.0.0.134 -> api.ipify.org` | `GET /`, `GET /?format=xml` | IE-like strings | Public IP discovery. |
| `21-20090` | `10.0.0.134 -> pariamarraire.ru` | `POST /8/forum.php` | `Mozilla/5.0 ... Trident/7.0` | Repeated Hancitor-style check-in, 13 POSTs. |
| `30` | `10.0.0.134 -> larn9kany.ru` | `GET /15.bin` | IE-like string | Reconstructable payload-like object. |
| `34` | `10.0.0.134 -> larn9kany.ru` | `GET /15s.bin` | IE-like string | Reconstructable payload-like object. |
| `43` | `10.0.0.134 -> larn9kany.ru` | `GET /f7h7jhhjbch.exe` | IE-like string | EXE download; Windows Defender blocked hash access during extraction. |
| `39` | `10.0.0.134 -> 162.244.83.95` | `GET /ZsDK` | MSIE 10 style | Reconstructable payload-like object. |
| `72` | `10.0.0.134 -> 162.244.83.95:443` | `GET /8mJm` | MSIE style | HTTP over port `443`; reconstructable payload-like object. |
| `678-20237` | `10.0.0.134 -> 5.252.177.17` | `GET /load` | `MSIE 9.0 ... yie9` | Repeated loader/C2 polling, 25 requests. |
| `749-20713` | `10.0.0.134 -> 5.252.177.17:443` | `GET /activity` | `MSIE 7.0 ... Windows NT 5.1` | Heavy beaconing, 1,689 requests. |
| `2206-3922` | `10.0.0.134 -> 5.252.177.17:443` | `POST /submit.php?id=159302031` | same MSIE 7 string | Suspected stealer upload/C2 submit, 6 POSTs. |

### Reconstructable Object Hashes
| Object | SHA256 | Notes |
|---|---|---|
| `15.bin` | `F9DD938DEF42C00911A7D3DA7A823BF3A9BA24998933763988492C711960B6FE` | From `larn9kany.ru`. |
| `15s.bin` | `8673EC9943347CDD6C0AEB5B3A8DA3E20110C0E6C500AC7DE39994417440BE75` | From `larn9kany.ru`. |
| `ZsDK` | `416639BFCF73C0CB7D06E7C70DA46BBCAF06B707114E2DDBD034AB40E9CCE293` | From `162.244.83.95`. |
| `8mJm` | `8B8F1CC2048E4C5C30E39D5E1C74B7E0AC67964AD029797DBEC609102CC57C9A` | From `162.244.83.95:443`. |
| `f7h7jhhjbch.exe` | Not captured safely | Export was detected by Windows Defender before hash access. Treat as malicious download evidence by URI and packet metadata, not by hash. |

JA3/JA3S fingerprints were not added because the local `tshark` workflow did not expose JA3 fields in the extracted packet fields. The report therefore relies on URI, host, domain, User-Agent, packet range, and file-hash evidence.

## Capture 2: Trickbot Infection with Cobalt Strike

### Time Bounds
| Field | Value |
|---|---|
| File | `2021-05-26-Trickbot-infection-with-Cobalt-Strike.pcap` |
| Frames | `26,644` |
| Bytes | `17,570,609` |
| Start | `2021-05-27 01:53:19 +0530` |
| End | `2021-05-27 03:13:28 +0530` |
| Duration | About `1 hour 20 minutes` |

### Protocol Summary
| Protocol | Frames | Bytes | Triage Meaning |
|---|---:|---:|---|
| TCP | `21,901` | `17,218,725` | Dominant transport for SMB, LDAP, HTTP, TLS, and C2. |
| SMB/SMB2 over NBSS | `2,659` | `868,899` | Internal Windows file sharing and lateral movement path. |
| TLS | `1,240` | `748,167` | Includes repeated SNI for `antivirusupdaty.com`. |
| DNS | `458` | `62,914` | AD lookups plus suspicious C2 domain resolution. |
| HTTP | `289` | `195,764` | Trickbot URI patterns and beaconing. |
| LDAP | `274` | `103,046` | Active Directory discovery/query activity. |
| Kerberos | `72` | `20,313` | Domain authentication traffic. |

### Key Endpoints
| IP | Packets | Bytes | Role |
|---|---:|---:|---|
| `10.5.26.4` | `17,004` | `11 MB` | Domain controller candidate: `VictoryPunk-DC`; later suspicious C2 source. |
| `10.5.26.132` | `10,880` | `8,292 kB` | Infected workstation: `DESKTOP-OG16DGY`. |
| `192.236.155.230` | `6,372` | `7,493 kB` | External HTTP payload/resource server. |
| `5.199.162.3` | `2,453` | `1,431 kB` | Infrastructure for `antivirusupdaty.com`. |
| `78.186.110.14` | `2,246` | `2,361 kB` | Suspicious external TCP conversation with `10.5.26.4`. |
| `91.83.88.122` | `2,037` | `2,347 kB` | Suspicious external TLS conversation with `10.5.26.132`. |

### HTTP/DNS/TLS Findings
| Frame / Range | Evidence | Assessment |
|---|---|---|
| `809`, `6801`, `10957`, `14873` | Queries to `api.ipify.org`, `api.ip.sb`, `ipinfo.io`, `myexternalip.com` | Public IP discovery. |
| `2910`, `3045` | `POST /rob87/DESKTOP-OG16DGY_W617601.D5D933B2E59558F3F46D9130BB091D98/90` to `36.95.27.243` and `103.102.220.50` | Trickbot-style bot identifier. |
| `13210`, `14613` | `POST /tot108/VICTORYPUNK-DC_W617601.B5EFB19D6BB313B551F3FB5897A283BE/90` | C2-like traffic from DC candidate. |
| `3052-17677` | GETs for `/images/redbutton.png`, `/ico/viodifot`, `/images/cutscroll.png` from `192.236.155.230` | Suspicious resource/payload retrieval. |
| `24097` | DNS query for `antivirusupdaty.com` | Typo-squatted suspicious domain. |
| `24102-26581` | TLS SNI `antivirusupdaty.com` to `5.199.162.3` | Repeated C2-like TLS. |
| `23179-26639` | Repeated `GET /logo?hour=true` and `POST /as` to `antivirusupdaty.com` | Beaconing and suspected upload channel. |

## Capture 3: RedLine Stealer Infection Traffic

### Time Bounds
| Field | Value |
|---|---|
| File | `2024-10-23-Redline-Stealer-infection-traffic.pcap` |
| Frames | `8,807` |
| Bytes | `6,064,544` |
| Start | `2024-10-24 00:45:32 +0530` |
| End | `2024-10-24 00:45:57 +0530` |
| Duration | About `25.7 seconds` |

### Triage Findings
| Evidence | Value | Assessment |
|---|---|---|
| Dominant conversation | `10.10.23.101:49697 <-> 188.190.10.10:55123` | `8,787` frames and about `6,057 kB`; this is the capture's central event. |
| Directionality | `10.10.23.101 -> 188.190.10.10` sends about `5,783 kB` | Strong internal-to-external upload pattern. |
| HTTP requests | Frames `7`, `14`, `4387`, `8786` show `POST /` to `188.190.10.10:55123` | Repeated direct-IP POSTs on a non-standard service port. |
| Public IP discovery | Frame `21` queries `api.ip.sb`; frame `26` uses TLS SNI `api.ip.sb` | Common stealer pre-exfil behavior. |
| Internal source | `10.10.23.101` | Infected workstation candidate. |

### Baseline vs Anomaly
The normal baseline is very small: one workstation, a gateway/DNS resolver, and a public IP service. The anomaly is the size and direction of the main flow: almost the entire capture is upload traffic from `10.10.23.101` to `188.190.10.10:55123`.

## Capture 4: Exfiltration PCAP

### Time Bounds
| Field | Value |
|---|---|
| File | `exfiltration.pcap` |
| Frames | `7,919` |
| Bytes | `2,907,391` |
| Start | `2018-11-27 13:29:46 +0530` |
| End | `2018-11-27 13:30:56 +0530` |
| Duration | About `1 minute 10 seconds` |

### Triage Findings
| Evidence | Value | Assessment |
|---|---|---|
| Internal host | `10.0.2.15` | Single client generating all significant external activity. |
| DNS resolver | `1.1.1.1` | `1,741` DNS packets; DNS is a loud baseline channel. |
| Top DNS queries | `dt.adsafeprotected.com`, `www.redditstatic.com`, `ophan.theguardian.com`, `i.guim.co.uk`, `assets.guim.co.uk` | Ad, analytics, media, and web-content baseline dominates. |
| Top TLS SNI | `dt.adsafeprotected.com`, `pixel.onaudience.com`, `ophan.theguardian.com`, `secure.adnxs.com`, `cdn.krxd.net` | Third-party ad/analytics TLS traffic creates noisy cover. |
| HTTP requests | OCSP POSTs to `ocsp.comodoca.com`, `status.rapidssl.com`, `ocsp.digicert.com`, `ocsp.trustwave.com` | Certificate validation baseline, not malicious by itself. |

### Baseline vs Anomaly
This capture is useful because most traffic looks like web browsing telemetry, ads, analytics, OCSP, and CDN activity. A SOC analyst should not label every third-party domain as malicious. The investigative value is showing how DNS/TLS volume can hide unusual egress unless query frequency, destination reputation, and upload direction are compared against baseline.

## Capture 5: FTP Credential and File Transfer

### Time Bounds
| Field | Value |
|---|---|
| File | `ftp2.pcap` |
| Frames | `182` |
| Bytes | `92,830` |
| Start | `2013-09-01 01:54:40 +0530` |
| End | `2013-09-01 01:55:30 +0530` |
| Duration | About `49.8 seconds` |

### Triage Findings
| Evidence | Packet(s) | Assessment |
|---|---:|---|
| FTP server banner | `4`, `37` | `Microsoft FTP Service` on `192.168.47.134`. |
| Username | `5`, `38` | `USER Administrator` exposed in cleartext. |
| Password | `7`, `40` | `PASS napier` exposed in cleartext. |
| Successful login | `8`, `41` | `230 User Administrator logged in.` |
| Upload | `50-123` | `STOR 111.png` followed by passive data transfer. |
| Download | `166-177` | `RETR manual.txt` followed by passive data transfer. |

### Baseline vs Anomaly
FTP control traffic is readable by design. The anomaly is not that FTP exists; the risk is privileged cleartext authentication plus file movement. In Meridian FinServe, this would be unacceptable for administrative or sensitive data flows.

## Capture 6: Nmap NULL Scan

### Time Bounds
| Field | Value |
|---|---|
| File | `nmap_null.pcap` |
| Frames | `2,036` |
| Bytes | `116,969` |
| Start | `2014-01-05 21:02:21 +0530` |
| End | `2014-01-05 21:02:24 +0530` |
| Duration | About `2.6 seconds` |

### Triage Findings
| Evidence | Value | Assessment |
|---|---|---|
| Scanner | `192.168.47.171` | Generates the probe burst. |
| Target | `192.168.47.134` | Receives broad port sweep. |
| NULL probes | `1,030` packets with `tcp.flags == 0x000` | Strong signature of NULL scan behavior. |
| Unique destination ports | `1,000` | Broad reconnaissance rather than normal application use. |
| Sample ports | `3306`, `3389`, `445`, `8080`, `135`, `21`, `80`, `443`, `25`, `23`, `5900`, `53`, `22`, `5432` | Common service discovery across database, remote access, file sharing, web, mail, DNS, SSH. |

### Baseline vs Anomaly
Normal client traffic does not send zero-flag TCP probes across 1,000 destination ports in under three seconds. This capture cleanly satisfies the scanning requirement.

## Cross-Capture Anomaly Pattern
Across all PCAPs, the baseline is normal Windows domain behavior, Microsoft update/telemetry, web browsing DNS/TLS, OCSP, and ordinary service banners. The anomalies only become meaningful against that baseline:

- Public IP discovery appears before or during malware activity.
- Suspicious domains and direct IP HTTP requests appear early.
- HTTP is decoded on port `443`, bypassing assumptions that port `443` means TLS.
- C2 uses simple repeated URI patterns, not complex web workflows.
- Internal Windows domain traffic appears near malware activity, creating lateral movement risk.
- RedLine shows high-volume upload to a direct external IP on non-standard port `55123`.
- FTP exposes privileged credentials and file movement in cleartext.
- Nmap NULL scanning produces a short, dense burst of zero-flag probes across 1,000 ports.

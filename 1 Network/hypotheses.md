# Hypothesis-Driven Deep Dive (A.3)

## Hypothesis 1: Hancitor Infection Followed by Ficker Stealer and Cobalt Strike Activity
**Statement:** `10.0.0.134` is infected through a Hancitor chain, retrieves follow-on payloads, performs stealer-style POST activity, and then maintains C2-like communication.

### What Would Confirm It
- Suspicious DNS before payload activity.
- Payload-like HTTP downloads.
- Repeated beaconing to a small set of external endpoints.
- POST traffic that suggests data submission or stealer upload.
- Internal domain contact after initial infection.

### Evidence Supporting
- DNS frames `10-14` repeatedly query `sciandwourgy.com`; frames `16`, `25`, and `840` query `pariamarraire.ru`, `larn9kany.ru`, and `pospvisis.com`.
- Frame `21` posts to `pariamarraire.ru /8/forum.php`; the same endpoint repeats through frame `20090`.
- Frames `30`, `34`, and `43` retrieve `/15.bin`, `/15s.bin`, and `/f7h7jhhjbch.exe` from `larn9kany.ru`.
- Frames `39` and `72` retrieve `/ZsDK` and `/8mJm` from `162.244.83.95`, including HTTP decoded on destination port `443`.
- `5.252.177.17` dominates the capture with `15,839` packets.
- `/activity` appears `1,689` times from `10.0.0.134` to `5.252.177.17:443`.
- `/submit.php?id=159302031` appears as six POSTs between frames `2206` and `3922`.
- `10.0.0.134` communicates with likely domain controller `10.0.0.7` over SMB, LDAP, and DCERPC.

### Evidence Against
- The PCAP metadata does not prove the exact stolen data content.
- Cobalt Strike configuration was not decrypted or extracted from TLS/HTTP payloads here.
- Some Microsoft TLS and AD traffic is normal baseline and must not be treated as malicious by itself.

### Verdict
**Confirmed with high confidence for Hancitor-style infection and C2.** Ficker Stealer activity is **strongly suspected** because of the capture label and repeated `/submit.php` POSTs, but exact stolen content is not proven from packet metadata. Cobalt Strike follow-on is **medium confidence** without decrypted beacon configuration.

---

## Hypothesis 2: Trickbot Infection with Cobalt Strike Follow-On Activity
**Statement:** `10.5.26.132` is infected with Trickbot, performs C2 over HTTP on port `443`, interacts with internal Windows domain services, and activity expands toward `10.5.26.4`.

### What Would Confirm It
- Trickbot-style bot identifiers in URI paths.
- Multiple external C2 destinations.
- Internal SMB/LDAP/Kerberos traffic near the suspicious sequence.
- Later C2-like traffic from another internal host.

### Evidence Supporting
- Frames `2910` and `3045` show `10.5.26.132` POSTing `/rob87/DESKTOP-OG16DGY_W617601.D5D933B2E59558F3F46D9130BB091D98/90` to `36.95.27.243` and `103.102.220.50`.
- Frames `13210` and `14613` show `10.5.26.4` POSTing `/tot108/VICTORYPUNK-DC_W617601.B5EFB19D6BB313B551F3FB5897A283BE/90`.
- DNS frame `24097` and TLS SNI frames `24102-26581` show `10.5.26.4` contacting `antivirusupdaty.com`.
- Frames `23179-26639` show repeated `GET /logo?hour=true` to `antivirusupdaty.com`.
- Frames such as `23758`, `24228`, `24246`, `24271`, `24296`, `24321`, `24966`, `24995`, `25037`, `25084`, and `25525` show repeated `POST /as`.
- The SMB conversation `10.5.26.132:49539 <-> 10.5.26.4:445` contains `2,537` frames and `1,790 kB`.

### Evidence Against
- Exact command output, credential material, or exfiltrated files are not decoded.
- Cobalt Strike is supported by the capture label and later beacon pattern, but this analysis does not extract a full beacon profile.

### Verdict
**Confirmed with high confidence for Trickbot-style C2 and internal expansion risk.** Cobalt Strike follow-on is **medium-to-high confidence** because the capture label and later beaconing align with post-exploitation behavior.

---

## Hypothesis 3: Benign Software Update or Administrative Activity
**Statement:** The traffic is normal enterprise software updates, Active Directory behavior, or administrative tooling.

### What Would Confirm It
- Domains belong to known vendors or internal services.
- Traffic uses expected protocols: TLS on `443`, normal update paths, and consistent enterprise User-Agents.
- Internal SMB/LDAP/Kerberos activity is isolated from suspicious external events.

### Evidence Supporting
- Both captures contain normal Windows domain traffic.
- The Hancitor capture includes Microsoft update and telemetry domains such as `fe2cr.update.microsoft.com`, `v10.events.data.microsoft.com`, and `login.microsoftonline.com`.
- Internal DNS, LDAP, SMB, Kerberos, DCERPC, ARP, and WPAD can be normal in Windows environments.

### Evidence Refuting
- `larn9kany.ru`, `pariamarraire.ru`, `sciandwourgy.com`, and `pospvisis.com` do not fit enterprise update naming.
- Payload-like paths `/15.bin`, `/15s.bin`, `/f7h7jhhjbch.exe`, `/ZsDK`, and `/8mJm` are not normal update workflows.
- The Hancitor capture has `1,689` repeated `/activity` requests and six `/submit.php?id=159302031` POSTs to `5.252.177.17`.
- The Trickbot capture has bot-ID URI paths containing hostnames and hash-like values.
- Both captures decode HTTP on destination port `443`, which is abnormal for legitimate HTTPS.
- User-Agent strings are inconsistent and suspicious, including old IE strings, `Winhttp 1/0`, `WinHTTP loader/1.0`, and an Android WebView string from a Windows domain controller candidate.

### Verdict
**Rejected.** Normal baseline traffic exists, but it does not explain the malicious domains, payload downloads, bot URI patterns, repeated C2 paths, or suspicious User-Agents.

---

## Hypothesis 4: Authorized Red Team or Training Exercise
**Statement:** The traffic represents a controlled exercise rather than a real malicious intrusion.

### What Would Confirm It
- Exercise markers in hostnames, domains, banners, or payloads.
- Internal red-team infrastructure or benign test domains.
- Documentation tying the traffic to a known simulation.

### Evidence Supporting
- Both PCAPs are public training/research captures from Malware-Traffic-Analysis.net.
- The attack chain is compact and suitable for classroom investigation.

### Evidence Refuting
- The packets themselves contain realistic malware-family behavior rather than obvious lab markers.
- The Hancitor capture uses suspicious external domains and binary downloads.
- The Trickbot capture uses malware-specific bot path structures and typo-squatted C2.
- A SOC analyst receiving only these packets would need to treat them as hostile until proven otherwise.

### Verdict
**Plausible as dataset context, rejected as the traffic explanation.** For the report, analyze the packets as a real incident capture and mention the public-corpus context only in A.1.

---

## Hypothesis 5: RedLine Stealer Performs Direct Exfiltration
**Statement:** `10.10.23.101` is infected with RedLine Stealer and uploads collected data to `188.190.10.10:55123`.

### What Would Confirm It
- A single internal host sending a large amount of data to an external host.
- HTTP POST activity to a direct IP or non-standard service port.
- Public IP discovery before or during upload behavior.

### Evidence Supporting
- `10.10.23.101:49697 <-> 188.190.10.10:55123` accounts for `8,787` frames and about `6,057 kB`.
- Directionality is heavily outbound: `10.10.23.101` sends about `5,783 kB` to `188.190.10.10`.
- Frames `7`, `14`, `4387`, and `8786` show HTTP `POST /` requests to `188.190.10.10:55123`.
- Frame `21` queries `api.ip.sb`, and frame `26` uses TLS SNI `api.ip.sb`, matching public-IP discovery behavior.

### Evidence Against
- The exact uploaded content is not decoded in this report.
- The PCAP is short, so post-exfil command activity is not visible.

### Verdict
**Confirmed with high confidence for stealer-like upload behavior.** Exact stolen file or credential contents are not proven, but the traffic pattern is not consistent with normal browsing.

---

## Hypothesis 6: The Exfiltration Capture Is Mostly Benign Web Noise Hiding Possible Egress Risk
**Statement:** `exfiltration.pcap` is not a clean malware trace by itself; it demonstrates how noisy DNS/TLS web activity can hide exfiltration unless a baseline is built first.

### What Would Confirm It
- Many DNS and TLS destinations tied to advertising, analytics, CDNs, and OCSP.
- No single obvious malware URI or plaintext credential.
- One internal client generating broad external web activity.

### Evidence Supporting
- `10.0.2.15` is the main internal host and sends DNS through `1.1.1.1`.
- Top DNS queries include `dt.adsafeprotected.com`, `www.redditstatic.com`, `ophan.theguardian.com`, `i.guim.co.uk`, and `www.google-analytics.com`.
- TLS SNI values include ad/analytics domains such as `pixel.onaudience.com`, `secure.adnxs.com`, `cdn.krxd.net`, and `sync.crwdcntrl.net`.
- HTTP requests are OCSP POSTs to certificate-status services such as `ocsp.digicert.com` and `ocsp.comodoca.com`.

### Evidence Against
- High DNS/TLS volume can still be abused for covert egress.
- The filename indicates exfiltration, so the correct analyst stance is cautious, not dismissive.

### Verdict
**Medium confidence as an exfiltration-training analogue, not high-confidence malware from metadata alone.** It supports the architecture recommendation for DNS analytics, TLS metadata logging, and egress DLP.

---

## Hypothesis 7: FTP Traffic Represents Credential Abuse and File Movement
**Statement:** `192.168.47.1` authenticates to `192.168.47.134` over FTP using exposed privileged credentials and transfers files.

### What Would Confirm It
- Cleartext username and password.
- Successful login response.
- FTP upload or download commands.

### Evidence Supporting
- Frame `5` shows `USER Administrator`.
- Frame `7` shows `PASS napier`.
- Frame `8` confirms `230 User Administrator logged in.`
- Frame `50` shows `STOR 111.png`, with transfer completion at frame `123`.
- Frame `166` shows `RETR manual.txt`, with transfer completion at frame `177`.

### Evidence Against
- The capture does not prove whether the actor was authorized.
- FTP may be a lab sample rather than a real company incident.

### Verdict
**Confirmed for credential exposure and file movement.** This is direct evidence for replacing plaintext FTP with managed secure transfer and credential controls.

---

## Hypothesis 8: Nmap NULL Scan Performs Internal Reconnaissance
**Statement:** `192.168.47.171` scans `192.168.47.134` using TCP NULL probes to discover exposed services.

### What Would Confirm It
- TCP probes with no flags set.
- Large number of destination ports in a short time window.
- One scanner and one target.

### Evidence Supporting
- `192.168.47.171` sends `1,030` packets with `tcp.flags == 0x000` to `192.168.47.134`.
- The scan touches `1,000` unique destination ports.
- The burst occurs in about `2.6` seconds.
- Sample target ports include `21`, `22`, `23`, `25`, `53`, `80`, `135`, `139`, `443`, `445`, `3306`, `3389`, `5432`, `5900`, and `8080`.

### Evidence Against
- This could be an authorized vulnerability scan if scheduled and documented.
- The capture alone does not show follow-on exploitation.

### Verdict
**Confirmed for scanning behavior.** Authorization is unknown, so a SOC report should label it as reconnaissance requiring validation against the approved scan calendar.

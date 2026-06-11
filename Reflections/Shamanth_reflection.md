# Reflections: Project KAVACH – Network Forensics
**Author: Shamanth**

## The Challenge
The most difficult part of this workstream was filtering through the "noise" of modern network traffic. PCAPs like `exfiltration.pcap` proved that thousands of DNS and TLS requests are normal in an enterprise environment. Finding the "needle in the haystack"—like the Hancitor payload download or the RedLine stealer POSTing to a direct IP on port 55123—required a very disciplined triage process.

## Key Technical Learnings
1.  **Malware doesn't always use standard ports:** I learned that RedLine Stealer used port `55123` for exfiltration, which would bypass many basic firewall rules.
2.  **Port 443 doesn't always mean HTTPS:** We found plaintext HTTP being sent over port 443. This was a huge realization—if you only look at the port and not the protocol, you miss the attack.
3.  **The "Pre-flight" Pattern:** I noticed a consistent pattern where malware first contacts `api.ipify.org` or `api.ip.sb` to discover its own public IP before starting its malicious staging. This is now a high-confidence alert trigger for me.

## Successes and "Aha" Moments
My biggest success was successfully reconstructing the file hashes for the staged malware binaries. Matching the SHA256 of `15.bin` to known Ficker Stealer artifacts confirmed our hypothesis about the infection chain. Another "aha" moment was seeing the Trickbot bot ID embedded directly in the URI path—it was like the malware was introducing itself.

## Strategic Impact
Working on the Network part made me realize that security isn't about stopping the first click; it's about breaking the *chain* of events that follows. By proposing "deny-by-default" egress rules and protocol-aware inspection, we can effectively "blind" the attacker's Command and Control even if a workstation gets infected.

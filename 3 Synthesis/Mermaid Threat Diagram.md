# Threat Model Diagram — Project KAVACH

```mermaid
flowchart TD

%% =====================
%% ATTACK PATH 1 (WEB → NETWORK)
%% =====================

A1[Attacker] --> B1[Exploit SQL Injection]
B1 --> C1[Access Backend Database]
C1 --> D1[Exploit IDOR\n(Unauthorized Data Access)]
D1 --> E1[Extract Customer Data]

E1 --> F1[Exfiltration Attempt]
F1 --> G1[HTTP POST / DNS Exfiltration]
G1 --> H1[External C2 / Attacker Server]

%% Evidence mapping
G1 --- N1[Network Evidence:\n• High outbound traffic\n• POST uploads\n• External IP 188.190.10.10\n• DNS anomalies]

D1 --- N2[Web Evidence:\n• SQL Injection\n• IDOR vulnerability]


%% =====================
%% ATTACK PATH 2 (NETWORK → WEB)
%% =====================

A2[Malware Infection\n(Hancitor / Trickbot)] --> B2[C2 Beaconing]
B2 --> C2[Credential Theft\n(Suspected)]
C2 --> D2[Lateral Movement\n(SMB / LDAP / Kerberos)]
D2 --> E2[Compromised Internal Host]

E2 --> F2[Access Customer Portal]
F2 --> G2[Exploit Weak Auth / IDOR]
G2 --> H2[Data Access / Abuse]

%% Evidence mapping
B2 --- N3[Network Evidence:\n• C2 beaconing\n• Suspicious DNS\n• Typosquatted domains]

D2 --- N4[Network Evidence:\n• SMB / LDAP / Kerberos activity\n• Internal movement risk]

G2 --- N5[Web Evidence:\n• Broken Access Control\n• Weak authentication]


%% =====================
%% SHARED INSIGHT
%% =====================

H1 --> Z[Potential Multi-Stage Attack Chain]
H2 --> Z

Z --> R[Business Impact:\n• Customer data exposure\n• Financial risk\n• Reputation damage]

%% Styling (optional but helps visually)
style A1 fill:#ffcccc
style A2 fill:#ffcccc
style Z fill:#ffe599
style R fill:#f4cccc
``
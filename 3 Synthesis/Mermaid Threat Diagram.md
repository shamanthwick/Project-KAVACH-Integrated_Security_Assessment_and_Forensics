# Mermaid Threat Diagram

Source image: `image.png`

```mermaid
flowchart LR
    A[PCAP Analysis Layer] --> B[Detect Traffic Anomalies]
    B --> C[Map Flows to HTTP<br/>Behavior]

    C --> D[IDOR Exploit<br/>(Unauthorized Data Access)]
    D --> E[Application Logic Issue<br/>(Broken Access Control)]
    E --> F[Weak Network Visibility<br/>(Auth Issue Not Fully Visible)]

    C --> G[SQL Injection Attempt<br/>(Malicious Input Payload)]
    G --> H[Database Manipulation Path]
    H --> I[Strong Network Indicators<br/>(Repeated Query Patterns)]

    F --> J[Correlation Gap]
    I --> J
    J --> K[AI Correlation Engine<br/>(Pattern Matching + Context Linking)]
    K --> L[Confidence Score<br/>(High / Medium / Low)]

    classDef process fill:#eef0ff,stroke:#8b5cf6,stroke-width:2px,color:#333;
    classDef gap fill:#f3f0ff,stroke:#8b5cf6,stroke-width:2px,color:#333;
    classDef engine fill:#eef0ff,stroke:#8b5cf6,stroke-width:2px,color:#333;

    class A,B,C,D,E,F,G,H,I,L process;
    class J gap;
    class K engine;
```

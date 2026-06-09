# Wireshark SSH Brute Force Analysis

Network-layer analysis of live SSH brute force traffic captured on an internet-exposed Azure VM. This lab captures real attacker traffic with `tcpdump`, analyzes it in Wireshark, fingerprints the tools attackers used, and correlates the packet evidence against Microsoft Sentinel logs — then hardens the firewall and validates the fix at the packet level.

This is the second lab in a blue-team detection series. The first lab detected the same attacks at the **log layer** in Microsoft Sentinel; this one analyzes them at the **network layer**.

**Workflow:** Detect, Analyze, Correlate, Harden and Validate

---

## What this lab demonstrates

- Capturing live attack traffic on an exposed cloud host without polluting the capture with your own session
- Reading a brute force attack from raw packets — connection attempts, failed-auth patterns, TCP resets
- Fingerprinting attacker tooling from unencrypted SSH version strings
- Mapping attacker source IPs geographically
- Correlating network evidence (pcap) against log evidence (Sentinel KQL) — the same attack seen through two independent data sources
- Hardening a Network Security Group and proving the control works with a zero-packet validation capture

---

## Environment

| Component | Detail |
|---|---|
| Target | Ubuntu 24.04 VM on Microsoft Azure, SSH exposed on port 22 |
| Capture tool | `tcpdump` (background capture via `nohup`) |
| Analysis tool | Wireshark |
| Correlation | Microsoft Sentinel (KQL against syslog) |
| Firewall | Azure Network Security Group |
| MITRE ATT&CK | [T1110.001 — Brute Force: Password Guessing](https://attack.mitre.org/techniques/T1110/001/) |

---

## Key findings

- **9 unique attacker IPs** captured hitting SSH in a single capture window
- Multiple clients fingerprinted as `SSH-2.0-libssh` — automated tooling, not human operators
- Attacker source IPs spanned multiple continents
- The same IPs appearing in the packet capture were confirmed in Microsoft Sentinel syslog — two tools, one attack
- After restricting the NSG to a single source IP, a re-capture filtered for external connection attempts returned **0 packets**, proving the control

---## Repository structure
.
├── README.md
├── LICENSE
├── queries/
│   └── attacker-ip-correlation.kql     # KQL: correlate pcap IPs vs Sentinel syslog
├── screenshots/                        # Annotated evidence (9 captures)
└── docs/
└── lab-guide.html                  # Full interactive lab walkthrough

---

## How to read this repo

1. Start with `docs/lab-guide.html` for the full step-by-step walkthrough
2. `queries/attacker-ip-correlation.kql` holds the correlation logic — the array of captured attacker IPs matched against Sentinel `Syslog`
3. `screenshots/` contains the annotated evidence for each phase: capture, fingerprinting, mapping, correlation, hardening, and validation

---

## Selected Wireshark filters used
tcp.port == 22
ip.addr != <analyst-ip>
frame contains "libssh"
tcp.flags.reset == 1
tcp.flags.syn==1 and tcp.flags.ack==0 and tcp.dstport==22 and ip.src != <analyst-ip>

---

## Related labs

- **Lab 1 — Microsoft Sentinel SSH Brute Force Detection** *(log layer)*
- **Lab 2 — Wireshark SSH Brute Force Analysis** *(network layer)* — this repo
- **Lab 3 — Active Directory Attack Detection** *(identity layer)* — coming soon

---

## A note on the data

The attacker IPs shown here were captured against an intentionally exposed lab host and are real internet scanning sources observed during the capture window. No internal or personal infrastructure detail is included; analyst/home IPs have been redacted from all screenshots and queries.

---

## License

Released under the [MIT License](LICENSE).



# Network Fundamentals & Reconnaissance — Nmap & Wireshark

## Objective
A hands-on networking and reconnaissance exercise performed in an isolated lab environment (Kali Linux inside VMware, alongside a Windows 10 target and Metasploitable2). The project covers core networking fundamentals, live packet capture and protocol analysis with **Wireshark**, active host/service reconnaissance with **Nmap**, and maps the techniques used to the **MITRE ATT&CK** framework with relevant CVE/CVSS context.

> All activity was performed against isolated lab VMs on a private, non-routable network (`192.168.198.0/24`) — no external systems were scanned.

## Environment
- **Attacker/analysis machine:** Kali Linux (`192.168.198.135/24`), running in VMware Workstation
- **Targets:** Windows 10 VM (`192.168.198.142`), Metasploitable2 VM (`192.168.120.129`, separate segment)
- **Tools:** Wireshark, Nmap

## Method
1. **Networking fundamentals review** — OSI model, TCP/IP suite, ports, protocols (TCP/UDP/ICMP/ARP/DNS/DHCP/HTTP-HTTPS) as the theoretical basis for the exercise.
2. **IP addressing** — confirmed the Kali machine's own private IP configuration and reviewed public vs. private addressing and subnetting.
3. **Packet capture** — ran a live Wireshark capture on `eth0` while generating browsing/ping traffic, then filtered by protocol (DNS, HTTP, ICMP) to identify and explain specific traffic.
4. **Active reconnaissance** — used Nmap to perform host discovery, service/version detection, OS fingerprinting, and an aggressive scan against a Windows target.
5. **Framework mapping** — mapped the reconnaissance techniques used to MITRE ATT&CK and connected the discovered SMB/NetBIOS exposure to real-world CVE/CVSS context.

## Wireshark Findings

| Filter | What it showed |
|---|---|
| *(none — live capture)* | ARP resolution, TCP handshake/teardown, TLSv1.2 encrypted traffic to an external host |
| `dns` | Kali (`192.168.198.135`) querying its resolver (`192.168.198.2`) for domains including `detectportal.firefox.com`, `safebrowsing.googleapis.com`, `o.pki.goog`, `ads.mozilla.org` — A/AAAA responses returned |
| `http` | OCSP (Online Certificate Status Protocol) traffic — the browser checking TLS certificate revocation — plus a plain HTTP GET/response pair |
| *(ICMP, cross-segment)* | Pings from Kali to Metasploitable2 produced no replies, since the two VMs sit on separate, unrouted network segments — a direct illustration of how subnetting affects reachability |

Capture exported to `captures.pcapng` (38.0 MiB) as a reusable artifact.

## Nmap Findings

| Scan | Command | Result |
|---|---|---|
| Host discovery | `nmap -sn 192.168.198.135/24` | 4 live hosts found: gateway (`.1`), a second host (`.2`), DHCP/broadcast responder (`.254`), Kali itself (`.135`) |
| Service/version detection | `nmap -sV 192.168.198.142` | 4 open ports on the Windows target: `135` (msrpc), `139` (netbios-ssn), `445` (microsoft-ds), `5357` (HTTPAPI) |
| OS detection | `nmap -O 192.168.198.142` | Correctly fingerprinted as Microsoft Windows 10 (build 1709–22H2) |
| Aggressive scan | `sudo nmap -A 192.168.198.142` | Combined service + OS detection, NetBIOS/SMB host script results (hostname `SHERY`, SMB2 signing enabled but not required), traceroute confirming a single hop to the target |

## MITRE ATT&CK Mapping

| Technique | Activity |
|---|---|
| **T1018 — Remote System Discovery** | Nmap host discovery sweep (`-sn`) — enumerating live systems on a network for further reconnaissance |
| **T1040 — Network Sniffing** | Wireshark packet capture — passively monitoring traffic to reveal configuration, services, and potential credentials in transit |

## CVE/CVSS Context
The open SMB/NetBIOS ports (`139`, `445`) discovered on the Windows target are the same class of service historically associated with high-severity vulnerabilities such as **CVE-2017-0144 ("EternalBlue", CVSS 8.1)**, affecting older SMBv1 implementations. No exploitation was attempted — the findings illustrate why exposed SMB services are routinely flagged in vulnerability assessments, and why patch level and SMB signing configuration matter for hardening.

## Screenshots

| # | Description |
|---|---|
| [`01-kali-ip-config.png`](./screenshots/01-kali-ip-config.png) | `ip a` output — Kali's private IP `192.168.198.135/24` |
| [`02-kali-lab-environment.png`](./screenshots/02-kali-lab-environment.png) | Kali Linux desktop — the analysis machine used throughout |
| [`03-wireshark-live-capture.png`](./screenshots/03-wireshark-live-capture.png) | Live capture on `eth0` — ARP, TCP, TLSv1.2 traffic |
| [`04-wireshark-dns-filter.png`](./screenshots/04-wireshark-dns-filter.png) | Filtered on `dns` — queries and responses |
| [`05-wireshark-http-ocsp.png`](./screenshots/05-wireshark-http-ocsp.png) | Filtered on `http` — OCSP + HTTP GET/response |
| [`06-wireshark-icmp-cross-segment.png`](./screenshots/06-wireshark-icmp-cross-segment.png) | ICMP to Metasploitable2 — no reply across segments |
| [`07-wireshark-exported-pcap.png`](./screenshots/07-wireshark-exported-pcap.png) | Exported `captures.pcapng` (38.0 MiB) |
| [`08-nmap-host-discovery.png`](./screenshots/08-nmap-host-discovery.png) | `-sn` host discovery — 4 live hosts |
| [`09-nmap-service-detection.png`](./screenshots/09-nmap-service-detection.png) | `-sV` service detection on Windows target |
| [`10-nmap-service-detection-cont.png`](./screenshots/10-nmap-service-detection-cont.png) | `-sV` output continued, start of `-O` |
| [`11-nmap-os-detection-aggressive.png`](./screenshots/11-nmap-os-detection-aggressive.png) | OS detection + `-A` aggressive scan, host script + traceroute |
| [`12-mitre-t1018-discovery.png`](./screenshots/12-mitre-t1018-discovery.png) | MITRE ATT&CK — T1018 Remote System Discovery |
| [`13-mitre-t1040-sniffing.png`](./screenshots/13-mitre-t1040-sniffing.png) | MITRE ATT&CK — T1040 Network Sniffing |

## Key Takeaway
Passive traffic analysis (Wireshark) and active scanning (Nmap) answer different questions — what's actually happening on the wire vs. what's reachable and running — and both map cleanly onto documented adversary techniques (MITRE ATT&CK), which is exactly how a SOC analyst would classify and communicate this kind of activity in a real investigation.

## Repository Structure
```
├── README.md
└── screenshots/
    ├── 01-kali-ip-config.png
    ├── 02-kali-lab-environment.png
    ├── 03-wireshark-live-capture.png
    ├── 04-wireshark-dns-filter.png
    ├── 05-wireshark-http-ocsp.png
    ├── 06-wireshark-icmp-cross-segment.png
    ├── 07-wireshark-exported-pcap.png
    ├── 08-nmap-host-discovery.png
    ├── 09-nmap-service-detection.png
    ├── 10-nmap-service-detection-cont.png
    ├── 11-nmap-os-detection-aggressive.png
    ├── 12-mitre-t1018-discovery.png
    └── 13-mitre-t1040-sniffing.png
```

---
**Tools:** Wireshark, Nmap, Kali Linux, VMware Workstation
**Skills demonstrated:** Packet analysis, protocol identification, active reconnaissance, OS/service fingerprinting, MITRE ATT&CK mapping, CVE/CVSS context

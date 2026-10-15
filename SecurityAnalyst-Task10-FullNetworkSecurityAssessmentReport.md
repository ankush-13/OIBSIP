# Full Network Security Assessment Report

**Task:** Task 10 – Full Network Security Assessment Report  
**File:** `network_security_assessment.md`  
**Focus:** End-to-end security assessment methodology, reconnaissance, traffic analysis, vulnerability scans, findings register, executive summary, and remediation roadmap.

---

## 1. Executive Summary

During October 2026, a structured end-to-end network security assessment was conducted against the local test network environment (`192.168.10.0/24`). The primary objective was to evaluate the overall security posture, uncover configuration flaws, identify unencrypted data transmissions, and review web application security controls.

### Overall Risk Posture
The test network currently exhibits a **High Risk** posture. This rating is driven by legacy unencrypted management services (Telnet) exposed on live hosts, plaintext administrative credentials traversing the wire, and unpatched web service banners combined with improperly secured backup directories.

### Key Metrics & Findings Breakdown
* **Critical:** 1
* **High:** 2
* **Medium:** 1
* **Low:** 1
* **Total Discovered Vulnerabilities:** 5

### Top Priority Remediation Focus
1. **Eliminate Legacy Protocols:** Immediately shut down and disable plaintext Telnet services across all endpoints, enforcing secure SSH v2 connections instead.
2. **Secure Data-in-Transit:** Mandate TLS 1.3 across all management and web endpoints to prevent cleartext credential sniffing.
3. **Web Server & Directory Hardening:** Remove public access from backup directories and suppress software version disclosure headers.

---

## 2. Assessment Scope & Methodology

### 2.1 Scope Definition
* **Target IP Range:** `192.168.10.0/24` (Local Lab Test Range)
* **Target Services:** TCP/UDP Ports (SSH, HTTP, HTTPS, DNS, FTP, Telnet)
* **Time Window:** October 15, 2026, from 09:00 AM to 02:00 IST

### 2.2 Industry Frameworks Referenced
* **Penetration Testing Execution Standard (PTES):** Guided the end-to-end engagement lifecycle from reconnaissance to reporting.
* **OWASP Testing Guide (WSTG):** Directed web component inspection, header verification, and directory enumeration.
* **CVSS v3.1:** Utilized for consistent, mathematical severity rating of all identified weaknesses.

---

## 3. Technical Report & Phase Findings

### Phase 1 — Reconnaissance & Discovery

An active port scan and service enumeration sweep was executed against the target range using Nmap:

```bash
nmap -sV -O 192.168.10.0/24
```

*Raw scan output saved to:* `nmap_results.txt`

#### Discovered Assets & Services Summary

| Host IP | State | Open Ports | Detected Service / Version | OS Fingerprint |
| :--- | :--- | :--- | :--- | :--- |
| `192.168.10.1` | Up | 53/TCP, 80/TCP | DNS (BIND 9.11), Apache HTTPD 2.4.41 | Linux (Ubuntu 20.04) |
| `192.168.10.15` | Up | 23/TCP, 80/TCP | Telnet (Linux telnetd), Nginx 1.18.0 | Embedded Linux |

---

### Phase 2 — Traffic Analysis (Wireshark Capture)

A 5-minute packet capture was gathered (`wireshark_capture.pcap`) and analyzed via protocol filters:

* **ARP Traffic:** Normal gratuitous and query broadcast behavior observed, with minor local vulnerability to ARP cache poisoning due to lack of Dynamic ARP Inspection (DAI).
* **DNS Traffic:** Standard iterative queries handled cleanly by local resolvers; no DNS tunneling anomalies detected.
* **HTTP/Telnet Traffic (Critical Observation):** Unencrypted cleartext credentials were observed traversing the wire on Port 23 (Telnet) and Port 80 (Basic Auth HTTP login forms). Plaintext usernames and passwords can be directly extracted from TCP stream payloads.

---

### Phase 3 — Web Vulnerability Assessment (Nikto)

Nikto was run against the discovered web server endpoint (`http://192.168.10.15`):

```bash
nikto -h http://192.168.10.15
```

#### Findings:
* Server banner leaks exact Apache/Nginx version strings.
* Default administration directory (`/admin/backup/`) is publicly accessible without authentication.
* Missing modern security headers (`X-Frame-Options`, `Content-Security-Policy`, `Strict-Transport-Security`).

---

## 4. Findings Register

| Finding ID | Description | Severity | Affected Asset | Recommended Fix |
| :--- | :--- | :--- | :--- | :--- |
| **FIN-01** | Plaintext Telnet Management Service Enabled | **Critical** (CVSS 9.8) | `192.168.10.15:23` | Disable Telnet completely; enforce secure SSH v2 access. |
| **FIN-02** | Sensitive Credentials Transmitted in Cleartext (HTTP/Telnet) | **High** (CVSS 8.1) | Network Segment `192.168.10.0/24` | Mandate TLS 1.3 for all web panels and tunnel CLI management over SSH. |
| **FIN-03** | Publicly Accessible Backup Directory (`/admin/backup/`) | **High** (CVSS 7.5) | `192.168.10.15:80` | Restrict directory permissions and implement robust role-based access control. |
| **FIN-04** | Web Server Version Disclosure via HTTP Headers | **Medium** (CVSS 5.3) | `192.168.10.1` | Suppress version tokens in Apache/Nginx global configuration files. |
| **FIN-05** | Missing Modern Security Headers (HSTS, CSP) | **Low** (CVSS 3.7) | `192.168.10.1` | Configure HTTP response headers to enforce HSTS and X-Frame-Options. |

---

## 5. Remediation Roadmap

Prioritized roadmap for engineering and IT security teams to systematically close identified security gaps:

1. **Phase A: Critical Remediations (Immediate / 24 Hours)** — *Effort: Easy*
   Disable the legacy Telnet daemon across all routers and servers. Shut down unencrypted management channels and block port 23 at the firewall perimeter. Verify success by running an Nmap port check confirming port 23 is closed.

2. **Phase B: Access Control & Secret Protection (Within 7 Days)** — *Effort: Medium*
   Remove or password-protect exposed web directories like `/admin/backup/`. Implement SSL/TLS certificates across all internal web services to prevent cleartext credential harvesting. Verify via Wireshark packet capture that no plaintext passwords appear in traffic streams.

3. **Phase C: Server Hardening & Header Tuning (Within 30 Days)** — *Effort: Easy*
   Update Apache and Nginx configuration files to hide version banners and inject secure response headers (`X-Content-Type-Options`, `Strict-Transport-Security`). Verify success via an automated Nikto scan rerun.

---

## 6. GitHub Repository Artifacts Checklist

The following deliverables have been prepared for commit to the version control repository:
* `network_security_assessment.md` (This complete report document)
* `nmap_results.txt` (Raw command-line output logs from network enumeration)
* `wireshark_capture.pcap` (Filtered network capture sample file)

---

## Submission Checklist

- [x] Scope defined in writing (IP ranges, services, time window)
- [x] Phase 1 Reconnaissance documented with Nmap scan results
- [x] Phase 2 Traffic Analysis documented with Wireshark insights
- [x] Phase 3 Web Vulnerability Scan documented with Nikto findings
- [x] Findings register produced with all required columns and CVSS ratings
- [x] 1-page non-technical Executive Summary included
- [x] Detailed technical report with findings for each phase
- [x] Remediation roadmap with prioritization and effort estimates
- [x] All GitHub files ready (`network_security_assessment.md`, `nmap_results.txt`, `wireshark_capture.pcap`)
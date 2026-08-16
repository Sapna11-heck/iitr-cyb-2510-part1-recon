# Task 7 — Penetration-Test Report

## (a) Executive Summary

This engagement assessed the security posture of a controlled lab network (192.168.42.0/24) representing a mid-sized technology company's infrastructure. Testing identified a live web/service host (Metasploitable2) running numerous outdated software versions, several of which carry publicly known, high-severity vulnerabilities — including at least one remotely exploitable backdoor. Immediate remediation of the highest-severity findings (detailed below) is recommended before this infrastructure profile is deployed in any production-equivalent environment.

## (b) Scope and Rules of Engagement

*(See `scope.md` in this repository for the full pre-engagement scope document.)*

- **Target range:** 192.168.42.0/24
- **In scope:** Passive OSINT, active host discovery, port scanning, DNS enumeration, automated vulnerability scanning
- **Out of scope:** Exploitation, post-exploitation, any target outside 192.168.42.0/24, denial-of-service testing
- **Rules of engagement:** Self-contained isolated lab; tester (Sapna) is the sole point of contact; scans run at default/reduced Nmap timing to minimize network noise

## (c) Methodology — PTES Phase Mapping

| PTES Phase | Tasks Performed |
|---|---|
| **Pre-engagement Interactions** | Task 1 — Scope definition, rules of engagement |
| **Intelligence Gathering** | Task 2 — Passive OSINT (DNS records, Shodan); Task 3 — Active host discovery (Nmap ping sweep); Task 5 — DNS enumeration (dnsmasq authoritative zone, dig queries, AXFR test, reverse DNS) |
| **Vulnerability Analysis** | Task 4 — Port scanning and service-version enumeration (SYN scan, `-sV`, `-O`); Task 6 — Automated vulnerability scan (Nessus Basic Network Scan) |

*(Exploitation and Post-Exploitation phases were explicitly out of scope per the Task 1 rules of engagement and were not performed.)*

## (d) Findings Table

*(Consolidated from Task 4 manual banner-grabbing and Task 6's automated Nessus scan, 65 total findings — 8 Critical, 28 High, 7 Medium.)*

| CVE / Reference | CVSS Base Score | Severity | Affected Host:Port | Remediation Summary |
|---|---|---|---|---|
| CVE-2011-2523 | 10.0 (Critical) | Critical | 192.168.42.129:21 | Upgrade vsftpd from the backdoored 2.3.4 build to a current, verified release; the 2.3.4 source distribution was compromised and contains a hardcoded backdoor triggered by a specific login string — no configuration change mitigates this, only replacing the binary does. |
| Nessus Plugin 61708 — VNC 'password' Password | 10.0 (Critical) | Critical | 192.168.42.129:5900 | Nessus successfully authenticated to the VNC service using the password "password", granting full remote desktop control. Set a strong unique password immediately or disable VNC; tunnel any legitimate remote-desktop need over SSH rather than exposing VNC directly. |
| Nessus Plugin 51988 — Bind Shell Backdoor | 9.8 (Critical) | Critical | 192.168.42.129:1524 | Nessus confirmed a fully unauthenticated root shell (`uid=0(root)`) listening on this port. This is a pre-planted backdoor, not a configuration issue — the host must be treated as already compromised via this vector and rebuilt from a clean image. |
| Nessus Plugin 20007 — SSLv2/SSLv3 Protocol Detection | 9.8 (Critical) | Critical | 192.168.42.129:25, 192.168.42.129:5432 | Disable SSLv2/SSLv3 on the SMTP and PostgreSQL services; require TLS 1.2+ with approved cipher suites only — these deprecated protocols carry known cryptographic weaknesses (e.g., POODLE-class padding attacks). |
| CVE-2010-2075 | 10.0 (Critical) | Critical | 192.168.42.129:6667 | Upgrade or remove the UnrealIRCd service; the affected build (3.2.8.1) contains a backdoored source archive allowing arbitrary remote command execution — replace with a clean, current build or disable the IRC service entirely if not business-critical. |

## (e) Risk Heat Map Description (Likelihood × Impact)

- **vsftpd 2.3.4 backdoor (CVE-2011-2523):** High Likelihood (publicly known, trivially triggered, no authentication required) × Critical Impact (direct root shell) → **Top-right quadrant: Critical Risk, immediate action required.**
- **UnrealIRCd backdoor (CVE-2010-2075):** High Likelihood (public exploit widely available) × Critical Impact (arbitrary command execution as the service user) → **Top-right quadrant: Critical Risk, immediate action required.**
- **Legacy service exposure (Telnet, rsh/rexec/rlogin, unauthenticated NFS, X11):** Medium Likelihood (requires network access, but no exploit needed — plaintext credentials or trust-based auth) × High Impact (credential interception, lateral movement) → **Upper-middle quadrant: High Risk, prioritize after critical findings.**
- **Outdated framework versions (Apache 2.2.8, MySQL 5.0.51a, PostgreSQL 8.3.x, Samba 3.x):** Medium Likelihood (many have historical CVEs but not all are trivially exploitable) × Medium Impact (varies by specific CVE) → **Center quadrant: Medium Risk, remediate in normal patch cycle.**

## (f) Remediation Priority List (Highest CVSS First)

1. **vsftpd 2.3.4 backdoor** (CVE-2011-2523, CVSS 10.0) — replace binary immediately
2. **VNC weak password "password"** (Nessus 61708, CVSS 10.0) — set a strong password or disable VNC immediately
3. **UnrealIRCd backdoor** (CVE-2010-2075, CVSS 10.0) — replace or disable service immediately
4. **Bind shell backdoor on port 1524** (Nessus 51988, CVSS 9.8) — treat host as compromised, rebuild from clean image
5. **SSLv2/SSLv3 enabled on SMTP and PostgreSQL** (Nessus 20007, CVSS 9.8) — disable deprecated protocols, enforce TLS 1.2+
6. Disable or restrict legacy plaintext-authentication services (Telnet, rsh/rexec/rlogin) — replace with SSH-only access
7. Patch/upgrade all remaining outdated service versions identified in Task 4 (Apache, Samba, MySQL, PostgreSQL, BIND) to current stable releases
8. Restrict NFS and X11 exposure to internal-only access with proper authentication, or disable if unused

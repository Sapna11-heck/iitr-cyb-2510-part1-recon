# Pre-Engagement Scope Definition

## Lab Setup Note
- **Kali Linux (attacker box):** 192.168.42.128
- **Metasploitable2 (target — intentionally vulnerable VM):** 192.168.42.129
- **Network:** VMware Host-only adapter, isolated from the host's production/internet-facing network
- **Target scope subnet:** 192.168.42.0/24

## (a) Target IP Range
The authorized target scope for this engagement is **192.168.42.0/24**, a private, isolated virtual-machine lab network created specifically for this assessment. No systems outside this range are in scope.

## (b) Techniques In Scope
- Passive OSINT reconnaissance (demonstrated against public sources only, as detailed in Task 2)
- Active host discovery (ICMP/ARP-based ping sweep) against 192.168.42.0/24
- Port scanning (TCP SYN scan, service-version detection, OS fingerprinting) against live hosts in 192.168.42.0/24
- DNS enumeration against a locally-configured authoritative DNS server within the lab
- Automated vulnerability scanning (Nessus/OpenVAS) against live hosts in 192.168.42.0/24

## (c) Techniques Out of Scope
- Exploitation of any discovered vulnerability (no Metasploit exploit modules, no manual exploitation)
- Post-exploitation activity of any kind (no privilege escalation, no lateral movement, no data exfiltration)
- Any scan, probe, or packet directed at IP addresses outside 192.168.42.0/24
- Denial-of-service testing or any technique that could disrupt availability
- Social engineering or physical security testing

## (d) Rules of Engagement
- **Scanning hours:** Testing is conducted only within the controlled lab environment on the tester's own machine; no time-of-day restriction applies since no production systems or third parties are affected.
- **Rate limits:** Nmap scans are run at default or reduced timing templates (T2–T3) to avoid saturating the virtual network and to produce clean, reproducible output for the report.
- **Contact point in case of accidental disruption:** Since this is a self-contained, isolated lab with no external stakeholders, the tester (Sapna) is the sole point of contact and is responsible for restoring any VM to a known-good snapshot if disruption occurs.
- **Authorization:** This engagement is self-authorized for academic/training purposes as part of the IITR-CYB-2510 capstone project. No third-party systems are involved.

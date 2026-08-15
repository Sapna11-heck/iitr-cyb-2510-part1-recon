# Task 2 — Passive OSINT (Technique Demonstration)

## Target: Public domain `rolls-roycemotorcars.com` (DNS) and `port:22 country:GB` (Shodan device class)

All data below was collected using publicly available passive sources only. No packets were sent to the private lab range (192.168.42.0/24).

## (a) DNS Record Enumeration (via `dig` against Google Public DNS 8.8.8.8)

| Record Type | Value | Source Command |
|---|---|---|
| A | 160.46.235.35 | `dig A rolls-roycemotorcars.com @8.8.8.8` |
| MX | 10 rollsroycemotorcars-com01c.mail.protection.outlook.com | `dig MX rolls-roycemotorcars.com @8.8.8.8` |
| NS | ns.bmw.de, ns2.m-online.net, ns3.m-online.net, ns4.m-online.net | `dig NS rolls-roycemotorcars.com @8.8.8.8` |
| CNAME | www.rolls-roycemotorcars.com → rolls-roycemotorcars.com.edgekey.net | `dig CNAME www.rolls-roycemotorcars.com @8.8.8.8` |
| TXT | SPF record: `v=spf1 include:2u60g8ujv.spf.checkpoint-spf.com include:spf.protection.outlook.com -all`, plus 19 domain-ownership verification tokens (Google, Cisco, GlobalSign, WebEx, OpenAI, Figma, SwissSign, MS) | `dig TXT rolls-roycemotorcars.com @8.8.8.8` |

## (b) Shodan Free-Tier Query

**Query used:** `port:22 country:GB`
**Total results:** 739,077 hosts
**Top cities:** London (360,180), Ipswich (131,526), Croydon (21,368)
**Top organizations:** EE Limited, DigitalOcean LLC, Orange WBC Broadband, Google LLC, OVH Ltd

**Sample findings:**

| IP | Organization | Location | Banner |
|---|---|---|---|
| 51.105.56.187 | Microsoft Limited UK | London | SSH-2.0-OpenSSH_9.9 |
| 34.89.7.182 | Google LLC | London | SSH-2.0-OpenSSH_10.0 |
| 31.77.146.171 | EE Limited | Ipswich | SSH-2.0-OpenSSH_9.6p1 Ubuntu-3ubuntu13.18 |
| 46.235.229.141 | Mythic Beasts Ltd | London | SSH-2.0-OpenSSH_9.2p1 Debian-2+deb12u10 |
| 87.83.100.156 | NAT customers at smnp exchange | London | SSH-2.0-OpenSSH_10.0p2 Debian-7 |

## (c) Structured Findings Table & Sensitivity Classification

| Finding | Source | Sensitivity (if disclosed to an attacker) | What an Attacker Could Infer |
|---|---|---|---|
| A record (160.46.235.35) | DNS (dig) | **Medium** | Identifies the live public IP fronting the domain — a direct starting point for further port scanning/service enumeration against that host. |
| MX record → Outlook Protection | DNS (dig) | **Low** | Reveals the organization uses Microsoft 365 for email, which narrows phishing/spoofing pretexts (e.g., crafting a fake O365 login page) but is extremely common and low-uplift on its own. |
| NS records (ns.bmw.de, m-online.net) | DNS (dig) | **Medium** | Confirms the domain's DNS is managed by BMW Group infrastructure — a strong OSINT signal of the parent-company relationship (Rolls-Royce Motor Cars is a BMW subsidiary), useful for an attacker mapping organizational/trust relationships for a supply-chain or pretexting attack. |
| CNAME → edgekey.net (Akamai) | DNS (dig) | **Low** | Shows the site sits behind Akamai CDN/WAF, meaning direct-IP attacks against the origin server are less likely to succeed — mildly useful for attack planning but mostly defensive information. |
| TXT — SPF record | DNS (dig) | **Medium** | Lists exact third-party mail-relay providers (Checkpoint, Outlook), which an attacker could use to identify which spoofing/relay paths are explicitly authorized vs. which would fail SPF checks — relevant for planning a convincing phishing campaign. |
| TXT — 19 verification tokens (Google, Cisco, GlobalSign, WebEx, Figma, OpenAI, etc.) | DNS (dig) | **High** | Even though individual tokens are inert, the *set* of services reveals the company's SaaS/vendor footprint (identity providers, collaboration tools, CDN/SSL vendors) — this is valuable reconnaissance for a supply-chain or vendor-impersonation attack, and is disclosed unnecessarily by leaving stale verification records in place after onboarding is complete. |
| Shodan: 739,077 SSH hosts in GB, banner-grabbed | Shodan | **High** | Each banner discloses the exact OpenSSH version and OS/distro (e.g., "Ubuntu-3ubuntu13.18", "Debian-7") for a specific IP — an attacker can directly cross-reference the version string against known CVEs for that OpenSSH build without ever touching the target, enabling pre-built exploit selection before any active engagement. |
| Shodan: organization/ASN mapping (Microsoft, Google, EE, OVH) | Shodan | **Medium** | Confirms which cloud/hosting provider a given IP belongs to, helping an attacker distinguish cloud-hosted assets (often behind additional provider-level protections) from traditional ISP-hosted infrastructure (often less monitored), which affects attack-path prioritization. |

## Summary

This exercise demonstrates that passive OSINT alone — without sending a single packet to the target — already surfaces enough information to significantly narrow an attacker's reconnaissance effort: infrastructure ownership and parent-company relationships (NS records), the organization's SaaS/vendor footprint (TXT tokens), mail-relay trust boundaries (SPF), and, in the Shodan case, exact software versions ripe for CVE lookup. The highest-risk findings are the ones that reveal exact software versions (enabling direct exploit-matching) and the ones that reveal a broad vendor/service footprint (enabling supply-chain and pretexting attacks) — both categories warrant active management (e.g., removing stale TXT records, minimizing banner disclosure via `sshd_config`'s `DebianBanner no` or version obfuscation) even though none of it was obtained through any active scanning technique.

# Task 3 — Active Host Discovery

## (a) Exact Nmap Command and Flag Justification

```bash
sudo nmap -sn 192.168.42.0/24
```

| Flag | Justification |
|---|---|
| `sudo` | Required because Nmap host discovery uses raw ICMP/ARP packets, which need root privileges to craft on Linux. |
| `-sn` | Performs a "ping scan" (host discovery only) without proceeding to port scanning — this is the correct flag for Task 3, which asks only for live-host identification, not service enumeration (that comes in Task 4). |
| `192.168.42.0/24` | Specifies the full authorized lab subnet (as defined in the Task 1 scope document) so every possible host in the range is checked. |

**Note on the lab subnet:** the assessment's example used `192.168.56.0/24`; this lab environment's actual VMware Host-only network was assigned `192.168.42.0/24` by VMware's DHCP configuration. The same range is used consistently across the scope document and every scan in this report.

## (b) Live Hosts Discovered

| IP Address | MAC Address | Vendor | Notes |
|---|---|---|---|
| 192.168.42.1 | 00:50:56:C0:00:01 | VMware | Virtual network gateway |
| 192.168.42.128 | (Kali's own interface) | — | Kali Linux (attacker box, no MAC shown for local interface) |
| 192.168.42.129 | 00:0C:29:FA:DD:2A | VMware | **Metasploitable2 (target)** |
| 192.168.42.254 | 00:50:56:ED:83:36 | VMware | VMware DHCP server |

**Full scan output:**
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-15 08:29 -0400
Nmap scan report for 192.168.42.1
Host is up (0.0022s latency).
MAC Address: 00:50:56:C0:00:01 (VMware)
Nmap scan report for 192.168.42.129
Host is up (0.0016s latency).
MAC Address: 00:0C:29:FA:DD:2A (VMware)
Nmap scan report for 192.168.42.254
Host is up (0.00038s latency).
MAC Address: 00:50:56:ED:83:36 (VMware)
Nmap scan report for 192.168.42.128
Host is up.
Nmap done: 256 IP addresses (4 hosts up) scanned in 11.01 seconds
## (c) Why Host Discovery Precedes Port Scanning in the PTES Intelligence-Gathering Phase

Host discovery is performed before port scanning because port scanning every possible port against every possible address in a /24 range (256 addresses × up to 65,535 ports) is expensive and largely wasted effort against hosts that are not even online — most of a subnet's address space is typically unassigned or offline at any given time. By first identifying which of the 256 possible addresses actually respond (in this case, 4 out of 256), the tester narrows the actual target list before committing time and network traffic to the much more expensive port-scanning and service-enumeration phase (Task 4). This mirrors the PTES Intelligence Gathering phase's core principle of progressively narrowing scope — from the full authorized range, to confirmed live hosts, to confirmed open services on those hosts — so that later, more intrusive techniques are only ever pointed at targets already known to exist, reducing both scan time and unnecessary network noise.

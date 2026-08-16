Task 5 — DNS Enumeration
DNS Server Setup

A local authoritative DNS server was configured on the Kali host using dnsmasq (v2.93), serving the lab domain lab.local.

dnsmasq.conf directives (relevant excerpt):

domain=lab.local
local=/lab.local/
address=/lab.local/192.168.42.129
host-record=www.lab.local,192.168.42.129
mx-host=lab.local,mail.lab.local,10
txt-record=lab.local,"v=spf1 -all"
cname=ftp.lab.local,www.lab.local
auth-zone=lab.local
auth-server=lab.local,eth0
Directive	Purpose
domain=lab.local	Declares the domain served by this instance.
local=/lab.local/	Restricts resolution of lab.local to locally-known records only (no forwarding upstream for this zone).
address=/lab.local/192.168.42.129	Provides the A record — resolves lab.local (and subdomains) to the Metasploitable host.
host-record=www.lab.local,192.168.42.129	Explicit A record for www.lab.local.
mx-host=lab.local,mail.lab.local,10	MX record for the domain, priority 10, pointing to mail.lab.local.
txt-record=lab.local,"v=spf1 -all"	TXT record containing a sample SPF policy.
cname=ftp.lab.local,www.lab.local	CNAME record aliasing ftp.lab.local to www.lab.local.
auth-zone=lab.local / auth-server=lab.local,eth0	Puts dnsmasq into authoritative mode for the lab.local zone, served on the eth0 interface.

The Kali host (192.168.42.128) acts as its own authoritative resolver for this exercise (queries directed at 127.0.0.1 / the local dnsmasq instance).

(a) All DNS Record Types Retrieved via dig

A record:

$ dig A www.lab.local @127.0.0.1
;; ANSWER SECTION:
www.lab.local.          600     IN      A       192.168.42.129

MX record:

$ dig MX lab.local @127.0.0.1
;; ANSWER SECTION:
lab.local.              600     IN      MX      10 mail.lab.local.

NS record:

$ dig NS lab.local @127.0.0.1
;; flags: qr aa rd ra;
;; ANSWER SECTION:
lab.local.              600     IN      NS      lab.local.

(Note the aa — Authoritative Answer — flag, confirming dnsmasq is authoritative for this zone.)

TXT record:

$ dig TXT lab.local @127.0.0.1
;; ANSWER SECTION:
lab.local.              600     IN      TXT     "v=spf1 -all"

CNAME record:

$ dig CNAME ftp.lab.local @127.0.0.1
;; ANSWER SECTION:
ftp.lab.local.          600     IN      CNAME   www.lab.local.

All five required record types (A, MX, NS, TXT, CNAME) were successfully retrieved from the authoritative server.

(b) Zone Transfer (AXFR) Attempt

Command:

$ dig axfr lab.local @127.0.0.1

Result:

; <<>> DiG 9.20.23-1-Debian <<>> axfr lab.local @127.0.0.1
;; global options: +cmd
; Transfer failed.

Security conclusion: The zone transfer failed, meaning AXFR is not permitted by this dnsmasq configuration by default. This is the secure and expected outcome — dnsmasq does not implement AXFR the way a full DNS server like BIND9 does, so unauthenticated zone transfers are effectively blocked out of the box. Had AXFR succeeded (as can happen with a misconfigured BIND9 server with an open allow-transfer directive), an attacker would gain the entire zone file in a single query — every hostname, subdomain, internal naming convention, and IP mapping in the domain — without needing to guess or brute-force individual names. This is a well-known high-severity DNS misconfiguration because it converts a domain's internal structure from something an attacker must painstakingly enumerate into something they can dump in one request. The fact that this lab's AXFR attempt failed demonstrates a secure-by-default posture for this record type.

(c) Additional DNS Enumeration Technique — Reverse DNS Lookup

Command:

$ dig -x 192.168.42.129 @127.0.0.1

Result:

;; flags: qr aa rd ra;
;; QUESTION SECTION:
;129.42.168.192.in-addr.arpa.   IN      PTR
;; ANSWER SECTION:
129.42.168.192.in-addr.arpa. 0  IN      PTR     www.lab.local.

Reverse DNS (PTR) resolution of the target IP 192.168.42.129 successfully returns www.lab.local, confirming the forward/reverse mapping is consistent. In a real-world engagement, systematic reverse lookups across a target's IP range are a low-noise way to discover hostnames that may not otherwise be advertised in forward zone records, effectively mapping infrastructure without needing the zone file directly.



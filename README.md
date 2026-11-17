# Enterprise Network Security Lab — Cisco IOS IPS Deployment
## What This Project Is
A three-router enterprise network built from scratch in Cisco Packet Tracer,
secured with Cisco IOS Intrusion Prevention System (IPS).
The IPS inspects traffic entering the 192.168.1.0 network, matches ICMP Echo
Request signatures, drops the packet, and sends alerts to a Syslog server.
The ping fails. The alert appears. That's IPS — prevention with proof.
Every cable, interface, and command was configured from an empty canvas and
verified at each step. No shortcuts.
---
## Topology
| Device | Model | Role |
|--------|-------|------|
| R1 | Cisco 1941 | Left LAN gateway + IPS inspection point |
| R2 | Cisco 1941 | Middle router joining two serial links |
| R3 | Cisco 1941 | Right LAN gateway |
| Left-SW | Cisco 2960 | Connects PC-A and Syslog Server |
| Right-SW | Cisco 2960 | Connects PC-B |
| PC-A | PC-PT | Left LAN host |
| PC-B | PC-PT | Right LAN host |
| Syslog Server | Server-PT | Centralized log destination |
---
## Network Addressing
| Device | Interface | IP Address | Mask |
|--------|-----------|------------|------|
| R1 | Gi0/0 | 192.168.1.1 | 255.255.255.0 |
| R1 | Serial0/1/0 | 10.1.1.1 | 255.255.255.252 |
| R2 | Serial0/1/0 | 10.1.1.2 | 255.255.255.252 |
| R2 | Serial0/1/1 | 10.2.2.1 | 255.255.255.252 |
| R3 | Gi0/0 | 192.168.3.1 | 255.255.255.0 |
| R3 | Serial0/1/1 | 10.2.2.2 | 255.255.255.252 |
| PC-A | Fa0 | 192.168.1.2 | 255.255.255.0 |
| PC-B | Fa0 | 192.168.3.2 | 255.255.255.0 |
| Syslog Server | Fa0 | 192.168.1.50 | 255.255.255.0 |
---
## Cabling Reference
### Copper Straight-Through
| Source | Port | Destination | Port |
|--------|------|-------------|------|
| PC-A | Fa0 | Left-SW | Fa0/1 |
| Syslog Server | Fa0 | Left-SW | Fa0/2 |
| Left-SW | Fa0/3 | R1 | Gi0/0 |
| PC-B | Fa0 | Right-SW | Fa0/1 |
| Right-SW | Fa0/2 | R3 | Gi0/0 |
### Serial DCE (Red Curved)
| Source | Port | Destination | Port |
|--------|------|-------------|------|
| R1 | Serial0/1/0 | R2 | Serial0/1/0 |
| R2 | Serial0/1/1 | R3 | Serial0/1/1 |
Port numbers must match on both ends.
### DCE/DTE Identification
| Link | DCE End | Clock Rate On |
|------|---------|---------------|
| R1–R2 | R1 Serial0/1/0 | R1 |
| R2–R3 | R2 Serial0/1/1 | R2 |
---
## Part 1 — Network Configuration
### R1 — Complete Block
enable
configure terminal
hostname R1
interface gigabitEthernet 0/0
ip address 192.168.1.1 255.255.255.0
no shutdown
exit
interface serial 0/1/0
ip address 10.1.1.1 255.255.255.252
clock rate 64000
no shutdown
exit
router rip
version 2
no auto-summary
network 192.168.1.0
network 10.0.0.0
exit
enable password level 15 cisco1
end
write memory
text
### R2 — Complete Block
enable
configure terminal
hostname R2
interface serial 0/1/0
ip address 10.1.1.2 255.255.255.252
no shutdown
exit
interface serial 0/1/1
ip address 10.2.2.1 255.255.255.252
clock rate 64000
no shutdown
exit
router rip
version 2
no auto-summary
network 10.0.0.0
exit
enable password level 15 cisco1
end
write memory
text
### R3 — Complete Block
enable
configure terminal
hostname R3
interface gigabitEthernet 0/0
ip address 192.168.3.1 255.255.255.0
no shutdown
exit
interface serial 0/1/1
ip address 10.2.2.2 255.255.255.252
no shutdown
exit
router rip
version 2
no auto-summary
network 192.168.3.0
network 10.0.0.0
exit
enable password level 15 cisco1
end
write memory
text
### End Devices
**PC-A:** 192.168.1.2 / 255.255.255.0 / 192.168.1.1
**PC-B:** 192.168.3.2 / 255.255.255.0 / 192.168.3.1
**Syslog Server:** 192.168.1.50 / 255.255.255.0 / 192.168.1.1
Enable Syslog: Services tab → Syslog → ON.
---
## Part 2 — IPS Configuration
### License Activation — R1
enable
show version
text
Security row reads `None`.
configure terminal
license boot module c1900 technology-package securityk9
text
Type `yes`.
end
copy running-config startup-config
reload
text
Press Enter.
After reboot:
enable
! password: cisco1
show version
text
Security row shows `securityk9`.
### IPS Configuration — R1 — Complete Block
enable
mkdir ipsdir
! press Enter
configure terminal
ip ips config location flash:ipsdir
ip ips name iosips
ip ips notify log
service timestamps log datetime msec
logging host 192.168.1.50
exit
ip ips signature-category
category all
retired true
exit
category ios_ips basic
retired false
exit
exit
interface gigabitEthernet 0/0
ip ips iosips out
exit
ip ips signature-definition
signature 2004 0
status
retired false
enabled true
exit
engine
event-action produce-alert
event-action deny-packet-inline
exit
exit
exit
end
show ip ips all
write memory
text
### R2 and R3 — Save Only
enable
write memory
text
---
## Verification Results
### show ip ips all — Key Sections
IPS Signature Configuration:
Signature 2004:0
Enabled: true
Retired: false
Event Action: deny-packet-inline
Event Action: produce-alert
IPS Rule Configuration:
Rule Name: iosips
IPS Configuration:
Config Location: flash:ipsdir
Interface Configuration:
GigabitEthernet0/0
Inbound IPS rule: None
Outbound IPS rule: iosips
text
### PC-B → PC-A — Ping Fails (IPS blocks)
Pinging 192.168.1.2 with 32 bytes of data:
Request timed out.
Request timed out.
Request timed out.
Request timed out.
Packets: Sent = 4, Received = 0, Lost = 4 (100% loss)
text
### PC-A → PC-B — Ping Succeeds
Pinging 192.168.3.2 with 32 bytes of data:
Reply from 192.168.3.2: bytes=32 time=1ms TTL=126
Reply from 192.168.3.2: bytes=32 time=1ms TTL=126
Reply from 192.168.3.2: bytes=32 time=1ms TTL=126
Reply from 192.168.3.2: bytes=32 time=1ms TTL=126
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
text
### Syslog Server — Alerts Appear
192.168.1.1 %IPS-4-SIGNATURE: Sig:2004 Subsig:0 Sev:2 ...
192.168.1.1 %IPS-3-SIGNATURE: Sig:2004 Subsig:0 Sev:2 ...
text
---
## IPS vs IDS — The One Command Difference
| Action | IDS | IPS |
|--------|-----|-----|
| `event-action produce-alert` | Yes | Yes |
| `event-action deny-packet-inline` | No | **Yes** |
Our configuration has both actions. The packet is dropped. The alert fires.
---
## Troubleshooting
| Symptom | Fix |
|---------|-----|
| Serial `down/down` | Check cable is in matching port on both routers |
| Serial `up/down` | Move `clock rate 64000` to the DCE end |
| No serial interfaces | Install HWIC-2T with router powered off |
| Security stays `None` | `copy run start` then `reload` |
| `% Invalid input` on `ip ips config location` | Enter `configure terminal`
first |
| No Syslog alerts | Apply `logging host 192.168.1.50` |
| No Syslog alerts | Services → Syslog → confirm ON |
| Ping succeeds when it should fail | Add `event-action deny-packet-inline` to
signature 2004 |
| No alerts at all | Check `show ip ips signature-definition` — signature 2004
must be enabled |
| Part 1 ping fails | Wait 30 seconds for RIP or run `clear ip route *` |
---
## Success Criteria
| Check | Command | Required Result | Status |
|-------|---------|-----------------|--------|
| Interfaces up | `show ip interface brief` | Gi0/0 and serials show `up up` |
✅ |
| RIP routes | `show ip route` | Each LAN visible as `R` or `C` | ✅ |
| Part 1 ping | PC-A → PC-B | 4/4 replies, TTL=126 | ✅ |
| License active | `show version` | Security row shows `securityk9` | ✅ |
| IPS configured | `show ip ips all` | Rule `iosips`, signature 2004 with deny
and alert | ✅ |
| IPS blocks inbound | PC-B → PC-A | 100% loss | ✅ |
| IPS allows outbound | PC-A → PC-B | 4/4 replies, TTL=126 | ✅ |
| Alerts arrive | Syslog Server → Services → Syslog | `%IPS-4-SIGNATURE:
Sig:2004` entries | ✅ |
All checks pass. Project verified.
---
## Three Rules That Guarantee Success
| # | Rule | Wrong Result |
|---|------|--------------|
| 1 | Clock rate 64000 on DCE ends only | Serial stuck down |
| 2 | /24 on LANs, /30 on serials | No cross-network connectivity |
| 3 | Signature 2004 with both `produce-alert` and `deny-packet-inline` |
Missing deny builds IDS, not IPS |
---
## Files Included
cisco-ios-ips-enterprise-lab/
│
├── README.md
├── enterprise-network-ips.pkt
│
├── configs/
│ ├── R1-config.txt
│ ├── R2-config.txt
│ └── R3-config.txt
│
└── screenshots/
├── 01-topology.png
├── 02-show-version-security.png
├── 03-ping-before-ips.png
├── 04-show-ips-all.png
├── 05-ping-blocked.png
├── 06-ping-allowed.png
└── 07-syslog-alerts.png
text
---
## Skills Demonstrated
- Cisco IOS CLI configuration
- RIPv2 dynamic routing
- Serial DCE/DTE clocking
- Cisco IOS IPS deployment
- Signature-based traffic inspection
- Packet dropping with `deny-packet-inline`
- Syslog centralized logging
- Network security policy enforcement
- End-to-end verification and troubleshooting
---
## Author
**Axton** — 14 years old
Networking and Cybersecurity Lab Portfolio
---
## License
Educational purposes.
---
Built from scratch. Verified end to end. Blocked and logged. That's IPS.

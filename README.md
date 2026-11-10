# IT Homelab Portfolio

Enterprise-style IT homelab: 8 hands-on projects covering networking, Active Directory, Linux services, packet analysis, firewalls/IDS, Python automation, SIEM/SOC, and troubleshooting. Built end-to-end with free tools.

These aren't 8 separate labs. They are **one environment built in layers**, where each project runs on top of the one before it. Every project folder has its own write-up with an architecture diagram, a step-by-step build, screenshots, and a "Problems Encountered & Fixes" section covering what actually went wrong and how it was fixed.

## Project Tracker

| # | Project | Status | What it demonstrates |
|---|---|---|---|
| 1 | [Multi-Subnet Enterprise Network](01-enterprise-network/) | ✅ Complete | Subnetting, 4 segmented zones, routing with RRAS, firewall policy between zones, tested rule by rule |
| 2 | [Active Directory](02-active-directory/) | ✅ Complete | Domain controller, DNS for a multi-homed DC, OUs and groups, Group Policy, least-privilege delegation |
| 3 | [Linux Services Network](03-linux-services/) | 🔄 In progress | DNS (BIND9), DHCP with relay, NTP (chrony) and a web server on Ubuntu, integrated with the domain |
| 4 | [Wireshark Traffic Analysis](04-wireshark-analysis/) | ⬜ Not started | Capturing and explaining DNS, DHCP, Kerberos, SMB and HTTPS traffic |
| 5 | [Firewall (pfSense)](05-firewall-pfsense/) | ⬜ Not started | Stateful firewall and real 802.1Q VLANs replacing the Project 1 design |
| 6 | [Python + Networking](06-python-networking/) | ⬜ Not started | Scripts that automate and monitor the environment |
| 7 | [SOC (Wazuh)](07-soc-wazuh/) | ⬜ Not started | Centralized logging and detections for failed logons, new admins, PowerShell and port scans |
| 8 | [Troubleshooting Challenge](08-troubleshooting-challenge/) | ⬜ Not started | Breaking the environment on purpose and fixing it, written up as incident reports |

## How the layers fit together

1. **Network foundation:** four isolated segments (Servers, Workstations, Management, Guest) routed by a Windows Server, with firewall rules between them.
2. **Active Directory:** the same server becomes the domain controller for `corp.local`, with a Windows 11 client joined across a routed segment.
3. **Linux services:** DNS, DHCP, NTP and web servers that the domain and the clients depend on.
4. **Wireshark:** analysing the traffic all of the above generates.
5. **pfSense:** replacing the basic router and firewall with a real stateful one.
6. **Python:** automating and monitoring what was built.
7. **Wazuh:** a SOC watching the logs from all of it.
8. **Troubleshooting:** intentional breakage to prove the environment is understood, not just followed.

## Environment

- VMware Workstation Pro on a Windows host
- Windows Server 2022 (evaluation), Windows 11, Ubuntu Server
- All software is free or open source

## Notes on this repository

Projects 1 and 2 were rebuilt from scratch as **v2** after earning CompTIA Network+, to deepen understanding and tighten the documentation. The original builds are preserved under the `v1-archive` tag.

The Project 1 network uses four separate virtual switches, because VMware Workstation can't carry 802.1Q VLAN tags. True VLAN tagging with a stateful firewall is built in Project 5.

## Repository layout

```
it-homelab-portfolio/
├── README.md                  ← this page
├── 01-enterprise-network/     ← README, diagrams/, screenshots/
├── 02-active-directory/       ← README, screenshots/
├── 03-linux-services/
├── 04-wireshark-analysis/
├── 05-firewall-pfsense/
├── 06-python-networking/
├── 07-soc-wazuh/
└── 08-troubleshooting-challenge/
```

VM disks and ISOs are not stored here (see `.gitignore`); everything text-based is.

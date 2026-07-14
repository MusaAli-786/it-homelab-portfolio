# Project 1 — Multi-Subnet Enterprise Network

## Overview
A segmented enterprise network built entirely in VMware Workstation Pro, isolating traffic into four distinct zones (Servers, Workstations, Management, Guest) with routing and firewall policy controlling what's allowed between them — mirroring how a real business network is designed rather than running everything on one flat network.

## Objectives
- Design a realistic private IP addressing scheme across 4 network segments
- Create isolated virtual network switches for each segment
- (Next) Route between segments using a central Windows Server
- (Next) Enforce firewall rules limiting cross-segment traffic

## Tools & Technologies
- VMware Workstation Pro (virtual networking / host-only switches)
- Windows Server (evaluation) — RRAS role (planned)
- Windows Defender Firewall with Advanced Security (planned)

## Build Process
1. **Designed the IP addressing scheme** before touching any software — 4 non-overlapping /24 subnets, one per network zone:
   | Segment | Subnet | Gateway |
   |---|---|---|
   | Servers | 10.10.10.0/24 | 10.10.10.1 |
   | Workstations | 10.10.20.0/24 | 10.10.20.1 |
   | Management | 10.10.30.0/24 | 10.10.30.1 |
   | Guest | 10.10.40.0/24 | 10.10.40.1 |
2. **Created 4 isolated virtual networks** in VMware's Virtual Network Editor (VMnet2–VMnet5), each configured as **Host-only** — meaning these switches are visible only to VMs attached to them, with no bridge to the physical/home network. DHCP was intentionally left disabled on each, since IP addressing will be centrally managed by the Windows Server router (via RRAS) rather than VMware itself — matching how a real business network assigns addressing from a central authority, not the switching layer.

## Problems Encountered & Fixes
- (to be filled in as the build continues)

## Key Takeaways / Skills Demonstrated
- Private IP addressing design (RFC 1918) across multiple network segments
- Understanding of network isolation at the virtual-switch layer
- Documentation practices matching real network design handoffs
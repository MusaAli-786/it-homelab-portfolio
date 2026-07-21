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
3. **Built the DC01 virtual machine** — Windows Server 2022 Standard (Desktop Experience), 4GB RAM, 60GB disk, with 4 network adapters attached (one each to VMnet2-5). Chose Desktop Experience over Server Core specifically to use the graphical AD/GPO/DNS management tools while learning these concepts for the first time — a production-realistic Server Core install is a reasonable follow-up exercise once comfortable with the underlying concepts.
4. **Installed the OS via Custom (clean) install**, not Upgrade, matching how real servers are always freshly provisioned rather than upgraded in place.
5. **Installed VMware Tools** post-install for proper mouse/display integration between host and VM.
6. **Assigned static IP addresses** to all 4 of DC01's network adapters, matching the subnet design — 10.10.10.1 (Servers), 10.10.20.1 (Workstations), 10.10.30.1 (Management), 10.10.40.1 (Guest), each with subnet mask 255.255.255.0 and no default gateway configured (since DC01 itself serves as the gateway for each subnet). Adapters were first matched to their correct VMnet using MAC address cross-referencing between Windows (`Get-NetAdapter`) and VMware's adapter settings, then renamed to self-documenting names (e.g. "Servers-VMnet2").
7. **Installed and configured RRAS** (Routing and Remote Access Service) on DC01 via Server Manager's "Remote Access" role, then enabled it through the Routing and Remote Access console using Custom Configuration → LAN Routing.
8. **Built a throwaway test VM** (TEST-Workstation01, Ubuntu Server, single adapter on VMnet3/Workstations, static IP 10.10.20.50) specifically to validate the design from an ordinary endpoint's perspective rather than testing from DC01 itself.
9. **Verified routing with two ping tests:** a same-subnet ping to DC01 (10.10.20.1, ttl=128 — 0 hops, direct connectivity) and a cross-subnet ping to DC01's Servers-side adapter (10.10.10.1, ttl=127 — 1 hop) from a VM with no direct connection to the Servers network at all. The successful cross-subnet result is the real proof: it's only possible if DC01 is actively receiving traffic on one interface and forwarding it out another.

**Current state:** routing works between all 4 VLANs with no restrictions in place yet — this is intentional, establishing a clean baseline before firewall rules are added next to actually enforce which VLANs may reach which others.

## Problems Encountered & Fixes
- **Issue:** The VM initially failed to boot from the ISO — VMware showed "Cannot connect virtual device sata0:1" and later timed out during the "Press any key to boot from CD or DVD" prompt.
  **Fix:** Corrected the CD/DVD device setting in VM hardware config from "Auto detect" (which looks for a physical host DVD drive) to "Use ISO image file," pointed explicitly at the ISO's path, and confirmed "Connect at power on" was checked. On reboot, clicked into the VM window immediately and pressed a key right as the boot screen appeared to catch the narrow timing window.
  **Lesson:** VMware's CD/DVD default assumes a physical drive exists on the host — always explicitly point it at the ISO file rather than relying on auto-detect.
  - **Issue:** The DC01 VM initially failed to boot from the Windows Server ISO ("Cannot connect virtual device sata0:1", then a missed "press any key" boot prompt).
  **Fix:** Corrected the CD/DVD setting from "Auto detect" to "Use ISO image file" with "Connect at power on" checked; caught the boot prompt on retry by clicking into the VM immediately and pressing a key the moment the screen changed.
- **Issue:** While attaching DC01's network adapters, repeated "Add" clicks created 7 adapters instead of 4, with 3 extras defaulting to NAT/VMnet0.
  **Fix:** Reviewed the full adapter list against the design before finalizing, removed the 3 unwanted adapters individually.
- **Issue:** Windows' generic adapter names ("Ethernet0/1/2/3") gave no indication of which VMnet each was actually plugged into.
  **Fix:** Cross-referenced MAC addresses between `Get-NetAdapter` (Windows) and each adapter's Advanced settings (VMware) to match them correctly, then renamed each adapter to a self-documenting name (e.g. "Servers-VMnet2").

## Key Takeaways / Skills Demonstrated
- Private IP addressing design (RFC 1918) across multiple network segments
- Understanding of network isolation at the virtual-switch layer
- Documentation practices matching real network design handoffs
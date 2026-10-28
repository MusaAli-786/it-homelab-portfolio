# Project 1 — Multi-Subnet Enterprise Network (v2)

> **Status: In progress.** Design, virtual networks, DC01 build, RRAS routing, and routing validation are complete. Firewall policy and rule validation are being added next.
>
> This is a from-scratch rebuild (v2) done after earning CompTIA Network+. The original build is preserved under the `v1-archive` tag of this repository.

## Overview
A segmented enterprise network built in VMware Workstation Pro, isolating traffic into four zones — Servers, Workstations, Management, and Guest — with a central Windows Server acting as the router between them. This mirrors how real business networks are designed: instead of one flat network where every device can reach every other device, traffic between zones passes through a single controllable checkpoint where policy can be enforced.

## Architecture Diagram
![Network diagram](diagrams/01-network-diagram.png)

*DC01 is a multi-homed router with one interface in each segment. Each interface address (.1) is the default gateway for the devices on that segment.*

## Objectives
- Design a private IP addressing scheme across 4 network segments
- Build an isolated virtual switch for each segment
- Build a multi-homed Windows Server to act as the router between segments (RRAS)
- Enforce firewall policy limiting which segments can reach which others
- Prove the design with real connectivity tests, not just configuration

## Tools & Technologies
- VMware Workstation Pro (host-only virtual networking)
- Windows Server 2022 Standard (Desktop Experience)
- RRAS — Routing and Remote Access Service
- RRAS IP Packet Filters *(upcoming)*
- Ubuntu Server (throwaway test VMs for validation)
- draw.io (network diagram)

## IP Addressing Scheme
| Segment | VMware Network | Subnet | Gateway (DC01) |
|---|---|---|---|
| Servers | VMnet2 | 10.10.10.0/24 | 10.10.10.1 |
| Workstations | VMnet3 | 10.10.20.0/24 | 10.10.20.1 |
| Management | VMnet4 | 10.10.30.0/24 | 10.10.30.1 |
| Guest | VMnet5 | 10.10.40.0/24 | 10.10.40.1 |

Each /24 leaves 8 host bits (254 usable addresses). The third octet identifies the segment, which keeps the scheme easy to read. The network address (.0) and broadcast address (.255) are reserved in every subnet.

## DC01 Interface Mapping
| Windows NIC Name | VMware Adapter | VMnet | Segment | MAC Address | IP Address |
|---|---|---|---|---|---|
| Servers-VMnet2 | Network Adapter 1 / Ethernet0 | VMnet2 | Servers | 00:0C:29:BF:90:8B | 10.10.10.1/24 |
| Workstations-VMnet3 | Network Adapter 2 / Ethernet1 | VMnet3 | Workstations | 00:0C:29:BF:90:95 | 10.10.20.1/24 |
| Management-VMnet4 | Network Adapter 3 / Ethernet2 | VMnet4 | Management | 00:0C:29:BF:90:9F | 10.10.30.1/24 |
| Guest-VMnet5 | Network Adapter 4 / Ethernet3 | VMnet5 | Guest | 00:0C:29:BF:90:A9 | 10.10.40.1/24 |

## Build Process

### 1. Design the network before touching any software
Four non-overlapping /24 subnets from private (RFC 1918) address space, one per zone, drawn in draw.io first. Planning the addressing up front means every later problem is either a typo or a design flaw, and the diagram makes it easy to tell which.

### 2. Create the four virtual networks
In VMware's Virtual Network Editor, created VMnet2–VMnet5 as host-only networks with the subnets above and renamed each after its segment. VMware's local DHCP service was disabled on every network, and no host adapter was attached, so the lab is fully isolated from the host PC and addressing is controlled manually.

Each VMnet is a separate virtual switch — an isolated Layer 2 segment.

![VMware Virtual Network Editor](screenshots/02-vmnet-editor.png)

### 3. Build DC01
Created a Windows Server 2022 Standard (Desktop Experience) VM with 2 vCPUs, 4 GB RAM, and a 60 GB disk, with four network adapters, one attached to each VMnet. Desktop Experience was chosen deliberately to use the graphical AD, GPO, and DNS tools in later projects.

![DC01 hardware settings](screenshots/03-dc01-vm-settings.png)

### 4. Match and rename the adapters
All four virtual NICs look identical inside Windows, so each one was matched to its VMnet by MAC address: the MAC listed under each adapter's Advanced settings in VMware was compared against `Get-NetAdapter` output in Windows. Each adapter was then renamed to a self-documenting name.

```powershell
Get-NetAdapter | Select Name, MacAddress

Rename-NetAdapter -Name "Ethernet0" -NewName "Servers-VMnet2"
Rename-NetAdapter -Name "Ethernet1" -NewName "Workstations-VMnet3"
Rename-NetAdapter -Name "Ethernet2" -NewName "Management-VMnet4"
Rename-NetAdapter -Name "Ethernet3" -NewName "Guest-VMnet5"
```

![Renamed adapters with MAC addresses](screenshots/04-dc01-nic-names.png)

### 5. Assign static IP addresses
Each adapter received the .1 address for its subnet with a /24 prefix. **No default gateway was set on any adapter**, because DC01 is itself the gateway for every segment — it has nowhere further to forward unknown traffic.

```powershell
New-NetIPAddress -InterfaceAlias "Servers-VMnet2"      -IPAddress 10.10.10.1 -PrefixLength 24
New-NetIPAddress -InterfaceAlias "Workstations-VMnet3" -IPAddress 10.10.20.1 -PrefixLength 24
New-NetIPAddress -InterfaceAlias "Management-VMnet4"   -IPAddress 10.10.30.1 -PrefixLength 24
New-NetIPAddress -InterfaceAlias "Guest-VMnet5"        -IPAddress 10.10.40.1 -PrefixLength 24
```

Assigning an address to each adapter also gives DC01 a connected route to each subnet automatically, so it knows all four networks directly.

![Static IP configuration](screenshots/05-dc01-static-ips.png)

### 6. Rename the server
Renamed the computer to match the diagram.

```powershell
Rename-Computer -NewName "DC01" -Restart
```

![Hostname confirmed](screenshots/06-dc01-hostname.png)

### 7. Install and configure RRAS
Installed the **Remote Access** role with the **Routing** role service through Server Manager. Installing the role only adds the software, so RRAS was then enabled in the **Routing and Remote Access** console: right-click DC01 → *Configure and Enable Routing and Remote Access* → **Custom configuration** → **LAN routing** → start the service.

The why: until RRAS is enabled, Windows receives a packet on one interface that is addressed to another network and simply drops it. Enabling RRAS turns on IP forwarding, which is what makes DC01 a router.

![RRAS enabled with all four interfaces](screenshots/07-rras-enabled.png)

### 8. Prove routing with test VMs
Testing from DC01 only proves it can reach its own interfaces. To prove it forwards traffic between segments, two throwaway Ubuntu Server VMs were placed on different networks, each using DC01 as its default gateway:

| VM | VMnet | Address | Default gateway |
|---|---|---|---|
| TestVM-Workstations | VMnet3 | 10.10.20.50/24 | 10.10.20.1 |
| TestVM-Servers (full clone) | VMnet2 | 10.10.10.50/24 | 10.10.10.1 |

There is no DHCP on these networks, so both VMs use static addresses.

![TestVM-Workstations hardware](screenshots/08-testvm-workstations-settings.png)

**Same-subnet test.** The first ping from TestVM-Workstations to DC01's interface (10.10.20.1) failed with 100% packet loss:

![Ping to DC01 failing](screenshots/09-ping-same-subnet-blocked.png)

`ip neigh show 10.10.20.1` showed DC01's MAC address, so ARP (Layer 2) was working and only ICMP was being dropped. Windows Server blocks inbound ping by default, so the echo-request rule was enabled on DC01:

```powershell
Enable-NetFirewallRule -DisplayName "File and Printer Sharing (Echo Request - ICMPv4-In)"
```

![ICMP echo rule enabled](screenshots/10-dc01-icmp-rule-enabled.png)

The ping then succeeded with `ttl=128`, the default for Windows:

![Same-subnet ping success](screenshots/11-ping-same-subnet-success.png)

**Cross-subnet test.** From TestVM-Workstations, pinged TestVM-Servers on a network it has no direct connection to:

```bash
ping -c 4 10.10.10.50
```

The replies came back with `ttl=63`. Linux starts at 64, and DC01 decremented it by 1 as it routed the packet from VMnet3 to VMnet2, so the TTL shows exactly one router hop. That confirms DC01 is forwarding between the isolated segments rather than just answering on its own interfaces.

![Cross-subnet ping success](screenshots/12-ping-cross-subnet.png)
### 9. Implement firewall policy with RRAS packet filters *(upcoming)*
### 10. Validate every rule *(upcoming)*

## Problems Encountered & Fixes
- **VMware wizard detected the wrong operating system.** The New Virtual Machine wizard remembered the Windows 11 ISO from the previous build and labelled the new VM "Windows 11 x64," with the wrong name and folder. Fixed by choosing *I will install the operating system later*, setting the guest OS to Windows Server 2022, and attaching the Server 2022 ISO afterward in the hardware settings.
- **Windows adapter order did not match VMware adapter order.** `Get-NetAdapter` listed the adapters in a different order from the VMware adapter numbering, so counting down the list would have mapped them to the wrong segments. MAC address matching resolved it, and the adapters were renamed immediately to avoid repeating the exercise.
- **Same-subnet ping to DC01 failed with 100% packet loss.** The static address and default route on the test VM were correct, and `ip neigh` showed DC01's MAC address, which ruled out a Layer 1/2 problem. The cause was DC01's own Windows firewall, which blocks inbound ICMP echo by default. Fixed by enabling the "File and Printer Sharing (Echo Request - ICMPv4-In)" rule. This rule only affects traffic addressed to DC01 itself; policy for traffic routed *between* segments is handled separately by RRAS packet filters.
- **Default gateway appeared to clear in the Ubuntu installer's network screen.** After entering the gateway and reopening the interface settings, the field looked empty. Verified after installation with `ip route` that the default route via 10.10.20.1 was in place before continuing.
- **Cloned VM failed to apply its new network config.** `netplan apply` returned "Cannot find unique matching interface for ens33." The installer had pinned the config to the original VM's MAC address with a `match: macaddress` block, and the clone received a new MAC from VMware. Fixed by removing the `match` and `set-name` lines so the config applies by interface name, then confirming the new address with `ip -br a`.

## Key Takeaways / Skills Demonstrated
*(To be completed when the project is finished.)*
- Private IP addressing design (RFC 1918) and subnetting across multiple segments
- Layer 2 segmentation with isolated virtual switches versus Layer 3 addressing and gateways
- Identifying network interfaces by MAC address and documenting an interface-to-segment mapping
- Building a multi-homed Windows Server for routing
- Proving a packet crossed a router using TTL (64 → 63 for Linux-to-Linux, 128 for a direct Windows reply)
- Separating Layer 2 problems (ARP, checked with `ip neigh`) from Layer 3 and ICMP problems when troubleshooting
- Configuring static addressing and a default route on Ubuntu with netplan

## Current Scope Note
True 802.1Q VLAN tagging is not used here. VMware Workstation does not support VLAN trunking, so this design uses four separate virtual switches rather than one trunked switch. Real VLAN tagging will be implemented in Project 5 using pfSense.

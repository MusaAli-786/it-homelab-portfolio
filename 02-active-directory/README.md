# Project 2 — Active Directory

## Overview
Building a real Active Directory domain on top of the Project 1 network — 
centralized identity management, Group Policy, and proper permission delegation.

## Progress Log

### Domain Controller Promotion
- Installed the AD DS server role on DC01, then promoted it via the AD DS 
  Configuration Wizard
- Created a new forest and domain: `corp.local`
- DC01 renamed from default hostname prior to promotion to keep it consistent 
  with lab naming conventions and avoid post-promotion rename complexity
- Verified with `dcdiag` — all tests passed
- DC01 now serves as domain controller, DNS server, and RRAS router 
  simultaneously — a valid small-business consolidation pattern, though 
  enterprise environments typically separate these roles across dedicated 
  servers

**Screenshots:**

![Promotion wizard review options](screenshots/01-adds-wizard-review-options.png)

![ADUC - corp.local domain tree](screenshots/02-aduc-corp-local-tree.png)

![CORP\Administrator login](screenshots/03-corp-administrator-login.png)

![DC01 registered as Domain Controller](screenshots/04-aduc-domain-controllers-dc01.png)

**Verification:**

![Get-Service health check](screenshots/05-get-service-health-check.png)

![Functional verification - ADWS/WinRM](screenshots/06-functional-verification.png)

  ### Problems Encountered & Fixes
**dcdiag flagged SystemLog test failures (ADWS/WinRM) after DC promotion**
- After promoting DC01, `dcdiag` repeatedly failed its SystemLog test, citing 
  WinRM SPN registration issues and an ADWS startup timeout (45s)
- Confirmed via `Get-Service` that all relevant services were actually Running, 
  despite the log language suggesting otherwise
- Rather than trust the diagnostic tool blindly, tested the actual functionality 
  directly: `Get-ADUser` (uses ADWS) returned valid results, `Test-WSMan` 
  confirmed WinRM was listening correctly
- Root cause: dcdiag's SystemLog test scans for stale error-log entries from a 
  resource-constrained boot (AD/DNS/RRAS/ADWS/WinRM all competing for startup 
  resources), not live service health — a one-time slow start, not an ongoing 
  failure
- No fix was required; verified functionally correct behavior over trusting a 
  diagnostic warning at face value

  ### Client VM Domain Join

- Built WS01 (Windows 11 Pro) on VMnet3 (Workstations, 10.10.20.0/24)
- Configured static IP, gateway, and DNS (pointed at DC01)
- Verified pre-join connectivity: ping to DC01 (0% loss) and `nslookup corp.local`
  (confirmed DNS resolution before attempting domain join)
- Renamed computer to WS01 and joined `corp.local` in a single step via
  System Properties
- Created a reverse lookup zone (`20.10.10.in-addr.arpa`) and ran
  `ipconfig /registerdns` on DC01 to populate a missing PTR record — resolved
  `nslookup` showing "Server: UnKnown" instead of "Server: DC01.corp.local"
- Verified domain membership from both sides: `whoami` on WS01 confirmed
  `corp\administrator` login; ADUC on DC01 shows WS01 registered under the
  default Computers container

**Screenshots:**

![WS01 ping test to DC01](screenshots/07-ws01-ping-dc01-test.png)

![WS01 nslookup corp.local (fixed)](screenshots/08-ws01-nslookup-corplocal-fixed.png)

![Domain join welcome message](screenshots/09-ws01-domain-join-welcome.png)

![WS01 registered in ADUC](screenshots/10-aduc-ws01-in-computers.png)

![WS01 whoami confirming domain login](screenshots/11-ws01-whoami-domain-login.png)

![DNS reverse lookup zone overview](screenshots/12-dns-reverse-zone-overview.png)

![DNS PTR record for DC01](screenshots/13-dns-ptr-record-dc01.png)

### Problems Encountered & Fixes

**Reverse DNS not resolving (nslookup showed "Server: UnKnown")**
- Created a reverse lookup zone, but no PTR record existed yet since the zone
  didn't exist when DC01 first came online
- Ran `ipconfig /registerdns` on DC01 to force re-registration; PTR record
  appeared and `nslookup` began correctly showing `Server: DC01.corp.local`

**Domain login failed with "Invalid credentials" despite correct password**
- Root cause: typed `CORP/Administrator` (forward slash) instead of the
  required `CORP\Administrator` (backslash) domain-login format
- Corrected the format; login succeeded, confirmed via `whoami`
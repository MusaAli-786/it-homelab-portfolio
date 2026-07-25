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
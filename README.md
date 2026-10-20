# Windows & Active Directory Lab

A hands-on Windows systems administration lab built with Oracle VM VirtualBox. This project documents a small Windows domain environment and the support tasks practiced in it: user and group administration, domain joining, Group Policy, and basic DNS/DHCP troubleshooting.

## Table of Contents
- [Overview](#overview)
- [Objectives](#objectives)
- [Lab Architecture](#lab-architecture)
- [Environment](#environment)
- [Implementation](#implementation)
- [Troubleshooting Runbook](#troubleshooting-runbook)
- [Validation Checklist](#validation-checklist)
- [Evidence and Screenshots](#evidence-and-screenshots)
- [Skills Demonstrated](#skills-demonstrated)
- [Safety and Limitations](#safety-and-limitations)

## Overview

This project uses one Windows Server virtual machine as a domain controller and two Windows client virtual machines to practice core Windows administration tasks in an isolated lab.

### Work documented
- Built a virtual environment with **3 virtual machines**.
- Managed **10 test user accounts across 2 groups**.
- Practiced password resets and account-lockout troubleshooting.
- Joined Windows client machines to a domain.
- Worked with user/group permissions and **2 Group Policy Objects (GPOs)**.
- Practiced basic DNS/DHCP configuration and troubleshooting.

This project demonstrates foundational skills relevant to entry-level IT Support, Help Desk, and Junior Systems Administration roles.

## Objectives

1. Understand how a Windows domain centralizes identity and access management.
2. Practice account-support tasks such as password resets and account-lockout investigation.
3. Join client computers to a domain and verify membership.
4. Apply and verify Group Policy in a controlled environment.
5. Use diagnostic tools to investigate DNS, DHCP, and domain connectivity issues.
6. Record repeatable procedures and troubleshooting results.

## Lab Architecture

![Windows & Active Directory Lab Network Architecture](windows-active-directory-lab-architecture.png)

```text
               Isolated Virtual Lab Network
                            |
             +--------------+--------------+
             |                             |
   +---------------------+       +---------------------+
   | Windows Server      |       | Windows Client 1    |
   | Domain Controller   |<----->| Domain-joined       |
   | AD DS / DNS          |       | Windows 10/11       |
   +---------------------+       +---------------------+
             |
             +--------------------+
                                  |
                       +---------------------+
                       | Windows Client 2   |
                       | Domain-joined      |
                       | Windows 10/11      |
                       +---------------------+

   Virtualization platform: Oracle VM VirtualBox
```

### Machine inventory

| Machine | Role | Operating system | Purpose |
|---|---|---|---|
| Server VM | Domain controller | REPLACE with exact version | Domain and identity administration |
| Client VM 1 | Domain client | REPLACE with exact version | Test domain sign-in, policies, and support tasks |
| Client VM 2 | Domain client | REPLACE with exact version | Second client for validation |

### Actual configuration

| Setting | Value |
|---|---|
| Domain DNS name | `REPLACE-WITH-YOUR-LAB-DOMAIN` |
| Server hostname | `REPLACE-WITH-SERVER-HOSTNAME` |
| Server IP address | `REPLACE-WITH-LAB-IP` |
| Client 1 hostname / IP | `REPLACE-WITH-VALUES` |
| Client 2 hostname / IP | `REPLACE-WITH-VALUES` |
| VirtualBox network mode | `REPLACE-WITH-ACTUAL-MODE` |
| DNS provider | `REPLACE-WITH-ACTUAL-SERVICE` |
| DHCP provider (if configured) | `REPLACE-WITH-ACTUAL-SERVICE-OR-NOT-CONFIGURED` |

## Environment

### Software
- Oracle VM VirtualBox
- One Windows Server virtual machine
- Two Windows 10/11 client virtual machines
- Sufficient host RAM, CPU, and storage for three VMs
- Administrative access to configure the lab

### Network considerations
- Keep the environment isolated from production networks.
- Ensure the clients can reach the domain controller over the lab network.
- Active Directory clients should use the DNS service configured for the domain so they can locate domain resources.
- Avoid competing DHCP services on the same lab segment unless this is intentional and documented.

## Implementation

### 1. Build the virtual machines
1. Create one Windows Server VM and two Windows client VMs in VirtualBox.
2. Assign each VM a clear hostname.
3. Connect the machines to the intended lab network.
4. Record OS versions, hostnames, IP addressing, and network mode in the table above.
5. Capture a screenshot of the VMs in VirtualBox.

**Evidence:** `screenshots/01-virtual-machines.png`

### 2. Configure the server and domain
1. Configure the server hostname and network settings.
2. Install and configure Active Directory Domain Services if performed in your lab.
3. Promote the server to a domain controller and record the lab domain name.
4. Configure DNS as required for the domain.
5. Verify that the domain controller is running and domain name resolution works.

**Evidence:** `screenshots/02-server-and-domain.png`

### 3. Create users and groups
1. Open **Active Directory Users and Computers**.
2. Create or identify the two test groups used in your lab.
3. Create 10 test user accounts and organize them according to your lab design.
4. Apply the intended group memberships.
5. Test account administration tasks, such as resetting a test user's password.
6. Practice investigating account lockout if you configured or encountered it.

**Evidence:** `screenshots/03-active-directory-users-groups.png`

### 4. Join client machines to the domain
1. Confirm each client can reach the domain controller.
2. Verify that each client uses the intended DNS server.
3. Join each client to the lab domain using an authorized account.
4. Restart if prompted.
5. Sign in with an appropriate test domain account.
6. Verify domain membership and record any errors encountered.

**Evidence:** `screenshots/04-domain-joined-client.png`

### 5. Configure and verify Group Policy
1. Open **Group Policy Management**.
2. Record each of the two GPOs' names, purpose, link location, and settings.
3. Apply GPOs only to the intended lab scope.
4. On a client, run `gpupdate /force`.
5. Use `gpresult /r` or `gpresult /h gpresult.html` to inspect applied policy.
6. Compare observed results with the intended configuration.

**Evidence:**
- `screenshots/05-group-policy-management.png`
- `screenshots/06-gpresult-client.png`

| GPO | Purpose | Scope / link location | Verification |
|---|---|---|---|
| GPO 1 — REPLACE with actual name | REPLACE with actual setting | REPLACE with actual scope | REPLACE with observed result |
| GPO 2 — REPLACE with actual name | REPLACE with actual setting | REPLACE with actual scope | REPLACE with observed result |

### 6. Configure and test DNS/DHCP
DNS checks can include identifying the configured DNS server and testing name resolution. DHCP checks can include reviewing the provider, scope/options (if configured), and the client's network configuration.

Useful client commands:

```powershell
ipconfig /all
nslookup <your-lab-domain>
ping <domain-controller-ip>
nltest /dsgetdc:<your-lab-domain>
```

If the client uses DHCP and it is safe to renew the lease:

```powershell
ipconfig /release
ipconfig /renew
```

A successful ping alone does not prove that DNS or Active Directory is working correctly.

**Evidence:** `screenshots/07-network-configuration.png`

## Troubleshooting Runbook

The following table outlines common diagnostic scenarios for Windows domain environments.

| Symptom | Checks | Possible causes |
|---|---|---|
| Client cannot join domain | Check IP settings, DNS, name resolution, reachability | Incorrect DNS, network isolation, typo, unavailable domain services |
| Domain user cannot sign in | Check username/domain, connectivity, account status | Incorrect credentials, locked/disabled account, connectivity |
| GPO does not apply | Run `gpupdate /force` and `gpresult /r`; check link and scope | Incorrect link, filtering, scope, refresh timing |
| Domain name does not resolve | Run `ipconfig /all` and `nslookup <your-lab-domain>` | Incorrect DNS server or record |
| Client has unexpected IP settings | Run `ipconfig /all`; check DHCP provider/lease | DHCP scope/options or client configuration |

### Troubleshooting case template

```text
Case ID: AD-TRBL-01
Issue:
Impact:
Initial checks:
Evidence / command output:
Root cause (confirmed or unconfirmed):
Resolution:
Validation after fix:
Prevention / lesson learned:
```

## Validation Checklist

- [ ] Three VMs and OS versions are documented.
- [ ] Server hostname and lab domain are recorded.
- [ ] Ten test users and two groups are documented.
- [ ] A test password reset was performed and checked.
- [ ] Both clients' domain membership was verified.
- [ ] Both GPOs and their actual settings are documented.
- [ ] Group Policy results were inspected on a client.
- [ ] DNS configuration and lookup results are recorded.
- [ ] DHCP setup and client lease were checked, if DHCP was configured.
- [ ] Troubleshooting cases include findings, cause, resolution, and validation.
- [ ] Genuine screenshots have been added and sensitive data removed.

## Evidence and Screenshots

| Filename | Suggested evidence |
|---|---|
| `01-virtual-machines.png` | Three VMs in VirtualBox |
| `02-server-and-domain.png` | Server identity and domain setup |
| `03-active-directory-users-groups.png` | Test users and groups |
| `04-domain-joined-client.png` | Client domain membership |
| `05-group-policy-management.png` | The two actual GPOs |
| `06-gpresult-client.png` | Policy results on a client |
| `07-network-configuration.png` | Relevant DNS/DHCP or IP configuration |

## Skills Demonstrated
- Windows Server and domain administration fundamentals
- Active Directory users and groups
- Password resets and account-lockout troubleshooting
- Domain joining and client validation
- Group Policy configuration and verification
- DNS and DHCP fundamentals
- Command-line network diagnostics
- Technical documentation and troubleshooting methodology

## Safety and Limitations
- This is a learning environment, not a production domain.
- Results may differ by Windows version and lab network mode.

## Repository Structure

```text
windows-active-directory-lab/
├── README.md
├── docs/
│   ├── lab-architecture.md
│   └── troubleshooting.md
└── screenshots/
    └── README.md
```

## Author

**Jericho Encinares**  

- GitHub: [@encinaresjericho](https://github.com/encinaresjericho)
- LinkedIn: [jericho-encinares](https://linkedin.com/in/jericho-encinares)

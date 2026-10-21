# Windows & Active Directory Lab

This project documents a small Active Directory environment with one Windows Server domain controller and two Windows client virtual machines. The reference design uses the domain `adlab.test` and subnet `192.168.10.0/24` to demonstrate centralized identity management, DNS, DHCP, Group Policy, domain joining, and structured troubleshooting.

## Project Overview

This lab is designed to practice Windows domain administration in an isolated virtual network. It uses one Windows Server virtual machine as a domain controller and two Windows client virtual machines to practice centralized user management, domain joining, Group Policy, and basic DNS/DHCP troubleshooting.

### Project at a glance

| Area | Reference configuration |
|---|---|
| Virtualization | Oracle VM VirtualBox |
| Lab design | One Windows Server VM and two Windows 11 Pro client VMs |
| Domain | `adlab.test` |
| Domain controller | `DC01` — `192.168.10.10` |
| Client computers | `CLIENT01` and `CLIENT02` |
| Test accounts | 10 fictional accounts across 2 security groups |
| Core technologies | Active Directory Domain Services (AD DS), Group Policy, DNS, and DHCP |
| Administration tasks | User and group management, password resets, domain joining, and account-lockout investigation |
| Troubleshooting focus | Domain sign-in, name resolution, client connectivity, and Group Policy application |
| Intended roles | IT Support, Help Desk, and Junior Systems Administration |

### Key skills

- Windows Server and Active Directory fundamentals
- User and group administration
- Domain joining and client validation
- Group Policy configuration and verification
- DNS/DHCP and network-connectivity troubleshooting
- Windows command-line diagnostics and technical documentation

## Table of Contents

- [Lab Architecture](#lab-architecture)
- [Environment](#environment)
- [Example Lab Configuration](#example-lab-configuration)
- [Implementation](#implementation)
- [Troubleshooting Runbook](#troubleshooting-runbook)
- [Validation Checklist](#validation-checklist)
- [Evidence and Screenshots](#evidence-and-screenshots)
- [Repository Structure](#repository-structure)
- [Lab Scope and Limitations](#lab-scope-and-limitations)

## Lab Architecture

![Windows & Active Directory Lab Network Architecture](windows-active-directory-lab-architecture.png)

The reference environment contains three virtual machines connected to an isolated VirtualBox **Internal Network** named `AD-Lab`.

```text
                    VirtualBox Internal Network: AD-Lab
                       Example subnet: 192.168.10.0/24

       +--------------------------+--------------------------+
       |                          |                          |
+------------------+     +------------------+      +------------------+
| DC01             |     | CLIENT01         |      | CLIENT02         |
| Windows Server   |     | Windows 11 Pro   |      | Windows 11 Pro   |
| AD DS / DNS /    |     | Domain client    |      | Domain client    |
| DHCP             |     | DHCP address     |      | DHCP address     |
| 192.168.10.10    |     | 192.168.10.101  |      | 192.168.10.102  |
+------------------+     +------------------+      +------------------+
```

### Machine inventory

| Machine | Role | Example operating system | Purpose |
|---|---|---|---|
| `DC01` | Domain controller | Windows Server 2022 Standard Evaluation | Active Directory, DNS, DHCP, and domain administration |
| `CLIENT01` | Domain client | Windows 11 Pro | Test domain sign-in, policies, and support tasks |
| `CLIENT02` | Domain client | Windows 11 Pro | Second client for validation |

Windows client editions must support joining an Active Directory domain; Windows Home editions are not suitable for this example.

## Environment

### Software and resources

- Oracle VM VirtualBox
- One Windows Server 2022 virtual machine
- Two Windows 11 Pro virtual machines
- Sufficient host RAM, CPU, and storage for three VMs
- Administrative access to configure the lab

### Network design

- VirtualBox network mode: **Internal Network**, with all three VMs connected to `AD-Lab`.
- Example subnet: `192.168.10.0/24`.
- The domain controller uses a static IP address.
- Client computers obtain their IP settings from the DHCP service on `DC01`.
- Client DNS points to `DC01` at `192.168.10.10`, so clients can locate Active Directory services.
- No default gateway is configured in this isolated example because no router or Internet connection is part of the topology.

## Example Lab Configuration

| Setting | Example value |
|---|---|
| Domain DNS name | `adlab.test` |
| NetBIOS domain name | `ADLAB` |
| Domain controller hostname | `DC01` |
| Domain controller IP address | `192.168.10.10/24` (static) |
| Client 1 hostname | `CLIENT01` |
| Client 1 example DHCP lease | `192.168.10.101/24` |
| Client 2 hostname | `CLIENT02` |
| Client 2 example DHCP lease | `192.168.10.102/24` |
| VirtualBox network mode | Internal Network: `AD-Lab` |
| DNS provider | `DC01` — `192.168.10.10` |
| DHCP provider | `DC01` — DHCP scope `192.168.10.100` to `192.168.10.199` |
| DHCP DNS option | `192.168.10.10` |
| DHCP DNS domain option | `adlab.test` |
| Default gateway | Not configured in this isolated example |

### User and security groups

| Security group | Purpose | Example membership |
|---|---|---|
| `GG-IT-Support` | Test accounts representing IT support staff | `it.user01` through `it.user05` |
| `GG-Staff-Users` | Test accounts representing standard staff | `staff.user01` through `staff.user05` |


### Group Policy Objects

| GPO name | Purpose | Scope / link location | Example verification |
|---|---|---|---|
| `Workstation - Screen Lock Policy` | Set the machine inactivity limit to 600 seconds (10 minutes) | Linked to `OU=Workstations,DC=adlab,DC=test`, which contains the client computer accounts | Use Group Policy Management and `gpresult /h gpresult.html` to inspect whether the policy applies and which settings are reported |
| `Staff - Control Panel Restriction` | Prevent staff users from opening Control Panel and PC settings | Linked to `OU=Staff,DC=adlab,DC=test`; security filtering limited to `GG-Staff-Users` | Sign in with a test staff account and confirm the restriction; use `gpresult /r` to inspect applied user policy |


## Implementation

### 1. Build the virtual machines

1. Create one Windows Server VM and two Windows 11 Pro client VMs in VirtualBox.
2. Name the machines `DC01`, `CLIENT01`, and `CLIENT02`.
3. Connect all three VMs to the `AD-Lab` Internal Network.
4. Assign `DC01` the static address `192.168.10.10/24`.
5. Record the actual Windows versions and build numbers used.

**Evidence:** `screenshots/01-virtual-machines.png`

### 2. Configure the server and domain

1. Configure the server hostname as `DC01` and set its static network configuration.
2. Install and configure Active Directory Domain Services (AD DS).
3. Promote the server to a domain controller for `adlab.test`.
4. Configure DNS for the domain.
5. Install and configure DHCP with a scope such as `192.168.10.100`–`192.168.10.199`.
6. Configure the DHCP scope to provide DNS server `192.168.10.10` and DNS suffix `adlab.test`.
7. Verify that the domain controller is running and DNS lookups work.

**Evidence:** `screenshots/02-server-and-domain.png`

### 3. Create users and groups

1. Open **Active Directory Users and Computers**.
2. Create the security groups `GG-IT-Support` and `GG-Staff-Users`.
3. Create 10 fictional test accounts: `it.user01`–`it.user05` and `staff.user01`–`staff.user05`.
4. Add the first five accounts to `GG-IT-Support` and the other five to `GG-Staff-Users`.
5. Test account administration tasks, such as resetting a test user's password.
6. Investigate account lockout if it is configured or encountered.

**Evidence:** `screenshots/03-active-directory-users-groups.png`

### 4. Join client machines to the domain

1. Confirm each client receives an address in the example subnet and can reach `192.168.10.10`.
2. Verify that each client uses `192.168.10.10` as its DNS server.
3. Join `CLIENT01` and `CLIENT02` to `adlab.test` using an authorized account.
4. Restart if prompted.
5. Sign in with an appropriate test domain account.
6. Verify domain membership and record any errors encountered.

**Evidence:** `screenshots/04-domain-joined-client.png`

### 5. Configure and verify Group Policy

1. Create an organizational unit named `Workstations` and place the client computer accounts there.
2. Create and link `Workstation - Screen Lock Policy` to the `Workstations` OU, setting the machine inactivity limit to 600 seconds.
3. Create an organizational unit named `Staff` and place the staff test user accounts there.
4. Create and link `Staff - Control Panel Restriction` to the `Staff` OU, and use security filtering for `GG-Staff-Users`.
5. On a client, run `gpupdate /force`.
6. Use `gpresult /r` or `gpresult /h gpresult.html` to inspect applied policy.
7. Compare the reported policy and observed client behavior with the intended settings.

**Evidence:**
- `screenshots/05-group-policy-management.png`
- `screenshots/06-gpresult-client.png`

### 6. Configure and test DNS/DHCP

For DNS, verify that `DC01` provides DNS for `adlab.test`. For DHCP, verify the scope and options and confirm that each client receives a valid lease.

Useful client commands:

```powershell
ipconfig /all
nslookup dc01.adlab.test
ping 192.168.10.10
nltest /dsgetdc:adlab.test
```

If a client uses DHCP and its lease needs refreshing:

```powershell
ipconfig /release
ipconfig /renew
```

A successful ping alone does not prove that DNS or Active Directory is working correctly.

**Evidence:** `screenshots/07-network-configuration.png`

## Troubleshooting Runbook

The five case studies below cover common Active Directory support scenarios in the reference environment: DNS resolution, Group Policy, DHCP addressing, account sign-in, and domain joining.

| Case ID | Scenario | Primary checks |
|---|---|---|
| `AD-TRBL-01` | Domain controller name does not resolve | `ipconfig /all`, `nslookup`, `nltest` |
| `AD-TRBL-02` | Group Policy does not apply | `gpupdate /force`, `gpresult`, OU and GPO scope |
| `AD-TRBL-03` | Client receives an unexpected IP address | `ipconfig /all`, DHCP scope and lease status |
| `AD-TRBL-04` | Domain user cannot sign in | Account status, lockout state, domain connectivity |
| `AD-TRBL-05` | Client cannot join the domain | Domain name, DNS server, domain-controller discovery |

Each case study follows a consistent workflow: symptom, checks, evidence, cause, resolution, and validation.

### AD-TRBL-01 — Domain controller name does not resolve

| Field | Reference case study |
|---|---|
| **Issue** | `CLIENT01` cannot resolve `dc01.adlab.test` when checking domain connectivity. |
| **Initial checks** | Run `ipconfig /all`, `nslookup dc01.adlab.test`, and `nltest /dsgetdc:adlab.test`. |
| **Evidence** | The example client is configured to use `192.168.10.1` for DNS instead of the domain controller at `192.168.10.10`; the lookup fails. |
| **Root cause** | The client is querying the wrong DNS server. |
| **Resolution** | Correct the client's DNS configuration or the DHCP scope's DNS-server option so it provides `192.168.10.10`. Renew the DHCP lease if the client uses DHCP. |
| **Validation** | Confirm `ipconfig /all` lists `192.168.10.10` as DNS, `nslookup dc01.adlab.test` returns the domain controller's address, and `nltest /dsgetdc:adlab.test` locates a domain controller. |
| **Lesson learned** | Domain clients need to use DNS that can resolve the Active Directory domain and its service records. |

**Suggested evidence:** `screenshots/08-dns-troubleshooting.png`

### AD-TRBL-02 — Group Policy does not apply

| Field | Reference case study |
|---|---|
| **Issue** | The screen-lock policy does not appear in the policy results for `CLIENT01`. |
| **Initial checks** | Run `gpupdate /force` and `gpresult /r /scope computer`. In Group Policy Management, inspect the GPO link and the computer account's OU. |
| **Evidence** | In this case study, `CLIENT01` remains in the default `Computers` container, while `Workstation - Screen Lock Policy` is linked to the `Workstations` OU. |
| **Root cause** | The computer account is outside the OU where the GPO is linked. |
| **Resolution** | Move the `CLIENT01` computer account into the `Workstations` OU, refresh policy, and restart the client if required by the setting. |
| **Validation** | Run `gpupdate /force` and `gpresult /r /scope computer` again. Confirm the intended GPO appears in the applied policy results, then verify the resulting policy behavior. |
| **Lesson learned** | When a GPO is missing, check its link, scope, filtering, and the location of the target computer or user account. |

**Suggested evidence:** `screenshots/09-gpo-troubleshooting.png`

### AD-TRBL-03 — Client receives an unexpected IP address

| Field | Reference case study |
|---|---|
| **Issue** | A client receives an automatic private IP address beginning with `169.254` rather than an address from the lab DHCP scope. |
| **Initial checks** | Run `ipconfig /all`. In the DHCP console on `DC01`, inspect the scope `192.168.10.100`–`192.168.10.199` and its activation state. |
| **Evidence** | In this case study, the client has a `169.254.x.x` address and the lab DHCP scope is inactive. |
| **Root cause** | The DHCP scope is not active. |
| **Resolution** | Activate the intended scope on `DC01`. On the client, run `ipconfig /release` followed by `ipconfig /renew`. |
| **Validation** | Run `ipconfig /all` again and confirm the client receives a `192.168.10.x` address and lists `192.168.10.10` as the DHCP-provided DNS server. |
| **Lesson learned** | Check the DHCP service, scope state, and lease details when a client does not receive the expected address. |

**Suggested evidence:** `screenshots/10-dhcp-troubleshooting.png`

### AD-TRBL-04 — Domain user cannot sign in

| Field | Reference case study |
|---|---|
| **Issue** | The test account `staff.user03` cannot sign in to `CLIENT01` and receives an account-lockout message. |
| **Initial checks** | Confirm the username and domain, verify the client can reach `DC01`, then inspect the account in Active Directory Users and Computers. If auditing is available, review the domain controller's Security log for the related lockout event. |
| **Evidence** | In this case study, the account's properties show that `staff.user03` is locked out. |
| **Root cause** | Repeated unsuccessful sign-in attempts caused the account to be locked. |
| **Resolution** | Unlock the test account in Active Directory Users and Computers, confirm the account is enabled, and use the correct test credentials. |
| **Validation** | Sign in to `CLIENT01` with the test domain account and confirm the desktop loads successfully. Recheck the account status if the failure recurs. |
| **Lesson learned** | Check account status and lockout information before resetting passwords or changing client configuration. |

**Suggested evidence:** `screenshots/11-domain-sign-in-troubleshooting.png`

### AD-TRBL-05 — Client cannot join the domain

| Field | Reference case study |
|---|---|
| **Issue** | `CLIENT02` cannot join the domain because the domain name entered in the join dialog is `adlab.local` instead of `adlab.test`. |
| **Initial checks** | Confirm the intended domain name, run `ipconfig /all`, use `nslookup dc01.adlab.test`, and run `nltest /dsgetdc:adlab.test` to verify domain-controller discovery. |
| **Evidence** | In this case study, `dc01.adlab.test` resolves correctly, but the join attempt uses the incorrect domain name `adlab.local`. |
| **Root cause** | The domain name entered during the join operation does not match the configured Active Directory domain. |
| **Resolution** | Enter `adlab.test` as the domain name and provide an authorized lab account when prompted. |
| **Validation** | Confirm the join completes, restart if prompted, and verify membership in System settings or with `Get-CimInstance Win32_ComputerSystem | Select-Object Name, Domain, PartOfDomain`. |
| **Lesson learned** | Confirm the exact DNS domain name and verify domain-controller discovery before retrying a domain join. |

**Suggested evidence:** `screenshots/12-domain-join-troubleshooting.png`

## Validation Checklist

- [ ] Three VMs are configured and actual operating system versions are recorded.
- [ ] Server hostname and lab domain are recorded.
- [ ] Ten fictional test users and two security groups are configured and documented.
- [ ] A test password reset was performed and checked.
- [ ] Both clients' domain membership was verified.
- [ ] Both GPOs and their actual settings are documented.
- [ ] Group Policy results were inspected on a client.
- [ ] DNS configuration and lookup results are recorded.
- [ ] DHCP scope, options, and client leases were checked.
- [ ] Troubleshooting exercises have been reproduced and documented with evidence, root-cause findings, resolution, and post-fix validation.
- [ ] Screenshots show the actual configuration or results.

## Evidence and Screenshots

The `screenshots/` directory contains evidence for the configuration and troubleshooting cases.

| Filename | Evidence shown |
|---|---|
| `01-virtual-machines.png` | Three VMs in VirtualBox |
| `02-server-and-domain.png` | Server identity and domain setup |
| `03-active-directory-users-groups.png` | Test users and groups |
| `04-domain-joined-client.png` | Client domain membership |
| `05-group-policy-management.png` | The two GPOs and their settings |
| `06-gpresult-client.png` | Group Policy results on a client |
| `07-network-configuration.png` | Relevant DNS/DHCP or IP configuration |
| `08-dns-troubleshooting.png` | DNS problem investigation and retest |
| `09-gpo-troubleshooting.png` | Group Policy investigation and retest |
| `10-dhcp-troubleshooting.png` | DHCP address investigation and retest |
| `11-domain-sign-in-troubleshooting.png` | Domain account sign-in investigation and retest |
| `12-domain-join-troubleshooting.png` | Domain-join investigation and retest |

## Repository Structure

```text
windows-active-directory-lab/
├── README.md
├── windows-active-directory-lab-architecture.png
└── screenshots/
    ├── 01-virtual-machines.png
    ├── 02-server-and-domain.png
    ├── 03-active-directory-users-groups.png
    ├── 04-domain-joined-client.png
    ├── 05-group-policy-management.png
    ├── 06-gpresult-client.png
    ├── 07-network-configuration.png
    ├── 08-dns-troubleshooting.png
    ├── 09-gpo-troubleshooting.png
    ├── 10-dhcp-troubleshooting.png
    ├── 11-domain-sign-in-troubleshooting.png
    └── 12-domain-join-troubleshooting.png
```

## Lab Scope and Limitations

- The reference topology uses three virtual machines on an isolated VirtualBox Internal Network, without a router or Internet connection.
- The domain, hostnames, IP addresses, group names, GPO settings, and troubleshooting outcomes define a consistent example environment.
- Windows edition capabilities and VirtualBox interface details can vary by software version.

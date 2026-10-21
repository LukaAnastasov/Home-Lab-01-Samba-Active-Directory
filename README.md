# Lab 01 — Linux Samba File Server & Active Directory Foundations

**Hands-on home lab | SMB file sharing • Linux permissions • Windows Server 2025 • Active Directory • Group Policy**

This project documents a two-phase home lab exploring authenticated SMB file sharing and foundational Windows domain administration. I configured a Samba share on Kali Linux, tested allowed and denied access from Windows, then built a separate Windows Server 2025 Active Directory environment with a domain-joined Windows workstation and a tested Sales Group Policy Object (GPO).

> **Scope note:** The original Phase 2 objective included integrating Samba with Active Directory and replacing local Samba accounts with domain authentication. The supplied documentation demonstrates the **AD deployment and GPO validation**, but does **not** demonstrate completion of Samba-to-AD integration. That remains a possible next step.

## Lab overview

| System | Purpose | Documented address |
|---|---|---|
| Kali Linux VM | Samba file server for Phase 1 | `192.168.0.181` |
| Windows PC | SMB client for Phase 1 | `192.168.0.210` |
| DC01 — Windows Server 2025 | Active Directory Domain Services and DNS | `192.168.0.10` |
| CLIENT01 — Windows workstation | Domain-joined endpoint and GPO validation | See source documentation |

**AD domain:** `anastasov.lab` · **NetBIOS:** `ANASTASOV` · **SMB port:** TCP `445`

## Phase 1 — Authenticated Samba file sharing

### 1. Validate network connectivity

I checked the Kali VM's IPv4 address and default gateway with `ip a` and `ip route`, then confirmed connectivity from Windows with `ipconfig` and `ping`.

![Connectivity verification](screenshots/network-connectivity.png)

### 2. Configure the file server

I installed and verified the Samba service, created `/srv/samba/shared`, configured a `sambashare` Linux group, and applied group ownership and permissions:

```bash
sudo apt update
sudo apt install samba
sudo systemctl start smbd
sudo mkdir -p /srv/samba/shared
sudo groupadd sambashare
sudo chown root:sambashare /srv/samba/shared
sudo chmod 2770 /srv/samba/shared
```

The `2770` mode allows owner/group access, denies other users, and sets the **setgid** bit so new items inherit the directory's group. The account was also enrolled in the group and assigned Samba credentials.

The share definition added to `/etc/samba/smb.conf` was:

```ini
[shared]
    path = /srv/samba/shared
    browseable = yes
    read only = no
    guest ok = no
    valid users = @sambashare
```

I checked the configuration with `testparm`, reloaded `smbd`, and verified SMB listening on TCP 445.

![Samba share configuration](screenshots/samba-configuration.png)

### 3. Verify successful file access

From Windows, I connected to `\\192.168.0.181\shared` using the authorized Samba account, created `windows-test.txt`, and confirmed the file and its contents were visible on Kali.

![Windows SMB access](screenshots/windows-smb-access.png)
![Cross-platform file verification](screenshots/cross-platform-file-verification.png)

### 4. Test access controls

I created `testuser` with Samba credentials **but without** `sambashare` group membership. A Windows `net use` connection attempt was denied. I also inspected the Samba service journal for server-side evidence. After adding the user to `sambashare`, I repeated the test and successfully accessed the share and created a file.

**Key finding:** Having valid Samba credentials alone did not grant access; membership in the authorized Linux group was also required.

![Unauthorized user denied](screenshots/unauthorized-access-denied.png)
![Authorized after group membership](screenshots/group-membership-access-granted.png)

### 5. Final checks

The documented final audit verified that `testparm` accepted the configuration and that the Samba service was enabled and active.

![Final Samba audit](screenshots/samba-final-audit.png)

## Phase 2 — Active Directory foundations

### 1. Deploy DC01

I deployed Windows Server 2025, configured a static IPv4 address of `192.168.0.10`, renamed the server `DC01`, and installed Active Directory Domain Services and DNS. I created a new forest, `anastasov.lab`, with NetBIOS name `ANASTASOV`.

![DC01 static IP configuration](screenshots/dc-static-ip.png)

### 2. Verify domain services

I verified the domain administrator context and used DNS queries to resolve the domain controller and its LDAP service (SRV) records.

![Domain and DNS verification](screenshots/domain-dns-verification.png)
![LDAP SRV lookup](screenshots/ad-srv-record.png)

### 3. Organize users and groups

To simulate a small organization, I created departmental organizational units (IT, Sales, Finance), along with Company Computers and Company Groups OUs. I created domain users and global security groups for the departments.

![AD users and groups](screenshots/ad-users-groups.png)

### 4. Apply and validate a Sales GPO

I created a **Sales - User Restrictions** GPO to block Control Panel and PC Settings for Sales users. I joined a Windows workstation (`CLIENT01`) to the domain, signed in using a Sales account, checked applied policy with `gpresult`, and tested the restriction in Windows.

![Sales GPO configuration](screenshots/sales-gpo.png)
![CLIENT01 joined to domain](screenshots/client-domain-join.png)
![Sales GPO verification](screenshots/sales-gpo-verification.png)

## Troubleshooting and lessons learned

- **Group membership refresh:** A new Linux group membership was not reflected in the existing shell. Running `newgrp sambashare` refreshed the group context, allowing file verification.
- **Authentication versus authorization:** `testuser` had a Samba password but could not access the share until added to the authorized group.
- **IP planning:** The domain controller was assigned an address outside the revised DHCP allocation range to reduce address-conflict risk.
- **Firewall behavior:** ICMP connectivity initially differed between the Windows Server and Kali directions; this was identified as likely firewall filtering rather than proof that the network was disconnected.
- **GPO verification:** Configuration alone was not the endpoint; I tested the policy from a domain-joined client and checked the applied policy.

## Outcomes

- Configured a functioning authenticated SMB share with group-based access control.
- Verified Windows-to-Linux file creation and retrieval.
- Demonstrated denied access followed by permitted access after a group membership change.
- Deployed an Active Directory forest and verified DNS/domain-controller discovery.
- Built an organizational structure and validated a Sales user policy on a domain-joined endpoint.

## Future extension

Complete and document Samba integration with Active Directory, replacing local Samba authentication with domain-backed authentication and validating domain-group authorization from Windows clients.

## Evidence and usage

All screenshots are page captures from my original lab report, retained to document the work. This is an **isolated, authorized home lab**; example IP addresses are private LAN addresses. The original report is the source of truth for the steps and observations. Screenshots may contain surrounding document text because they were extracted from the PDF pages.

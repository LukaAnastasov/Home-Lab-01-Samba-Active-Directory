# Project notes

## Source and scope

This repository was adapted from the provided 46-page PDF, *Threat Defense Home Labs*, Lab 1. The PDF includes Phase 1 (Linux Samba File Server) and Phase 2 (Active Directory Integration). The latter title describes an intended extension, not a fully evidenced domain-joined Samba deployment.

## Key commands demonstrated

```bash
ip a
ip route
sudo apt update
sudo apt install samba
sudo systemctl status smbd
sudo mkdir -p /srv/samba/shared
sudo groupadd sambashare
sudo chown root:sambashare /srv/samba/shared
sudo chmod 2770 /srv/samba/shared
sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.backup
testparm
sudo smbpasswd -a kali
sudo pdbedit -L
sudo systemctl reload smbd
sudo ss -tlnp | grep smbd
newgrp sambashare
sudo smbpasswd -a testuser
sudo usermod -aG sambashare testuser
```

Windows-side tests included `ipconfig`, `ping`, SMB Explorer access, `net use`, `whoami`, `hostname`, `nslookup`, and `gpresult`.

## Screenshot index

Screenshots are full-page PDF extracts rather than individually cropped application screenshots, to preserve original evidence and context.

| File | Original PDF page | Evidence |
|---|---:|---|
| network-connectivity.png | 2 | IP route and Windows-to-VM ping |
| samba-configuration.png | 6 | Share configuration |
| windows-smb-access.png | 10 | Windows authentication and shared folder |
| cross-platform-file-verification.png | 12 | File created in Windows seen on Kali |
| unauthorized-access-denied.png | 14 | Access denied for nonmember |
| group-membership-access-granted.png | 16 | Access after adding user to group |
| samba-final-audit.png | 17 | Samba configuration and service verification |
| dc-static-ip.png | 24 | DC01 static address |
| domain-dns-verification.png | 35 | Domain logon and DNS lookup |
| ad-srv-record.png | 36 | AD LDAP SRV lookup |
| ad-users-groups.png | 39 | Domain users and security groups |
| sales-gpo.png | 41 | Sales GPO creation |
| client-domain-join.png | 43 | CLIENT01 domain join |
| sales-gpo-verification.png | 45 | Applied GPO and Control Panel restriction |

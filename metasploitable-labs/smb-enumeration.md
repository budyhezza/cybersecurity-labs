# Metasploitable 2 - SMB Enumeration & Anonymous Share Access

## Lab Information

Target Machine: Metasploitable2

Attacker Machine: Kali Linux

Target IP: 192.168.56.101

---

## Objective

Identify SMB services and enumerate available shares to discover potential misconfigurations and security weaknesses.

---

## Reconnaissance

### Nmap Scan

Focused enumeration was performed on ports 139 and 445, which are commonly used by Samba and NetBIOS services.

Command used:

nmap -sC -sV 192.168.56.101

### Key findings

| Port | Service     | Version                  |
|------|-------------|--------------------------|
| 139  | Netbios-SSN | Samba smbd 3.X - 4.X     |
| 445  | Netbios-SSN | Samba smbd 3.0.20-Debian |

---

## Enumeration

### SMB Enumeration with Nmap Scripts

Command used:

nmap -p 139,445 --script smb-enum-shares,smb-enum-users 192.168.56.101

### Findings

- Multiple shares discovered:
  - tmp
  - opt
  - IPC$
  - print$
  - ADMIN$

- Identified some system users (e.g., daemon, ircd, games)

---

### SMB Client Enumeration

Command used:

smbclient -L //192.168.56.101/ -N

Result:
- No credentials required (Anonymous login successful)
- Share listing accessible without authentication

---

## Vulnerability

The SMB service is misconfigured to allow unauthenticated (anonymous) access to shared resources.

Moreover, the `tmp` share allows unauthenticated write access, allowing arbitrary file uploads.

---

## Exploitation (Misconfiguration Abuse)

### Accessing Writable Share

Command:

smbclient //192.168.56.101/tmp -N

### Actions performed

- Listed directory contents using `ls`
- Uploaded file using:

put test.txt

### Result

Successfully uploaded the file to the target system without authentication.

---

## Attack Flow Summary

1. Identified SMB service
2. Enumerated shares
3. Discovered anonymous (unauthenticated) access
4. Found writable share
5. Successfully uploaded a file to target system

---

## Detection Opportunities (SOC Perspective)

Many detection opportunities are found, such as:

- SMB anonymous login attempts (null sessions)
- Share enumeration behavior from a single host
- Repeated access to multiple SMB shares
- Unauthorized file upload activity
- Suspicious SMB traffic on ports 139/445

---

## Lessons Learned

- SMB enumeration can reveal critical misconfigurations
- Anonymous access significantly increases attack surface
- Not all attacks require exploits; misconfigurations are often enough
- Writable shares can be abused for staging attacks

---

## Mitigation

- Disable anonymous SMB access
- Restrict share (read/write) permissions
- Enforce authentication
- Monitor SMB activity and logs
- Apply proper access control policies
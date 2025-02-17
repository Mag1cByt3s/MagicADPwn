# MagicADPwn

<br>

## Overview
MagicADPwn is a standalone Bash script designed to automate the enumeration and exploitation of Active Directory environments. It integrates multiple tools to identify misconfigurations, escalate privileges, perform lateral movement, and establish persistence.

<br>

---

<br>

## Features
- Automated Active Directory enumeration and exploitation
- Uses `netexec` under the hood for authentication and enumeration
- Integrates with `certipy`, `bloodhound-python`, `ldapdomaindump`, and `bloodyAD`
- Supports various authentication methods:
  - Username/password
  - NTLM hash authentication
  - Kerberos authentication
  - Guest/anonymous fallback
- User and Group enumeration
- Dumping user descriptions
- AS‑REP Roasting (runs anonymously using a users file, if available, or with credentials)
- Kerberoasting (requires valid credentials)
- Automated attack path discovery and execution
- Generates structured reports in JSON, CSV, Markdown, and HTML
- Optional interactive HTML report with visual attack paths (future feature)
- Smart password spraying detection for SMB/LDAP
- **Vulnerability Scanning** for known exploits like Zerologon, PrintNightmare, SMBGhost, and MS17-010
- **SMB Enumeration**:
  - Enumerate **readable and writable** SMB shares
  - List all readable files using the **spider_plus** module

<br>

---

<br>

## Installation
For now, MagicADPwn is a standalone Bash script with no installation required. Ensure the necessary dependencies are installed:
```bash
impacket
bloodhound-python
ldapdomaindump
certipy
bloodyad
```

<br>

---

<br>

## Usage
```bash
./MagicADPwn -t <target_ip/hostname> [-u <username>] [-p <password> | -H <hash> | -k [--no-pass]] [--local-auth] [-v]
```

Required:
  - `-t`, `--target <IP/hostname>`: Specify the Domain Controller

Optional:
  - `-u`, `--user <username>`: Specify a username (default: guest/anonymous)
  - `-p`, `--pass <password>`: Specify a password
  - `-H`, `--hash <NTLM hash>`: Use an NTLM hash instead of a password
  - `k` / `--kerberos`: Use Kerberos authentication. Requires a valid Kerberos ticket.
    - If using Kerberos ticket cache (no password or hash), set the `KRB5CCNAME` environment variable to the path of your ticket and use `--no-pass`.
  - `--no-pass`: Skip password or hash when using Kerberos authentication. Requires `-k`.
  - `--local-auth`: Use local authentication (optional).
  - `--spray-users <file>`: Supply a file with usernames for password spraying.
  - `--spray-passwords <file>`: Supply a file with passwords for password spraying.
  - `--no-recon`: Skip initial enumeration and go straight to attacks.
  - `--report <format>`: Generate a report (json, csv, markdown, html).
  - `-v`, `--verbose`: Enable verbose debugging output.

<br>

---

<br>

## Attack Flow
- **Authentication Check**: Validate credentials or fallback to guest/anonymous.
- **Reconnaissance**: Extract user and group information, SPNs, shares, ACLs, GPOs, etc.
- **Privilege Escalation Checks**: Identify vulnerabilities including Kerberoasting, AS‑REP roasting, RBCD, and AD CS issues.
- **Exploitation**: Automate privilege escalation, lateral movement, and persistence.
- **Vulnerability Scanning**: Scan for known vulnerabilities like Zerologon, PrintNightmare, SMBGhost, and MS17-010.
- **SMB Enumeration**:
  - Identify readable and writable SMB shares
  - Use spider_plus to list all readable files (with optional share exclusion)
- **Group Membership Analysis**: Automatically check group membership and, if the user is in Backup Operators, run the backup_operator module.
- **Roasting**:
  - **AS‑REP Roasting**: Runs either anonymously using a generated users file (if non‑empty) or with provided credentials.
  - **Kerberoasting**: Requires valid credentials.
- **Reporting**: Generate structured reports for further analysis.

<br>

---

<br>

## Examples

Password auth:
```bash
./MagicADPwn -t 192.168.1.100 -u administrator -p SuperSecretPass123
```

<br>

Pass-the-Hash:
```bash
./MagicADPwn -t 192.168.1.100 -u administrator -H '0123456789abcdef0123456789abcdef'
```

<br>

Local auth:
```bash
./MagicADPwn -t 192.168.1.120 -u administrator -p P@ssw0rd --local-auth
```

<br>

Pass-the-Ticket (Kerberos auth):
```bash
KRB5CCNAME=administrator@cifs_dc.company.com@COMPANY.COM.ccache ./MagicADPwn -t dc.company.com -u administrator -k --no-pass
```

<br>

Kerberos auth with password:
```bash
./MagicADPwn -t dc.company.com -u administrator -p 'StrongPassword123' -k
```

<br>

Password Spraying (SMB/LDAP auto-detection):

- Username and password lists:
  ```bash
  ./MagicADPwn -t 192.168.1.100 --spray-users users.txt --spray-passwords passwords.txt
  ```
- Username list with a single password:
  ```bash
  ./MagicADPwn -t 192.168.1.100 --spray-users users.txt -p 'Password123'
  ```
- Password list with a single username:
  ```bash
  ./MagicADPwn -t 192.168.1.100 -u admin --spray-passwords passwords.txt
  ```

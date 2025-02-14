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
- Automated attack path discovery and execution
- Generates structured reports in JSON, CSV, Markdown, and HTML
- Optional interactive HTML report with visual attack paths (future feature)
- Smart password spraying detection for SMB/LDAP
- **Vulnerability Scanning** for known exploits like Zerologon, PrintNightmare, SMBGhost, and MS17-010

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
- Authentication Check: Validate credentials or fallback to guest/anonymous.
- Reconnaissance: Extract user/group information, SPNs, shares, ACLs, GPOs, etc.
- Privilege Escalation Checks: Identify Kerberoasting, AS-REP roasting, RBCD, and AD CS vulnerabilities.
- Exploitation: Automate privilege escalation, lateral movement, and persistence.
- Reporting: Generate structured reports for review.
- **Password Spraying Logic:**
  - If SMB (port 445) is open, `nxc smb` is used for authentication attempts.
  - If LDAP (port 389) is open, `nxc ldap` is used for authentication attempts.
  - If neither service is available, password spraying is skipped.
  - The tool can spray using a username list and password list, a single password with a user list, or a single username with a password list.

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

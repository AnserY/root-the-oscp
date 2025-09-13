**IP:**

---

## OS / Technology
- Ubuntu
- PHP

---

## Users / Passwords
- ben: (password recovered during post-exploitation)
- admin (CrushFTP admin created via exploit)

---

## Services
- **80 HTTP**
  - Hostname: `soulmate.htb`
  - Source code reviewed — nothing immediately interesting
  - Fuzzing: no results
  - Login page and registration available
  - Image upload functionality present (upload attempted with `.php`)
  - Uploaded file path not immediately obvious; revisited later during exploitation
- **22 SSH**
  - Open (standard)
- **FTP subdomain**
  - `ftp.soulmate.htb` discovered during subdomain fuzzing
  - Powered by CrushFTP — vulnerable to CVE-2025-31161 (auth bypass)

---

## Nmap
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.13
80/tcp open  http    nginx 1.18.0 (Ubuntu)
```

---


## FTP / CrushFTP Pivot
- Subdomain `ftp.soulmate.htb` was discovered using fuzzing.
- Identified an **authentication bypass** vulnerability (CVE-2025-31161) affecting CrushFTP that allows creating an admin-level account.
- Exploited the CVE to create an admin user in CrushFTP.

### With CrushFTP admin:
- Connected to the FTP management panel and set/change passwords for existing users.
- Changed password for user `ben` and used that account to upload a PHP reverse shell to the web application (or used the admin to identify the upload location and place the shell).

---

## Gaining a Shell
- Triggered the uploaded PHP shell by requesting the uploaded file URL from the web app.
- Received a reverse shell as `www-data`.

---

## Post-Exploitation
- Searched filesystem and found password for `ben` in:
```
/usr/local/lib/erlang/start_erlang.sh
```
- Used `ben`'s credentials to connect to an SSH service listening on **port 2222** (discovered via `ss -tulpn`).
- The service on 2222 was an Erlang shell (or an Erlang-based service) that provided a shell context under root.
- Verified `id` → UID 0 (root). Root shell obtained.

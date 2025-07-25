
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

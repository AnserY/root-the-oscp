# Box: 10.10.109.142

---

## Service Enumeration
| Port   | State | Service | Notes                    |
| ------ | ----- | ------- | ------------------------ |
| **53** | open  | domain  | DNS authoritative server |
| **80** | closed| http    | –                        |

---

## DNS Enumeration

### 1. CNAME lookup
```bash
dig givemetheflag.com CNAME
```
Output shows no CNAME but an SOA authority record.

### 2. NS records
```bash
dig givemetheflag.com NS
```
Returns:
- ns1.afternic.com.
- ns2.afternic.com.

### 3. Public A records
```bash
dig givemetheflag.com A
```
Returns:
- 76.223.54.146
- 13.248.169.48

### 4. Direct query to the box
```bash
dig @10.10.109.142 givemetheflag.com
```
```
;; ANSWER SECTION:
givemetheflag.com. 0 IN TXT "flag{0767ccd06e79853318f25aeb08ff83e2}"
```


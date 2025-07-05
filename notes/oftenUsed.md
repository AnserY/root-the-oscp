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



During Exploit: 

wow shell: 
python3 -c 'import pty;pty.spawn("/bin/bash")'
 


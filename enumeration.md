# Enumeration cheatsheet

## Summary
* [Enumeration factors](#enumeration-factor-to-check)
* [Enumerate web application](#enumerate-web-application)
	* Enumerate directories
	* Nikto scanning
	* Bruteforce credentials
	* Enumerate subdomains
	* Enumerate parameters
	* Enumerate file names & file path
	* Enumerate POST data
* [Hydra](#hydra)
	* Bruteforce network services
	* Bruteforce basic auth
	* Bruteforce credentials
	* Bruteforce POST data
* [Encoding & hashes](#encoding--hashes)
	* Crack Shadow hashes
	* Generate hashes
	* Crack SSH key
	* Crack hashes
	* Decode base64
	* Crack zip file
	* Generate SSH key
	* Crack PGP key
	* Identify hashes
	* Generate wordlists
* [Wordpress](#wordpress)
	* Enumerate Wordpress site
	* Enumerate Wordpress users
	* Enumerate Wordpress passwords
* [Git](#interacting-git-on-cli)

### Enumeration factor to check
```
apache default page
hidden directory
page source
inspect page
subdomains
parameter
filename
web requests with different methods
401,403,404 enumeration
Javascript files for any web paths
functionality in application
application or system's version
```
### Enumerate web application

Enumerate directories
```
gobuster dir -u [url] -w [wordlists] -t 10 -x txt,php,sh,cgi,html,zip,bak,sql,old,ppm -b 400,404 -U [user] -P [password] -r
gobuster dir -u http://10.10.211.97/ -w /usr/share/wordlists/dirb/common.txt -s "204,301,302,307,401,403" --timeout 20s
dirb [url]
feroxbuster -u http://example.com/ -d 0 -r -C 403,404 -w /wordlists/common.txt -t 40
feroxbuster -u http://example.com/ -C 403,404 -w /wordlists/common.txt -t 40
ffuf -u http://[ip]/FUZZ -c -w /usr/share/seclists/Discovery/Web-Content/raft-small-directories-lowercase.txt -e .txt,.php -fc 403
```
Scan for vulnerabilities with nikto
```
nikto -h [domain or ip]
nikto -h [domain or ip] -ssl ==> scan an SSL-enable website
nikto -h [ip or domain] -Format msf+ ==> pair scans with metasploit
```
Bruteforcing credentials
```
ffuf -w [wordlists] -X [request method] -d "username=FUZZ&email=x&password=x&cpassword=x" -H "Content-Type: application/x-www-form-urlencoded" -u [url] -mr "username already exists"
ffuf -w [username.txt]:W1,[password.txt]:W2 -X [request method] -d "username=W1&password=W2" -H "Content-Type: application/x-www-form-urlencoded" -u [url] -fc 200
```
Enumerate subdomains
```
wfuzz -c --hw 977 -u [url] -H "Host: FUZZ.[domain]" -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt
wfuzz -c -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -u "[url]" -H "Host: FUZZ.[hostname]" --hw 290
ffuf -u http://vulnnet.thm -H "Host: FUZZ.vulnnet.thm" -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -fs 5829
ffuf -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -u http://FUZZ.wekor.thm -of -o result
ffuf -w [wordlists] -H "Host:FUZZ.domain" -u [url]
amass enum --passive -d example.com
dnsrecon -n 10.10.142.241 -d undiscovered.thm -D /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -t brt
wfuzz -c -f dns.txt -w wordlist.txt -u 'http://example.com' -H 'Host:FUZZ.example.com' --hw 290 --hc 404,302
gobuster vhost -u http://example.com/ -w wordlists.txt | grep 200
gobuster dns -d example.com -w wordlist.txt
```
Enumerate url parameters
```
wfuzz -c -z file,/usr/share/wordlists/wfuzz/general/medium.txt --hc 400 -X POST -u [url]\?FUZZ\=test
ffuf -u http://vulnnet.thm/?FUZZ=/etc/passwd -w /usr/share/wordlists/SecLists/Discovery/Web-Content/burp-parameter-names.txt -fs 5829
wfuzz -w /usr/share/wordlists/dirb/common.txt --hc=404 "http://10.10.244.142:5000/api/v1/resources/books?FUZZ=/etc/passwd"
ffuf -u "http://example.com?FUZZ=id" -w /usr/share/wordlists/SecLists/Discovery/Web-Content/burp-parameter-names.txt -fw 1
```
Enumerate file name & file path
```
wfuzz -w /data/src/wordlists/directory-list-2.3-medium.txt --hc 404 -u [url/FUZZ.[filename]]
wfuzz -w /data/src/wordlists/directory-list-2.3-medium.txt --hc 404 -c --hh 3834 -t 1 http://example.com/FUZZ.bak.txt
ffuf -u http://example.com/FUZZ.pdf -w /usr/share/wordlists/SecLists/Discovery/Web-Content/common.txt
wfuzz -c -w /usr/share/wordlists/SecLists/Fuzzing/LFI/LFI-gracefulsecurity-linux.txt -u [url/example.php?example=FUZZ] --hw=0
```
Enumerate POST data
```
wfuzz -c -z file,numbers.txt -H "X-Originating-IP: 127.0.0.1" -d "number=FUZZ" --hw 81 http://10.10.60.231:8085/
```

### Hydra
Bruteforcing network services
```
hydra -s [port] [service{ftp/ssh}]://[ip] -l [username] -P [password wordlists] -t 4 -V
hydra -l [username] -P /data/src/wordlists/fasttrack.txt pop3://[ip]:[port]
hydra -L users.txt -P password.txt -vV -o ssh.log -e ns IP ssh
hydra IP ftp -l username -P wordlist -t thread(default 16) -vV
hydra IP ftp -l username -P wordlist -e ns -vV
hydra -m /index.php -l username -P pass.txt IP https
hydra -l username -P wordlist -s portnumber -vV ip teamspeak
hydra -P pass.txt IP cisco
hydra -m cloud -P pass.txt 192.168.1.11 cisco-enable
hydra -l administrator -P pass.txt IP smb
hydra -l muts -P pass.txt my.pop3.mail pop3
hydra IP rdp -l administrator -P pass.txt -V
hydra -l admin -P pass.txt http-proxy://192.168.1.111
hydra IP telnet -l username -P wordlist -t 32 -s 23 -e ns -f -V
```
Bruteforcing basic auth website
```
hydra -l [username] -P [password wordlists] [ip] -s [port] http-get [url]
hydra -l username -P wordlist -e ns IP http-get /admin/
hydra -l username -P wordlist -e ns -f IP http-get /admin/index.php
hydra -C [userpass file] http-get://[url]
```
Bruteforcing POST data
```
hydra -l admin -P pass.lst -o ok.lst -t 1 -f 127.0.0.1 http-post-form “index.php:name=^USER^&pwd=^PASS^:<title>invalido</title>”
hydra -l [username] -p [password] example.com http-post-form "/[url]:log=^USER^&pwd=^PASS^:[error]:H=[cookie]"
hydra -l [username] -P [wordlists] 127.0.0.1 http-post-form "/console/mfa.php:code=^PASS^:F=Incorrect code:H=Cookie: PHPSESSID=gckm1it42pqrbk2hikjk1oseos; user=jason_test_account; pwd=violet"
```

### Encoding & hashes
#### https://www.guballa.de/vigenere-solver
Crack hashes from shadow file
```
copy /etc/shadow and /etc/passwd in 2 different files 
unshadow the 2 files  {unshadow passwd shadow > unshadowed.txt}
john --wordlist=/usr/share/wordlists/rockyou.txt --format=sha512crypt unshadowed.txt
```
Create hash value for a password
```
openssl passwd -1 -salt [user] [password]
python3 -c 'import crypt;crypt.crypt("password123")'
```
Crack private SSH key
```
ssh2john [private key] > [save file]
john --wordlist=<wordlist file> <save file>
```
Crack a hash value
```
john --wordlist=[wordlist] [hash file]
hashcat -a 0 -m <mode> '<hash>' -w /usr/share/wordlists/rockyou.txt
```
Decode base64
```
echo -n "[text]" | base64 -d
```
Crack zip file
```
zip2john [file] > output.txt
john --wordlist=<wordlist file> output.txt
```
Generate SSH private & public key
```
ssh-keygen -f id_rsa
```
Crack PGP private key
```
gpg2john [private key] > output.txt
john --wordlist=/usr/share/wordlists/rockyou.txt output.txt
```
Identify hashes strings
```
hashid -m '<hash>'
hash-identifier '<hash>'
```
Generate a wordlists of password from a web page
```
cewl http://example.com/simeon -d 5 -m 5 -w pass.txt
cewl http://example.com/simeon > pass.txt
```

### Wordpress
Enumerate Wordpress site
```
wpscan --url [url] --no-banner
wpscan --url [url] -e u,vp,vt
```
Enumerate Wordpress users
```
wpscan --url [url] --enumerate u
```
Enumerate Wordpress passwords
```
wpscan --url [url] --passwords [pass_file] --usernames [username]
wpscan -U users.txt -P /usr/share/wordlists/rockyou.txt --url http://example.com
```

### Interacting Git on CLI
```
git branch
git log 'commit' --oneline
git show 'hashes'
```


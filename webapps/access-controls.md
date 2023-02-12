# Access controls

## Summary
* [Vertical access controls](#vertical-access-controls)
    * [User role controlled by request parameter](#user-role-controlled-by-request-parameter)
    * [User role can be modified in user profile](#user-role-can-be-modified-in-user-profile)
    * [Broken access control from platform misconfiguration](#broken-access-control-from-platform-misconfiguration)
    * [Method-based broken access control](#method-based-broken-access-control)
* [Horizontal access controls](#horizontal-access-controls)
    * [Userid controlled by request parameter with unpredictable user id](s#userid-controlled-by-request-parameter-with-unpredictable-user-id)
    * [Userid controlled by request parameter with data leakage in redirect](#userid-controlled-by-request-parameter-with-data-leakage-in-redirect)
* [IDOR](#idor)
* [Access control in multi-step process](#access-control-in-multi-step-processes)
* [Referer-based access control](#referer-based-access-control)
* [Location-based access control](#location-based-access-control)
* [Read forbidden page source code](#read-forbidden-page-source-code)
* [Bypassing 403 response](#bypass-403-response)
    * [Using "X-Original-URL" header](#using-x-original-url-header)
    * [Appending "%2e" after the first slash](#appending-2e-after-the-first-slash)
    * [Add dot (.), slash (/) & semicolon (;) in URL](#add-dot--slash---semicolon--in-url)
    * [Add "..;/" after path name](#add--after-path-name)
    * [Uppercase the characters in URL](#uppercase-the-characters-in-url)

## Vertical access controls
### User role controlled by request parameter
```
> Navigate to any site with authenticated user
> Intercept the request and notice the cookies contain 2 parameter, sessions and admin
> Change the value of admin to true and go back to the app, now you can access the admin's page
```
### User role can be modified in user profile
```
> Navigate to any site with authenticated user
> Submit a request to change the user's email and intercept the request with burpsuite
> In the get request body, there is a Json body which contains the email that is going to be change
> Add "roleid"=2 into the Json body and send the request
> Now the user can access the admin's page
```
### Broken access control from platform misconfiguration
Some applications enforce access controls at the platform layer by restricting access to specific URLs and HTTP methods based on the user's role
```
DENY: POST, /admin/deleteUser, managers
#This denies access to the POST method on the URL for users in the managers group
```
```
> An attacker can bypass this by providing X-Original-URL or X-Rewrite-URL in the request header
> Assume an attacker wants to delete a user using admin's functionality with a normal user
> Make a request for the / page and intercept with burpsuite
```
In the intercepted request, add another request header
```
X-Original-URL: /admin
#This will redirect the normal user to the /admin page ignoring the original URL
```
To delete a user, an attacker will click on the delete function and intercept the request with burpsuite and the original URL should be
```
GET /admin/delete?username=carlos
-- remove the /admin/delete and another request header X-Original-URL: /admin/delete
```
This will delete the user Carlos without the normal user authentication actually has the admin's functionality
### Method-based broken access control
```
> Log in using the admin credentials, promote carlos and send the HTTP request to Burp Repeater
> Open a private/incognito window and log in into the app with non-admin credentials and copy that user's session cookie
> go back to the Burp Repeater and replace the user's cookie with the current one
> send the request and observe that the response says "Unauthorized"
> change the method from POST to POSTX and the response says "Missing Parameter"
> convert the request method by right-clicking and selecting change request method
> change the username parameter to the non-admin user and send the request
```

## Horizontal access controls
### Userid controlled by request parameter with unpredictable user id
```
> some applications use GUIDs to identify users and these GUIDs can sometimes be disclosed elsewhere in the application where users are referenced, such as user messages, comments or reviews
> log in to the app with authenticated credentials and go to the page where users are posting their blogs
> notice when the page reloads, the url changes from http://example.com to http://example.com?guids=fdhd-fdf-fdfd
> that is the GUID's for the user that posted the blog
> go the user profile page for your own account and paste the GUID in the URL and observe you can now access another user profile page
```
### Userid controlled by request parameter with data leakage in redirect
```
> log in using any authenticated credentials and navigate to user profile page
> send the request to Burp Repeater
> change the id parameter to carlos
> Observe that although the response is now redirecting you to the home page, it contains some informations of the user carlos's profile
```

## Idor
Assume a website that saves chat message transcripts to disk using an incrementing file name and allow users to retrieve these by visiting a URL like "http://example.com/static/12.txt". An attacker could simply change the file name into something else and possibly find a crucial information about other users

## Access control in multi-step processes
```
> log in using the admin credentials
> promote carlos and send the confirmation HTTP request to Burp Repeater
> Open a private browser window and log in with non-admin credentials and copy the user's session cookie
> go back to Burp Repeater and replace the user's cookie with the current one
> change the username parameter to yours and send the request
```

## Referer-based access control
```
> suppose an application enforces access control over the main administrative page at /admin but for sub-pages like /admin/delete only inspects the Referer header
> if the Referer header contains the main /admin URL, then the request is allowed
```

## Location-based access control
```
> some application enforces access controls over resources based on user's geographical location
> this access controls can be circumvented by the use of web proxies,VPNs or manipulation of client-side geolocation mechanisms
```

## Read forbidden page source code
Append "~" to a filename to read its source code
Append "s" to a .php file
```
http://example.com?file=/etc/passwd~
```

## Bypassing 403 response
### Using "X-Original-URL" header
```
X-Original-URL: /admin
```
### Appending "%2e" after the first slash
```
http://example.com/%2e/admin
```
### Add dot (.), slash (/) & semicolon (;) in URL
```
http://example.com/secret/.
http://example.com/secret//
http://example.com/./secret/..
http://example.com/;/secret/
http://example.com/.;/secret
http://example.com//;//secret
```
### Add "..;/" after path name
```
http://example.com/admin..;/
```
### Uppercase the characters in URL
```
http://example.com/AdmIN
```


# Web-sockets

## Summary
* [Intercept & modify websocket message](#intercepting--modifying-websocket-messages)
* [Replaying & generating new websocket messages](#replaying--generating-new-websocket-messages-with-burp-repeater)
* [Manipulating websocket connection](#manipulating-websocket-connection)
* [Using Cross-site websockets](#using-cross-site-websockets)

## Intercepting & modifying websocket messages
Intercept the application function that uses WebSockets & modify the content that is going to be send
``` 
<img src=1 onerror='alert(1)'>
```
What happends in the process is that we first intercept the content that is going to be send then a second request displaying what is going to be send to the server. Then, a third request showing that the server successfully accepted the request from the website with websocket function & finally fourth interception, the result of what is going to be send by the server.

## Replaying & generating new websocket messages with Burp Repeater

## Manipulating websocket connection
Intercept the application function that uses WebSockets and modify the content that is going to be send
```
> <img src=1 onerror='alert(1)'>
```
Notice the attack failed because of XSS filtered and the websocket connection has been terminated & your IP address has been banned from using the site again. Then, modify the content that is going to be send.
```
> <img src=1 oNeRrOr=alert'1'>
```

## Using Cross-site websockets

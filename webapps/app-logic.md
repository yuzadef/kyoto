# Application logic

## Summary
* [Excessive trust in client-side controls](#excessive-trust-in-client-side-controls)
    * [Changing the price value for an item](#changing-the-price-value-for-an-item)
    * [2FA broken application logic](#2fa-broken-application-logic)
* [Failing to handler unconventional input](#failing-to-handle-unconventional-input)
    * [Decrese the total price of item's cart](#decrease-the-total-price-of-items-cart)
    * [Increase the price until maximum value set in the back-end](#increase-the-price-until-maximum-value-set-in-the-back-end)
    * [Gaining administrative access](#gaining-administrative-access-from-inconsistent-handling-of-exceptional-input)
* [Flawed assumptions on user behaviour](#flawed-assumptions-about-user-behaviour)
    * [Inconsistent security controls](#inconsistent-security-controls)
    * [Weak isolation on dual-use endpoint](#weak-isolation-on-dual-use-endpoint)
    * [Password reset broken logic](#password-reset-broken-logic)
    * [2FA simple bypass](#2fa-simple-bypass)
    * [Insufficient workflow validation](#insufficient-workflow-validation)
    * [Authentication bypass via flawed state machine](#authentication-bypass-via-flawed-state-machine)
* [Domain specific flaws](#domain-specific-flaws)
    * [Flawed encorcement of business rules](#flawed-enforcement-of-business-rules)

## EXCESSIVE TRUST IN CLIENT-SIDE CONTROLS
### Changing the price value for an item
```
> suppose you want to buy an item which the price has exceeded your account balance
> add your item into the cart and intercept with Burp Proxy
> notice in the request, there is a few parameter that needs to be noted, but the most important in this context is the price parameter
> change the price value to 10 and forward the request
> in the application, go to the /cart page and notice that the price for your item is 10 and now you can pay for it eventhough the price is originally high
```
### 2FA broken application logic
```
> log in using the provided credentials and the app is asking for a 2FA code that has been sent to the user's email
> submit an invalid 2FA code and send the request to Burp Intruder
> notice the verify parameter is used to determine which user's account is being accessed
> change the value to other user and add a payload position to the 2FA-code parameter
> brute force the verification code
```

## FAILING TO HANDLE UNCONVENTIONAL INPUT
### Decrease the total price of item's cart
```
> suppose you want to buy an item which the price has exceeded your account balance
> add your item into the cart, since your account balance is insufficient we need to decrease the total price of the item's cart
> an attacker can do this by adding another item of different prices and set the quantity into negative value, then add it to cart
> notice the total price has decreased a little because of the negative value for quantity
> add a suitable negative quantity of the another item to reduce the total price to match your account balance
```
### Increase the price until maximum value set in the back-end
```
> suppose you want to buy an item but you only have $100 store credit
> add your item into the cart, since your account balance is insufficient we need to increase the total price so it exceeds the maximum value and it will loops back and falls between $0 and $100
> add another item of different prices and send the request to burp repeater
> set the quantity parameter to 99 and clear the payload's position
> on the payload's tab, set the payload type as Null Payloads and select Continue Indefinitely
> start the attack and go back to the /cart page and refresh
> notice now the total price has exceeded the maximum value allowed in the back-end so it loops back to the minimum possible value which is a negative value (-2,157,574,584)
> refresh the page again and notice now the price has fallen between $0 and $100, now you can check out the items 
```
### Gaining administrative access from inconsistent handling of exceptional input
```
> navigating to the /admin page says that only DontWannaCry users can access
> try registering with any username and password but create a very long string for the email
    # ffffffffff{280 chars}@gmail.com
> confirm the registration and log in into your account
> notice that the rest of your 35 characters from your very long string email has been truncated because the website only accepts the first 255 characters
> since we dont have a DontWannaCry account, we still want to register and receive the confirmation email using @gmail.com
    # register a new account with another long email address but this time includes dontwannacry.com as subdomain in the email address
    # makes sure that the final letter 'm' from dontwannacry.com is character 255 exactly
    # ffffffffff{238 char}@dontwannacry.com.gmail.com and click register
> note that the email will still be send to our email client only because the inconsistent handling of exceptional input
    # go to your email account and click the confirmation link and we are registered
> now go to the user profile and look at the email
# ffffff{238char}@dontwannacry.com but the rest of the string (.gmail.com) is not included since it gets truncated
> since the email in the user profile ends with the subdomain of dontwannacry, this means the user can access the admin page
```

## FLAWED ASSUMPTIONS ABOUT USER BEHAVIOUR
### Inconsistent security controls
```
> navigating to the /admin page says that only DontWannaCry users can access the page
> try registering with any username,password and email then confirm the registration
> log in to your account and go the /profile page
> notice that there is a functionality for users to change their email
> since only dontwannacry users can access admin's panel, maybe we can change our email
    # example@dontwannacry.com
> this will give us the privileges of adminstrator
> this happens because the developers assume that trusted users can always remain trustworthy
```
### Weak isolation on dual-use endpoint
```
> log in with any authenticated users and navigate to the profile page
> notice that to change your password, you must fill in all parameter's values
    # username
    # current-password
    # new-password1
    # new-password2
> submit a request to change your user's password and send the request to burp proxy
> in burp proxy, remove the parameter current-password with its value and forward the request
> observe that the submission to change the password is still carried out
> this means we can change some of the values to get admistrative access
    # change the username into admistrator
    # set the current-password with arbitrary values
    # set a new password for the user administrator and confirm
> submit and send the request to burp proxy
> remove the parameter current-password with its value and forward the request
> looking at the response, the password change is successful
> log out from current user and log in with admistrator credentials
> this happens because the developers assume that users will always supply mandatory input
```
### Password reset broken logic
```
> in the log in page, click the forget password and enter a valid email
> go to your email client and click the link
> submit a new password and confirm, then send the request to burp proxy
> in burp proxy, notice there is username parameter aside from the password's parameter
> change the username value to other carlos
> log in with carlos's credentials with the newly updates password
> this happens because the developers assume that users will always supply mandatory input
```
### 2FA simple bypass
```
> log in with your account and submit the 2FA code
> take note of the URL and log out
> now log in with other user's credentials
    # carlos:montoya
> in the URL, change the /login to /my-account
> this will make the application skips the 2FA and go straight into logging in into carlos's account
> this happens because the developers assume that users will always follow the intended sequence
```
### Insufficient workflow validation
```
> suppose you want to buy an item of price that has exceeded your account balance
> add any item that you can afford into the cart and observe you are redirect to confirmation page
> take note of the URL
    # example.com/cart/order-confirmation?order-confirmation=true
> now clear your cart and add the item that is more expensive
> check out for the item and send the request to burp proxy
> replace the POST /cart/checkout
    # POST /cart/order-confirmation?order-confirmation=true
> now change the request method to GET since the confirmation page only supports GET
> observe that your items has been checked out
> this happens because the developers assume that users will always follow the intended sequence
```
### Authentication bypass via flawed state machine
```
> log in with any authenticated user and you will be redirected to /role-selector page to choose your user role
> log out and log in again, in the /role-selector page send the request to burp proxy
> in burp proxy, drop the request and browse to the home page of the website
> notice that you get the administrative access to the application because the application assume that the user will complete the process of selecting role
```

## DOMAIN SPECIFIC FLAWS
### Flawed enforcement of business rules
```
> log in and notice there is a coupon code for new users
> at the bottom of the page, you can receive another coupon code by signing up for the newsletter
> add an item to the cart
> go the checkout and apply both coupon codes
> trying to apply the codes more than once was a failure however we can alternate between the two codes to bypass this kind of control
> reuse the two codes enough time until it drops low enough
```

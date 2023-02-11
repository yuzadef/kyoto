# Google dorking

## Summary
* [Find text from a webpage](#find-text-from-a-webpage)
* [Find for texts with multiple keywords](#find-for-texts-with-multiple-keywords)
* [Find files with extension](#find-files-with-extension)
* [Find for texts from a file with specific extension](#find-for-texts-from-a-file-with-specific-extension)
* [Find title with a keyword](#find-title-with-a-keyword)
* [Find for other similar titles](#find-for-other-similar-titles)
* [Find links with specified keywords](#find-links-with-specified-keywords)
* [Find links with multiple keywords](#find-links-with-multiple-keywords)
* [Find for specific website](#find-for-specific-websites)
* [Find files with extension](#find-files-with-extension)
* [Find for keywords within the post](#find-for-keywords-within-the-post)
* [Check a website cached data](#check-a-website-cached-data)
* [Find pages with inbound pages](#find-pages-with-inbound-pages-with-inbound-links-that-contain-anchor-text)
* [Search only on the social media platform](#search-only-on-the-social-media-platform)
* [Search related contents on multiple site](#search-related-contents-on-multiple-site)
* [Get info on a domain name](#get-info-on-a-domain-name)
* [See devices that share weather details](#see-devices-that-share-weather-details)
* [Find a zip SQL file](#find-a-zipped-sql-file)
* [Find admin login page](#find-a-site-admin-login-page)
* [Find a network service](#find-a-network-service-like-ftp)
* [Accessing online cameras](#accessing-online-cameras)
* [Search for sites contain any keywords](#search-for-sites-contain-any-keywords)
* [Search for sites contain all keywords](#search-for-sites-contain-all-keywords)
* [Combine all & any to search for sites](#combine-all--any-to-search-for-sites)
* [Common keywords for dorking](#common-keywords-for-dorking)

## Find text from a webpage
```
intext:usernames
```
## Find for texts with multiple keywords
```
allintext:"username""password"
```
# Find files with extension
```
filetype:log
```
## Find for texts from a file with specific extension
```
allintext:username filetype:pdf
```
## Find title with a keyword
```
intitle:"ip camera"
```
## Find for other similar titles
```
allintitle:"how to draw" "how to sing"
```
## Find links with specified keywords
```
inurl:tesla
```
## Find links with multiple keywords
```
allinurl:tesla lambo
```
## Find for specific websites
```
site:https://google.com
```
## Find files with extension
```
ext:pdf
```
## Find for keywords within the post
```
inposttitle:good morning
```
## Check a website cached data
```
cache:http://example.com
```
## Find pages with inbound pages with inbound links that contain anchor text
```
allinanchor:"How to draw anime"
```
## Search only on the social media platform
```
messi @instagram
```
## Search related contents on multiple site
```
"Related:terrorists"
```
## Get info on a domain name
```
"Info:example.com"
```
## See devices that share weather details
```
intitle:"Weather Wing WS-2"
```
## Find a zipped SQL file
```
"Index of" "database.sql.zip"
```
## Find a site admin login page
```
intitle:"Index of" wp-admin
```
## Find a network service like FTP
``` 
intitle:"index of" inurl:ftp
```
## Accessing online cameras
```
intitle:"webcamXP 5"
```
## Search for sites contain any keywords
```
site:facebook.com | site:twitter.com
```
## Search for sites contain all keywords
```
site:facebook.com & site:twitter.com
```
## Combine all & any to search for sites
```
(site:facebook.com | site:twitter.com) & intext:"login"
```
## Common keywords for dorking
```
login
register
upload
contact
feedback
join
signup
profile
user
comment
api
developer
affiliate
careers
mobile
upgrade
passwordreset
php
aspx
jsp
txt
xml
bak
```

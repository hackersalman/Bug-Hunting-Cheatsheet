### Improper implementation of the implicit grant type
In the implicit flow, this POST request is exposed to attackers via their browser. As a result, this behavior can lead to a serious vulnerability if the client application doesn't properly check that the access token matches the other data in the request. In this case, an attacker can simply `change the parameters` sent to the server to impersonate any user.

### Exploiting Absense of State Parameter
In Oauth mechanism, `state` parameter in authorization request work as a CSRF token. Absense of this parameter can be exploitable to CSRF attack.
<br>
**Here is an example of authorization request without state parameter**
```
GET /authorization?client_id=12345&redirect_uri=https://client-app.com/callback&response_type=code&scope=openid%20profile HTTP/1.1
Host: oauth-authorization-server.com
```

### OAuth Account Hijacking via `redirect_uri`

Depending on the grant type, the `redirect_uri` parameter is used to send the authorization code to the specified domain in the `redirect_uri`. If an OAuth mechanism allows arbitrary domains to be specified in the `redirect_uri`, an attacker could exploit this flow to hijack the authorization code.

For example:  
```
redirect_uri=https://client-app.net
```
The code is sent to `client-app.net`.  
```
redirect_uri=https://attacker.com
```
The code is sent to `attacker.com`.<br>
<br>
**Note:** If this process not work, try with `ssrf defense bypass` technique and `parameter pollution`.
<br>Check this for more details >> [Flawed redirect_uri validation](https://portswigger.net/web-security/oauth#leaking-authorization-codes-and-access-tokens)

### Exploiting response_mode and redirect_uri

In some cases, changing `response_mode` can let you to bypass `redirect_uri` validation. You can change the value with `query`, `fragment`, `web_message`. Also if you notice that the `web_message` response mode is already in the request, this often allows a wider range of subdomains in the `redirect_uri`.
<br>
```
https://auth.example.com/authorize?
client_id=12345
&redirect_uri=https://client-app.net
&response_type=code
&response_mode=web_message
```

### Stealing OAuth access tokens via an open redirect

Check this for more details : [Portswigger Lab](https://portswigger.net/web-security/oauth/lab-oauth-stealing-oauth-access-tokens-via-an-open-redirect)



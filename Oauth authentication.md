### Improper implementation of the implicit grant type
In the implicit flow, this POST request is exposed to attackers via their browser. As a result, this behavior can lead to a serious vulnerability if the client application doesn't properly check that the access token matches the other data in the request. In this case, an attacker can simply `change the parameters` sent to the server to impersonate any user.

### Exploiting Absense of State Parameter
In Oauth mechanism, `state` parameter in authorization request work as a CSRF token. Absense of this parameter can be exploitable to CSRF attack.

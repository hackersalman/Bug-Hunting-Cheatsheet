## Index

1. [Unprotected functionality](Unprotected-functionality)
2. [Parameter based access control](Parameter-based-access-control)
3. [Broken access control resulting from platform misconfiguration Url and Method based](Broken-access-control-resulting-from-platform-misconfiguration-Url-and-Method-based)
4. [Referer based access control](Referer-based-access-control)
___

### Unprotected functionality

    admin
    administrator
    administrator-panel
    robots.txt
___
### Parameter based access control 
Some applications determine the user's access rights or role at login, and then store this information in a user-controllable location. This could be:

    A hidden field.
    A cookie.
    A preset query string parameter.

The application makes access control decisions based on the submitted value. For example:
```
https://insecure-website.com/login/home.jsp?admin=true
https://insecure-website.com/login/home.jsp?role=1
https://insecure-website.com/myaccount?id=123
```
This approach is insecure because a user can modify the value and access functionality they're not authorized to, such as administrative functions.

In some applications, the exploitable parameter does not have a predictable value. For example, instead of an incrementing number, an application might use globally unique identifiers (GUIDs) to identify users. This may prevent an attacker from guessing or predicting another user's identifier. However, the GUIDs belonging to other users might be disclosed (Information Discloser) elsewhere in the application where users are referenced, such as user messages or reviews.
___
### Broken access control resulting from platform misconfiguration Url and Method based
Some applications enforce access controls at the platform layer. they do this by restricting access to specific URLs and HTTP methods based on the user's role. Some application frameworks support various non-standard HTTP headers that can be used to override the URL in the original request, such as X-Original-URL and X-Rewrite-URL.
```txt
POST / HTTP/1.1
X-Original-URL: /admin/deleteUser
```
An alternative attack relates to the HTTP method used in the request. In this situation, try to change request method and see response.
___
### Referer based access control
 Some websites base access controls on the Referer header submitted in the HTTP request. The Referer header can be added to requests by browsers to indicate which page initiated a request.

For example, an application robustly enforces access control over the main administrative page at /admin, but for sub-pages such as /admin/deleteUser only inspects the Referer header. If the Referer header contains the main /admin URL, then the request is allowed.
```Referrer: https://example.com/admin/deleteUser```
In this case, the Referer header can be fully controlled by an attacker. This means that they can forge direct requests to sensitive sub-pages by supplying the required Referer header, and gain unauthorized access.
___

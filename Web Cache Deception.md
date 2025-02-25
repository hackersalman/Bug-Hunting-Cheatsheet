# **Web Cache Deception Techniques**

## **Table of Contents**

1. [Detecting Cached Responses](#detecting-cached-responses)  
2. [Cache Rules](#cache-rules)  
    - [1. Exploiting Static File Extension Rules](#1-exploiting-static-file-extension-rules)  
        - [Path Mapping Discrepancies](#path-mapping-discrepancies)  
        - [Path Delimiter Discrepancies](#path-delimiter-discrepancies)  
        - [Path Delimiter Decoding Discrepancies](#path-delimiter-decoding-discrepancies)  
        - [Web Cache Deception Delimiter Lists](#web-cache-deception-delimiter-lists)  
    - [2. Exploiting Static Directory Rules](#2-exploiting-static-directory-rules)  
        - [Normalization Discrepancies by the Origin Server](#normalization-discrepancies-by-the-origin-server)  
        - [Normalization Discrepancies by the Cache Server](#normalization-discrepancies-by-the-cache-server)  
    - [3. Exploiting File Name Cache Rules](#3-exploiting-file-name-cache-rules)  

---

**Note: Always Use Url Encoded Delimeter**

---

## **1. Detecting Cached Responses**

To confirm caching behavior, inspect response headers and time differences:

- **`X-Cache: hit`** – Response served from the cache.  
- **`X-Cache: miss`** – Response fetched from the origin server and potentially cached.  
- **`X-Cache: refresh`** – Cache was outdated and revalidated.

Additionally, check `Cache-Control` headers for caching directives like `public` with a non-zero `max-age`.

---

## **2. Cache Rules**

Attackers exploit web server cache rules in several ways. Below are common techniques:

---

### **1. Exploiting Static File Extension Rules**

#### **Description**  
Caching mechanisms often rely on file extensions such as `.css`, `.js`, or `.jpg` to categorize content as static. Attackers can use discrepancies to cache sensitive dynamic data.

---

#### **Path Mapping Discrepancies**

1. Add `/wcd` (an arbitrary path) after the original path.  
2. If the response is the same, append a static file extension like `.js` or `.css`.  
3. Use the crafted URL for exploitation.  

---

#### **Path Delimiter Discrepancies**

1. Add a delimiter (`;`) after the original path.  
2. If the response is the same, append `/wcd` with a file extension like `.js` or `.css`.  
3. Use the crafted URL for exploitation.

---

#### **Path Delimiter Decoding Discrepancies**

1. Similar to delimiter discrepancies but use encoded delimiters like `%3b` (URL-encoded `;`).  
2. Test the response and confirm caching behavior.  
3. Craft and exploit the malicious URL.

---

### **Web Cache Deception Delimiter Lists**

The following list includes common delimiters that can help identify vulnerabilities:

```txt
!
"
#
$
%
&
'
(
)
*
+
,
-
.
/
:
;
<
=
>
?
@
[
\
]
^
_
`
{
|
}
~
%00
%21
%22
%23
%24
%25
%26
%27
%28
%29
%2A
%2B
%2C
%2D
%2E
%2F
%3A
%3B
%3C
%3D
%3E
%3F
%40
%5B
%5C
%5D
%5E
%5F
%60
%7B
%7C
%7D
%7E
```

---

### **2. Exploiting Static Directory Rules**

#### **Description**  
Many web servers store static resources in predefined directories (e.g., `/static/`, `/assets/`, `/scripts/`). Cache rules often target these directories, making them susceptible to Web Cache Deception.

---

#### **Normalization Discrepancies by the Origin Server**

1. Add `/<static-directory-prefix>/..%2f<dynamic-path>` to the URL.  
   - **Example**: `/assets/..%2fprofile`.  

The origin server resolves the path traversal, serving dynamic content, while the cache stores it as static.

---

#### **Normalization Discrepancies by the Cache Server**

1. Use the reverse pattern: `/<dynamic-path>%2f%2e%2e%2f<static-directory-prefix>`.  
   - **Example**: `/profile%2f%2e%2e%2fassets`.  

The cache interprets it differently, potentially exposing sensitive data.

---

### **3. Exploiting File Name Cache Rules**

#### **Description**  
Certain files (e.g., `robots.txt`, `index.html`, `favicon.ico`) are often cached due to their infrequent changes. Attackers can exploit these caching mechanisms by targeting specific file names.

---

#### **Steps**  

1. Follow the same patterns used for normalization discrepancies but target filenames like:
   - `robots.txt`
   - `index.html`
   - `favicon.ico`.

Example crafted URL: `/robots.txt%2f%2e%2e%2fprofile`.  

The cache might store sensitive data while treating it as a standard file.

---

#### **Exploit**
```
<script>document.location="https://example.net/my-account/wcd.js"</script>
```

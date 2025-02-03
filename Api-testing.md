# Table of Contents
1. [API Reconnaissance](#api-reconnaissance)
   - 1.1. [Discovering API Documentation](#discovering-api-documentation)
   - 1.2. [Fuzzing](#fuzzing)
   - 1.3. [Burp Scanner](#burp-scanner)
   - 1.4. [Changing HTTP Methods](#changing-http-methods)
2. [Mass Assignment](#mass-assignment)
3. [Testing for Server-Side Parameter Pollution (SSPP)](#testing-for-server-side-parameter-pollution-sspp)
   - 3.1. [Testing the Query String](#testing-the-query-string)
   - 3.2. [Overriding Existing Parameters](#overriding-existing-parameters)
   - 3.3. [Testing in REST Paths](#testing-in-rest-paths)
   - 3.4. [Testing in Structured Data Formats](#testing-in-structured-data-formats)
4. [Examples and Techniques](#examples-and-techniques)
   - 4.1. [Truncating Query Strings](#truncating-query-strings)
   - 4.2. [Injecting Invalid Parameters](#injecting-invalid-parameters)
   - 4.3. [Injecting Valid Parameters](#injecting-valid-parameters)
5. [Special Cases](#special-cases)
   - 5.1. [Structured Data Formats (JSON, XML)](#structured-data-formats-json-xml)

---

**Related Topic:** [Server-Side-Parameter-Pollution](https://github.com/tasin-zucced/Bug-Hunting-Cheatsheet/blob/main/Server%20Side%20Parameter%20Pollution.md)

---

# API Reconnaissance

### 1.1. Discovering API Documentation
- Identify potential API documentation endpoints such as `/api-docs`, `/swagger`, `/openapi`, etc.
- Use automated tools or wordlists to fuzz for common API paths.
- Check for exposed API documentation, like OpenAPI or Swagger files, which can reveal endpoints, parameters, and authentication details.

### 1.2. Fuzzing
- Use wordlists based on common API naming conventions and industry-specific terms.
- Tools like Burp Suite, OWASP ZAP, or ffuf can help discover hidden or undocumented endpoints.

### 1.3. Burp Scanner
- Configure Burp Suite to perform automated vulnerability scans on API endpoints.
- Customize scan configurations to focus on specific vulnerabilities, such as authentication issues or sensitive data exposure.

### 1.4. Changing HTTP Methods
- Test different HTTP methods (GET, POST, PUT, DELETE, etc.) on the same endpoint.
- Check if changing the method alters the response or bypasses certain access controls.

---

# Mass Assignment

Mass assignment vulnerabilities occur when an application blindly accepts user input and sets multiple attributes on an object without validation. Attackers can exploit this to manipulate parameters or escalate privileges.

**Example Attack Scenario:**
A web app allows users to update their profile with fields like `name`, `email`.
```json
{
    "name": "Alice",
    "email": "alice@example.com",
}
```
If the server does not validate the incoming data. An attacker could add the `"isAdmin": true` field, thereby escalating privileges if the application does not restrict this.
```json
{
    "name": "Alice",
    "email": "alice@example.com",
    "isAdmin": true
}
```

---

# Testing for Server-Side Parameter Pollution (SSPP)

### 3.1. Testing the Query String
- Insert special characters like `#`, `&`, and `=` to observe the app's response.
- Test how parameters are parsed and if they can be used to manipulate server-side logic.

   **Examples:**
   - Using a `#` to truncate the server-side query:
     ```
     http://example.com/search?name=alice%23additionalData
     ```
   - Attempting to add an extra parameter:
     ```
     http://example.com/search?name=attacker&email=test@example.com
     ```

### 3.2. Overriding Existing Parameters
- Try injecting duplicate parameters to see which one the server processes:
   ```
   name=attacker&name=originalUser
   ```
- The server's behavior varies by technology:
   - **PHP/Apache:** Uses the last parameter.
   - **ASP.NET:** Concatenates the parameters.
   - **Node.js/Express/Apache Tomcat:** Uses the first parameter.
     
### 3.3. Testing in REST Paths
- Test RESTful APIs by manipulating URL path parameters.
- Use path traversal sequences:
   ```
   /users/carlos/../../../../../
   ```
- Fuzz for common API definition files:
   ```
   /../../../../openapi.json
   ```

### 3.4. Testing in Structured Data Formats
- Test requests containing structured data formats (e.g., JSON, XML).
- Inject parameters directly into JSON requests:
   ```json
   {
       "name": "bob",
       "access_level": "administrator"
   }
   ```

---

# Examples and Techniques

### 4.1. Truncating Query Strings
- Use the `#` character to attempt truncation:
   ```
   http://example.com/resource?param=value%23additional
   ```

### 4.2. Injecting Invalid Parameters
- Add an unexpected or invalid parameter:
   ```
   &invalidParam=unexpectedValue
   ```

### 4.3. Injecting Valid Parameters
- Add a known, valid parameter to observe how it is processed:
   ```
   name=attacker&email=attacker@example.com
   ```

---

# Special Cases

### 5.1. Structured Data Formats (JSON, XML)
- **JSON Example:** Modify an API request that uses JSON:
   ```json
   POST /myaccount
   {
       "name": "john",
       "role": "admin"
   }
   ```
- **XML Example:** Manipulate XML data for injection:
   ```xml
   <user>
       <name>john</name>
       <role>admin</role>
   </user>
   ```
- Structured data injection can be leveraged in request or response data.

---

## DNS Testing

1. [Testing Zone Transfer Vulnerability](#Testing-Zone-Transfer-Vulnerability)

## 1. Testing Zone Transfer Vulnerability

**Find the domain's nameservers (NS records)**

```bash
dig ns example.com
```
This will list the nameservers (e.g., `ns1.example.com`, `ns2.example.com`).

**Try zone transfer with each nameserver**

```bash
dig AXFR example.com @ns1.example.com
```
If not vulnerable, you'll see something like:

```
; Transfer failed.
```

**Tools for Automated Testing** (Recommended)

* **dnsenum**
  ```bash
  dnsenum example.com
  ```

## 2. Testing Subdomain Takeover

**Check CNAME of every 404 domains of target**

```bash
dig CNAME sub.example.com
```
If the 404 domains pointing to a third-party service, but there is no services running on that site, that means the domain is vulnerable to subdomain takeover.

## DNS Testing

1. [Zone Transfer](#zone-transfer)
2. [Subdomain Takeover](#subdomain-takeover)

---

## 1. Zone Transfer

**Find the domain's nameservers (NS records):**

```bash
dig ns example.com
```

This will list the nameservers (e.g., `ns1.example.com`, `ns2.example.com`).

**Try a zone transfer with each nameserver:**

```bash
dig AXFR example.com @ns1.example.com
```

If not vulnerable, you'll see something like:

```
; Transfer failed.
```

**Tools for Automated Testing (Recommended):**

* **dnsenum**

  ```bash
  dnsenum example.com
  ```

---

## 2. Subdomain Takeover

**Check the CNAME of every 404 subdomain on the target:**

```bash
dig CNAME sub.example.com
```

If the 404 subdomain points to a third-party service but there is no service running on that site, it means the domain is vulnerable to subdomain takeover.

**Tools for Automated Testing (Recommended):**

* **subjack**

  ```bash
  subjack -w all.txt -t 50 -timeout 30 -ssl -v -c /usr/share/wordlists/fingerprints.json -o subjack.txt
  ```

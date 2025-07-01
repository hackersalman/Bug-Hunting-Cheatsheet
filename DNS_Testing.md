## DNS Testing

1. [Testing Zone Transfer Vulnerability](#Testing-Zone-Transfer-Vulnerability)

## 1. Testing Zone Transfer Vulnerability

**Find the domain's nameservers (NS records)**

```bash
dig -t ns example.com
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

**Tools for Automated Testing**

* **dnsrecon**

  ```bash
  dnsrecon -d example.com -t axfr
  ```

# 2FA Bypass Testing Techniques

## **Index**
1. [Status Code Changes](#1-status-code-changes)
2. [Brute-Force OTP](#2-brute-force-otp)
3. [OTP Reuse](#3-otp-reuse)
4. [Cross-Account Token Test](#4-cross-account-token-test)
5. [Direct Dashboard Access](#5-direct-dashboard-access)
6. [Search for 2FA Codes](#6-search-for-2fa-codes)
7. [CSRF/Clickjacking on 2FA](#7-csrfclickjacking-on-2fa)
8. [Session Persistence](#8-session-persistence)
9. [OAuth 2FA Bypass](#9-oauth-2fa-bypass)
10. [Disabling 2FA Without Verification](#10-disabling-2fa-without-verification)
11. [Password Reset Without 2FA](#11-password-reset-without-2fa)
12. [Test Default OTPs](#12-test-default-otps)
13. [Request Manipulation](#13-request-manipulation)
14. [OpenID Misconfiguration](#14-openid-misconfiguration)
15. [OTP Expiry Check](#15-otp-expiry-check)
16. [Backup Code Abuse](#16-backup-code-abuse)
17. [Sensitive Info Exposure](#17-sensitive-info-exposure)
18. [Permanent Denial of Service (DoS) on Accounts](#18-permanent-denial-of-service-dos-on-accounts)
19. [Authenticated Actions Without 2FA](#19-authenticated-actions-without-2fa)
20. [Bulk OTP Testing in JSON](#20-bulk-otp-testing-in-json)
21. [X-Forwarded-Host](#X-Forwarded-Host)

---

## 1. Status Code Changes
- Check if altering response or status codes (e.g., 200, 403) during 2FA verification allows bypass.

## 2. Brute-Force OTP
- Test if the application allows repeated attempts to guess OTPs without blocking.

## 3. OTP Reuse
- Verify if the OTP can be reused after it has already been used once.

## 4. Cross-Account Token Test
- Request two OTPs for different accounts and see if one account's OTP can be used for another account.

## 5. Direct Dashboard Access
- Attempt accessing the dashboard URL directly without completing 2FA.
- If blocked, include the 2FA page as a referrer header and retry.

## 6. Search for 2FA Codes
- Use tools like **Burp Suite** to search response or JavaScript files for exposed 2FA codes.

## 7. CSRF/Clickjacking on 2FA
- Test if attackers can disable 2FA using CSRF (Cross-Site Request Forgery) or clickjacking attacks.

## 8. Session Persistence
- Check if enabling 2FA logs out all active sessions. If not, report the issue.

## 9. OAuth 2FA Bypass
- Check if using OAuth logins bypasses the need for 2FA. (This is rare but worth testing.)

## 10. Disabling 2FA Without Verification
- Test if 2FA can be disabled without entering a 2FA code.

## 11. Password Reset Without 2FA
- Test if resetting the account password through "Forgot Password" bypasses 2FA.

## 12. Test Default OTPs
- Try entering "000000" (or similar default codes) to see if the app accepts it as a valid OTP.

## 13. Request Manipulation
- Manipulate JSON requests to bypass 2FA:
  - Send a null value.
  - Change `"otprequired": true` to `false`.
  - Remove the 2FA-related code or parameter.
  - Use unexpected inputs (e.g., an email as an array).

## 14. OpenID Misconfiguration
- Check for misconfigurations in OpenID that might allow bypassing 2FA.

## 15. OTP Expiry Check
- Verify if OTPs remain valid for an excessive period (e.g., more than a few minutes).

## 16. Backup Code Abuse
- After logging in, generate a backup code request and check if it leaks valid codes.

## 17. Sensitive Info Exposure
- Check if the 2FA page reveals sensitive information (e.g., phone numbers or email addresses).

## 18. Permanent Denial of Service (DoS) on Accounts
- Abuse the system to lock an account:
  - Create an account with someone else's email (if email verification isn’t required) and enable 2FA.
  - If verification is required, use a verified account to enable 2FA, then change the email to the victim's.

## 19. Authenticated Actions Without 2FA
- Test if you can perform authenticated actions (e.g., update profile, create API tokens) without completing 2FA.

## 20. Bulk OTP Testing in JSON
- Test if multiple OTP values can be sent in a single request:
  ```json
  {
    "code": ["1000", "1001", "1002", ..., "9999"]
  }
  ```
## 21. X-Forwarded-Host
Using this header may send otp to attacker controlled server.
```
X-Forwarded-Host: burpcollaborator.com
```
---

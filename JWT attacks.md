# Table of Contents
1. [Important Paths That Reveal Server's Public/Private Key](#important-paths-that-reveal-servers-publicprivate-key)
2. [JWT](#jwt)
3. [Accepting Arbitrary Signatures](#accepting-arbitrary-signatures)
4. [Accepting Tokens with No Signature](#accepting-tokens-with-no-signature)
5. [JWT Authentication Bypass via Weak Signing Key (Bruteforcing)](#jwt-authentication-bypass-via-weak-signing-key-bruteforcing)
6. [JWT Header Parameter Injections](#jwt-header-parameter-injections)
    - [Injecting Self-Signed JWTs via the `jwk` Parameter](#injecting-self-signed-jwts-via-the-jwk-parameter)
    - [Injecting Self-Signed JWTs via the `jku` Parameter](#injecting-self-signed-jwts-via-the-jku-parameter)
    - [Injecting Self-Signed JWTs via the `kid` Parameter](#injecting-self-signed-jwts-via-the-kid-parameter)
7. [Algorithm Confusion](#algorithm-confusion)
    - [Performing an Algorithm Confusion Attack via Exposed Key](#performing-an-algorithm-confusion-attack-via-exposed-key)
    - [Deriving Public Keys from Existing Tokens (Without Exposed Keys)](#deriving-public-keys-from-existing-tokens-without-exposed-keys)

---

## Important Paths That Reveal Server's Public/Private Key
```
/jwks.json
/.well-known/jwks.json
```

## JWT
JSON Web Tokens (JWTs) are a standard format for sending cryptographically signed JSON data between systems. They're commonly used in authentication, session management, and access control mechanisms. If an attacker can successfully modify a JWT, they may be able to escalate their privileges or impersonate other users. 

**JWT Format**: `header.payload.signature`

## Accepting Arbitrary Signatures
JWT libraries typically provide two methods: one for verifying tokens and another that just decodes them. For example, the Node.js library `jsonwebtoken` has `verify()` and `decode()`. Sometimes, developers mistakenly use `decode()` without verifying the signature, effectively allowing arbitrary values in the JSON data, enabling an attacker to modify user roles and escalate privileges.
<br>
<br>
**Steps:**
1. Add a character `x` after signature and check the response.
2. If the response maintain a logged in session wothout logging out, it means the token is vulnerable.
3. Now edit the payload info with other users data and send the request.

## Accepting Tokens with No Signature
1. Set the `alg` parameter value to `none`.
2. Set or change `username`, `role`, `id` in the payload `(Replace payload with another user data)`.
3. Remove the signature from the JWT, but leave the trailing dot after the payload and send the request.

## JWT Authentication Bypass via Weak Signing Key (Bruteforcing)
Developers sometimes forget to change default secrets or hardcoded example secrets in JWT applications. This makes it easy for attackers to brute-force the server's secret using a wordlist. Then craft a new jwt token with that secret key.

Example command using `jwt_tool/jwt`: `(Recommended)`
```
jwt <JWT Token> -C -d /usr/share/wordlists/rockyou.txt
```

Example command using `hashcat`:
```
hashcat -a 0 -m 16500 eyJraWQiOiIyZTI4MDIyYy02NWQ3LTRjNTEtYjU3Ni1mYWY3Mjg4MzVhMTUiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJwb3J0c3dpZ2dlciIsImV4cCI6MTcxMTIyMDc5MSwic3ViIjoiYWRtaW5pc3RyYXRvciJ9.gteqm9hkHA6PdSv0pqbXBeUCON_8kwelR7Be1NI6WMs ~/Wordlist/jwt.secrets.list
```
[Wordlist of common secret keys](https://github.com/wallarm/jwt-secrets/blob/master/jwt.secrets.list)

**Generate a forged signing key**

1. In Burp, go to the JWT Editor Keys tab.
2. Click New Symmetric Key.
3. Check `Specify secret` and add the secret key in the box.
4. Click Generate to generate a new key in JWK format.
5. Click OK to save the key.
6. In Burp `Repeater` >> `JSON Web Token` >> `Sign` >> `Don't modify header` >> `Send` request. 

## JWT Header Parameter Injections
JWT headers, also known as JOSE headers, often contain several parameters of interest to attackers:

- **`jwk` (JSON Web Key)**: Provides an embedded JSON object representing the key.
- **`jku` (JSON Web Key Set URL)**: Provides a URL from which servers can fetch a set of keys containing the correct key.
- **`kid` (Key ID)**: Provides an ID that servers use to identify the correct key.

### Injecting Self-Signed JWTs via the `jwk` Parameter
Servers should ideally use a limited whitelist of public keys to verify JWT signatures. However, misconfigured servers may accept any key embedded in the `jwk` parameter. This can be exploited by signing a modified JWT using your own RSA private key and embedding the matching public key in the `jwk` header.

**Steps**:
1. Send the request to Repeater.
2. Go to the JSON Web Token tab to change the payload info.
3. Generate a new RSA key in the JWT Editor.
4. In the `JSON Web Token` tab, select `Attack` -> `Embedded JWK` and add the generated `kid`.
5. Sign the key.
6. Send the HTTP request.

### Injecting Self-Signed JWTs via the `jku` Parameter
Instead of embedding public keys using the `jwk` parameter, some servers accept the `jku` parameter, allowing attackers to reference a JWK Set URL. If the server fetches the key from this URL, you can bypass authentication using the `jku` header.

**Steps**:
1. Send the request to Repeater.
2. Change payload info in the JSON Web Token tab.
3. Generate a new RSA key.
4. Host the RSA key in a JSON format on your exploit server at `/.well-known/jwks.json`.
   ```json
   {
       "keys": [
           {
               "kty": "RSA",
               "kid": "your-key-id",
               "n": "your-key-value",
               "e": "AQAB"
           }
       ]
   }
   ```
5. Change `kid` and `n` in exploit server to match your RSA key's `kid`, `n`.
6. Add the `jku` parameter, pointing to your exploit server in header of Json Web Tokens in Burp.
   ```
   "jku": "https://exploit.server/.well-known/jwks.json"
   ```
7. Sign the JWT with `Update/generate "alg", "typ" and "kid" parameters`.
8. Send the request.

### Injecting Self-Signed JWTs via the `kid` Parameter
If the server allows directory traversal in the `kid` parameter, an attacker can force the server to use an arbitrary file, like `/dev/null`, as the verification key. If the server supports symmetric algorithms, the JWT can be signed using an empty string.

**Steps**:
1. In the JWT header, set `kid` to `../../../../../../../dev/null`.
2. Change the payload `sub` claim to the desired user.
3. In the JSON Web Token tab, select Attack -> Sign with empty key.
4. Send the request.

**Alternative Approach with Generating a empty Symmetric Key**:
1. In the JWT header, set `kid` to `../../../../../../../dev/null`.
2. Change the payload `sub` claim to the desired user.
3. Generate new symmetric key in JWT editor.
4. Remove the value of `"K":""` and save the key.
5. Sign the JWT with generated new symmetric key with ```Don't modify Header```.
6. Send the request.

## Algorithm Confusion
Algorithm confusion vulnerabilities occur when JWT libraries rely on the `alg` parameter to determine the verification method. An attacker can trick the server into using a different algorithm, such as HS256, treating the public key as an HMAC secret.

### Performing an Algorithm Confusion Attack via Exposed Key

**If the server exposes public key at `jwks.json` path, you can do this attack.**
1. Copy the JWK object from inside the keys array.
**Example:**
```
{"kty":"RSA",
..
..
..
"n":"t2vWc1Vy8UpMF5TswF1xx4n2cOKil4VUqZL08eduWOQ5AkHJelK5wl6dAIuJqR8gkhciNH6wIFKw3uuSlWcREJ4eIrTYkS3x1RHPQFnTOrTpgbYFK9SDlc45PKMK5m5ZnKqMox4-U29NYemPMRiCnDZv8EmwTCfxuy17pRWEZfNH8PJaTpZFAfU2qvvWrf5YOPhpfg80Jvq52EUetpf3GdeMZP8NVkZNCvcIM0Q7eGKVnz6j1n_Ozc5DvYNVWUpv-MiCFIUyTXQKRaVsqg7qsDleY94BjRIqd6u0u48H5Bze1w3bI1m5mSwj3Sayj2H2F5bM-14x6nf2Nwkbq3T-6w"}
```
3. In Burp, go to the JWT Editor.
4. Select `create a new RSA key` and paste the JWK object that you copied and select is as a `PEM` format.
5. Copy and encode the PEM key to Base64 and create a new symmetric key in the JWT Editor.
6. Replace the `"k"` parameter with the Base64-encoded PEM key.
7. Set the `alg` header to `HS256`.
8. Modify the payload `(Don't modify header)` and sign the key with symmetric key you created.
9. Send the request.

### Deriving Public Keys from Existing Tokens (Without Exposed Keys)
1. Copy session cookie values from two different login attempts.
2. Run `sudo docker run --rm -it portswigger/sig2n <cookie1> <cookie2>`.
3. Identify the valid tempered JWT by using it and see if the JWT keeps you logged in.
4. Create a new symmetric key and paste the base64 verson of the identified tempered JWT in the `k` parameter.
5. Set the `alg` header to `HS256` and sign the key.
6. Modify the payload and send the request.

---

# Browser Security & Storage Notes

---

## 1. Browser Guardrails

### 1.1 SOP (Same-Origin Policy)

The **Same-Origin Policy (SOP)** is a critical security mechanism that isolates different websites from each other. It restricts read access to sensitive data across different origins.

**Example:** `malicious-site.com` cannot read response of an API call of `my-ecom-store.com/products`

**Rule:**  
A web browser permits scripts (mainly JavaScript) contained in one web page to access data in a second web page, but only if both pages have the same origin.

#### Definition of "Origin"
**Protocol + Domain + Port must match exactly**

- `https://bank.com/account` and `https://bank.com/settings` → Same Origin  
- `http://bank.com` and `https://bank.com` → Different Origin (**Protocol mismatch**)  
- `https://api.bank.com` and `https://bank.com` → Different Origin (**Subdomain mismatch**)

---

### 1.2 CORS (Cross-Origin Resource Sharing)

**Cross-Origin Resource Sharing (CORS)** is a browser-level security feature that allows a web server to explicitly authorize cross-origin requests.

#### How it works

1. **Simple Requests**
   - Browser sends request
   - Blocks response unless server includes:
     - `Access-Control-Allow-Origin: https://your-app.com`

2. **Pre-flight (OPTIONS)**
   - Used for risky requests (`PUT`, `DELETE`, custom headers like `X-CSRF-Token`)
   - Browser sends a "ping" request first
   - If server does not respond with:
     - `Allow-Headers`
     - `Allow-Methods`
   - The real request is **never sent**

---

### 1.3 CSP (Content Security Policy)

**Content Security Policy (CSP)** is an HTTP response header that acts as a browser-side security layer, allowing site owners to define trusted sources of content.

#### How it works

- **No Inline Scripts**
  - Blocks:
    - `<script>alert('hacked')</script>`

- **Domain Whitelisting**
  - Example:
    - Only allow scripts from `apis.google.com`

- **Anti-Exfiltration**
  - Restrict where browser can send data using:
    - `connect-src`

---

## 2. XSS (Cross-Site Scripting) — *The Inside Attack*

**XSS** is a vulnerability where attackers inject malicious client-side scripts into trusted websites.

### Potential Impact

- **Session Hijacking** → Stealing session tokens  
- **Credential Theft** → Fake login forms  
- **Defacement** → Changing UI  
- **Malware Distribution** → Redirecting users  

### Prevention Methods

- **Output Encoding**
  - Convert `<` → `&lt;`
  - Prevents execution

- **Input Validation / Sanitization**
  - Remove malicious code

- **Content Security Policy (CSP)**
  - Restricts script sources

- **Cookie Security**
  - Use `HttpOnly`

---

## 3. CSRF (Cross-Site Request Forgery) — *The Outside Attack*

**CSRF (XSRF)** tricks a logged-in user’s browser into performing unintended actions.

### How CSRF Works

1. **Authentication**
   - User logs into `bank.com`
   - Receives session cookie

2. **Manipulation**
   - Visits malicious site

3. **Forged Request**
   - Hidden request sent to `bank.com`

4. **Execution**
   - Browser auto-sends cookies
   - Server accepts request as valid

### Prevention Methods

- **Anti-CSRF Tokens**
  - Unique, secret, unpredictable token per request

- **SameSite Cookie**
  - `Lax` or `Strict` instructs the browser not to send cookies with cross-site requests.

- **Double Submit Cookies**
  - Same value in cookie + request param

---

## 4. Browser Storage Mechanisms

- **`localStorage`**
  - No expiration
  - Persistent

- **`sessionStorage`**
  - Per tab/session
  - Cleared on close

- **`IndexedDB`**
  - Large structured storage
  - Supports files/blobs

- **Cookies**
  - Sent with every request
  - Used for auth/session

- **Cache API**
  - Stores network responses
  - Used in service workers

### Comparison Table

| Feature        | LocalStorage     | SessionStorage    | Cookies               |
|----------------|----------------|------------------|-----------------------|
| Persistence    | Permanent       | Per-session (tab) | User-defined / Expire |
| Capacity       | 5–10 MB         | ~5 MB             | ~4 KB                 |
| Sent to Server | No              | No                | Yes                   |

---

## 5. Handling Sensitive Tokens

### 5.1 Storing Tokens in `localStorage`

⚠️ Vulnerable to **XSS**

Any script can access `localStorage`.

#### Hardening Measures

- **CSP**
- **Short Expiry**
- **Subresource Integrity (SRI)**

---

### 5.2 `HttpOnly` Cookies

Most secure approach for storing authentication tokens.

- `HttpOnly` → Not accessible via JavaScript  
- `Secure` → HTTPS only  
- `SameSite=Strict/Lax` → Prevents CSRF  

---

### 5.3 In-Memory Storage (Access Tokens)

Store access tokens in JavaScript variables.

#### Workflow

1. Store **Refresh Token** in `HttpOnly` cookie  
2. Store **Access Token** in memory  
3. On refresh:
   - Call `/refresh`
   - Get new Access Token  

---

## 6. Token Storage Comparison

| Feature         | LocalStorage                  | HttpOnly Cookie              | In-Memory (JS Var)      |
|----------------|-----------------------------|------------------------------|--------------------------|
| XSS Protection | ❌ Vulnerable                | ✅ Protected                  | ✅ Protected             |
| CSRF Protection| ✅ Immune                    | ❌ Needs SameSite/Token       | ✅ Immune                |
| Ease of Use    | High                        | Moderate                     | Low                     |
| Best For       | Preferences                 | Refresh Tokens / Sessions    | Short-lived Access Token |

## Browser's Guardrails

### SOP (Same-Origin Policy)

The Same-Origin Policy (SOP) is a critical security mechanism that isolates different websites from each other. It restricts read access to sensitive data across different origins. Example: `malicious-site.com` cannot read response of an API call of `my-ecom-store.com/products`

The Rule: A web browser permits scripts (mainly JS) contained in one web page to access data in a second web page, but only if both pages have the same origin.

#### Definition of "Origin": 

Protocol + Domain + Port must match exactly.

- `https://bank.com/account` and `https://bank.com/settings` → Same Origin
- `http://bank.com` and `https://bank.com` → Different Origin (Protocol mismatch)
- `https://api.bank.com` and `https://bank.com` → Different Origin (Subdomain mismatch)

### CORS (Cross-Origin Resource Sharing)

Cross-Origin Resource Sharing (CORS) is a browser-level security feature that allows a web server to explicitly authorize cross-origin requests from domains other than its own.

#### How it works
1. Simple Requests: For basic GET or POST requests, the browser sends the request but blocks the response unless the server includes the header: `Access-Control-Allow-Origin: https://your-app.com`
2. Pre-flight (OPTIONS): For "risky" requests (like those with custom headers such as `X-CSRF-Token` or `PUT/DELETE`), the browser sends a "ping" first.
   - If the server doesn't respond with a "thumbs up" (Allow-Headers, Allow-Methods), the real request is never even sent.

### CSP (Content Security Policy)

Content Security Policy (CSP) is an HTTP response header that acts as a browser-side security layer, allowing site owners to define trusted sources of content. By restricting which scripts, styles, and images can load, it serves as a "browser firewall".

How it works
The server sends a Content-Security-Policy header that tells the browser which sources of content (scripts, styles, images) are trusted.

- No Inline Scripts: By default, a strong CSP blocks `<script>alert('hacked')</script>` inside your HTML.
- Domain Whitelisting: You can tell the browser: "Only run scripts that come from `apis.google.com` or my own domain."
- Anti-Exfiltration: You can restrict where the browser is allowed to send data (using connect-src), preventing a hacker's script from sending your data to evil-hacker.com.

## XSS (Cross-Site Scripting) - The inside attack

Cross-Site Scripting (XSS) is a vulnerability where attackers inject malicious client-side scripts (usually JavaScript) into trusted websites. When victims visit the infected page, the script executes, allowing attackers to steal session cookies, impersonate users, or deface websites.

### Potential Impact

- Session Hijacking: Stealing session tokens to log in as the victim.
- Credential Theft: Capturing login credentials via fake forms.
- Defacement: Altering the website's appearance.
- Malware Distribution: Redirecting users to malicious sites.

### Prevention Methods

- Output Encoding: Encode untrusted data before rendering it in the browser to prevent it from being interpreted as active content (e.g., convert < to &lt;). React handles this automatically
- Input Validation/Sanitization: Filter user input to remove or neutralize malicious code.
- Content Security Policy (CSP): Implement a CSP header to restrict where scripts can be loaded from and prevent the execution of unauthorized inline scripts.
- Cookie Security: Use the `HttpOnly` flag on cookies to prevent them from being accessed by JavaScript.

## CSRF (Cross-Site Request Forgery) - The outside attack

Cross-Site Request Forgery (CSRF or XSRF) is a vulnerability where an attacker tricks a logged-in user’s browser into performing unwanted actions on a trusted website, such as changing passwords or transferring funds, without their consent. It exploits the automatic inclusion of cookies in HTTP requests. The most effective defense is using unpredictable, user-specific anti-CS RF tokens.

### How CSRF works

- Authentication: The user logs in to a vulnerable website (e.g., bank.com) and receives a session cookie.
- Manipulation: An attacker tricks the user into visiting a malicious site (e.g., via a link or advertisement).
- Forged Request: The malicious site sends a hidden request to bank.com (e.g., to transfer money).
- Execution: Because the browser automatically includes the user's bank.com session cookies, bank.com believes the request is authorized and executes it.

### Prevention Methods
- Anti-CSRF Tokens (Synchronizer Token Pattern): The most effective method is including a unique, secret, and unpredictable token in each request that the server validates.
- SameSite Cookie Attribute: Setting SameSite to Lax or Strict instructs the browser not to send cookies with cross-site requests.
- Double Submit Cookies: A fallback method where a random value is sent in both a cookie and a request parameter.

## Browser Storage Mechanisms

- `LocalStorage`: Stores data with no expiration date. The data remains even after the browser is closed. It is ideal for long-term user settings or site preferences.

- `SessionStorage`: Stores data only for the duration of the page session. The data is cleared when the tab or browser is closed.

- `IndexedDB`: A powerful, NoSQL-like database for storing large amounts of structured data, including files and blobs.

- `Cookies`: Small data files, typically used for session management and authentication. They are automatically sent with every server request.

- `Cache API`: Stores network requests and responses, often used by service workers for offline support and app performance.

| Feature        | LocalStorage     | SessionStorage    | Cookies               |
| -------------- | ---------------- | ----------------- | --------------------- |
| Persistence    | Permanent        | Per-session (tab) | User-defined / Expire |
| Capacity       | 5–10 MB (varies) | ~5 MB             | ~4 KB                 |
| Sent to Server | No               | No                | Yes                   |

## Handling sensitive tokens

### Storing sensitive tokens (like JWTs or session IDs) in `localStorage`

It is a common practice, but it's one that leaves your application vulnerable to **Cross-Site Scripting (XSS)**. Because any script running on your page can access `localStorage`, a single malicious dependency or injected script can compromise your user's entire session.

If you have no choice but to use `localStorage` (e.g., a legacy system or specific architectural constraint), you must harden your application:

- Content Security Policy (CSP): Implement a strict CSP to prevent unauthorized scripts from executing or exfiltrating data.
- Short Expiry: Keep token lifetimes extremely short.
- Subresource Integrity (SRI): Ensure third-party scripts (like analytics or UI libraries) haven't been tampered with.

### `HttpOnly` Cookies

The most secure way to store an authentication token is in an `HttpOnly`, `Secure`, and `SameSite cookie`.

- `HttpOnly`: This flag ensures that the cookie cannot be accessed through JavaScript (`document.cookie`). If an attacker injects a script, they still can't "read" the token.
- `Secure`: Ensures the cookie is only sent over encrypted (HTTPS) connections.
- `SameSite=Strict/Lax`: Prevents the cookie from being sent in cross-site requests, providing a strong defense against Cross-Site Request Forgery (CSRF).

### Use In-Memory Storage for Access Tokens

If your architecture requires the frontend to manually attach a token to the Authorization header (typical for APIs), keep the Access Token in memory (a simple JavaScript variable).

#### The Workflow:

1. Store the Refresh Token in an `HttpOnly` cookie.
2. Keep the Access Token in a variable inside your React/Vue/Svelte state.
3. When the user refreshes the page, the Access Token is lost. Your app should automatically call a /refresh endpoint (which reads the HttpOnly cookie) to get a new Access Token.

---

| Feature         | LocalStorage                     | HttpOnly Cookie                    | In-Memory (JS Var)            |
| --------------- | -------------------------------- | ---------------------------------- | ----------------------------- |
| XSS Protection  | Vulnerable (Scripts can read it) | Protected (Scripts can't read it)  | Protected (Scoped to closure) |
| CSRF Protection | Immune (Manual header needed)    | Vulnerable (Needs SameSite/Tokens) | Immune (Manual header needed) |
| Ease of Use     | High (localStorage.getItem)      | Moderate (Server-side setup)       | Low (Requires State Mgmt)     |
| Best For        | Theme, UX preferences            | Session IDs / Refresh Tokens       | Short-lived Access Tokens     |
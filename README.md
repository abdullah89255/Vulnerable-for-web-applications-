# Vulnerable-for-web-applications-
Great question! Here's a **focused breakdown of methods that are typically vulnerable in web applications** — by "methods," we can refer to both **HTTP methods** (like GET, POST) and **programming logic methods/functions** that are commonly exploited.

---

## 🔥 **Common Vulnerable Methods in Web Applications**

### 🧩 1. **HTTP Methods Misconfigurations**
These methods can be misused if not properly restricted:

- **GET** – Vulnerable to:
  - Information disclosure
  - Reflected XSS
  - IDOR (Insecure Direct Object Reference)
  
- **POST** – Often used in:
  - SQLi
  - CSRF
  - Broken authentication
  
- **PUT / DELETE / PATCH** – Should be **disabled unless explicitly needed**
  - PUT → Upload arbitrary files
  - DELETE → Remove sensitive data
  - PATCH → Modify server resources
  - These can lead to full **RCE** or server takeover

- **OPTIONS** – May reveal supported methods via **CORS misconfigurations**

---

### 💥 2. **Vulnerable Application Methods/Functions**
These are coding patterns or logic commonly exploited:

| Method/Function | Vulnerability | Description |
|------------------|------------------|-------------|
| `eval()` / `exec()` (JS, PHP, Python) | RCE | Executes arbitrary code |
| `system()` / `popen()` | Command Injection | Executes system commands |
| `include()` / `require()` (PHP) | LFI/RFI | Can include malicious files |
| `deserialize()` (Python, PHP, Java) | Insecure Deserialization | Leads to code execution |
| `file_get_contents()` (PHP) | SSRF / RFI | Used to read external/internal resources |
| URL parameters (e.g., `?id=123`) | IDOR | Changing values may expose other users' data |
| Unvalidated `redirect()` calls | Open Redirect | Redirects users to external malicious sites |

---

### 🎯 Examples of Exploitable Methods

#### ✅ GET
```http
GET /profile?id=1001
→ Change id to another user = IDOR
```

#### ✅ POST
```http
POST /transfer
body: amount=1000&to=12345
→ If no CSRF token = CSRF attack
```

#### ✅ PUT (Misconfigured)
```bash
curl -X PUT -d @shell.php http://target.com/uploads/shell.php
```

#### ✅ eval() Misuse
```php
$input = $_GET['cmd'];
eval($input); // Dangerous!
```

---

### 🧪 How to Test These Vulnerabilities
- **Burp Suite** – Intercept traffic & test for injection points.
- **Nmap --script http-methods** – Detect enabled HTTP methods.
- **Nikto** – Detect dangerous HTTP methods.
- **FFUF / Dirsearch** – Find hidden endpoints or methods.

---

### 🛡️ Mitigation Tips
- Limit HTTP methods to only those required.
- Use parameterized queries (to avoid SQLi).
- Sanitize all user input.
- Disable dangerous functions (e.g., `eval`, `exec`).
- Implement strong access control & auth checks.
- Validate redirects and uploaded files strictly.

---

Let me know if you want a cheat sheet or real examples of exploiting these! 🛠️

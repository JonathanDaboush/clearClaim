# Security Implementation Plan

**How to secure your e-signature platform** (Step-by-step guide)

---

## 🎯 Quick Overview

| Threat | Status | Effort |
|--------|--------|--------|
| SQL Injection | ✅ Solved | 0 days (automatic) |
| XSS Attacks | 📋 Ready to implement | 2 days |
| CSRF Attacks | 📋 Ready to implement | 2 days |
| File Upload Abuse | 📋 Ready to implement | 3 days |

**Total implementation:** ~1 week of focused work

---

## ✅ Already Solved

### SQL Injection Protection

**Status:** ✅ **Automatically handled by SQLAlchemy**

SQLAlchemy parameterizes all queries - you don't need to do anything special.

**Just remember:**
```python
# ✅ SAFE - automatic parameterization
user = User.query.filter_by(email=email).first()
docs = Document.query.filter(Document.user_id == user_id).all()

# ❌ NEVER DO THIS - SQL injection risk
# query = f"SELECT * FROM users WHERE email = '{email}'"
```

**That's it.** As long as you use SQLAlchemy's ORM methods, you're protected.

---

## 📋 To Implement (Plans Ready)

### 1. XSS Protection (2 days)

**What it prevents:** Attackers injecting malicious JavaScript

**3-Layer Implementation:**

#### Layer 1: Content Security Policy (30 min)
Add headers to all responses:
```python
Content-Security-Policy: default-src 'self'; script-src 'self'; object-src 'none'
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
```

#### Layer 2: Input Sanitization (4 hours)
```python
import bleach

# Sanitize before storing
clean_input = bleach.clean(user_input, tags=[], strip=True)
```

**Install:** `pip install bleach`

#### Layer 3: Output Encoding (2 hours)
```python
# In Jinja2 templates (auto-escapes by default)
{{ user_name }}  # Automatically escaped

# For React/Vue
// Use dangerouslySetInnerHTML ONLY for trusted content
```

**That's it.** 3 layers = XSS protection.

---

### 2. CSRF Protection (2 days)

**What it prevents:** Attackers tricking users into unwanted actions

**2-Part Implementation:**

#### Part 1: CSRF Tokens (6 hours)
```python
# Option A: Use Flask-WTF (easiest)
from flask_wtf import FlaskForm, CSRFProtect

csrf = CSRFProtect(app)

# Option B: Custom tokens
import secrets
token = secrets.token_urlsafe(32)
session['csrf_token'] = token
```

**Install:** `pip install Flask-WTF`

#### Part 2: Secure Cookies (2 hours)
```python
app.config.update(
    SESSION_COOKIE_SECURE=True,      # HTTPS only
    SESSION_COOKIE_HTTPONLY=True,    # No JavaScript access
    SESSION_COOKIE_SAMESITE='Strict' # Same-origin only
)
```

**That's it.** Tokens + secure cookies = CSRF protection.

---

### 3. File Upload Security (3 days)

**What it prevents:** Malicious PDFs, viruses, exploits

**7-Layer Validation:**

#### Layer 1: Filename Check (1 hour)
```python
import os
import re

def safe_filename(filename):
    # Remove path components
    filename = os.path.basename(filename)
    # Only allow safe characters
    filename = re.sub(r'[^a-zA-Z0-9._-]', '', filename)
    return filename
```

#### Layer 2: Extension Check (30 min)
```python
ALLOWED_EXTENSIONS = {'pdf'}

def allowed_file(filename):
    return '.' in filename and \
           filename.rsplit('.', 1)[1].lower() in ALLOWED_EXTENSIONS
```

#### Layer 3: Magic Number Check (2 hours)
```python
import magic

def is_real_pdf(file_path):
    mime = magic.from_file(file_path, mime=True)
    return mime == 'application/pdf'
```

**Install:** `pip install python-magic-bin` (Windows) or `python-magic` (Linux/Mac)

#### Layer 4: Size Check (15 min)
```python
MAX_FILE_SIZE = 10 * 1024 * 1024  # 10 MB

if len(file.read()) > MAX_FILE_SIZE:
    raise ValueError("File too large")
```

#### Layer 5: Virus Scan (4 hours)
```python
import pyclamd

cd = pyclamd.ClamdAgentSocket()
scan_result = cd.scan_file(file_path)

if scan_result:
    raise ValueError("Virus detected")
```

**Install:** `pip install pyclamd` + Install ClamAV on server

#### Layer 6: PDF Structure Check (2 hours)
```python
import PyPDF2

def validate_pdf_structure(file_path):
    with open(file_path, 'rb') as f:
        try:
            pdf = PyPDF2.PdfReader(f)
            # Check has pages
            if len(pdf.pages) == 0:
                return False
            return True
        except:
            return False
```

**Install:** `pip install PyPDF2`

#### Layer 7: Secure Storage (2 hours)
```python
import os
import uuid

# Generate random filename
unique_filename = f"{uuid.uuid4()}.pdf"

# Store outside web root
UPLOAD_FOLDER = '/var/uploads/'  # Not accessible via web

# Save with restricted permissions
file_path = os.path.join(UPLOAD_FOLDER, unique_filename)
os.chmod(file_path, 0o600)  # Owner read/write only
```

**That's it.** 7 layers = secure file uploads.

---

## 🛡️ Additional Security (Optional)

### Rate Limiting (1 day)
```python
from flask_limiter import Limiter

limiter = Limiter(app, key_func=lambda: request.remote_addr)

@app.route("/upload", methods=["POST"])
@limiter.limit("10 per hour")
def upload_file():
    # ...
```

**Install:** `pip install Flask-Limiter`

**Limits:**
- 10 uploads per hour
- 100 API calls per minute
- 5 login attempts per 15 min

---

## 📦 Complete Package List

```bash
pip install sqlalchemy          # SQL injection (automatic)
pip install bleach              # XSS protection
pip install Flask-WTF           # CSRF protection
pip install python-magic-bin    # File type detection (Windows)
pip install pyclamd             # Virus scanning
pip install PyPDF2              # PDF validation
pip install Flask-Limiter       # Rate limiting
```

---

## 🔐 Authentication Flow

### Login (with MFA)
1. User enters **email + password**
2. System validates credentials
3. System prompts for **TOTP code** (Google Authenticator)
4. Validate TOTP code
5. Create session (15 min timeout)
6. Set secure cookies (HttpOnly, Secure, SameSite)

### Signing a Document
1. User clicks "Sign" button
2. Check if session < 15 min old
3. If session expired: **Re-enter password** (not TOTP)
4. Collect device fingerprint
5. Record signature (user, timestamp, IP, device hash)
6. Add to blockchain (immutable audit entry)
7. Check unanimous consent status
8. Send notifications

**Key point:** TOTP only required at **login**, not for every signature.

---

## 🧪 Testing Checklist

### SQL Injection Tests
- ✅ Try `' OR '1'='1` in all inputs
- ✅ Try `'; DROP TABLE users; --`
- ✅ Verify all queries use ORM methods

### XSS Tests
- ✅ Try `<script>alert('XSS')</script>` in all inputs
- ✅ Try `<img src=x onerror=alert('XSS')>`
- ✅ Verify CSP headers present

### CSRF Tests
- ✅ Try POST request without token
- ✅ Try reusing old token
- ✅ Verify SameSite cookie setting

### File Upload Tests
- ✅ Try uploading .exe renamed to .pdf
- ✅ Try uploading 100 MB file
- ✅ Try uploading EICAR test virus
- ✅ Try uploading corrupted PDF

---

## 🚀 Implementation Order

**Week 1:** Core security
1. Day 1-2: XSS protection (CSP + bleach + encoding)
2. Day 3-4: CSRF protection (tokens + cookies)
3. Day 5-7: File upload security (7 layers)

**Week 2:** Additional hardening
1. Rate limiting
2. Security headers
3. Testing
4. Documentation

---

## 📚 Resources

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [SQLAlchemy Security](https://docs.sqlalchemy.org/en/20/faq/security.html)
- [Flask Security Guide](https://flask.palletsprojects.com/en/2.3.x/security/)
- [GDPR Article 17](https://gdpr-info.eu/art-17-gdpr/)

---

**Bottom line:** Most security is handled by libraries. Follow this guide = secure platform.

# Security Plan (Realistic)

**Building "secure enough" not "Fort Knox"**

---

## 🎯 Security Philosophy

**Goal:** Secure enough for most use cases

**Not building:** Bank-grade, military-grade, blockchain-everything

**Reality:** Most security comes from frameworks and libraries

---

## ✅ What Frameworks Handle

### SQLAlchemy = SQL Injection Protection

**You get this for free:**
```python
# ✅ Safe - SQLAlchemy parameterizes automatically
user = User.query.filter_by(email=email).first()
docs = Document.query.filter(Document.id == doc_id).all()
```

**Just don't do this:**
```python
# ❌ NEVER - manual SQL strings
query = f"SELECT * FROM users WHERE email = '{email}'"
```

**That's it.** Use the ORM = protected.

---

## 🔐 Authentication (Do These)

### 1. Password Hashing (30 min)

**Use bcrypt or argon2:**
```python
from werkzeug.security import generate_password_hash, check_password_hash

# Storing password
hashed = generate_password_hash(password)

# Checking password
valid = check_password_hash(hashed, password)
```

**Don't:**
- ❌ Roll your own crypto
- ❌ Store plaintext
- ❌ Use MD5 or SHA1

**Install:** `pip install flask[async]` (includes werkzeug)

---

### 2. Session Management (1 hour)

**Use Flask sessions or JWT:**

**Option A: Flask sessions (simpler)**
```python
from flask import session

app.config.update(
    SESSION_COOKIE_SECURE=True,      # HTTPS only
    SESSION_COOKIE_HTTPONLY=True,    # No JavaScript access
    SESSION_COOKIE_SAMESITE='Lax',   # CSRF protection
    PERMANENT_SESSION_LIFETIME=900   # 15 min timeout
)
```

**Option B: JWT (if you prefer)**
```python
pip install PyJWT
```

Pick one. Don't overthink it.

---

### 3. 2FA / TOTP (2-3 hours)

**Use pyotp:**
```python
import pyotp

# Setup (user enrolls)
secret = pyotp.random_base32()
totp = pyotp.TOTP(secret)
qr_code_uri = totp.provisioning_uri(user.email, issuer_name="ClearClaim")

# Verification (user logs in)
totp = pyotp.TOTP(user.totp_secret)
valid = totp.verify(user_entered_code)
```

**Install:** `pip install pyotp qrcode`

**When to require TOTP:**
- ✅ At login
- ❌ NOT for every signature (user fatigue)

---

## 📁 File Upload Security (4-6 hours)

### Layer 1: Extension Check (15 min)
```python
ALLOWED_EXTENSIONS = {'pdf'}

def allowed_file(filename):
    return '.' in filename and \
           filename.rsplit('.', 1)[1].lower() in ALLOWED_EXTENSIONS
```

### Layer 2: Size Check (15 min)
```python
MAX_SIZE = 10 * 1024 * 1024  # 10 MB

if len(file.read()) > MAX_SIZE:
    raise ValueError("File too large")
```

### Layer 3: Magic Number Check (1-2 hours)
```python
import magic

mime = magic.from_buffer(file.read(2048), mime=True)
if mime != 'application/pdf':
    raise ValueError("Not a PDF")
```

**Install:** `pip install python-magic-bin`

### Layer 4: Secure Storage (1 hour)
```python
import os
import uuid

# Random filename
filename = f"{uuid.uuid4()}.pdf"

# Store outside web root
UPLOAD_FOLDER = '/var/app/uploads/'  # Not /var/www/
filepath = os.path.join(UPLOAD_FOLDER, filename)

# Save with restricted permissions
with open(filepath, 'wb') as f:
    f.write(file.read())
os.chmod(filepath, 0o600)  # Owner read/write only
```

**That's enough for MVP.**

**Add later:**
- Virus scanning (ClamAV)
- Advanced PDF validation
- File type detection improvements

---

## 🛡️ Input Validation (Ongoing)

### Never Trust User Input

**Always validate:**
```python
from werkzeug.utils import secure_filename

# Filenames
safe_filename = secure_filename(user_filename)

# Emails
if '@' not in email or '.' not in email:
    raise ValueError("Invalid email")

# IDs
if not isinstance(doc_id, int) or doc_id < 1:
    raise ValueError("Invalid ID")
```

**Use framework validators:**
```python
from flask_wtf import FlaskForm
from wtforms import StringField, validators

class ProjectForm(FlaskForm):
    name = StringField('Name', [validators.Length(min=1, max=100)])
    description = StringField('Description', [validators.Length(max=500)])
```

**Install:** `pip install Flask-WTF`

---

## 🌐 HTTPS Only (1 hour setup)

**For production:**
- Use Let's Encrypt (free SSL)
- Redirect HTTP → HTTPS
- Set secure cookie flags

**For development:**
- HTTP is fine locally
- Use ngrok for testing

**Don't:**
- ❌ Roll your own SSL
- ❌ Use self-signed certs in production

---

## 🚫 What You DON'T Need (Yet)

### Skip These in MVP:

| Feature | Why Not MVP |
|---------|-------------|
| Device fingerprinting | Adds complexity, questionable value |
| Rate limiting | Add when you have traffic |
| WAF | Overkill for MVP |
| DDoS protection | Cloudflare free tier later |
| Penetration testing | Do after V1 |
| Security audit | Expensive, do when profitable |
| Blockchain anything | Overengineered |
| Advanced monitoring | Basic logs are fine |

**All of these can wait.** Focus on core security first.

---

## ✅ MVP Security Checklist

**Before launch, verify:**

- [ ] Passwords hashed with bcrypt/argon2
- [ ] HTTPS only in production
- [ ] Session cookies marked Secure + HttpOnly + SameSite
- [ ] 2FA (TOTP) working at login
- [ ] SQL queries use SQLAlchemy ORM (no raw SQL)
- [ ] File uploads validated (extension + size + magic number)
- [ ] Files stored outside web root
- [ ] Input validation on all forms
- [ ] Email verification working
- [ ] Password reset working
- [ ] Basic audit logging (who did what)

**That's a solid MVP security baseline.**

---

## 🧪 Testing Security

### Basic Tests:

**SQL Injection:**
```
Try: email = "' OR '1'='1"
Expected: Login fails, no SQL error
```

**File Upload:**
```
Try: Upload .exe renamed to .pdf
Expected: Rejected
```

**Session:**
```
Try: Copy session cookie to different browser
Expected: Works (that's normal)
Try: Use expired session
Expected: Redirected to login
```

**2FA:**
```
Try: Wrong TOTP code
Expected: Login fails
Try: Old TOTP code (30s+ old)
Expected: Login fails
```

---

## 📦 Security Dependencies

```bash
# Core
pip install flask
pip install sqlalchemy
pip install psycopg2-binary  # PostgreSQL

# Auth
pip install pyotp
pip install qrcode
pip install Flask-WTF

# File handling
pip install python-magic-bin  # Windows
# or python-magic  # Linux/Mac
pip install PyPDF2

# Utils
pip install python-dotenv  # Environment variables
```

---

## ⚖️ Security Claims (What to Say)

### ✅ Safe to Say:
- "Password encryption"
- "Two-factor authentication"
- "Secure file storage"
- "Activity logging"
- "Designed with security in mind"

### ❌ Don't Say:
- "Military-grade encryption"
- "Unhackable"
- "Bank-level security"
- "Fully compliant with [regulation]"
- "Blockchain-secured"

**Why:** Claims require proof. Stick to facts.

---

## 🔄 Security Updates

**Plan to revisit:**
- Update dependencies monthly
- Check for CVEs in libraries
- Review Flask security guide annually

**Tools:**
```bash
# Check for known vulnerabilities
pip install safety
safety check
```

---

## 🧠 Remember

**You're building:** Secure enough for most teams

**You're not building:** Fort Knox

**Your edge:** Unanimous agreement, not security theater

**The trap:** Overengineering security instead of shipping

**Ship MVP → Get users → Iterate on security based on real needs**

---

## 📚 Resources

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Flask Security Guide](https://flask.palletsprojects.com/en/2.3.x/security/)
- [SQLAlchemy Security](https://docs.sqlalchemy.org/en/20/faq/security.html)

**Read these. Don't reinvent security.**

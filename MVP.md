# MVP Feature Breakdown

**What to build in the next 6-10 weeks**

---

## 🎯 Core Principle

> **Does this help someone confidently sign or reject an agreement?**

If no → it's not in MVP.

---

## ✅ Features to Build

### 1. User Accounts (Week 1)

**Bare minimum:**
- Email + password registration
- Email verification (simple link)
- Login/logout
- Password reset
- Profile (name, email)

**Authentication:**
- TOTP 2FA (Google Authenticator)
- Enable at login, that's it
- No TOTP per action

**Libraries:**
- `Flask-Login` or JWT
- `pyotp` for TOTP
- `bcrypt` for password hashing

---

### 2. Projects (Week 1-2)

**Simple structure:**
- User creates project
- Project has name + description
- User can delete projects they own
- List all projects user has access to

**No:**
- ❌ Subprojects
- ❌ Complex permissions
- ❌ Templates

**Database:**
```
projects:
  id, name, description, owner_id, created_at
```

---

### 3. Groups (Week 2)

**Keep it simple:**
- Group = named collection of users
- User can create group
- Add/remove members from group
- That's it

**No:**
- ❌ Nested groups
- ❌ Permission inheritance
- ❌ Role hierarchies at group level

**Database:**
```
groups:
  id, name, project_id, created_at

group_members:
  group_id, user_id
```

**Roles applied at document level**, not group level.

---

### 4. Documents (Week 3)

**Upload workflow:**
1. User uploads PDF (max 10 MB)
2. Extract text for comparison later
3. Store file securely
4. Record metadata

**What to store:**
- Original filename
- File hash (SHA-256)
- Size
- Upload timestamp
- Extracted text (for diffs)

**Validation:**
- File extension (.pdf only)
- File size (< 10 MB)
- Magic number check (really a PDF)
- That's enough for MVP

**No:**
- ❌ Virus scanning (add later)
- ❌ Advanced PDF validation
- ❌ OCR for scanned PDFs

**Database:**
```
documents:
  id, project_id, filename, file_hash, size_bytes
  uploaded_by, uploaded_at, extracted_text, status
  
status: 'draft' | 'ready' | 'signing' | 'complete'
```

---

### 5. Participants & Roles (Week 3-4)

**Simple roles:**
- **Admin** - Can manage document, assign participants
- **Signer** - Required to sign (this is key)
- **Viewer** - Can view only

**Assignment:**
- Admin assigns users to document
- Admin sets role per user
- System tracks who needs to sign

**Database:**
```
document_participants:
  document_id, user_id, role, assigned_at
  
role: 'admin' | 'signer' | 'viewer'
```

---

### 6. Signing System (Week 4-5)

**Core workflow:**
1. User with 'signer' role views document
2. Clicks "Sign" button
3. Re-enter password (if session > 15 min)
4. System records signature
5. Check if all signers have signed
6. If yes → mark document 'complete'

**What to record:**
```
signatures:
  id, document_id, user_id
  signed_at, ip_address
```

**Certificate generation:**
- Simple PDF with:
  - Document name
  - All signers + timestamps
  - File hash
- Generate when document complete

---

### 7. Unanimous Agreement Enforcement (Week 5)

**The differentiator:**

System tracks:
- Total signers required
- Total signers who signed
- Who hasn't signed yet

**Rules:**
- Document can't be marked 'complete' until ALL sign
- Show clear status: "3 of 5 signed"
- Email reminders to unsigned participants
- Admin can't override (that's the point)

**Implementation:**
```python
def check_unanimous_consent(document_id):
    required = count_required_signers(document_id)
    signed = count_signatures(document_id)
    return required == signed
```

Simple. No complexity needed.

---

### 8. Basic Audit Log (Week 5-6)

**Just record:**
- Who (user_id)
- Did what (action: 'created', 'uploaded', 'signed', 'deleted')
- To what (resource_type + resource_id)
- When (timestamp)
- Where (ip_address)

**Database:**
```
audit_log:
  id, user_id, action, resource_type
  resource_id, timestamp, ip_address
```

**No:**
- ❌ Blockchain
- ❌ Immutability proofs
- ❌ Cryptographic signatures

**Just a simple append-only table.**

---

### 9. Simple Line Comparison (Week 6-7)

**When document updated:**
1. Extract text from new PDF
2. Run basic text diff against old version
3. Store diff results
4. Show "X lines added, Y lines removed"

**Libraries:**
- `PyPDF2` or `pdfplumber` for text extraction
- `difflib` for text diffing (Python built-in)

**Don't aim for:**
- ❌ Perfect visual mapping  
- ❌ 100% accuracy
- ❌ Page-by-page comparison

**Aim for:**
- ✅ "Roughly what changed"
- ✅ Useful enough to catch major edits

**Database:**
```
document_versions:
  id, document_id, version_number
  file_hash, extracted_text, created_at
  
document_diffs:
  id, document_id, from_version, to_version
  added_lines, removed_lines, changed_lines
```

**Keep it simple:**
```python
import difflib

old_text = get_text_from_pdf(old_version)
new_text = get_text_from_pdf(new_version)

diff = difflib.unified_diff(
    old_text.splitlines(),
    new_text.splitlines()
)

# Store basic stats
added = sum(1 for line in diff if line.startswith('+'))
removed = sum(1 for line in diff if line.startswith('-'))
```

---

## 📊 Storage Estimates (MVP)

**For 1,000 users, 10,000 documents:**

| Component | Size |
|-----------|------|
| PDFs | 10 GB (1 MB avg) |
| Extracted text | 500 MB |
| Diffs | 200 MB |
| User data | 10 MB |
| Audit logs | 50 MB |
| **TOTAL** | **~11 GB** |

**Monthly cost:** $200-300 (database + storage + small app server)

---

## ⏱️ Development Timeline

| Week | Focus |
|------|-------|
| 1 | User accounts + auth + 2FA |
| 2 | Projects + groups (simple) |
| 3 | Document upload + validation |
| 4 | Participants + roles |
| 5 | Signing workflow + unanimous enforcement |
| 6 | Audit log + notifications |
| 7 | Line comparison (basic) |
| 8-10 | Testing + polish + deployment |

**Total: 6-10 weeks to working MVP**

---

## 🚫 What's NOT in MVP

These can wait:

| Feature | Why Not MVP |
|---------|-------------|
| Blockchain audit | Overengineered, adds weeks |
| Device fingerprinting | Adds complexity without clear value |
| Virus scanning | Can add when needed |
| OCR | Most PDFs have selectable text |
| Subgroups | Flat structure is simpler |
| Advanced permissions | 3 roles is enough |
| Mobile app | Web works on mobile |
| API for integrations | Build after proving product |
| Analytics dashboard | Audit log is enough |
| Custom branding | Not needed for validation |

👉 **All of these can come in V2.** Focus on MVP first.

---

## ✅ MVP Success Criteria

**MVP is done when:**

A user can:
1. ✅ Sign up with email + password + 2FA
2. ✅ Create a project
3. ✅ Upload a PDF
4. ✅ Assign 3 people as signers
5. ✅ See who signed, who hasn't
6. ✅ System marks complete when all sign
7. ✅ Download completion certificate
8. ✅ See basic audit log

**If you can demo that flow → MVP is done.**

---

## 🎯 After MVP

**Only after MVP works, consider:**
- Advanced security (device fingerprinting, etc.)
- Blockchain audit trail
- Advanced diff visualization
- Mobile apps
- API for integrations
- Enterprise features

**But not before.** Prove the core first.

---

## 🧠 Remember

**Focus:** Unanimous agreement enforcement  
**Timeline:** 6-10 weeks  
**Test:** Can someone sign a document with required unanimous approval?  
**Trap:** Feature creep

Ship it. Get users. Iterate.

# What to Build

Simplified e-signature platform specification.

---

## Core Features (Keep It Simple)

### 1. User Accounts
- Email + password login
- TOTP authentication (Google Authenticator)
- Email verification
- Password reset
- Session management (4-hour timeout)

### 2. Projects
- Create/delete projects
- Flat structure (no subgroups)
- Invite users to projects

### 3. Roles (Only 3)
- **Admin:** Full control
- **Member:** Can view and sign
- **Guest:** View only (expires in 30 days)

### 4. Documents
- PDF upload only (10 MB max)
- One draft version (overwrites on edit)
- Final signed version (locked, immutable)
- Download signed documents

### 5. Signing
- Click "Sign" button
- Re-enter TOTP if session > 15 min old
- System records: user, timestamp, IP, document hash
- Generate completion certificate

### 6. Audit Trail
- Append-only log (can't be modified)
- Records: who, what, when, IP address
- 7-year retention
- Admins can export as CSV

---

## Technical Stack

### Backend
- **Python 3.10+**
- **Flask** or **FastAPI**
- **PostgreSQL** + **SQLAlchemy**
- **Redis** (sessions + rate limiting)

### Security
- SQLAlchemy (SQL injection prevention)
- bleach (XSS protection)
- Flask-WTF (CSRF protection)
- python-magic + ClamAV (file security)
- Flask-Limiter (rate limiting)

### Storage
- Local storage or AWS S3
- Encrypted (AES-256)
- Outside web root

---

## Database Schema (Simple)

### Users
- id, email, password_hash, totp_secret
- created_at, last_login

### Projects
- id, name, created_at

### Project_Members
- project_id, user_id, role
- (role: 'admin', 'member', 'guest')

### Documents
- id, filename, file_hash, size_bytes
- project_id, uploaded_by, status
- (status: 'draft', 'ready', 'signed', 'archived')

### Signatures
- id, document_id, user_id
- signed_at, ip_address

### Audit_Logs
- id, timestamp, user_id, action
- resource_type, resource_id, ip_address

---

## Key Endpoints

```
POST   /auth/register
POST   /auth/login
POST   /auth/logout
POST   /auth/reset-password

GET    /projects
POST   /projects
DELETE /projects/:id

POST   /projects/:id/members
DELETE /projects/:id/members/:user_id

GET    /projects/:id/documents
POST   /projects/:id/documents (upload)
GET    /documents/:id
POST   /documents/:id/sign
GET    /documents/:id/download

GET    /audit-logs (admin only)
```

---

## Security Implementation

### Phase 1: Core Security (Week 1)
- [x] SQLAlchemy (SQL injection) - automatic
- [ ] XSS protection (CSP + bleach)
- [ ] CSRF protection (tokens)
- [ ] File upload security (magic numbers + ClamAV)

### Phase 2: Additional Security (Week 2)
- [ ] Rate limiting (10 uploads/hour, 100 API calls/min)
- [ ] Security headers (HSTS, CSP, X-Frame-Options)
- [ ] Session security (HttpOnly, Secure, SameSite)

---

## UI Pages

1. **Login / Register**
2. **Dashboard** (list of projects)
3. **Project View** (documents + members)
4. **Document Viewer** (PDF viewer)
5. **Sign Page** (review + TOTP + sign button)
6. **Settings** (change password, TOTP)

---

## File Upload Flow

1. User uploads PDF
2. **Validate filename** (no path traversal)
3. **Check extension** (.pdf only)
4. **Check magic numbers** (verify it's really a PDF)
5. **Check size** (max 10 MB)
6. **Virus scan** (ClamAV)
7. **Validate PDF structure** (has %PDF and %%EOF)
8. **Generate random filename** (UUID)
9. **Store securely** (chmod 600, outside web root)
10. **Save metadata** to database (hash, size, etc.)

---

## Signing Flow

1. User clicks "Sign" on document
2. **Check session** (< 15 min old?)
3. If needed: **Re-enter TOTP**
4. **Show document** for review
5. User clicks **"I agree to sign"**
6. **Record signature** (user_id, timestamp, IP, doc_hash)
7. **Update document status** (if all signed → "signed")
8. **Generate certificate** (PDF with all signatures)
9. **Send notifications** (email to all parties)

---

## Rate Limits

- **Login attempts:** 5 per 15 minutes
- **API calls:** 100 per minute
- **File uploads:** 10 per hour
- **Failed logins:** Auto-ban 1 hour after 20 attempts

---

## Storage Estimates

### For 1,000 users, 10,000 documents, 1 year:
- User data: ~3 MB
- Documents: ~10 GB (1 MB average per PDF)
- Audit logs: ~50 MB
- **Total: ~10.5 GB**

vs. original design: **~51 GB** (80% savings)

---

## Compliance Features

### GDPR
- Hard delete (right to be forgotten)
- Data export (CSV/JSON)
- Privacy policy + ToS
- Consent tracking

### ESIGN/UETA/eIDAS
- Electronic signature capture
- Intent to sign (button click)
- Audit trail (who, when, IP)
- Tamper-evident (digital signature + hash)

---

## Development Timeline

### Month 1: Authentication
- User registration/login
- TOTP setup
- Email verification
- Password reset

### Month 2: Projects & Documents
- Project CRUD
- Role management
- Document upload (with security)
- PDF viewing

### Month 3: Signatures
- Signature assignment
- Signing workflow
- Certificate generation
- Email notifications

### Month 4: Security & Audit
- Complete security implementation
- Audit log system
- Security monitoring
- Admin dashboard

### Month 5: Polish & Launch
- GDPR compliance features
- Testing (penetration test)
- Documentation
- **Launch** 🚀

---

## What NOT to Build

Don't waste time on these (removed features):
- ❌ Blockchain audit logs
- ❌ Line-by-line diffs
- ❌ Complex role hierarchies (> 3 roles)
- ❌ Subgroups
- ❌ Device fingerprinting
- ❌ Soft-deletion
- ❌ TOTP for every action
- ❌ Revision history (except final versions)

---

## Success Metrics

| Metric | Target |
|--------|--------|
| Time to sign | < 30 seconds |
| Storage per 1K users | < 15 GB |
| API response time | < 100ms |
| Uptime | 99.9% |
| Cost per user/month | < $0.50 |

---

**Bottom Line:** Simple feature set. Strong security. Full compliance. 4 months to launch. 🎯

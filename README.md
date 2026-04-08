# ClearClaim - Enterprise E-Signature Platform

**Court-admissible, GDPR-compliant, blockchain-backed e-signatures**

---

## 🚀 What We're Building

**Enterprise e-signature platform** with full audit trail, blockchain verification, and GDPR-compliant data handling.

**Only simplification:** 5-level role hierarchy → 3 roles (Admin/Member/Guest)

---

## ✨ Key Features

✅ **Blockchain audit logs** - Tamper-proof chain of custody  
✅ **Line-by-line diffs** - Track every document change  
✅ **Device fingerprinting** - Flag suspicious activity  
✅ **Subgroup structures** - Nested project organization  
✅ **Unanimous consent** - Enforced signature workflows  
✅ **GDPR hybrid deletion** - EU: hard-delete | Non-EU: soft-delete  
✅ **MFA at login** - TOTP via Google Authenticator  
✅ **AES-256 encryption** - Data protection  

---

## 📊 Project Stats

| Metric | Value |
|--------|-------|
| Storage | 51 GB (1K users, 10K docs) |
| Monthly cost | $1,500 |
| Dev timeline | 8 months |
| Security | Enterprise-grade |
| Legal compliance | ESIGN, UETA, eIDAS, GDPR, PIPEDA, HIPAA |

---

## 📚 Documentation

| File | Read Time | Purpose |
|------|-----------|---------|
| [START_HERE.md](START_HERE.md) | 3 min | Overview & quick start |
| [DECISIONS.md](DECISIONS.md) | 5 min | Feature decisions & rationale |
| [SECURITY_PLAN.md](SECURITY_PLAN.md) | 8 min | Security implementation guide |
| [BUILD_SPEC.md](BUILD_SPEC.md) | 15 min | Complete technical specification |
| [LEGAL.md](LEGAL.md) | 5 min | Legal compliance verification |

---

## 🔒 Security Implementation

| Threat | Status | Solution |
|--------|--------|----------|
| SQL Injection | ✅ Solved | SQLAlchemy ORM (automatic) |
| XSS | 📋 Ready | CSP headers + bleach + encoding |
| CSRF | 📋 Ready | Tokens + SameSite cookies |
| File uploads | 📋 Ready | 7-layer validation + virus scan |

---

## 🛠️ Tech Stack

- **Backend:** Python 3.11+, Flask/FastAPI
- **Database:** PostgreSQL 14+, SQLAlchemy 2.0+
- **Cache:** Redis
- **Security:** bleach, Flask-WTF, python-magic, ClamAV
- **Storage:** S3-compatible (encrypted)

---

## 🎯 Next Steps

1. Read [START_HERE.md](START_HERE.md) for overview
2. Review [SECURITY_PLAN.md](SECURITY_PLAN.md) for implementation
3. Follow [BUILD_SPEC.md](BUILD_SPEC.md) for development

---

**Built for:** Healthcare, Legal, Finance, and regulated industries  
**Focus:** Security, compliance, and court-admissible evidence

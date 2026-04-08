# Start Here - ClearClaim

## What Is This?

**ClearClaim**: Enterprise e-signature platform with **one simplification** - 3 roles instead of 5.

Everything else is **full enterprise-grade** features.

---

## 🎯 What We're Building

### Core Features
1. **Blockchain audit logs** - Tamper-proof, court-admissible evidence
2. **Line-by-line diffs** - Track every document change (Git-style)
3. **Device fingerprinting** - Flag suspicious logins automatically
4. **Subgroup structures** - Nested project organization
5. **Unanimous consent** - Enforce signature workflows
6. **Hybrid deletion** - EU users: hard-delete | Non-EU: soft-delete (GDPR compliant)

### Only Simplification
- **5 roles → 3 roles** (Admin/Member/Guest)

### Security
- MFA at login (TOTP via Google Authenticator)
- Password re-entry for signatures after 15 min
- AES-256 encryption
- Device fingerprinting on every action

---

## 📊 Quick Numbers

| What | Amount |
|------|--------|
| Storage needed | 51 GB (for 1K users, 10K docs) |
| Monthly cost | $1,500 |
| Development time | 8 months |
| Security level | Enterprise-grade |

---

## ⚖️ Legal Compliance

✅ **ESIGN Act** (US)  
✅ **UETA** (US)  
✅ **eIDAS** (EU) - AdES signatures  
✅ **GDPR** (EU) - Hybrid deletion approach  
✅ **PIPEDA** (Canada)  
✅ **HIPAA** (Healthcare)

**GDPR Strategy:**
- Track user region at registration
- EU users: hard-delete on request
- Non-EU users: soft-delete (7-year retention)

---

## 🔒 Security Status

| Issue | Status |
|-------|--------|
| SQL Injection | ✅ Solved (SQLAlchemy) |
| XSS Attacks | 📋 Ready to implement |
| CSRF Attacks | 📋 Ready to implement |
| File Upload Abuse | 📋 Ready to implement |

---

## 📂 Next Steps

1. **[DECISIONS.md](DECISIONS.md)** - Why we kept/removed features
2. **[SECURITY_PLAN.md](SECURITY_PLAN.md)** - How to implement security
3. **[BUILD_SPEC.md](BUILD_SPEC.md)** - Complete technical spec
4. **[LEGAL.md](LEGAL.md)** - Legal compliance verification

---

**Bottom Line:** Build the simple version. It's cheaper, faster, more secure, and fully compliant. 🚀

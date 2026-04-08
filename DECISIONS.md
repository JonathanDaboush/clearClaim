# What to Keep & Remove

Simple decision guide for your e-signature platform.

---

## ❌ SIMPLIFY These Features

| Feature | Original | Simplified | Why? |
|---------|----------|------------|------|
| **Role hierarchy** | 5 levels | **3 roles** (Admin/Member/Guest) | Prevent privilege escalation bugs while maintaining access control |

**Note:** This is the ONLY simplification. All other features are kept at full enterprise level.

---

## ⚠️ SIMPLIFY These Features

| Original | Simplified | Why? |
|----------|------------|------|
| 5 roles | 3 roles (Admin/Member/Guest) | Simpler permissions |
| All document versions | Final + 1 draft only | 90% less storage |
| TOTP every signature | TOTP at login only | Less friction, still secure |
| Complex analytics | Simple charts | Easier to build |

---

## ✅ KEEP These Features

| Feature | Why Keep? | Legal/Security Requirement? |
|---------|-----------|-----------------------------|
| **Blockchain audit logs** | Tamper-proof chain of custody | eIDAS best practice |
| **Line-by-line file diffs** | Track all document changes | Audit compliance |
| **Device fingerprinting** | Additional security layer | Enhanced security |
| **Soft-deletion** | Retain for audit trail | ⚠️ **GDPR risk - use hybrid approach** |
| **Unanimous consent** | All parties must approve | Workflow security |
| **Subgroup structures** | Nested project organization | Enterprise feature |
| Email + Password + TOTP | MFA at login | eIDAS best practice |
| Comprehensive audit trail | Who, what, when, IP, device | ESIGN, UETA, eIDAS |
| AES-256 encryption | Data protection | GDPR, HIPAA |
| PDF digital signatures | Tamper-evident | ESIGN, UETA, eIDAS |
| 3 simple roles | Access control (simplified from 5) | GDPR |
| PDF-only uploads | Security & consistency | Best practice |

### Soft-Deletion Strategy (GDPR-compliant hybrid):
- **EU users:** Hard-delete on request (GDPR Article 17)
- **Non-EU users:** Soft-delete (retained for audit)
- Track user jurisdiction at registration
- Implement `is_deleted` flag + `deletion_type` field

---

## 💾 Storage Estimate (1,000 users, 10,000 docs)

| Component | Minimal | ClearClaim (Full Enterprise) |
|-----------|---------|------------------------------|
| **Blockchain logs** | 50 MB | 2 GB (full chain with hashes) |
| **Documents** | 10 GB | 10 GB |
| **Line-by-line diffs** | 0 GB | 33 GB (all versions) |
| **Device fingerprints** | 0 MB | 100 MB |
| **Soft-deleted data** | 0 GB | 6 GB (7-year retention) |
| **Subgroup metadata** | 0 MB | 50 MB (nested structures) |
| **TOTAL** | **10 GB** | **51 GB** |

**Note:** Full enterprise feature set = maximum audit trail and compliance.

---

## 🔒 Security Impact

### Vulnerabilities ELIMINATED:
1. ✅ Privilege escalation (complex roles)
2. ✅ Race conditions (consent logic)
3. ✅ Device spoofing
4. ✅ Custom crypto bugs
5. ✅ Diff injection attacks
6. ✅ GDPR violations
7. ✅ Auth fatigue exploits
8. ✅ Token timing attacks
9. ✅ Permission inheritance bugs
10. ✅ Data leakage (old versions)

### To Implement (Plans Ready):
1. ✅ SQL injection prevention - **SQLAlchemy solves this!**
2. ⚠️ XSS protection (CSP + sanitization)
3. ⚠️ CSRF protection (tokens)
4. ⚠️ File upload security (magic numbers + virus scan)

**Net Result:** 10 vulnerabilities eliminated, 3 to implement (with ready plans)

---

## 💰 Cost Impact

**Annual Savings:**
- Database: $4,800/year
- Storage: $1,920/year
- Compute: $7,200/year
- **Total: ~$14,000/year**

---

## ⚖️ Legal Check

| Requirement | Original Design | Simplified Design | Winner |
|-------------|-----------------|-------------------|--------|
| ESIGN Act | ✅ | ✅ | Tie |
| UETA | ✅ | ✅ | Tie |
| eIDAS | ✅ | ✅ | Tie |
| GDPR | ❌ **VIOLATED** | ✅ Compliant | **Simplified** |
| PIPEDA | ⚠️ Risky | ✅ Compliant | **Simplified** |

**Simplified design is MORE legally compliant.**

---

## 🎯 Decision Framework

When evaluating any feature, ask:

1. **Is it legally required?** → If NO, consider removing
2. **Do competitors have it?** → If NO, probably not needed
3. **What's the memory cost?** → If HIGH, strong case to remove
4. **What's the security risk?** → If HIGH, remove or simplify
5. **Can users live without it?** → If YES, remove

---

## ✅ Bottom Line

**Remove:** Everything on the "Remove" list above  
**Simplify:** Everything on the "Simplify" list  
**Keep:** Only what's legally required + essential security

**Result:** Faster, cheaper, more secure, legally compliant ✅

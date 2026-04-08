# Feature Decisions

**What we're keeping and why** (Quick Reference)

---

## 🎯 Quick Summary

**One simplification:** 5 roles → 3 roles  
**Everything else:** Full enterprise feature set

---

## ⚠️ Only Simplification

| Original | Changed To | Why? |
|----------|------------|------|
| 5-level role hierarchy | **3 roles** (Admin/Member/Guest) | Simpler permissions, prevents privilege escalation bugs |

---

## ✅ Features We're Keeping

### 🔒 Security & Audit
| Feature | Purpose | Legal/Business Need? |
|---------|---------|---------------------|
| **Blockchain audit logs** | Tamper-proof chain of custody | Court-admissible evidence |
| **Device fingerprinting** | Flag suspicious logins | Security best practice |
| **MFA at login (TOTP)** | Prevent unauthorized access | Industry standard |

### 📄 Document Management
| Feature | Purpose | Legal/Business Need? |
|---------|---------|---------------------|
| **Line-by-line diffs** | Track all changes (Git-style) | Audit compliance |
| **Full version history** | Keep all document versions | Legal evidence |
| **PDF digital signatures** | Tamper-evident signatures | ESIGN, UETA, eIDAS |

### 👥 Organization
| Feature | Purpose | Legal/Business Need? |
|---------|---------|---------------------|
| **Subgroup structures** | Nested project organization | Enterprise feature |
| **3 simple roles** | Access control | GDPR requirement |
| **Unanimous consent** | Workflow enforcement | Business requirement |

### 🗑️ Data Retention
| Feature | Purpose | Legal/Business Need? |
|---------|---------|---------------------|
| **Hybrid deletion** | EU: hard-delete, non-EU: soft-delete | GDPR Article 17 compliance |
| **7-year audit retention** | Compliance requirement | ESIGN, UETA, SOX |

---

## 📊 Storage Breakdown

**For 1,000 users, 10,000 documents:**

| Component | Size | What It Stores |
|-----------|------|----------------|
| Documents (PDFs) | 10 GB | Actual PDF files |
| Line-by-line diffs | 33 GB | All version changes |
| Blockchain logs | 2 GB | Immutable audit chain |
| Soft-deleted data | 6 GB | Non-EU deleted records |
| Device fingerprints | 100 MB | Security data |
| Subgroup metadata | 50 MB | Organization structure |
| **TOTAL** | **51 GB** | |

**Monthly cost:** ~$1,500 (database + storage + compute)

---

## 💰 Cost vs Value

### What You Get for $1,500/month:
- ✅ Court-admissible blockchain evidence
- ✅ Complete audit trail (every change tracked)
- ✅ GDPR compliant (better than DocuSign/Adobe)
- ✅ Enhanced security (device fingerprinting)
- ✅ Enterprise org structure (subgroups)

### Target Customers:
- Healthcare (HIPAA compliance)
- Legal firms (court evidence)
- Finance (regulatory compliance)
- Enterprise (full audit requirements)

**Charge:** $25-50 per user/month = profitable at 50-100 users

---

## ⚖️ Legal Compliance

| Requirement | Status | How We Comply |
|-------------|--------|---------------|
| ESIGN Act (US) | ✅ | Intent to sign + audit trail |
| UETA (US) | ✅ | Electronic signature equivalence |
| eIDAS (EU) | ✅ | AdES-level digital signatures |
| GDPR (EU) | ✅ | Hybrid deletion (EU=hard, non-EU=soft) |
| PIPEDA (Canada) | ✅ | Data protection + deletion rights |
| HIPAA (Healthcare) | ✅ | Encryption + audit logs |

---

## 🔐 Security Impact

### What We Gain:
- ✅ Blockchain = tamper-evident audit trail
- ✅ Device fingerprinting = catch account hijacking
- ✅ Line-by-line diffs = prove who changed what
- ✅ Unanimous consent = prevent partial signatures

### What We Avoid:
- ❌ 5-level roles = no privilege escalation bugs
- ❌ TOTP per signature = no user fatigue
- ❌ Unnecessary complexity = smaller attack surface

---

## 🎯 Development Timeline

| Phase | Duration | Focus |
|-------|----------|-------|
| Month 1-2 | 2 months | Auth + MFA + Projects |
| Month 3-4 | 2 months | Documents + Signatures |
| Month 5-6 | 2 months | Blockchain + Diffs |
| Month 7-8 | 2 months | Security + Compliance + Testing |

**Total:** 8 months to MVP

---

## ❓ FAQ

**Q: Why blockchain for audit logs?**  
A: Court-admissible evidence. Any tampering breaks the chain.

**Q: Why keep soft-deletion?**  
A: Non-EU customers need audit trail retention. EU users get hard-deletion.

**Q: Why line-by-line diffs?**  
A: Proves exactly what changed and who changed it (legal evidence).

**Q: Why only 3 roles?**  
A: Simpler = more secure. Admin/Member/Guest covers 99% of use cases.

**Q: Why device fingerprinting?**  
A: Catches stolen sessions. Flags logins from new devices automatically.

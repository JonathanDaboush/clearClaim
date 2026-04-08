# Start Here

## The Core Idea

**ClearClaim = unanimous agreement platform**

Nothing gets approved unless everyone required agrees.

That single focus drives every decision.

---

## 🧭 Product Philosophy

### The Rule

Before building anything, ask:

> **"Does this help someone confidently sign or reject an agreement?"**

If no → delay it.

---

## 🎯 What We're Building (MVP)

### Core Workflow
1. User creates project
2. Uploads PDF document
3. Assigns participants with roles
4. Participants review & sign
5. **System enforces unanimous agreement**
6. Document marked complete when all sign

### That's It
No blockchain. No device fingerprinting. No overengineering.

Just: **upload → assign → sign → unanimous → done**

---

## ✅ What's In MVP

| Feature | Why? |
|---------|------|
| User accounts | Can't sign if you're not a user |
| 2FA (TOTP) | Industry standard security |
| Projects | Organize documents |
| Groups | Organize users (simple collections) |
| PDF upload | The documents being signed |
| Participant roles | Admin/Member/Viewer |
| Signing | The core action |
| Unanimous enforcement | **The differentiator** |
| Basic audit log | Who did what, when |
| Simple line comparison | Track changes (basic text diff) |

---

## ❌ What's NOT in MVP

| Feature | Why Cut? |
|---------|----------|
| Blockchain audit | Overengineered for MVP |
| Device fingerprinting | Adds complexity, not value |
| Complex deletion logic | GDPR hybrid is overkill for MVP |
| Deep permission trees | You're not building AWS IAM |
| Perfect visual diff | "Useful" beats "technically perfect" |
| Subgroups | Flat structure is simpler |

👉 **All of these can come later.** None help you get users.

---

## ⚠️ What We Simplify

### Groups
**Don't build:**
- Deep permission trees
- Inheritance logic
- Complex role hierarchies

**Do build:**
- Group = collection of users
- Roles applied at **document level**
- Simple: Admin/Member/Viewer

### Line Comparison
**Don't build:**
- Perfect PDF visual mapping
- Complex diff algorithms
- 100% accuracy

**Do build:**
- Extract text from PDF
- Basic text diff (even rough is useful)
- Show what changed

👉 Get 80% of the value with 20% of the work.

---

## 🔐 Security (Realistic)

**Do:**
- Password hashing (bcrypt or argon2)
- JWT or session auth
- Input validation everywhere
- File upload restrictions (type + size)
- SQLAlchemy (prevents SQL injection)
- HTTPS only

**Don't:**
- Assume frameworks "handle everything"
- Overclaim security coverage
- Build "bank-grade" security

👉 You're building **"secure enough"** not **"Fort Knox"**

---

## 📊 Audit Log (Keep It Simple)

Just store:
- `user_id` - Who did it
- `action` - What they did
- `document_id` - What document
- `timestamp` - When
- `ip_address` - Where from

That's ALL you need.

No blockchain. No immutability proofs. Just a simple database table.

---

## ⚖️ Legal Positioning

### What You Say
"Designed with compliance in mind"

### What You DON'T Say
- "Fully GDPR compliant"
- "HIPAA certified"
- "Enterprise-grade compliance"

👉 Claiming full compliance requires audits. Don't go there yet.

---

## ⏱️ Timeline

**If you follow this plan:**
- 6-10 weeks → working product
- Users can sign up, upload, assign, sign
- Unanimous agreement works

**If you drift into overbuilding:**
- 6-12 months → unfinished system
- No users because product doesn't exist

---

## 🎯 Success Metric

**MVP is successful when:**

A user can:
1. Sign up
2. Upload a PDF
3. Assign 3 people to sign
4. Track who hasn't signed yet
5. System marks complete when all 3 sign

**That's the test.** Everything else is noise.

---

## 📂 Next Steps

1. **[MVP.md](MVP.md)** - Detailed feature breakdown
2. **[SECURITY.md](SECURITY.md)** - Security implementation
3. **[API.md](API.md)** - Technical specification

---

## 🧠 Remember

**The edge:** Unanimous agreement enforcement

**The timeline:** 6-10 weeks

**The test:** Does this help someone sign or reject?

**The trap:** Overengineering

Stay focused. Ship fast. Iterate based on real users.

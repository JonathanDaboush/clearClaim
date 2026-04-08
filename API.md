# Technical Implementation (API)

**Database schema, API endpoints, and implementation guide**

---

## 🗄️ Database Schema (MVP)

### Users
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    totp_secret VARCHAR(32),          -- For 2FA
    email_verified BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT NOW(),
    last_login TIMESTAMP
);
```

---

### Projects
```sql
CREATE TABLE projects (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    owner_id INTEGER REFERENCES users(id),
    created_at TIMESTAMP DEFAULT NOW()
);
```

---

### Groups
```sql
CREATE TABLE groups (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    project_id INTEGER REFERENCES projects(id),
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE group_members (
    group_id INTEGER REFERENCES groups(id),
    user_id INTEGER REFERENCES users(id),
    added_at TIMESTAMP DEFAULT NOW(),
    PRIMARY KEY (group_id, user_id)
);
```

**Note:** Roles applied at document level, not group level.

---

### Documents
```sql
CREATE TABLE documents (
    id SERIAL PRIMARY KEY,
    project_id INTEGER REFERENCES projects(id),
    filename VARCHAR(255) NOT NULL,
    file_path VARCHAR(500) NOT NULL,    -- Actual file location
    file_hash VARCHAR(64) NOT NULL,     -- SHA-256
    size_bytes INTEGER NOT NULL,
    extracted_text TEXT,                 -- For diffs
    status VARCHAR(20) DEFAULT 'draft',  -- draft|ready|signing|complete
    uploaded_by INTEGER REFERENCES users(id),
    uploaded_at TIMESTAMP DEFAULT NOW()
);
```

---

### Document Participants
```sql
CREATE TABLE document_participants (
    id SERIAL PRIMARY KEY,
    document_id INTEGER REFERENCES documents(id),
    user_id INTEGER REFERENCES users(id),
    role VARCHAR(20) NOT NULL,           -- admin|signer|viewer
    assigned_at TIMESTAMP DEFAULT NOW(),
    UNIQUE(document_id, user_id)
);
```

---

### Signatures
```sql
CREATE TABLE signatures (
    id SERIAL PRIMARY KEY,
    document_id INTEGER REFERENCES documents(id),
    user_id INTEGER REFERENCES users(id),
    signed_at TIMESTAMP DEFAULT NOW(),
    ip_address VARCHAR(45),              -- IPv4 or IPv6
    UNIQUE(document_id, user_id)
);
```

---

### Document Versions (for diffs)
```sql
CREATE TABLE document_versions (
    id SERIAL PRIMARY KEY,
    document_id INTEGER REFERENCES documents(id),
    version_number INTEGER NOT NULL,
    file_path VARCHAR(500) NOT NULL,
    file_hash VARCHAR(64) NOT NULL,
    extracted_text TEXT,
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE document_diffs (
    id SERIAL PRIMARY KEY,
    document_id INTEGER REFERENCES documents(id),
    from_version INTEGER,
    to_version INTEGER,
    changes_summary TEXT,                -- JSON with added/removed/changed
    created_at TIMESTAMP DEFAULT NOW()
);
```

---

### Audit Log
```sql
CREATE TABLE audit_log (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    action VARCHAR(50) NOT NULL,         -- created|updated|deleted|signed|etc
    resource_type VARCHAR(50),           -- project|document|user|etc
    resource_id INTEGER,
    ip_address VARCHAR(45),
    timestamp TIMESTAMP DEFAULT NOW()
);
```

---

## 🌐 API Endpoints (MVP)

### Authentication

```
POST   /api/auth/register
POST   /api/auth/login
POST   /api/auth/logout
POST   /api/auth/reset-password
GET    /api/auth/verify-email?token=...

POST   /api/auth/totp/enroll           -- Setup 2FA
POST   /api/auth/totp/verify           -- Verify 2FA code
```

---

### Projects

```
GET    /api/projects                   -- List user's projects
POST   /api/projects                   -- Create project
GET    /api/projects/:id               -- Get project details
PUT    /api/projects/:id               -- Update project
DELETE /api/projects/:id               -- Delete project
```

---

### Groups

```
GET    /api/projects/:id/groups        -- List project groups
POST   /api/projects/:id/groups        -- Create group
DELETE /api/groups/:id                 -- Delete group

POST   /api/groups/:id/members         -- Add member
DELETE /api/groups/:id/members/:userId -- Remove member
```

---

### Documents

```
GET    /api/projects/:id/documents     -- List documents
POST   /api/projects/:id/documents     -- Upload document
GET    /api/documents/:id              -- Get document metadata
PUT    /api/documents/:id              -- Update document
DELETE /api/documents/:id              -- Delete document

GET    /api/documents/:id/download     -- Download PDF
GET    /api/documents/:id/versions     -- List versions
GET    /api/documents/:id/diff/:v1/:v2 -- Get diff between versions
```

---

### Participants

```
GET    /api/documents/:id/participants     -- List participants
POST   /api/documents/:id/participants     -- Assign participant
PUT    /api/documents/:id/participants/:uid -- Update role
DELETE /api/documents/:id/participants/:uid -- Remove participant
```

---

### Signing

```
POST   /api/documents/:id/sign             -- Sign document
GET    /api/documents/:id/signatures       -- List signatures
GET    /api/documents/:id/status           -- Check unanimous status
GET    /api/documents/:id/certificate      -- Download completion cert (if complete)
```

---

### Audit

```
GET    /api/audit                          -- Get audit log (admin only)
GET    /api/audit?resource_type=document&resource_id=123
```

---

## 📝 Example Request/Response

### Sign a Document

**Request:**
```http
POST /api/documents/123/sign
Authorization: Bearer <jwt_token>
Content-Type: application/json

{
  "password_confirmation": "user_password"  // If session > 15 min
}
```

**Response (Success):**
```json
{
  "success": true,
  "signature": {
    "id": 456,
    "document_id": 123,
    "user_id": 789,
    "signed_at": "2026-04-07T15:30:00Z",
    "ip_address": "192.168.1.1"
  },
  "unanimous_status": {
    "required_signers": 5,
    "completed_signers": 3,
    "is_complete": false,
    "pending_users": [
      {"id": 101, "email": "user1@example.com"},
      {"id": 102, "email": "user2@example.com"}
    ]
  }
}
```

**Response (Document Complete):**
```json
{
  "success": true,
  "signature": { /* ... */ },
  "unanimous_status": {
    "required_signers": 5,
    "completed_signers": 5,
    "is_complete": true,
    "completed_at": "2026-04-07T15:30:00Z"
  },
  "certificate_url": "/api/documents/123/certificate"
}
```

---

## 🔧 Implementation Notes

### File Upload Handler

```python
from flask import Flask, request
from werkzeug.utils import secure_filename
import hashlib
import uuid
import os

@app.route('/api/projects/<int:project_id>/documents', methods=['POST'])
@login_required
def upload_document(project_id):
    # Validate file
    if 'file' not in request.files:
        return {'error': 'No file provided'}, 400
    
    file = request.files['file']
    
    # Check extension
    if not file.filename.endswith('.pdf'):
        return {'error': 'Only PDF files allowed'}, 400
    
    # Check size
    file.seek(0, os.SEEK_END)
    size = file.tell()
    file.seek(0)
    
    if size > 10 * 1024 * 1024:  # 10 MB
        return {'error': 'File too large'}, 400
    
    # Generate unique filename
    unique_name = f"{uuid.uuid4()}.pdf"
    file_path = os.path.join(UPLOAD_DIR, unique_name)
    
    # Save file
    file.save(file_path)
    
    # Calculate hash
    with open(file_path, 'rb') as f:
        file_hash = hashlib.sha256(f.read()).hexdigest()
    
    # Extract text
    from PyPDF2 import PdfReader
    reader = PdfReader(file_path)
    text = '\\n'.join([page.extract_text() for page in reader.pages])
    
    # Save to database
    doc = Document(
        project_id=project_id,
        filename=secure_filename(file.filename),
        file_path=file_path,
        file_hash=file_hash,
        size_bytes=size,
        extracted_text=text,
        uploaded_by=current_user.id
    )
    db.session.add(doc)
    db.session.commit()
    
    return {'success': True, 'document': doc.to_dict()}, 201
```

---

### Unanimous Agreement Check

```python
def check_unanimous_agreement(document_id):
    # Get all required signers
    required = DocumentParticipant.query.filter_by(
        document_id=document_id,
        role='signer'
    ).count()
    
    # Get signatures
    signed = Signature.query.filter_by(
        document_id=document_id
    ).count()
    
    return {
        'required': required,
        'signed': signed,
        'is_complete': required == signed and required > 0
    }
```

---

### Simple Diff Implementation

```python
import difflib

def calculate_diff(document_id, version1, version2):
    v1 = DocumentVersion.query.filter_by(
        document_id=document_id,
        version_number=version1
    ).first()
    
    v2 = DocumentVersion.query.filter_by(
        document_id=document_id,
        version_number=version2
    ).first()
    
    if not v1 or not v2:
        return None
    
    # Basic text diff
    diff = list(difflib.unified_diff(
        v1.extracted_text.splitlines(),
        v2.extracted_text.splitlines(),
        lineterm=''
    ))
    
    # Count changes
    added = sum(1 for line in diff if line.startswith('+') and not line.startswith('+++'))
    removed = sum(1 for line in diff if line.startswith('-') and not line.startswith('---'))
    
    return {
        'from_version': version1,
        'to_version': version2,
        'lines_added': added,
        'lines_removed': removed,
        'diff': diff  # Full diff output
    }
```

---

## 🔐 Authentication Middleware

```python
from functools import wraps
from flask import request, jsonify

def login_required(f):
    @wraps(f)
    def decorated_function(*args, **kwargs):
        # Check session or JWT
        user_id = session.get('user_id')
        if not user_id:
            return jsonify({'error': 'Authentication required'}), 401
        
        # Load user
        user = User.query.get(user_id)
        if not user:
            return jsonify({'error': 'Invalid session'}), 401
        
        # Set current_user for route
        g.current_user = user
        
        return f(*args, **kwargs)
    return decorated_function
```

---

## 📊 Performance Considerations

### Indexes to Add

```sql
CREATE INDEX idx_documents_project ON documents(project_id);
CREATE INDEX idx_documents_status ON documents(status);
CREATE INDEX idx_participants_document ON document_participants(document_id);
CREATE INDEX idx_participants_user ON document_participants(user_id);
CREATE INDEX idx_signatures_document ON signatures(document_id);
CREATE INDEX idx_audit_user ON audit_log(user_id);
CREATE INDEX idx_audit_timestamp ON audit_log(timestamp);
```

---

## 🧪 Testing

### Unit Tests

```python
def test_unanimous_agreement():
    # Create document
    doc = create_test_document()
    
    # Assign 3 signers
    assign_signer(doc.id, user1.id)
    assign_signer(doc.id, user2.id)
    assign_signer(doc.id, user3.id)
    
    # Check status
    status = check_unanimous_agreement(doc.id)
    assert status['required'] == 3
    assert status['signed'] == 0
    assert status['is_complete'] == False
    
    # Sign
    sign_document(doc.id, user1.id)
    sign_document(doc.id, user2.id)
    
    status = check_unanimous_agreement(doc.id)
    assert status['signed'] == 2
    assert status['is_complete'] == False
    
    # Final signature
    sign_document(doc.id, user3.id)
    
    status = check_unanimous_agreement(doc.id)
    assert status['signed'] == 3
    assert status['is_complete'] == True
```

---

## 🚀 Deployment

### Environment Variables

```bash
# .env file
DATABASE_URL=postgresql://user:pass@localhost/clearclaim
SECRET_KEY=your-secret-key-here
UPLOAD_DIR=/var/app/uploads
MAX_FILE_SIZE=10485760
ALLOWED_EXTENSIONS=pdf

# Production only
FLASK_ENV=production
SESSION_COOKIE_SECURE=True
```

### Run

```bash
# Development
flask run

# Production (use gunicorn)
gunicorn -w 4 -b 0.0.0.0:8000 app:app
```

---

## 🧠 Remember

**This is MVP.** 

- Focus on core functionality
- Don't optimize prematurely
- Ship working product first
- Iterate based on real usage

**Get users → Get feedback → Improve**

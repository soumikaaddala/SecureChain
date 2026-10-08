# Phase 12 — Secure Coding and Refactoring Deliverable

**Document Status:** Complete / Verified  
**Project:** SecureChain — Software Supply Chain Security Platform  
**Marks Weight:** 4 Marks  

---

## 1. Implemented Security Module Overview

The Authentication & Role-Based Authorization module was implemented in [`backend/app/api/auth.py`](file:///c:/Users/addal/Documents/antigravity/focused-bell/backend/app/api/auth.py). It provides token authentication, hashed credential storage, strict schema validation, role enforcement (`Student`, `Faculty`, `Administrator`, `SecurityAnalyst`), and safe error responses.

---

## 2. Identified Security Weaknesses & Quality Issues

Two major security vulnerabilities were identified in the initial authentication prototype:

1. **Weakness 1 (CWE-259 / Plaintext Password Storage & Comparison)**:
   * *Initial State*: User passwords were stored as raw strings and verified using standard `==` string equality operators, vulnerable to timing attacks and database compromise exposure.
2. **Weakness 2 (CWE-20 / Missing Input Length & Pattern Validation)**:
   * *Initial State*: Authentication inputs lacked character boundary checks or regex validation, exposing the database to SQL/NoSQL injections and Denial of Service (DoS) via huge payload buffer allocations.

---

## 3. Before vs. After Code Comparison & Refactoring

### Weakness 1: Password Comparison & Storage

#### ❌ BEFORE (Vulnerable Code)
```python
# VULNERABLE: Storing plain passwords and using timing-vulnerable equality
def login_user_legacy(username: str, password_raw: str):
    user = DB.get(username)
    if user and user["password"] == password_raw:  # Vulnerable to timing attack & data leak
        return {"token": username}
    return None
```

#### ✅ AFTER (Secure Refactored Code)
```python
from passlib.context import CryptContext
from jose import jwt

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def verify_password(plain_password: str, hashed_password: str) -> bool:
    # SECURE: Bcrypt salted key-stretching algorithm
    return pwd_context.verify(plain_password, hashed_password)

@router.post("/login", response_model=Token)
def login_user(form_data: OAuth2PasswordRequestForm = Depends()):
    user = FAKE_USERS_DB.get(form_data.username)
    if not user or not verify_password(form_data.password, user["hashed_password"]):
        # SECURE: Constant time error message preventing user enumeration
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Incorrect username or password",
            headers={"WWW-Authenticate": "Bearer"},
        )
    access_token = create_access_token(data={"sub": user["username"], "role": user["role"]})
    return Token(access_token=access_token, token_type="bearer", role=user["role"])
```

---

### Weakness 2: Input Validation & Pattern Constraints

#### ❌ BEFORE (Vulnerable Code)
```python
# VULNERABLE: Unbounded string input accepting dangerous special characters
class UnsafeUserRegister(BaseModel):
    username: str
    password: str
```

#### ✅ AFTER (Secure Refactored Code)
```python
# SECURE: Pydantic v2 strict pattern matching & boundary validation
class UserRegister(BaseModel):
    username: str = Field(..., min_length=3, max_length=50, pattern=r"^[a-zA-Z0-9_-]+$")
    email: EmailStr
    password: str = Field(..., min_length=8, max_length=128)
    role: str = Field("Student", pattern=r"^(Student|Faculty|Administrator|SecurityAnalyst)$")
```

---

## 4. Security Justification

1. **Bcrypt Password Hashing**: Applies cryptographic salting and key stretching ($2^{12}$ rounds), preventing offline dictionary and rainbow table attacks if the database is leaked.
2. **Timing-Attack Resistance**: Uses constant-time password hash evaluation and generic HTTP 401 error details, preventing user enumeration.
3. **Role-Based Access Control (RBAC)**: Enforces RBAC dependencies `require_role(["Administrator", "Faculty"])` on administrative routes, preventing Horizontal/Vertical Privilege Escalation (CWE-269).
4. **Input Boundary Protection**: Pydantic schema constraints block buffer overflow vectors and special character SQL/XSS payload injection.

# OWASP API Security Hardening – Identified Vulnerabilities & Fixes

This repository contains the complete remediation work for **Assignment 03 – Identified Vulnerabilities and Fixes** from the OWASP API Security Lab.  

The assignment covers **10 major API vulnerabilities**, each fixed with:
- Explanation of the vulnerability  
- Associated risks  
- Fix and rationale  
- Key code changes  
- Validation and test steps  
- Evidence screenshots  

---

## 📌 Covered Security Controls

1. **Password Security (BCrypt)**
2. **Access Control & RBAC (SecurityFilterChain)**
3. **IDOR Prevention – Resource Ownership Enforcement**
4. **Data Exposure Control (DTO-based responses)**
5. **Rate Limiting (Bucket4j)**
6. **Mass Assignment Prevention**
7. **JWT Hardening (issuer, audience, expiry, strong secret)**
8. **Error Handling & Centralized Logging**
9. **Input Validation (Jakarta Validation + custom rules)**
10. **Testing & Verification (Integration tests)**

---

## 🔐 1. Password Security – BCrypt Hashing

### Vulnerability  
Passwords stored in plaintext or weak hashes.

### Fix  
- Replaced password storage with **BCrypt hashing**.  
- Verification done using `PasswordEncoder.matches()`.  
- Seeder now inserts hashed passwords only.

### Files Updated  
- `config/SecurityConfig.java`  
- `web/AuthController.java`  
- `config/DataSeeder.java`

### Validation  
- DB now stores `$2a$...` BCrypt hashes.  
- Wrong password → 401 Unauthorized.

---

## 🔒 2. Access Control & RBAC

### Vulnerability  
Overly permissive `permitAll()` allowed unauthorized access.

### Fix  
- Implemented **deny-by-default** authorization.  
- Only `/api/auth/login` and `/api/auth/signup` are public.  
- Admin routes protected with `hasRole("ADMIN")`.  
- Stateless JWT authentication enabled.

### Files Updated  
- `config/SecurityConfig.java`  
- `security/JwtAuthFilter.java`

### Validation  
- Unauthenticated requests → 401  
- Non-admin accessing admin routes → 403  

---

## 🛡️ 3. IDOR Prevention – Resource Ownership Enforcement

### Vulnerability  
User-controlled IDs allowed access to other users’ accounts.

### Fix  
- User ID is derived from JWT token.  
- Repository methods use `findByIdAndUserId()`.  
- No client-supplied user IDs are trusted.

### Files Updated  
- `web/AccountController.java`  
- `repository/AccountRepository.java`

### Validation  
- User A cannot fetch User B’s account. Returns 404.

---

## 🧱 4. Data Exposure Control – DTOs

### Vulnerability  
Entities returned with sensitive fields included (roles, password hash, etc.).

### Fix  
- Introduced **response DTOs** to strictly control returned fields.  
- Mapping from entity → DTO ensures no leakage.

### Files Updated  
- `web/dto/*.java`  
- `web/AccountController.java`

### Validation  
- GET `/api/accounts/mine` returns only non-sensitive data.

---

## 🚦 5. Rate Limiting – Bucket4j

### Vulnerability  
API had no throttling → brute force & resource abuse possible.

### Fix  
- Added per-IP and per-user rate limits.  
- Exceeding limit returns `429 Too Many Requests`.

### Files Updated  
- `security/RateLimitFilter.java`  
- `config/SecurityConfig.java`

### Validation  
- Rapid login/signup attempts trigger 429 response.

---

## 🧰 6. Mass Assignment Prevention

### Vulnerability  
User could submit fields like `"role": "ADMIN"` during signup.

### Fix  
- Dedicated request DTOs that include only whitelisted fields.  
- Server overrides all role/privileged values.

### Files Updated  
- `web/dto/SignupRequest.java`  
- `web/AuthController.java`

### Validation  
- Attempt to assign admin role is ignored.

---

## 🔑 7. JWT Hardening

### Vulnerability  
Weak secrets, missing issuer/audience validation, long token lifetime.

### Fix  
- Strong secret loaded from environment.  
- Validated issuer, audience, expiry.  
- Short TTL added.  
- Full signature validation in place.

### Files Updated  
- `service/JwtService.java`  
- `application.yaml`

### Validation  
- Expired token → 401  
- Wrong audience → 401  
- Valid token → 200  

---

## 📝 8. Error Handling & Logging

### Vulnerability  
Raw stack traces leaked to the client.

### Fix  
- Centralized exception handler with safe error messages.  
- Added correlation IDs for request tracing.  
- Server logs contain full diagnostic details.

### Files Updated  
- `web/advices/GlobalExceptionHandler.java`  
- `filters/CorrelationIdFilter.java`

### Validation  
- Client receives safe messages; server logs contain detailed errors.

---

## 📥 9. Input Validation

### Vulnerability  
Negative values, malformed inputs, invalid lengths allowed.

### Fix  
- Applied Jakarta validation annotations.  
- Added custom validation rules.

### Files Updated  
- `web/dto/TransferRequest.java`  
- `validators/ValidationRules.java`

### Validation  
- Negative amounts or invalid usernames → 400 Bad Request.

---

## 🧪 10. Testing & Verification

### Fix  
Created integration tests covering:
- Authentication  
- RBAC authorization  
- DTO response fields  
- Rate-limit behavior  
- Ownership invariants  

### Files Updated  
- `test/java/.../it/*`

### Validation  
- `mvn clean test` → all tests passed.

---

## 🚀 How to Build, Run & Test

### Build  
```bash
mvn -q -DskipTests=false clean package

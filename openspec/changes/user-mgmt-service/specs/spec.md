# User Management Service – Technical Specification

## Purpose

This specification defines requirements for the User Management Service of the Medication Management System. The service SHALL provide secure user identity and access management, profile management, and related user functions accessible via REST APIs, supporting integration with other services and client applications.

## Technologies and Runtime Stack

- Language: Java (Spring Boot)
- Database: PostgreSQL (encrypted at rest)
- Cache: Redis (session data)
- Authentication: OAuth 2.0 (PKCE/JWT, OpenID Connect)
- Containerized deployment (Kubernetes, Docker)
- Metrics/Monitoring: Prometheus/Grafana
- Logging/Audit: ELK Stack

---

### Requirement 1: User Identity & Authentication

#### Scenario 1.1: User Login

**Given** a registered user,  
**When** a POST request is made to `/auth/login` with valid credentials,  
**Then** the service SHALL verify credentials using OAuth2, issue a JWT access token (short-lived), and a refresh token (longer-lived, revocable), and log the event.

#### Scenario 1.2: Token Refresh

**Given** a valid refresh token,  
**When** a POST is sent to `/auth/refresh`,  
**Then** the service SHALL validate the token, issue a new access token, and record the refresh event in the audit log.

#### Scenario 1.3: MFA Enrollment

**Given** a logged-in user,  
**When** MFA setup is requested at `/auth/mfa/enroll`,  
**Then** the service SHALL initiate and validate MFA (e.g., TOTP, as configured), updating user profile and audit log.

#### Scenario 1.4: Logout/Session Invalidation

**Given** an authenticated session,  
**When** a POST is sent to `/auth/logout`,  
**Then** the session SHALL be invalidated, token(s) revoked, and the event recorded.

---

### Requirement 2: User Profile Management

#### Scenario 2.1: Register New User

**Given** valid user registration details,  
**When** a POST is made to `/users` (protected endpoint),  
**Then** the service SHALL create the user in PostgreSQL (ensuring uniqueness of email/ID), hash and store credentials, and return the user object excluding sensitive fields.

#### Scenario 2.2: Retrieve/Update User Profile

- **GET /users/{user_id}:** Returns full user profile (excluding private security attributes).
- **PUT /users/{user_id}:** Updates profile with allowed fields (e.g., name, language, accessibility).
- **DELETE /users/{user_id}:** Soft-deletes or anonymizes user, per configured policy.

All profile operations SHALL:
- Enforce RBAC (user can only manage own data unless admin role).
- Log all changes with before/after snapshots and user context.
- Only expose notifiable fields in API responses.

---

### Requirement 3: Role-Based Access Control

#### Scenario 3.1: Access Control Enforcement

- Each API SHALL require explicit scopes/permissions, enforced per endpoint (e.g., only admins can list all users).
- Scopes managed in user profile (PostgreSQL), validated on each request.
- Unauthorized activities SHALL return 403 with audit log entry.

---

### Requirement 4: Accessibility & Preferences

#### Scenario 4.1: Configure Preferences

- User can PATCH `/users/{user_id}/preferences` to update language, notification channel, or emergency contacts.
- Preferences saved in encrypted PostgreSQL fields, retrievable in profile.

---

### Requirement 5: Session and Token Lifecycle

- All sessions SHALL be tracked (session ID, creation, expiry, user agent, IP).
- Forced logout endpoints available for users and admins.
- Inactive sessions auto-expire after configurable TTL.

---

### Requirement 6: Auditing and Compliance

- All authentication events, profile changes, and access attempts SHALL be logged to ELK (tamper-evident).
- Audit logs SHALL record: timestamp, user, action, client/app, IP address.
- Sensitive fields masked in logs.

---

### APIs (REST Endpoints)

- **POST /auth/login** — User login, returns tokens
- **POST /auth/refresh** — Refresh JWT token
- **POST /auth/mfa/enroll** — Enroll/validate MFA option
- **POST /auth/logout** — User or session logout
- **POST /users** — Register new user
- **GET /users/{user_id}** — View user profile
- **PUT /users/{user_id}** — Update user profile
- **PATCH /users/{user_id}/preferences** — Update accessibility/comms preferences
- **DELETE /users/{user_id}** — Deactivate/anonymize user
- **GET /users/me/sessions** — List active sessions for user
- **DELETE /users/me/sessions/{session_id}** — Invalidate session (admin/self)

---

### Data Models

**User**
- user_id (UUID, PK, immutable)
- email (unique, required)
- password_hash (write-only, never returned by API)
- display_name (string)
- roles (array: ['user', 'admin', ...])
- language (string, ISO code)
- accessibility_settings (json: font_size, color_scheme, etc)
- notification_preferences (json)
- emergency_contacts (json, encrypted)
- mfa_enabled (bool)
- created_at, updated_at (timestamps)

_Enforce all invariants (e.g., unique email/user_id, minimum password strength) in API validation._

---

### Runtime & Dependency Interactions

- Connects to PostgreSQL for user CRUD.
- Stores session data (tokens, state) in Redis.
- Validates upstream OAuth tokens with provider (as needed).
- Provides user info/permissions to internal services via secure REST.
- Monitors and logs metrics/events for audit and compliance.
- Handles errors (e.g., authentication failure, constraint violation) with standardized error responses (RFC 7807), appropriate status codes, and audit logging.
- All sensitive operations are rate-limited (API Gateway/Redis).

---

### Key Flows

**Example: User Login Flow (5-10 steps)**
1. User submits credentials to `/auth/login` via API Gateway.
2. API Gateway forwards request to User Management Service.
3. Service validates credentials against PostgreSQL (hashed+salted).
4. If valid, generates JWT token with appropriate claims/scopes.
5. Records login event in ELK Stack (audit).
6. Caches session data in Redis.
7. Returns JWT token + refresh token to client.
8. Client uses JWT for subsequent authenticated requests.
9. Regular token validation/refresh cycle managed via `/auth/refresh`.
10. All failed attempts logged with reason (e.g., bad password, locked account).

---

## TODOs

- [ ] Detail exact OpenAPI (request/response shapes).
- [ ] Confirm all RBAC roles/scopes and error message formats with security/UX team.
- [ ] Define configuration for maximum concurrent sessions and session TTL.

---
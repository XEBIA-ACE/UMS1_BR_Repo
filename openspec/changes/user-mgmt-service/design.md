# User Management Service – Design Document

## Technical Approach & Architecture Decisions

### Overview

The User Management Service is realized as a stateless Spring Boot REST service, containerized for Kubernetes deployment, and integrated with PostgreSQL (user data), Redis (session caching), and external OAuth2 identity providers. It exposes well-defined REST endpoints via the API Gateway (Kong), enforces RBAC for all operations, and logs all security-relevant events for compliance.

### Data Flow

1. **Authentication:** Users authenticate via `/auth/*` endpoints; credentials are verified and tokens (JWT, refresh) are issued. Session metadata is stored in Redis.
2. **Profile CRUD:** `/users/*` APIs perform create/read/update/delete directly on PostgreSQL, enforcing unique constraints and field-level encryption for sensitive attributes.
3. **Preferences/Accessibility:** PATCH requests allow update of profile settings, persisted with fine-grained access controls.
4. **Role/Permission Checks:** Every endpoint includes role verification before processing.
5. **Session Lifecycle:** Session tokens are stored in Redis (with TTL), enabling forced logout, concurrent session control, and session activity tracking.
6. **Audit Logging:** All API calls and critical operations are logged to ELK with full context, masking sensitive fields.

### Main Component/Module Responsibilities

- **REST Controllers:** Map HTTP requests to domain service methods, validate inputs, structure API responses, handle errors.
- **Domain Services:** Contain core logic for authentication, session, profile, and preference management.
- **Repository Layer:** Handles all SQL/Redis access for user, session, and preferences data, using JPA/Hibernate and Redis Java client.
- **Integration Clients:** Manage communication with external OAuth2/OIDC providers.
- **Audit & Metrics:** Service pushes structured logs and metrics to the monitoring/ELK stack.

### Security Considerations

- Encryption for all at-rest (PostgreSQL, Redis) and in-flight data.
- Short-lived JWT with revocable refresh tokens.
- All external and inter-service communication over TLS 1.3.
- Strict field-level validation and sanitization.
- Rate limiting using API Gateway and Redis.
- Default audit log masking of any PII except for authorized audit staff.

### API & File Changes

- API endpoints as per spec (`/auth/*`, `/users/*`)
- Entities: User, UserPreferences, Session, Role
- Repository and service-level interfaces
- Logging/audit integration

---

## TODOs

- Finalize OpenAPI schema and error responses.
- Gate access to sensitive endpoints (list all users, RBAC changes) with admin-only scopes.
- Confirm monitoring hooks in all authentication flows.

---
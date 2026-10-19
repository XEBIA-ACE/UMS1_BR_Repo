# Implementation Plan: US-001

## Technical Context
- Codebase grounding snapshot (86af296112dd23f3)
- Accepted impact analysis (sha256:2e121b348ebd00552c148e09997e559c5a61ef936fe9c3cc96a401e1300b8e1c)
- unresolved reference (provider-object:1036)
- unresolved reference (provider-object:1484)
- unresolved reference (provider-object:986)
- unresolved reference (provider-object:2428)
- unresolved reference (component:login)
- unresolved reference (component:LoginRequestDto)
- unresolved reference (component:Promise)
- unresolved reference (component:createAuthRouter)
- unresolved reference (component:§{main_sources}§/New_Prompt_UMS-develop/BACKEND/src/controllers/auth.controller.ts)
- unresolved reference (api/v1/auth/login/)
- unresolved reference (LoginErrorResponse)
- unresolved reference (LoginSuccessResponse)
- unresolved reference (LoginGuard)
- unresolved reference (DefaultLoginGuard)
- Implement only against Accepted impact analysis (revision 1) (fingerprint sha256:2e121b348ebd00552c148e09997e559c5a61ef936fe9c3cc96a401e1300b8e1c)

## Grounded Components
- LoginRequestDto
- Promise
- createAuthRouter
- login
- §{main_sources}§/New_Prompt_UMS-develop/BACKEND/src/controllers/auth.controller.ts

## Design Steps
1. Step 1: Extend provider-confirmed interface LoginRequestDto to include optional MFA verification code field and MFA session token field for two-step authentication flow
2. Step 2: Modify provider-confirmed method login in auth.controller.ts to implement two-phase authentication: Phase 1 validates credentials and initiates MFA code generation/delivery; Phase 2 validates MFA code and grants access
3. Step 3: Create MFA service module to handle code generation, delivery channel selection (email/SMS), code validation, and expiration management - integrating with login method
4. Step 4: Extend provider-confirmed function createAuthRouter to add new endpoint POST api/v1/auth/mfa/verify for MFA code verification step, keeping existing login endpoint for credential validation
5. Step 5: Implement MFA session state management using provider-confirmed Promise patterns for async MFA code delivery and verification with timeout handling for 2-second performance constraint
6. Step 6: Add MFA-specific error responses extending existing LoginErrorResponse transaction pattern for failed MFA scenarios with screen-reader compatible error messages
7. Step 7: Implement rate limiting on MFA verification attempts to prevent brute force attacks on the login and MFA verification endpoints
8. Step 8: Add audit logging for all MFA-related events (initiation, success, failure) within the auth.controller.ts authentication flow

## Implementation Readiness Items (Must Resolve or Accept)
- Candidate verification could not confidently judge the selected focus object for this requirement
- Candidate verification was uncertain about the selected focus object: The requirement is about implementing multifactor authentication for admin login. The selected object "login" (id=1036) is a Typescript Method that could plausibly be related to authentication, but without additional context about whether this specific login method handles admin authentication vs. other login methods (id=1190, id=1192), it's unclear if this is the correct target. Multiple "login" methods exist in the candidate list, and we cannot determine which one specifically handles admin dashboard authentication.
- Requirement traceability incomplete for US-001-ac-1, US-001-ac-2, US-001-ac-3, US-001-ac-4
- No existing MFA infrastructure exists based on structural analysis
- Email and SMS delivery services will need to be integrated or are available
- Admin users have email/phone registered for MFA delivery
- The 20 'database objects' returned appear to be UI components and DTOs rather than actual database tables
- No MFA-related components detected in current codebase structure
- No email/SMS service integrations visible in structural facts
- Intra-object structural facts unavailable - cannot assess internal login method complexity
- Transaction call graphs unavailable - cannot trace full authentication flow
- No performance baseline data to validate 2-second constraint feasibility
- What email/SMS service will be used for MFA delivery?
- Should MFA be enforced on every login or allow 'remember this device' functionality?
- What is the MFA code validity period?
- Is there a fallback mechanism if email/SMS delivery fails?
- How will admin users configure their MFA delivery preference?

## Contract Changes
- POST api/v1/auth/login/ — Must be modified to support MFA flow - either extended to handle MFA verification or split into credential validation and MFA verification steps
- LoginRequestDto interface may need extension to support MFA token/code fields — change required
- New data structures required for MFA session state, verification codes, and delivery method preferences — change required
- Potential new storage for MFA configuration per admin user (email/phone for delivery) — change required

## Security Obligations
- MFA codes must be transmitted securely via email/SMS
- MFA verification codes should have short expiration times
- Failed MFA attempts should be rate-limited to prevent brute force attacks
- MFA session state must be protected against session fixation attacks
- Audit logging required for all MFA attempts (success and failure)
- Validate credential and personal-data handling against the Jira requirement
- Validate credential and personal-data handling against the story's acceptance criteria

## Quality Attribute Obligations
- Verify the changed user flow against the stated accessibility criteria

## Accessibility Obligations
- MFA code input field must be keyboard accessible
- Error messages for MFA failure must be screen-reader compatible
- Alternative MFA delivery methods should be available for accessibility

## Verification Plan
- **US-001-ac-1**: Given the administrator enters valid credentials, when the multifactor authentication step is initiated, then a second factor authentication request is sent via email or text.
- **US-001-ac-2**: Given the administrator completes the multifactor authentication, when they attempt to log in, then access to the admin dashboard is granted.
- **US-001-ac-3**: When the multifactor authentication fails, then access is denied, and an error message is displayed.
- **US-001-ac-4**: Ensure that login attempts including multifactor authentication are processed within 2 seconds to meet performance constraints.
- **IMPACT**: Unit tests for MFA code generation and validation
- **IMPACT**: Integration tests for email/SMS delivery of MFA codes
- **IMPACT**: End-to-end tests for complete MFA login flow
- **IMPACT**: Performance tests to verify 2-second response time constraint
- **IMPACT**: Security tests for MFA bypass attempts
- **IMPACT**: Tests for MFA failure scenarios and error message display

## Evidence References
- 86af296112dd23f3
- sha256:2e121b348ebd00552c148e09997e559c5a61ef936fe9c3cc96a401e1300b8e1c
- provider-object:1036
- provider-object:1484
- provider-object:986
- provider-object:2428
- component:login
- component:LoginRequestDto
- component:Promise
- component:createAuthRouter
- component:§{main_sources}§/New_Prompt_UMS-develop/BACKEND/src/controllers/auth.controller.ts
- api/v1/auth/login/
- LoginErrorResponse
- LoginSuccessResponse
- LoginGuard
- DefaultLoginGuard
# Task 3 — Security Remediation Plan

## Implemented Security Fixes

| ID | Finding | Severity (provisional) | Remediation |
|---|---|---|---|
| SEC-001 | SQL injection in scan search | High | Replaced unsafe SQL construction with parameterized queries in `app/database.py`. |
| SEC-002 | Unauthorized cross-user scan search | High | Enforced scan ownership using the authenticated user's ID. |
| SEC-003 | Shared-link password exposure (Task 1) | High, deployment-dependent | Changed password transmission from URL query parameters to the `X-Share-Password` header. |
| SEC-004 | Unsafe JWT algorithm configuration | High, subject to verification | Removed the `none` algorithm from JWT verification in `app/auth.py`. |

Severity ratings and distinctness must be verified against Task 2 findings.

## Unresolved Security Findings

### 1. Hardcoded Credentials

**Affected file:** `app/config.py`

**Residual risk:** Exposed credentials may allow unauthorized database access or forged authentication tokens.

**Reason not fixed:** Requires secure secret injection, deployment configuration changes, and credential rotation.

**Estimated remediation effort:** 2–4 hours.

**Compensating controls:** No existing controls verified. Restrict access and rotate exposed credentials as soon as possible.

### 2. Server-Side Request Forgery (SSRF)

**Affected component:** Notification service.

**Residual risk:** An attacker controlling webhook destinations may cause requests to internal services or cloud metadata endpoints.

**Reason not fixed:** Requires URL validation, DNS/IP restrictions, redirect protection, and outbound network controls.

**Estimated remediation effort:** 4–8 hours.

**Compensating controls:** Network egress restrictions are recommended but not verified as implemented.

### 3. Missing Notification Service Authentication

**Affected component:** `notify/src/index.js`

**Residual risk:** Unauthorized users may access notification operations if the service is exposed.

**Reason not fixed:** Requires authentication middleware and authorization checks.

**Estimated remediation effort:** 2–4 hours.

**Compensating controls:** Private network access restrictions are recommended but not verified.

### 4. Permissive CORS Configuration

**Affected file:** `app/main.py`

**Residual risk:** Untrusted browser origins may be permitted to make credentialed cross-origin requests.

**Reason not fixed:** Requires a trusted-origin allowlist and deployment-specific frontend configuration.

**Estimated remediation effort:** 1–2 hours.

**Compensating controls:** No existing controls verified.

### 5. Missing Rate Limiting

**Affected components:** Authentication and shared-report endpoints.

**Residual risk:** Attackers may repeatedly guess login credentials or shared-link passwords.

**Reason not fixed:** Requires rate-limiting middleware or gateway controls, ideally with shared state across instances.

**Estimated remediation effort:** 3–5 hours.

**Compensating controls:** Long, randomly generated share tokens reduce token-guessing risk. Password-attempt rate limiting is not currently implemented.

## Verification

The following checks are required before submission:

- Confirm Python files compile successfully.
- Run the application test suite.
- Verify SQL injection attempts are treated as search input.
- Verify users cannot search another user's scan records.
- Verify invalid or unsigned JWTs are rejected.
- Verify share-link passwords are accepted through the HTTP header and rejected when supplied only through URL parameters.

## Conclusion

The selected remediations address database query safety, scan ownership, JWT verification configuration, and Task 1 shared-link password exposure.

Remaining findings require additional engineering work and security validation before production deployment.
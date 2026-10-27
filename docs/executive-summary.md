# Executive Security Summary

**Application:** VulnTracker  
**Assessment:** Security engineering take-home exercise  
**Overall assessment:** Improved controls; production readiness not yet demonstrated

## Business overview

Before this work, the application had weaknesses that could expose customer vulnerability reports, allow users to see information outside their accounts, and make it easier for attackers to abuse connected services. The work has focused on reducing those exposures and adding safer deployment defaults. These are meaningful improvements, but they do not yet establish that the service is safe to operate with sensitive production data. End-to-end testing and deployment verification remain necessary.

## Security posture: before and after

| Area | Before | After the proposed changes |
| --- | --- | --- |
| Report access | Search functionality could expose records outside the requesting user's account. | Search queries are designed to restrict results to the signed-in account and use safe query parameters. |
| Shared report links | Passwords could be supplied in URLs, where they may appear in logs or browser history. | The shared-report endpoint is designed to accept passwords in a request header instead. |
| Login tokens | Token verification permitted an unnecessarily broad set of algorithms. | Verification is restricted to the configured algorithm. |
| Deployment | Production security controls were not consistently defined. | A non-root container and a Kubernetes deployment configuration with resource and network restrictions have been proposed. |

**Verification status:** These changes reflect the remediation work discussed, not a confirmed production deployment. The most recent Docker run failed during application startup because of a Python import issue. The complete test suite, successful container startup, and Kubernetes deployment still need to be verified before the improvements can be treated as operational.

## Top three remaining business risks

**1. Secrets and account compromise.** Sensitive configuration may still rely on weak defaults or locally supplied values. If a signing key or service credential is exposed, an attacker could impersonate users or access protected data. **Why deferred:** A complete fix requires a production secrets manager, credential rotation, and verified application integration—not just a configuration-file change.

**2. Abuse of the notification service.** Notification endpoints and outbound requests may lack sufficient authentication and destination restrictions. An attacker could potentially trigger unwanted messages or cause the service to contact unintended internal systems. **Why deferred:** This needs coordinated changes across the separate notification service, its callers, and its network permissions, followed by abuse-case testing.

**3. Excessive access and service disruption.** Broad cross-origin access rules and missing request limits can increase the impact of account misuse and automated traffic. This could expose sensitive information in some deployment scenarios or interrupt service for legitimate users. **Why deferred:** Safe restrictions depend on confirmed client origins, expected traffic, and operational monitoring; applying guessed values could break legitimate workflows.

## Recommended next steps

1. **Prove the fixes work:** Resolve the current application startup failure, run the full automated test suite, and add tests for cross-account access, shared-link expiry, password handling, and unauthorized requests.
2. **Protect credentials and connected services:** Integrate a managed secrets store, rotate credentials, require authentication for notification operations, and restrict outbound destinations.
3. **Harden public access:** Allow only approved web origins, introduce request limits for login and shared-link access, and enforce encrypted connections in production.
4. **Validate deployment before launch:** Build and run the container successfully, review the Kubernetes network rules and access permissions, and confirm logging, alerting, backups, and recovery procedures.

## Decision requested

**Do not approve production use with sensitive customer data yet.** Approve a short, focused hardening and verification phase, with release conditional on passing security tests, successful container deployment, and closure or explicit acceptance of the remaining high-impact risks.

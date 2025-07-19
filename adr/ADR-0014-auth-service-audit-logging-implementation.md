# ADR-0014: Auth Service Audit Logging Implementation

## Status
Accepted

## Context
The jms-auth-service was not properly logging authentication events (login, logout, token refresh, etc.) to the audit system despite having the infrastructure in place. The service had:

1. Two competing audit implementations:
   - A custom IAuditService interface for permission/role changes
   - The shared JMS.Common.Audit infrastructure for comprehensive audit logging

2. Missing authentication event logging in the AuthenticationService class

3. Configuration issues:
   - Incorrect audit schema script path in Program.cs
   - AWS EventBridge publishing commented out due to circular dependency
   - No audit interceptors configured for automatic entity change tracking

4. Schema confusion between `jms.audit_logs` and `jms_audit.audit_logs` tables

## Decision
We have decided to:

1. **Keep both audit systems** but with clear separation of concerns:
   - Custom IAuditService for authorization-specific events (permission/role changes)
   - New IAuthenticationAuditService for authentication events (login/logout/refresh)
   - Both systems write to the shared `jms_audit.audit_logs` table via PostgresAuditRepository

2. **Implement comprehensive authentication event logging** by:
   - Creating IAuthenticationAuditService interface with methods for all auth events
   - Implementing AuthenticationAuditService using JMS.Common.Audit infrastructure
   - Injecting and using this service in AuthenticationService for all auth operations

3. **Fix configuration issues**:
   - Correct the audit schema script path from `01_CreateAuditSchema.sql` to `AddAuditSchema.sql`
   - Keep AWS EventBridge publishing disabled until circular dependency is resolved
   - Note: Audit interceptors are already configured via `modelBuilder.ApplyAuditConfigurations()`

4. **Standardize on the `jms_audit` schema** for all audit tables as defined in ADR-0006

## Consequences

### Positive
- Authentication events are now properly logged for security and compliance
- Clear separation between authentication and authorization audit concerns
- Leverages existing JMS.Common.Audit infrastructure for consistency
- Maintains backward compatibility with existing permission audit code
- Provides comprehensive audit trail for all auth-related operations

### Negative
- Two audit interfaces may cause initial confusion (mitigated by clear naming)
- AWS EventBridge publishing remains disabled (temporary limitation)
- Additional service registration required in DI container

### Neutral
- Audit data is stored locally until EventBridge integration is fixed
- Performance impact is minimal due to async audit operations

## Alternatives Considered

1. **Replace custom IAuditService entirely** - Rejected because it would require significant refactoring of existing permission audit code

2. **Extend existing IAuditService** - Rejected because it would mix authentication and authorization concerns in a single interface

3. **Create separate audit tables** - Rejected because it would complicate querying and violate the unified audit architecture from ADR-0006

## References
- ADR-0006: Audit Logging Architecture
- ADR-0013: Authorization Architecture
- JMS.Common.Audit package documentation
- Implementation status report in x-project-docs/audit-logging-implementation

## Notes
- The circular dependency with ResilientAuditEventPublisher needs to be resolved in JMS.Common.Audit.AWS package
- Consider adding IP address and user agent capture from HttpContext in a future enhancement
- The audit interceptor configuration could be enhanced to automatically track entity changes

---
**Date**: 2025-07-19  
**Author**: Claude  
**Reviewers**: Timothy Ross
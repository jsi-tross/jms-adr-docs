# ADR-0005: Court Scheduling Service Placement

## Status
Proposed

## Context
The JMS system requires a comprehensive court scheduling and calendar management system (Story JJS-007) that will:
- Manage scheduling for 60 court locations
- Handle calendar years, exceptions, and emergency overrides
- Process bulk operations
- Maintain audit trails
- Implement approval workflows

We need to decide whether this functionality should be added to an existing service or implemented as a new microservice.

Currently, we have these services:
- **jms-juror-service**: Manages juror records and summoning
- **jms-auth-service**: Handles authentication and authorization
- **jms-tenant-service**: Manages multi-tenancy and tenant configuration
- **jms-mock-services**: Provides test data

## Decision
Implement court scheduling and calendar management functionality within the existing **jms-juror-service**, recognizing that court scheduling is an integral part of jury administration rather than a separate domain.

## Consequences

### Positive
- **Domain Cohesion**: Keeps tightly coupled jury administration functionality together
- **Performance**: No network overhead for operations that need both scheduling and juror data
- **Transaction Simplicity**: Local transactions for complex workflows (summons, deferrals, FTAs)
- **Single Source of Truth**: One service owns the complete jury administration lifecycle
- **Reduced Operational Complexity**: No additional service to deploy, monitor, and maintain

### Negative
- **Service Size**: jms-juror-service will grow significantly with added functionality
- **Team Coordination**: Multiple features being developed in the same codebase
- **Deployment Risk**: Changes to any jury feature require deploying the entire service
- **Potential for Coupling**: Risk of tightly coupling scheduling with juror management code

### Neutral
- **Internal Organization**: Will require clear module boundaries within the service
- **Database Schema**: Can use separate schemas (jms.jurors, jms.court_scheduling) within same database
- **Future Refactoring**: Can extract to separate service later if genuine need arises

## Alternatives Considered

1. **Create new jms-court-service**
   - Rejected: Would create artificial service boundary through the middle of jury workflows
   - Would require constant cross-service communication for tightly coupled operations
   - Distributed transactions for operations like summons generation and deferrals

2. **Split into jms-court-service (orchestration) and jms-juror-service (data)**
   - Rejected: Creates anemic service anti-pattern with juror-service as a database wrapper
   - Performance overhead for every operation requiring both services
   - Complex distributed transaction coordination

3. **Create multiple fine-grained services** (scheduling-service, summons-service, deferral-service)
   - Rejected: Over-engineering for tightly coupled domain
   - Extreme coordination complexity for simple operations
   - Violates bounded context principles - these are all part of jury administration

## References
- [ADR-0004: Court Scheduling Calendar Management](./ADR-0004-court-scheduling-calendar-management.md)
- [Story JJS-007 Implementation Plans](../x-project-docs/court-scheduling-JJS-007/)
- [Cross-Team Coordination Patterns](../x-project-docs/cross-team-coordination-plan-v3.md)

## Notes
### Internal Organization Strategy
The jms-juror-service will be organized with clear module boundaries:

```
jms-juror-service/
├── src/
│   ├── JurorManagement/          # Existing juror records functionality
│   │   ├── Controllers/
│   │   ├── Services/
│   │   └── Models/
│   ├── CourtScheduling/          # New scheduling functionality
│   │   ├── Controllers/
│   │   ├── Services/
│   │   └── Models/
│   ├── Workflows/                # Future: summons, deferrals, FTAs
│   │   ├── Summons/
│   │   ├── Deferrals/
│   │   └── FTA/
│   └── Shared/                   # Cross-cutting concerns
│       ├── Data/
│       └── Infrastructure/
```

### Key Design Principles
1. **Module Independence**: Each module should be independently testable
2. **Clear Interfaces**: Define service interfaces between modules
3. **Shared Database, Separate Schemas**: Use jms.jurors and jms.court_scheduling schemas
4. **Future Extraction**: Design modules so they could be extracted if needed

### Integration Approach
1. Court scheduling functionality can reference juror data directly
2. Use domain events for loose coupling between modules
3. Shared transaction boundaries for workflow operations
4. Common infrastructure for auth, tenant context, and audit logging

---
**Date**: 2025-01-17  
**Author**: Claude (AI Assistant)  
**Reviewers**: [Pending]
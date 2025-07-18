# ADR-0008: Court Scheduling Phase 1 Data Model Implementation

## Status
Accepted

## Context
Following the architectural decisions in ADR-0004 (Court Scheduling Calendar Management) and ADR-0005 (Court Scheduling Service Placement), we are implementing Phase 1 of the court scheduling feature within the jms-juror-service. This phase focuses on establishing the foundational data model, repository patterns, and tenant isolation for managing court schedules across 60 locations.

The implementation must align with existing JMS patterns for:
- Tenant isolation using PostgreSQL Row-Level Security (RLS)
- Audit trail requirements with 7-year retention
- Repository pattern for data access
- BaseEntity inheritance for consistent entity tracking
- Integration with existing authentication and authorization

## Decision
We have implemented the court scheduling data model as a module within jms-juror-service with the following structure:

### Module Organization
```
src/CourtScheduling/
├── Models/
│   ├── Entities/         # Core domain entities
│   └── DTOs/            # Data transfer objects
├── Interfaces/          # Repository interfaces
├── Data/
│   ├── Repositories/    # Repository implementations
│   └── Configurations/  # EF Core configurations
├── Controllers/         # API controllers (Phase 2)
├── Services/           # Business logic (Phase 2)
└── Workflows/          # Approval workflows (Phase 3)
```

### Core Entities
1. **CourtLocation**: Manages physical court locations with timezone support
2. **CalendarYear**: Defines fiscal year boundaries and lock status
3. **CourtSchedule**: Daily schedule entries with availability and capacity
4. **ScheduleException**: System-wide or location-specific exceptions (holidays, emergencies)
5. **ScheduleOverride**: Approval-based schedule modifications
6. **ScheduleAudit**: Comprehensive audit trail for all schedule changes

### Technical Decisions
- All entities inherit from `TenantAwareEntity` for automatic tenant isolation
- PostgreSQL JSONB columns for flexible configuration and audit data
- Composite unique indexes for tenant + business key combinations
- Foreign key relationships with appropriate cascade behaviors
- Repository interfaces follow existing JMS patterns

## Consequences

### Positive
- **Consistency**: Follows established JMS patterns for entities, repositories, and tenant isolation
- **Maintainability**: Clear module boundaries within jms-juror-service
- **Performance**: Optimized indexes for common query patterns
- **Flexibility**: JSONB columns allow schema evolution without migrations
- **Security**: Automatic tenant isolation through RLS inheritance
- **Audit Compliance**: Built-in audit trail from day one

### Negative
- **Module Size**: Adds significant functionality to jms-juror-service
- **Migration Complexity**: Large initial migration with multiple related tables
- **Testing Overhead**: Requires comprehensive test coverage for tenant isolation
- **Documentation Need**: Complex relationships require clear documentation

### Neutral
- **Database Schema**: Uses existing `jms` schema with new tables
- **Performance Impact**: Additional tables and indexes in main database
- **Development Workflow**: Multiple developers working in same service codebase

## Alternatives Considered

1. **Separate CourtScheduling project within solution**
   - Rejected: Would complicate deployment and violate monorepo pattern
   
2. **Single Schedule table with type discriminators**
   - Rejected: Would complicate queries and reduce type safety
   
3. **NoSQL approach with document storage**
   - Rejected: Loses transactional consistency and complicates reporting

4. **Minimal entities without audit built-in**
   - Rejected: Would require retrofitting audit functionality later

## References
- [ADR-0004: Court Scheduling Calendar Management](./ADR-0004-court-scheduling-calendar-management.md)
- [ADR-0005: Court Scheduling Service Placement](./ADR-0005-court-scheduling-service-placement.md)
- [Phase 1 Implementation Plan](../x-project-docs/court-scheduling-JJS-007/phase1-data-model-implementation.md)
- [JMS Tenant Isolation Patterns](../x-project-docs/cross-team-coordination-plan-v3.md)

## Notes
### Implementation Progress
- ✅ Entity models created with proper inheritance
- ✅ Repository interfaces defined following JMS patterns
- ✅ Repository implementations with tenant awareness
- ✅ JurorDbContext updated with new entities and configurations
- ⏳ Database migrations pending
- ⏳ Unit tests pending
- ⏳ Integration tests pending

### Key Design Patterns
1. **Repository Pattern**: Abstracts data access with testable interfaces
2. **Tenant Isolation**: Automatic filtering through EF Core query filters
3. **Audit Interception**: Leverages existing JMS audit infrastructure
4. **Soft Deletes**: Uses `IsActive` flags instead of physical deletion

### Migration Strategy
1. Create new tables in `jms` schema
2. Apply RLS policies matching existing patterns
3. Create indexes for performance optimization
4. Seed initial calendar year if needed

### Next Phase Dependencies
Phase 2 (Calendar Management API) will build upon:
- Completed entity models
- Working repository implementations
- Successful database migrations
- Basic integration tests

---
**Date**: 2025-01-18  
**Author**: Claude (AI Assistant)  
**Reviewers**: [Pending]
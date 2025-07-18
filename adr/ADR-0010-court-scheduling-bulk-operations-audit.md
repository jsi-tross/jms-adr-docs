# ADR-0010: Court Scheduling Bulk Operations and Comprehensive Audit Trail

## Status
Accepted

## Context
The court scheduling system needs to handle large-scale operations efficiently while maintaining a complete audit trail for compliance and operational transparency. Phase 4 of the court scheduling implementation (JJS-007) requires:

1. **Bulk Operations**: Courts need to create, update, and delete schedules in bulk for entire calendar periods, multiple locations, or based on specific criteria
2. **Performance**: Operations involving thousands of schedule records must complete within reasonable timeframes
3. **Audit Trail**: All scheduling actions must be tracked for compliance, including who made changes, when, and why
4. **Data Retention**: Audit data must be retained for 7 years per regulatory requirements
5. **Reporting**: Administrators need to view, search, and export audit data for compliance reporting

The existing jms-juror-service already has:
- Basic audit infrastructure using JMS.Common.Audit
- Multi-tenant architecture with row-level security
- ScheduleAudit entity for court scheduling specific audits

## Decision
We will implement bulk operations and enhanced audit trail capabilities within the existing jms-juror-service by:

1. **Bulk Operations Architecture**:
   - Create dedicated `BulkOperationsController` with endpoints for bulk create, update, delete operations
   - Implement `BulkScheduleService` using batch processing with configurable batch sizes (default 1000)
   - Use database transactions to ensure atomicity of bulk operations
   - Disable EF Core change tracking during bulk operations for performance

2. **Audit Trail Enhancement**:
   - Extend existing `ScheduleAudit` infrastructure with `CourtSchedulingAuditService`
   - Implement `CourtSchedulingAuditInterceptor` to automatically capture all entity changes
   - Create `AuditController` with endpoints for searching, viewing, and exporting audit data
   - Store audit metadata as JSONB for flexible querying

3. **Performance Optimizations**:
   - Implement `BulkOperationOptimizer` for efficient database operations
   - Use raw SQL for very large bulk inserts (>5000 records)
   - Process operations in parallel batches where appropriate
   - Implement read operations with `AsNoTracking()` and `AsSplitQuery()`

4. **Data Retention**:
   - Implement `AuditRetentionService` as a hosted service
   - Archive audit records older than 7 years to cold storage
   - Run cleanup operations during off-peak hours (2 AM daily)

## Consequences

### Positive
- **Performance**: Bulk operations can process thousands of records efficiently
- **Compliance**: Complete audit trail meets regulatory requirements
- **Maintainability**: Leverages existing audit infrastructure in jms-juror-service
- **Scalability**: Batch processing prevents memory issues with large datasets
- **Flexibility**: JSONB metadata allows for future audit enhancements without schema changes
- **User Experience**: Dashboard and reporting features provide operational insights

### Negative
- **Complexity**: Additional services and interceptors increase codebase complexity
- **Storage**: Comprehensive audit trail requires significant database storage
- **Performance Impact**: Audit interceptor adds slight overhead to all write operations
- **Migration**: Existing data may need to be migrated to include audit metadata

### Neutral
- **Testing**: Requires comprehensive integration and performance tests
- **Monitoring**: Need to monitor bulk operation performance and audit storage growth
- **Documentation**: API documentation must be updated for new endpoints

## Alternatives Considered

1. **Separate Audit Service**:
   - Rejected: Would require cross-service communication and complicate transaction management
   - Would lose the benefit of existing audit infrastructure

2. **Event Sourcing**:
   - Rejected: Too complex for current requirements
   - Would require significant architectural changes

3. **Stored Procedures for Bulk Operations**:
   - Rejected: Would bypass business logic and audit trail
   - Less maintainable and harder to test

4. **Message Queue for Bulk Operations**:
   - Rejected: Adds unnecessary complexity for synchronous operations
   - Users expect immediate feedback on bulk operations

## References
- [ADR-0005: Court Scheduling Service Placement](ADR-0005-court-scheduling-service-placement.md)
- [ADR-0006: Audit Logging Architecture](ADR-0006-audit-logging-architecture.md)
- [ADR-0008: Court Scheduling Phase 1 Implementation](ADR-0008-court-scheduling-phase1-implementation.md)
- [Phase 4 Implementation Plan](../../x-project-docs/court-scheduling-JJS-007/phase4-bulk-operations-audit-trail.md)
- [Entity Framework Core Performance Best Practices](https://docs.microsoft.com/en-us/ef/core/performance/)

## Notes
- The 1000 record batch size was chosen based on typical SQL Server parameter limits
- The BulkOperationOptimizer can be extended for other bulk operations in the future
- Audit export supports CSV and JSON formats initially, with Excel format planned
- The audit retention service is designed to be extensible for other retention policies
- Performance testing should be conducted with realistic data volumes

---
**Date**: 2025-07-18  
**Author**: AI Assistant  
**Reviewers**: Pending
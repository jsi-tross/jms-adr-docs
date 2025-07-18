# ADR-0009: No-Code Business Rules Engine for Multi-Tenant Configuration

## Status
Accepted

## Context
The JMS system requires extensive business rule validation across multiple functional areas, particularly in court scheduling. Each tenant (court system) has unique operational requirements, thresholds, and policies that need to be enforced. Examples include:

- Maximum capacity overrides (e.g., some courts allow 20% over capacity, others only 10%)
- Notice period requirements (e.g., 48 hours vs 72 hours advance notice)
- Exception approval thresholds
- Scheduling constraints specific to local court procedures

Initially, these rules were hard-coded in the application, requiring code changes and deployments whenever a tenant needed different rules. This approach was:
- Inflexible and slow to adapt to changing requirements
- Risk-prone, as code changes could introduce bugs
- Expensive to maintain with multiple tenant-specific code branches
- Unable to support rapid onboarding of new tenants with unique requirements

## Decision
We have implemented a JSON-based, database-driven business rules engine that allows tenants to configure their own business rules without requiring code changes. The solution includes:

1. **JSON Expression Language**: Rules are defined as JSON expressions stored in the database
2. **Dynamic Rule Evaluator**: A recursive expression evaluator that interprets JSON rule definitions at runtime
3. **Tenant-Scoped Rules**: Each rule is associated with a specific tenant through the existing multi-tenant infrastructure
4. **Rule Categories**: Rules are organized by functional area (scheduling, capacity, exceptions, etc.)
5. **Parameter Support**: Rules can have configurable parameters that can be adjusted without changing the rule expression
6. **Caching Layer**: Rules are cached for 5 minutes with manual reload capability for performance

Example rule expression:
```json
{
  "type": "logical",
  "operator": "and",
  "conditions": [
    {
      "type": "numeric",
      "field": "AdjustedMaxJurors",
      "operator": "lessThanOrEqual",
      "baseField": "MaxJurors",
      "percentage": 120
    },
    {
      "type": "permission",
      "permission": "CourtScheduling.Overrides.EmergencyApprove"
    }
  ]
}
```

## Consequences

### Positive
- **Rapid Configuration**: New business rules can be added or modified through API calls without code deployment
- **Tenant Autonomy**: Each tenant can configure rules to match their specific requirements
- **Reduced Risk**: No code changes means less risk of introducing bugs when adjusting rules
- **Audit Trail**: All rule changes are tracked with who changed what and when
- **Performance**: Caching ensures minimal performance impact despite database storage
- **Testability**: Rules can be tested in isolation before activation
- **Self-Service**: Tenant administrators can potentially manage their own rules through a UI

### Negative
- **Complexity**: The expression evaluator adds complexity to the codebase
- **Debugging**: Runtime-evaluated rules can be harder to debug than compiled code
- **Type Safety**: JSON expressions lose compile-time type checking
- **Learning Curve**: Administrators need to understand the JSON expression syntax
- **Migration Effort**: Existing hard-coded rules need to be migrated to the new system

### Neutral
- **Database Dependency**: Rules are now stored in the database rather than in code
- **Schema Evolution**: The JSON expression schema may need to evolve over time
- **Documentation**: Comprehensive documentation is required for rule syntax and available operators

## Alternatives Considered

1. **Hard-coded Rules with Feature Flags**
   - Rejected: Still requires code changes and deployments for new rules
   
2. **Scripting Engine (JavaScript/Lua)**
   - Rejected: Security concerns with arbitrary code execution; more complex than needed

3. **Rules Engine Library (Drools, NRules)**
   - Rejected: Heavy dependencies; overkill for our use case; licensing concerns

4. **SQL-based Rules**
   - Rejected: Limited expressiveness; difficult to compose complex logical conditions

5. **Configuration Files**
   - Rejected: Still requires deployment; doesn't support per-tenant configuration easily

## References
- Phase 3 Court Scheduling Implementation (JJS-007)
- ADR-0001: Usage of ADRs for Architectural Decisions
- [JSON Expression Language Specification](internal documentation)
- [Multi-Tenant Architecture Patterns](https://docs.microsoft.com/en-us/azure/architecture/guide/multitenant)

## Notes
**Implementation Details**:
- The rule evaluator supports comparison, logical, date, numeric, and permission operators
- Rules are evaluated synchronously but could be made async if needed
- The caching layer uses IMemoryCache with sliding expiration
- Rule templates are provided for common patterns
- Statistics are tracked for rule execution and violations

**Future Enhancements**:
- UI for rule management
- Rule versioning and rollback capabilities
- A/B testing of rules
- Machine learning for rule optimization
- Rule conflict detection

**Migration Strategy**:
- Existing hard-coded rules have been preserved as "static" rules
- Default rules are seeded via migration for all tenants
- Gradual migration allows both systems to coexist during transition

---
**Date**: 2025-01-18  
**Author**: Claude (AI Assistant)  
**Reviewers**: Timothy Ross
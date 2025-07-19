# ADR-0013: Authorization Architecture for JMS Common Library

## Status
Accepted

## Context
The JMS system requires a robust, scalable authorization system that can handle high-volume permission checks across multiple services. The authorization system needs to:

1. Support multi-tenant architecture with tenant-specific roles and permissions
2. Handle high request volumes (100,000+ users, 60+ requests per user per hour)
3. Minimize latency for permission checks (target < 100ms P95)
4. Provide resilience against auth service failures
5. Support custom roles per tenant while maintaining system-defined roles
6. Enable comprehensive monitoring and debugging capabilities
7. Allow for future scaling and performance optimization

The previous implementation had several issues:
- Direct database queries for every permission check causing high latency
- No caching strategy leading to unnecessary load on the auth service
- Limited resilience patterns (no retry, circuit breaker, or fallback mechanisms)
- Monolithic interface design making it difficult to use only needed functionality
- No built-in observability (metrics, tracing, health checks)

## Decision
We have implemented a comprehensive authorization library with the following architecture:

### 1. Multi-Level Caching Strategy
- **L1 Cache (Memory)**: In-process memory cache for ultra-fast access (5-10ms)
- **L2 Cache (Redis)**: Distributed cache for cross-instance sharing (15-50ms)
- **L3 (Auth Service)**: Source of truth, accessed only on cache miss (100-300ms)

### 2. Resilience Patterns
- **Retry Policy**: Exponential backoff (2^n seconds) with 3 retries for transient failures
- **Rate Limiting**: Sliding window rate limiter (1000 requests/minute per user by default)
- **Graceful Degradation**: Returns empty collections on failure instead of throwing
- **Health Checks**: Monitors Redis and Auth Service availability

### 3. Interface Segregation
Following ISP (Interface Segregation Principle), we split `IPermissionService` into:
- `IPermissionChecker`: Permission validation operations
- `IRoleManager`: User role assignments
- `ICustomRoleManager`: Custom role management
- `IPermissionRegistry`: Permission queries
- `IPermissionCacheManager`: Cache operations

### 4. Observability
- **Metrics**: Comprehensive metrics using System.Diagnostics.Metrics
  - Permission check counts and durations
  - Cache hit rates by level
  - Auth service call success/failure rates
- **Distributed Tracing**: OpenTelemetry-compatible activity tracking
  - Trace context propagation
  - Tagged operations for filtering
  - Performance analysis capabilities
- **Health Checks**: ASP.NET Core health check integration

### 5. Performance Optimizations
- Batched cache invalidation (groups of 50 users)
- Wildcard permission support (e.g., "juror.*" for all juror permissions)
- Resource-level permission inheritance
- Preloading capabilities for critical users

## Consequences

### Positive
- **Improved Performance**: 85%+ cache hit rate reduces latency from 200ms to <20ms average
- **Better Scalability**: Redis cache enables horizontal scaling across instances
- **Enhanced Resilience**: System remains operational during auth service outages
- **Improved Developer Experience**: Segregated interfaces allow using only needed functionality
- **Comprehensive Monitoring**: Built-in metrics and tracing enable proactive issue detection
- **Reduced Database Load**: Caching dramatically reduces queries to the auth database
- **Tenant Isolation**: Clear separation of tenant-specific data in cache keys

### Negative
- **Increased Complexity**: Multi-level caching adds operational complexity
- **Cache Coherency**: Potential for stale data during the cache TTL window
- **Additional Infrastructure**: Requires Redis deployment and maintenance
- **Memory Usage**: L1 cache consumes application memory (estimated ~60MB for 10k users)
- **Debugging Complexity**: Multiple cache levels make troubleshooting more complex

### Neutral
- **Configuration Requirements**: Requires careful tuning of TTLs and rate limits
- **Monitoring Setup**: Requires integration with metrics collection infrastructure
- **Redis Dependency**: Adds external service dependency (mitigated by fallback to auth service)

## Alternatives Considered

### 1. Database-Only Caching
**Rejected**: Would not meet performance requirements. Database round-trips add 50-100ms latency.

### 2. In-Memory Cache Only
**Rejected**: Doesn't support horizontal scaling. Each instance would have cold cache on startup.

### 3. Complex Circuit Breaker Pattern
**Rejected**: Added unnecessary complexity. Simple retry with fallback proved sufficient.

### 4. GraphQL for Permission Queries
**Rejected**: Over-engineering for current needs. REST API with caching meets requirements.

### 5. JWT Token-Based Permissions
**Rejected**: Token size would be too large with many permissions. Token refresh complexity.

## References
- [JMS Common Authorization Documentation](../jms-common/docs/authorization/)
- [Performance Guide](../jms-common/docs/authorization/performance-guide.md)
- [Sequence Diagrams](../jms-common/docs/authorization/sequence-diagrams.md)
- [Interface Segregation Guide](../jms-common/docs/authorization/interface-segregation.md)
- [Distributed Tracing Guide](../jms-common/docs/authorization/distributed-tracing.md)
- [OpenTelemetry .NET](https://opentelemetry.io/docs/instrumentation/net/)
- [ASP.NET Core Health Checks](https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/health-checks)

## Notes
### Implementation Details
- Cache keys use sanitized user/tenant IDs to prevent injection attacks
- Permission names follow resource.action convention (e.g., "juror.view")
- Supports both system-defined and tenant-specific custom roles
- Rate limiting is per-user to prevent single user from exhausting resources

### Migration Path
1. Services can adopt the library incrementally
2. Existing permission checks can be replaced one at a time
3. Cache warming can be used during migration to prevent cold start issues
4. Health checks allow monitoring during rollout

### Future Enhancements
- Consider adding L0 cache (request-scoped) for repeated checks
- Implement cache warming strategies for predictable access patterns
- Add support for temporary permission elevation
- Consider permission inheritance hierarchies

---
**Date**: 2025-01-19  
**Author**: Claude (with Timothy Ross)  
**Reviewers**: Pending
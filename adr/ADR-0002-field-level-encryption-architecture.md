# ADR-0002: Field-Level Encryption Architecture for PII Data

## Status
Accepted

## Context
The JMS (Jury Management System) handles sensitive Personally Identifiable Information (PII) including Social Security Numbers (SSN) and Driver's License numbers. These fields require encryption at rest to meet security and compliance requirements. The system must:

- Encrypt PII data at the field level in the database
- Maintain searchability for exact match queries on encrypted fields
- Support per-tenant key isolation for multi-tenant architecture
- Provide audit trails for all PII access
- Integrate with AWS KMS for key management
- Minimize performance impact on the application

The JMS.Common.Encryption package exists in GitHub but is not accessible via the NuGet feed (returns 401 Unauthorized), requiring an immediate solution for the phase 2.1 encryption implementation.

## Decision
We implemented a local field-level encryption service within the juror service using the following architecture:

1. **Encryption Algorithm**: AES-256-GCM for authenticated encryption
2. **Key Management**: AWS KMS for master keys, AWS Secrets Manager for data encryption keys
3. **Search Strategy**: HMAC-SHA256 deterministic hashing for exact match searches
4. **Implementation Layer**: Entity Framework Core interceptors for automatic encryption/decryption
5. **UI Pattern**: Decrypt-on-demand with mandatory audit reason collection
6. **Key Isolation**: Per-tenant encryption keys with separate KMS keys and secrets

The implementation consists of:
- Local AesFieldEncryptionService in juror service (instead of using the inaccessible package)
- AwsKeyManager for AWS KMS/Secrets Manager integration
- EncryptionInterceptor for automatic field encryption on save
- React EncryptedField component for secure display with audit trails
- Permission-based access control through auth service

## Consequences

### Positive
- **Immediate Implementation**: Local implementation avoided blocking on package access issues
- **Security**: PII never stored in plaintext, encrypted at rest with AES-256-GCM
- **Searchability**: Preserved exact match search capability through deterministic hashing
- **Audit Trail**: Complete tracking of who accesses PII data and why
- **Performance**: Key caching (5 minutes) minimizes AWS API calls
- **User Experience**: Clear visual indicators and smooth decrypt/encrypt transitions
- **Multi-tenant Isolation**: Each tenant has completely separate encryption keys

### Negative
- **Code Duplication**: Local implementation may duplicate logic from JMS.Common.Encryption
- **Maintenance**: Need to maintain encryption code in multiple places if other services need it
- **Migration Effort**: Will need to migrate to common package when it becomes available
- **Limited Search**: Only exact match searches supported (no partial/wildcard searches on encrypted fields)

### Neutral
- **AWS Dependency**: Requires AWS KMS and Secrets Manager (already part of infrastructure)
- **Key Rotation**: Manual process initially, can be automated later
- **Performance Impact**: Minimal due to caching, but adds ~50-100ms for decrypt operations

## Alternatives Considered

1. **Wait for Package Access**: Rejected due to urgent timeline for phase 2.1
2. **Database-Level Encryption Only**: Rejected as it doesn't provide field-level granularity or audit trails
3. **Implement in Common Project**: Rejected as it would require modifying shared infrastructure
4. **Use Simpler Encryption (AES-CBC)**: Rejected as GCM provides authentication and integrity
5. **Client-Side Encryption**: Rejected due to key distribution complexity and search limitations

## References
- Phase 2.1 Encryption Planning Documents: `/x-project-docs/jms-juror-service-phase2/phase2.1-encryption/`
- AWS KMS Best Practices: https://docs.aws.amazon.com/kms/latest/developerguide/best-practices.html
- NIST SP 800-38D (AES-GCM): https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-38d.pdf
- ADR-0001: Use of ADR Process

## Notes

### Implementation Details
- Encryption happens at Entity Framework save operations via interceptor
- Decryption is on-demand only, never automatic
- 30-second auto-hide timer on decrypted values in UI
- Hash fields are indexed for performance
- AWS resources required per tenant:
  - KMS Key: `alias/jms-tenant-{tenantId}`
  - Secret: `jms/tenant/{tenantId}/data-encryption-key`

### Future Considerations
1. When JMS.Common.Encryption becomes available:
   - Create migration plan to use common package
   - Ensure backward compatibility with existing encrypted data
   - Update all services to use common implementation

2. Enhanced search capabilities:
   - Consider homomorphic encryption for range queries
   - Implement encrypted indexes for prefix searches
   - Add support for encrypted full-text search

3. Key rotation automation:
   - Implement automated key rotation with AWS KMS
   - Create re-encryption jobs for data key changes
   - Add monitoring for key age and usage

### Security Considerations
- Never log decrypted PII values
- Ensure TLS for all data in transit
- Regular security audits of encryption implementation
- Monitor for unusual PII access patterns
- Implement rate limiting on decrypt operations

---
**Date**: 2025-01-16  
**Author**: Claude (AI Assistant)  
**Reviewers**: JMS Development Team
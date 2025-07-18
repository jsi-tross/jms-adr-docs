# ADR-0007: AWS Infrastructure Architecture for JMS Encryption Services

## Status
Proposed

## Context
The JMS (Jury Management System) has implemented field-level encryption for PII data as described in ADR-0002. The jms-common library provides encryption capabilities that require specific AWS services for key management, secrets storage, and audit logging. Currently, there is no centralized infrastructure-as-code solution to provision and manage these AWS resources consistently across environments.

Key challenges include:
- Multiple services (auth, tenant, juror) need consistent AWS access
- Per-tenant encryption requires dynamic AWS resource provisioning
- Compliance requirements demand comprehensive audit trails
- Infrastructure must support multi-environment deployments (dev, staging, prod)
- Cost optimization needed for scaling to 1000+ tenants
- No existing Terraform modules for JMS-specific AWS resources

## Decision
We will create a dedicated jms-infrastructure repository containing Terraform modules to provision and manage all AWS resources required by the JMS encryption system. The infrastructure will follow these architectural principles:

1. **Modular Terraform Design**: Separate modules for each AWS service (KMS, Secrets Manager, IAM, CloudWatch, CloudTrail)
2. **Per-Tenant Resource Isolation**: Each tenant gets dedicated KMS keys and secrets with no cross-tenant access
3. **Environment Separation**: Distinct infrastructure for dev, staging, and production environments
4. **Centralized State Management**: Terraform state stored in S3 with DynamoDB locking
5. **GitOps Workflow**: Infrastructure changes through pull requests with automated validation

The architecture includes:
- **AWS KMS**: Customer Master Keys (CMKs) per tenant with automatic rotation
- **AWS Secrets Manager**: Data Encryption Key (DEK) storage with KMS encryption
- **IAM Roles**: Service-specific roles with least privilege access
- **CloudWatch**: Centralized logging and metrics for encryption operations
- **CloudTrail**: Immutable audit trail for all cryptographic operations

## Consequences

### Positive
- **Consistency**: All environments provisioned from same Terraform code
- **Security**: Infrastructure-as-code enables security reviews and compliance validation
- **Scalability**: Automated tenant provisioning supports growth to 1000+ tenants
- **Auditability**: All infrastructure changes tracked in version control
- **Cost Visibility**: Resource tagging enables per-tenant cost allocation
- **Disaster Recovery**: Infrastructure can be recreated quickly in any region

### Negative
- **Initial Complexity**: Requires Terraform expertise for maintenance
- **State Management**: Terraform state becomes critical infrastructure component
- **Migration Effort**: Existing manually-created resources need migration
- **Learning Curve**: Development teams need basic Terraform knowledge

### Neutral
- **AWS Lock-in**: Architecture specifically designed for AWS services
- **Operational Overhead**: Requires infrastructure pipeline maintenance
- **Version Management**: Terraform and provider versions need regular updates

## Alternatives Considered

1. **Manual AWS Console Management**
   - Rejected: Error-prone, not repeatable, no audit trail

2. **CloudFormation**
   - Rejected: Less flexibility than Terraform, limited community modules

3. **AWS CDK (Cloud Development Kit)**
   - Rejected: Requires additional programming language expertise, less mature

4. **Pulumi**
   - Rejected: Smaller community, fewer existing modules, steeper learning curve

5. **Embedded Infrastructure Code in Services**
   - Rejected: Violates separation of concerns, harder to maintain consistency

## References
- [ADR-0002: Field-Level Encryption Architecture](../../../jms-adr-docs/adr/ADR-0002-field-level-encryption-architecture.md) - Defines encryption requirements driving infrastructure needs
- [ADR-0003: Migration to Common Encryption Package](../../../jms-adr-docs/adr/ADR-0003-migration-to-common-encryption-package.md) - Related to jms-common library requirements
- [AWS KMS Best Practices](https://docs.aws.amazon.com/kms/latest/developerguide/best-practices.html)
- [Terraform AWS Provider Documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- [AWS Well-Architected Framework - Security Pillar](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/welcome.html)

## Notes
Implementation details:
- Terraform modules will be versioned using semantic versioning
- Each module will include comprehensive variable validation
- Output values will expose necessary resource identifiers for service configuration
- Module documentation will include example usage for common scenarios
- Integration tests will validate infrastructure provisioning
- Cost estimation will be performed before production deployment

The infrastructure repository structure will follow Terraform best practices with separate directories for modules, environments, and documentation. All AWS resources will be tagged according to JMS tagging standards for cost allocation and compliance tracking.

---
**Date**: 2025-01-17  
**Author**: Claude (on behalf of jms-infrastructure team)  
**Reviewers**: [Pending]
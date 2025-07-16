# ADR-0003: Direct Usage of JMS.Common.Encryption Package

## Status
Accepted

## Context
In ADR-0002, we documented the decision to implement field-level encryption locally within the juror service because the JMS.Common.Encryption package was not accessible via NuGet (401 Unauthorized error). This was a temporary solution that allowed us to proceed with phase 2.1 encryption implementation without being blocked.

The JMS.Common.Encryption package has now become available via GitHub Packages with proper authentication configured. Since we are in the early stages of development and not concerned with backward compatibility, we can adopt the common package directly without maintaining local implementations or adapter patterns.

This approach allows us to:
- Eliminate all code duplication across services
- Ensure consistent encryption implementation
- Simplify maintenance significantly
- Standardize on a single encryption approach
- Remove unnecessary abstraction layers

## Decision
We have migrated all JMS services to use the JMS.Common.Encryption package directly, removing all local encryption implementations and adapter patterns. The migration approach:

1. **Direct Integration**: All services now use JMS.Common.Encryption interfaces directly
2. **Complete Removal**: Deleted all local encryption implementations, interfaces, and adapters
3. **Unified Configuration**: All services use the same encryption configuration pattern
4. **Consistent Registration**: Standardized dependency injection across all services

The implementation:
- Added JMS.Common.Encryption package reference (v1.0.0) to all services
- Configured GitHub Packages authentication using fine-grained tokens
- Updated all code to use `JMS.Common.Encryption.Interfaces` directly
- Removed all local encryption-related code
- Standardized AWS service registration across all services

## Consequences

### Positive
- **Zero Code Duplication**: Completely eliminated all duplicate encryption logic
- **Perfect Consistency**: All services use identical encryption implementation
- **Minimal Maintenance**: Single codebase for all encryption logic
- **Automatic Updates**: All services benefit from package improvements
- **Cleaner Architecture**: Removed unnecessary adapter layers and local implementations
- **Simplified Testing**: Only need to test the common package
- **Reduced Complexity**: Fewer interfaces and implementations to understand

### Negative
- **Package Dependency**: All services depend on the common package availability
- **No Backward Compatibility**: Cannot decrypt data encrypted with different implementations
- **GitHub Authentication Required**: Developers need GitHub tokens configured

### Neutral
- **One-time Migration**: Completed for all services in a single effort
- **Standardized Configuration**: All services use the same encryption settings
- **AWS Service Dependencies**: All services now require AWS KMS and Secrets Manager

## Alternatives Considered

1. **Adapter Pattern**: Create adapters to maintain local interfaces while using common implementation
   - Rejected: Adds unnecessary complexity when backward compatibility is not required

2. **Keep Local Implementation**: Continue using local encryption code
   - Rejected: Defeats purpose of having a common package and increases maintenance

3. **Gradual Migration**: Slowly phase out local code over multiple releases
   - Rejected: More complex than direct migration when not constrained by compatibility

## References
- ADR-0002: Field-Level Encryption Architecture for PII Data
- JMS.Common.Encryption Package: https://github.com/jsi-tross/jms-common
- Phase 2.1 Encryption Implementation Documentation

## Notes

### Migration Steps Completed
1. ✅ Configured GitHub Packages authentication with fine-grained token
2. ✅ Added JMS.Common.Encryption package reference (v1.0.0) to all services
3. ✅ Updated all code to use JMS.Common.Encryption interfaces directly
4. ✅ Removed all local encryption implementations and interfaces
5. ✅ Deleted adapter pattern implementations
6. ✅ Standardized service registration across all projects
7. ✅ Verified successful builds for all services

### Services Updated
1. **Juror Service**
   - Removed local encryption service implementations
   - Updated EncryptionInterceptor to use common interfaces
   - Configured AWS services and encryption options

2. **Auth Service**
   - Added encryption package to Infrastructure project
   - Registered encryption services in Program.cs
   - Ready for encryption permission validation

3. **Tenant Service**
   - Added encryption package
   - Registered encryption services
   - Encryption configuration endpoints already implemented

### Configuration Requirements
All services must include the following configuration in their appsettings.json:
```json
{
  "Encryption": {
    "AwsRegion": "us-east-1",
    "KmsKeyAliasPrefix": "alias/jms-tenant",
    "SecretPrefix": "jms/tenant",
    "EnableAuditing": true,
    "CacheKeys": true,
    "KeyCacheDuration": "00:05:00"
  }
}
```

### Developer Setup
1. Configure GitHub authentication:
   ```powershell
   [Environment]::SetEnvironmentVariable("GITHUB_USERNAME", "your-username", "User")
   [Environment]::SetEnvironmentVariable("GITHUB_TOKEN", "your-token", "User")
   ```

2. Run setup script (available in each service):
   ```powershell
   .\setup-nuget-auth.ps1
   ```

3. Restore packages:
   ```bash
   dotnet restore
   ```

### Package Versioning Strategy
- All services must use the same version of JMS.Common.Encryption
- Updates should be coordinated across all services
- Use semantic versioning for breaking changes

---
**Date**: 2025-01-16  
**Author**: Claude (AI Assistant)  
**Reviewers**: JMS Development Team
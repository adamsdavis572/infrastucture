# Project Structure and Conventions

## Repository Structure

### Top-Level Directory Organisation
```
infrastucture/                          # Root repository
├── .github/                           # GitHub workflows and templates
│   ├── workflows/                     # CI/CD pipeline definitions
│   ├── chatmodes/                     # GitHub Copilot chat modes
│   └── ISSUE_TEMPLATE/                # Issue templates
├── .kiro/                             # Kiro SDD framework files
│   ├── steering/                      # Project steering documents
│   │   ├── product.md                 # Product vision and strategy
│   │   ├── tech.md                    # Technology stack decisions
│   │   └── structure.md               # This file - project structure
│   ├── requirements.md                # Business and functional requirements
│   ├── design.md                      # Technical design documentation
│   └── tasks.md                       # Development task breakdown
├── compositions/                      # Crossplane composition definitions
│   ├── xapi/                         # API management compositions
│   ├── xingress/                     # Ingress and networking compositions
│   └── [service-name]/               # Additional service compositions
├── crossplane-provider/              # Provider configurations and functions
├── docs/                             # User and operator documentation
├── examples/                         # Usage examples and templates
├── tests/                            # Automated tests (to be created)
├── scripts/                          # Utility and automation scripts (to be created)
└── README.md                         # Repository overview and getting started
```

### Composition Directory Structure
Each service composition follows a consistent structure:
```
compositions/[service-name]/
├── [service-name]-composition.yaml    # Main composition definition
├── [service-name]-definition.yaml     # Composite Resource Definition (XRD)
├── README.md                          # Service-specific documentation
├── examples/                          # Usage examples for this service
│   └── example-[service-name].yaml   # Basic usage example
└── tests/                            # Service-specific tests
    ├── unit/                         # Unit tests for composition logic
    └── integration/                  # Integration tests
```

## Naming Conventions

### File and Directory Naming
- **Directories**: Use kebab-case (lowercase with hyphens)
  - ✅ `api-management`, `network-services`
  - ❌ `ApiManagement`, `network_services`
- **YAML Files**: Use kebab-case with descriptive names
  - ✅ `xapi-composition.yaml`, `azure-network-policy.yaml`
  - ❌ `comp1.yaml`, `AzureNetworkPolicy.yaml`
- **Documentation Files**: Use UPPERCASE for standard docs, kebab-case for specific guides
  - ✅ `README.md`, `API_REFERENCE.md`, `troubleshooting-guide.md`

### Crossplane Resource Naming
- **Composite Resources**: Use descriptive names with organisation prefix
  - Format: `X[ServiceName]` (e.g., `XApi`, `XIngress`)
- **Compositions**: Match the composite resource name with environment suffix if needed
  - Format: `x[service-name]-[environment]` (e.g., `xapi-production`, `xingress-development`)
- **Provider Configurations**: Use provider type and purpose
  - Format: `[provider]-[purpose]` (e.g., `azure-default`, `aws-networking`)

### Kubernetes Resource Naming
- **Namespaces**: Use kebab-case with clear purpose indication
  - Format: `[purpose]-[environment]` (e.g., `platform-system`, `tenant-development`)
- **Custom Resources**: Follow Kubernetes conventions with descriptive names
- **Labels and Annotations**: Use reverse DNS notation for custom labels
  - Format: `platform.company.com/[purpose]`

## Code Organisation Patterns

### Composition Structure Pattern
Each Crossplane composition should follow this internal organisation:
```yaml
# Header with metadata and labels
apiVersion: apiextensions.crossplane.io/v1
kind: Composition
metadata:
  name: xapi-composition
  labels:
    platform.company.com/service: api-management
    platform.company.com/version: v1.0.0

# Composite type reference
spec:
  compositeTypeRef:
    apiVersion: platform.company.com/v1alpha1
    kind: XApi

# Resources organised by logical grouping
  resources:
    # Core service resources first
    - name: api-management-service
      # ... resource definition
    
    # Supporting infrastructure second
    - name: resource-group
      # ... resource definition
    
    # Networking components third
    - name: virtual-network
      # ... resource definition
    
    # Security and access control last
    - name: access-policy
      # ... resource definition
```

### Documentation Organisation Pattern
Each service should maintain documentation in this order:
1. **Overview**: What the service provides
2. **Prerequisites**: Dependencies and requirements
3. **Configuration**: Available parameters and options
4. **Examples**: Common usage patterns
5. **Troubleshooting**: Common issues and solutions

## Variable and Parameter Naming

### Composition Parameters
- **Input Parameters**: Use camelCase for consistency with Kubernetes
  - ✅ `apiName`, `resourceGroupName`, `enableLogging`
  - ❌ `api_name`, `resource-group-name`
- **Internal Variables**: Use descriptive names that clearly indicate purpose
  - ✅ `storageAccountName`, `networkSecurityGroupRules`
  - ❌ `name`, `rules`

### Configuration Values
- **Environment Variables**: Use UPPER_SNAKE_CASE
  - ✅ `AZURE_SUBSCRIPTION_ID`, `DEFAULT_RESOURCE_GROUP`
  - ❌ `azureSubscriptionId`, `default-resource-group`
- **Helm Values**: Use camelCase matching Kubernetes conventions
- **ConfigMap Keys**: Use kebab-case for readability
  - ✅ `api-endpoint-url`, `max-retry-attempts`

## Architectural Patterns

### Multi-Tenancy Pattern
- **Namespace Isolation**: Each tenant gets dedicated namespaces
  - Format: `tenant-[tenant-id]-[environment]`
- **Resource Naming**: Include tenant identifier in resource names
  - Format: `[tenant-id]-[resource-type]-[unique-suffix]`
- **Label Strategy**: Consistent labelling for tenant identification
  ```yaml
  labels:
    platform.company.com/tenant: "tenant-123"
    platform.company.com/environment: "production"
    platform.company.com/service: "api-management"
  ```

### Composition Layering Pattern
- **L1 - Provider Resources**: Direct cloud provider resource definitions
- **L2 - Service Compositions**: Business service abstractions (current focus)
- **L3 - Application Compositions**: Higher-level application platform abstractions

### Configuration Management Pattern
- **Base Configurations**: Common settings applicable to all environments
- **Environment Overlays**: Environment-specific configuration overrides
- **Tenant Customisations**: Tenant-specific parameter variations

## Git Workflow Patterns

### Branch Naming
- **Feature Branches**: `feature/[TASK-ID]-[short-description]`
  - ✅ `feature/TASK-001-api-management-composition`
- **Bugfix Branches**: `bugfix/[TASK-ID]-[short-description]`
  - ✅ `bugfix/TASK-015-network-policy-fix`
- **Release Branches**: `release/v[major].[minor].[patch]`
  - ✅ `release/v1.2.0`

### Commit Message Format
```
[type]: [short description] ([TASK-ID], [REQ-ID])

[Optional longer description]

Relates to: TASK-XXX
Implements: REQ-XXX
```

Types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`

### File Organisation Principles
1. **Separation of Concerns**: Each file should have a single, clear purpose
2. **Logical Grouping**: Related resources should be co-located
3. **Reusability**: Common patterns should be extracted to shared locations
4. **Testability**: Structure should support automated testing
5. **Documentation**: Each component should be self-documenting with clear examples

## Quality Standards

### YAML Formatting
- **Indentation**: 2 spaces (no tabs)
- **Line Length**: Maximum 120 characters
- **Comments**: Use `#` for explanatory comments above complex sections
- **Consistency**: Maintain consistent ordering of fields within resource types

### Documentation Standards
- **Markdown Formatting**: Follow standard Markdown conventions
- **Code Examples**: Include working examples with explanations
- **API Documentation**: Auto-generate where possible from OpenAPI specs
- **Versioning**: Document version compatibility and migration guides

### Testing Standards
- **Unit Tests**: Test individual composition logic
- **Integration Tests**: Test end-to-end provisioning scenarios
- **Validation Tests**: Ensure configurations meet security and compliance requirements
- **Performance Tests**: Validate provisioning times and resource usage

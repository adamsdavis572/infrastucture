---
applyTo:
  - "compositions/**/*.yaml"
  - "examples/**/*.yaml"
  - "**/*-definition.yaml"
  - "**/*-composition.yaml"
---

# Crossplane Code Style and Structure Guidelines

## Directory Structure

### Standard Layout
```
compositions/
├── {resource-type}/              # One directory per composite resource type
│   ├── {resource-type}-definition.yaml    # CompositeResourceDefinition (XRD)
│   └── {resource-type}-composition.yaml   # Composition
crossplane-provider/
├── function-{function-name}.yaml          # Function definitions
├── provider-{provider-name}.yaml          # Provider installations
└── providerconfig-{config-name}.yaml      # Provider configurations
examples/
├── example-{resource-type}.yaml           # Working examples
docs/
└── {TOPIC}.md                             # Documentation files
```

### Observed Patterns
- `compositions/xingress/` - Shared gateway infrastructure
- `compositions/xapi/` - Individual API management
- Provider files prefixed with `provider-azure-{service}`

## File Naming Conventions

### Definition Files
- **Pattern**: `{resource-type}-definition.yaml`
- **Examples**: 
  - `xingress-definition.yaml`
  - `xapi-definition.yaml`

### Composition Files
- **Pattern**: `{resource-type}-composition.yaml`
- **Examples**:
  - `xingress-composition.yaml` 
  - `xapi-composition.yaml`

### Provider Files
- **Pattern**: `provider-{cloud}-{service}.yaml`
- **Examples**:
  - `provider-azure-apimanagement.yaml`
  - `provider-azure-network.yaml`
  - `provider-azure-storage.yaml`

### Function Files
- **Pattern**: `function-{function-name}.yaml`
- **Example**: `function-patch-and-transform.yaml`

## Resource Naming Conventions

### CompositeResourceDefinition (XRD)
```yaml
metadata:
  name: {plural}.{group-domain}
spec:
  group: {group-domain}
  names:
    kind: {PascalCase}
    plural: {lowercase-plural}
  claimNames:        # Only for claimable resources
    kind: {PascalCase}Claim
    plural: {lowercase-plural}claims
```

**Examples**:
- `xingresses.azure.platform.my-org.co` → `XIngress`
- `apiinstances.platform.my-org.co` → `ApiInstance` / `ApiClaim`

### Composition Names
```yaml
metadata:
  name: {descriptive-name}
```

**Patterns**:
- Infrastructure: `{resource-type}-composite` (e.g., `xingress-composite`)
- API/Claims: Use full domain (e.g., `apiinstances.platform.my-org.co`)

### Managed Resource Names in Compositions
```yaml
resources:
  - name: {service-purpose}  # kebab-case, descriptive
```

**Patterns**:
- **Infrastructure**: `api-management`, `api-gateway`, `vnet-dmz`, `subnet-apim`, `resource-group-dmz`
- **API Resources**: `api`, `gateway-api`, `policy`, `backend`

## Composition Structure Standards

### Required Pipeline Mode
```yaml
spec:
  mode: Pipeline
  pipeline:
  - step: patch-and-transform
    functionRef:
      name: function-patch-and-transform
    input:
      apiVersion: pt.fn.crossplane.io/v1beta1
      kind: Resources
```

### PatchSet Patterns
```yaml
patchSets:
  - name: common-labels
    patches:
      - type: FromCompositeFieldPath
        fromFieldPath: spec.name
        toFieldPath: metadata.labels.{resource-type}
```

### Resource Reference Pattern
```yaml
spec:
  forProvider:
    {resource}Selector:
      matchControllerRef: true  # Always use for cross-resource refs
```

## Transform Patterns

### Location Mapping (Required)
```yaml
transforms:
  - type: map
    map:
      UK: "UK South"
      USEast: "East US"
```

### String Formatting for Resource Names
```yaml
transforms:
  - type: string
    string:
      type: Format
      fmt: "{prefix}-%s"  # Use consistent prefix patterns
```

**Prefix Patterns**:
- Resource Groups: `rg-`
- Virtual Networks: `vnet-`
- Subnets: `subnet-{purpose}-` (e.g., `subnet-apim-`)

### External Name Annotation
```yaml
toFieldPath: "metadata.annotations[crossplane.io/external-name]"
```

## API Group Conventions

### Group Domains
- **Infrastructure**: `{cloud}.platform.{org-domain}` (e.g., `azure.platform.my-org.co`)
- **Application**: `platform.{org-domain}` (e.g., `platform.my-org.co`)

### Version Strategy
- Start with `v1alpha1`
- Use semantic versioning principles
- Always set `served: true` and `referenceable: true`

## Field Naming Standards

### Required Fields Pattern
```yaml
properties:
  {field}:
    type: string
    description: "{Clear description}"
required:
  - {field}
```

### Location Field (Standard)
```yaml
location:
  type: string
  oneOf:
    - pattern: '^UK$'
    - pattern: '^USEast$'
```

### Optional Overrides Pattern
```yaml
resourceNames:
  type: object
  description: "Optional: Override auto-generated resource names"
  properties:
    {resourceType}:
      type: string
      description: "Name for the {resource} (default: {prefix}-{name})"
```

## Provider Configuration Standards

### Provider Installation
```yaml
apiVersion: pkg.crossplane.io/v1
kind: Provider
metadata:
  name: provider-{cloud}-{service}
spec:
  package: xpkg.crossplane.io/{org}/{provider}:{version}
```

### Provider Config
```yaml
apiVersion: {provider}.upbound.io/v1beta1
kind: ProviderConfig
metadata:
  name: default
spec:
  credentials:
    source: Secret
    secretRef:
      namespace: crossplane-system
      name: {cloud}-secret
      key: creds
```

## Code Organization Best Practices

### Resource Order in Compositions
1. PatchSets (at top)
2. Core infrastructure (Resource Groups, Networks)
3. Application services (API Management, etc.)
4. Application-specific resources (APIs, Policies)

### Commenting Standards
```yaml
# Commented resources for future implementation
# - name: policy
#   base:
#     # ... commented implementation
```

### Label Consistency
Apply `common-labels` patchSet to all managed resources for consistent labeling and tracking.

## Validation Patterns

### Enum Validation
```yaml
enum:
  - "Consumption"
  - "Developer_1" 
  - "Basic_1"
  - "Standard_1"
  - "Premium_1"
```

### Array Defaults
```yaml
default: ["https"]  # Always provide sensible defaults
```

### Required vs Optional
- Mark infrastructure essentials as `required`
- Provide defaults for operational settings
- Use `resourceNames` pattern for optional overrides

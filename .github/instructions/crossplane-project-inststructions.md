---
applyTo: "block"
---
# AI Coding Agent Instructions for Crossplane Azure Infrastructure Platform

## Project Overview
This is a Crossplane-based platform for managing Azure API Management infrastructure declaratively through Kubernetes. The platform provides two main abstractions:

- **XIngress**: Shared API gateway infrastructure (Azure API Management + networking)
- **ApiClaim**: Individual APIs that reference an XIngress gateway

## Architecture Patterns

### Resource Hierarchy
```
XIngress (Shared Gateway) → Resource Group → VNet → Subnet → API Management Service → Gateway
                                ↓
ApiClaim (Individual APIs) → API + Gateway Association
```

### Key Design Principles
- **Shared Infrastructure**: One XIngress supports multiple ApiClaims for cost optimization
- **Location Abstraction**: Use `UK` or `USEast` (maps to `UK South` and `East US` Azure regions)
- **Pipeline Mode**: Compositions use `mode: Pipeline` with `function-patch-and-transform`

## Critical File Structure

```
compositions/
├── xingress/           # Shared gateway infrastructure
│   ├── xingress-definition.yaml    # XRD for XIngress
│   └── xingress-composition.yaml   # Creates RG, VNet, Subnet, API Management, Gateway
└── xapi/              # Individual API management
    ├── xapi-definition.yaml        # XRD for ApiInstance/ApiClaim
    └── xapi-composition.yaml       # Creates API + Gateway association
```

## Development Workflows

### Adding New Resources to XIngress
1. Edit `compositions/xingress/xingress-composition.yaml`
2. Add new resource to `spec.pipeline[0].input.resources[]`
3. Use `matchControllerRef: true` for resource references
4. Apply `common-labels` patchSet for consistent labeling

### Region Mapping Pattern
Always use location transforms in compositions:
```yaml
transforms:
  - type: map
    map:
      UK: "UK South"
      USEast: "East US"
```

### Resource Naming Convention
- Auto-generated: `rg-{name}`, `vnet-{name}`, `subnet-apim-{name}`
- Override via `spec.resourceNames` in XIngress
- Use `crossplane.io/external-name` annotation for custom names

## Configuration Patterns

### Required Fields
- **XIngress**: `location`, `name`
- **ApiClaim**: `ingressRef`, `apiName`, `apiPath`, `swaggerDefinition`, `backendUrl`

### Default Values
- API Management SKU: `Developer_1`
- VNet address space: `["10.0.0.0/16"]`
- Subnet prefix: `["10.0.1.0/24"]`
- API protocols: `["https"]`

### SKU Options
Development: `Developer_1`, `Consumption`
Production: `Basic_1`, `Standard_1`, `Premium_1`

## Provider Configuration

### Azure Providers Required
```bash
kubectl apply -f crossplane-provider/provider-azure-*.yaml
```

### Secret Format
Azure credentials secret expects JSON with:
```json
{
  "clientId": "...",
  "clientSecret": "...", 
  "subscriptionId": "...",
  "tenantId": "..."
}
```

## Testing and Debugging

### Resource Status Commands
```bash
# Check XIngress status
kubectl get xingress -o wide

# Check ApiClaim status  
kubectl get apiclaim -o wide

# Check underlying Azure resources
kubectl get managed

# Debug failed resources
kubectl describe <resource-type> <resource-name>
```

### Common Issues
- **Provider not ready**: Check `kubectl get providers`
- **Missing credentials**: Verify azure-secret in crossplane-system namespace
- **Location issues**: Ensure using `UK` or `USEast` (not Azure region names)
- **Resource conflicts**: Check for duplicate external-name annotations

## Documentation Navigation
- `examples/`: Working resource examples
- `docs/INSTALLATION.md`: Complete setup guide
- `docs/CONFIGURATION.md`: All configuration options
- `docs/API_REFERENCE.md`: Complete API specs
- `crossplane-role-definition.json`: Required Azure permissions

## Integration Points
- **Azure API Management**: Core service for API gateway functionality
- **Azure Networking**: VNet/Subnet for network isolation
- **Crossplane Providers**: Multiple Azure providers for different resource types
- **Kubernetes**: All resources managed as Kubernetes custom resources

When modifying compositions, always test with example resources and verify the underlying Azure resources are created correctly.

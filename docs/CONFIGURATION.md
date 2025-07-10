# Configuration Reference

## XIngress Configuration

The XIngress resource manages the shared API gateway infrastructure.

### Basic Configuration

```yaml
apiVersion: azure.platform.my-org.co/v1alpha1
kind: XIngress
metadata:
  name: my-gateway
spec:
  name: my-gateway
  location: UK
```

### Complete Configuration

```yaml
apiVersion: azure.platform.my-org.co/v1alpha1
kind: XIngress
metadata:
  name: my-gateway
spec:
  name: my-gateway
  location: UK
  apiManagement:
    publisherEmail: devops@my-org.co
    publisherName: DevOps Team
    skuName: Developer_1
  networking:
    vnetAddressSpace:
      - 10.0.0.0/16
    subnetPrefix:
      - 10.0.1.0/24
```

### XIngress Field Reference

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `name` | string | Yes | - | Identifier for the XIngress instance |
| `location` | string | Yes | - | Azure region (UK or USEast) |
| `apiManagement.publisherEmail` | string | No | "company@terraform.io" | API Management publisher email |
| `apiManagement.publisherName` | string | No | "My Company" | API Management publisher name |
| `apiManagement.skuName` | string | No | "Developer_1" | API Management SKU |
| `networking.vnetAddressSpace` | []string | No | ["10.0.0.0/16"] | Virtual network address space |
| `networking.subnetPrefix` | []string | No | ["10.0.1.0/24"] | Subnet address prefix |

### Supported Locations

| Location Value | Azure Region |
|----------------|--------------|
| `UK` | UK South |
| `USEast` | East US |

### Supported SKUs

| SKU | Description | Use Case |
|-----|-------------|----------|
| `Consumption` | Pay-per-use model | Development, testing |
| `Developer_1` | Development tier (1 unit) | Development |
| `Basic_1` | Basic tier (1 unit) | Small production |
| `Standard_1` | Standard tier (1 unit) | Production |
| `Premium_1` | Premium tier (1 unit) | Enterprise |

## ApiClaim Configuration

The ApiClaim resource manages individual APIs within the gateway.

### Basic Configuration

```yaml
apiVersion: platform.my-org.co/v1alpha1
kind: ApiClaim
metadata:
  name: my-api
spec:
  ingressRef: my-gateway
  apiName: "My API"
  apiPath: "myapi"
  backendUrl: "https://my-backend.azurewebsites.net/api"
  swaggerDefinition: |
    openapi: 3.0.0
    info:
      title: My API
      version: 1.0.0
    paths:
      /health:
        get:
          responses:
            '200':
              description: OK
```

### Complete Configuration

```yaml
apiVersion: platform.my-org.co/v1alpha1
kind: ApiClaim
metadata:
  name: my-api
spec:
  ingressRef: my-gateway
  apiName: "My API"
  apiPath: "myapi"
  apiVersion: "v1"
  apiRevision: "1"
  apiProtocols: 
    - "https"
  swaggerDefinition: |
    openapi: 3.0.0
    info:
      title: My API
      version: 1.0.0
    paths:
      /health:
        get:
          responses:
            '200':
              description: OK
  swaggerFormat: "openapi"
  backendUrl: "https://my-backend.azurewebsites.net/api"
  serviceUrl: "https://my-backend.azurewebsites.net/api"
```

### ApiClaim Field Reference

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `ingressRef` | string | Yes | - | Reference to XIngress resource |
| `apiName` | string | Yes | - | Display name for the API |
| `apiPath` | string | Yes | - | Base path for the API |
| `apiVersion` | string | No | "v1" | API version |
| `apiRevision` | string | No | "1" | API revision |
| `apiProtocols` | []string | No | ["https"] | Supported protocols |
| `swaggerDefinition` | string | Yes | - | OpenAPI/Swagger definition |
| `swaggerFormat` | string | No | "openapi" | Format of the definition |
| `backendUrl` | string | Yes | - | Backend service URL |
| `serviceUrl` | string | No | `backendUrl` | Service URL (defaults to backendUrl) |

### Supported Protocols

- `http`
- `https`
- `ws` (WebSocket)
- `wss` (WebSocket Secure)

### Swagger Formats

- `openapi` - OpenAPI 3.0
- `swagger` - Swagger 2.0
- `wadl` - Web Application Description Language
- `wsdl` - Web Services Description Language

## Networking Configuration

### Virtual Network Configuration

```yaml
networking:
  vnetAddressSpace:
    - 10.0.0.0/16
    - 10.1.0.0/16  # Multiple address spaces
  subnetPrefix:
    - 10.0.1.0/24
    - 10.0.2.0/24  # Multiple subnets
```

### Common Network Configurations

#### Small Environment
```yaml
networking:
  vnetAddressSpace:
    - 10.0.0.0/24
  subnetPrefix:
    - 10.0.0.0/26
```

#### Medium Environment
```yaml
networking:
  vnetAddressSpace:
    - 10.0.0.0/16
  subnetPrefix:
    - 10.0.1.0/24
    - 10.0.2.0/24
```

#### Large Environment
```yaml
networking:
  vnetAddressSpace:
    - 10.0.0.0/8
  subnetPrefix:
    - 10.0.1.0/16
    - 10.0.2.0/16
```

## Resource Naming Conventions

### XIngress Resources

- Resource Group: `rg-<name>-<location>`
- Virtual Network: `vnet-<name>-<location>`
- Subnet: `subnet-<name>-<location>`
- API Management: `apim-<name>-<location>`
- Gateway: `gateway-<name>-<location>`

### ApiClaim Resources

- API: `api-<apiPath>-<ingressRef>`
- Gateway API: `gateway-api-<apiPath>-<ingressRef>`

## Environment-Specific Configurations

### Development Environment

```yaml
apiVersion: azure.platform.my-org.co/v1alpha1
kind: XIngress
metadata:
  name: dev-gateway
spec:
  name: dev-gateway
  location: UK
  apiManagement:
    publisherEmail: dev@my-org.co
    publisherName: Dev Team
    skuName: Developer_1
  networking:
    vnetAddressSpace:
      - 10.0.0.0/24
    subnetPrefix:
      - 10.0.0.0/26
```

### Production Environment

```yaml
apiVersion: azure.platform.my-org.co/v1alpha1
kind: XIngress
metadata:
  name: prod-gateway
spec:
  name: prod-gateway
  location: UK
  apiManagement:
    publisherEmail: ops@my-org.co
    publisherName: Operations Team
    skuName: Standard_1
  networking:
    vnetAddressSpace:
      - 10.0.0.0/16
    subnetPrefix:
      - 10.0.1.0/24
      - 10.0.2.0/24
```

## Configuration Validation

### Required Fields Validation

The platform validates that all required fields are present:

- XIngress: `name`, `location`
- ApiClaim: `ingressRef`, `apiName`, `apiPath`, `swaggerDefinition`, `backendUrl`

### Format Validation

- Email addresses must be valid
- URLs must be valid HTTP/HTTPS URLs
- Network addresses must be valid CIDR notation
- API paths must not start with '/'

### Business Logic Validation

- XIngress names must be unique within the cluster
- ApiClaim `ingressRef` must reference an existing XIngress
- API paths must be unique within the same gateway

## Advanced Configuration

### Custom Resource Labels

```yaml
metadata:
  name: my-gateway
  labels:
    environment: production
    team: platform
    cost-center: engineering
```

### Resource Annotations

```yaml
metadata:
  name: my-gateway
  annotations:
    platform.my-org.co/contact: "devops@my-org.co"
    platform.my-org.co/documentation: "https://docs.my-org.co/gateway"
```

### Composition Selection

```yaml
spec:
  compositionRef:
    name: xingress-composition-v2
```

### Composition Revision

```yaml
spec:
  compositionRevisionRef:
    name: xingress-composition-abc123
```

## Best Practices

### Naming Conventions

- Use descriptive names for resources
- Include environment in resource names
- Use consistent naming patterns across resources

### Network Planning

- Plan IP address ranges carefully
- Avoid overlapping address spaces
- Leave room for future growth

### Security

- Use HTTPS for all API communications
- Implement proper authentication and authorization
- Monitor API usage and access patterns

### Cost Optimization

- Choose appropriate SKUs for your workload
- Share XIngress resources across teams
- Monitor and optimize resource usage

## Common Configuration Patterns

### Multi-Environment Setup

```yaml
# Development
apiVersion: azure.platform.my-org.co/v1alpha1
kind: XIngress
metadata:
  name: dev-gateway
spec:
  name: dev-gateway
  location: UK
  apiManagement:
    skuName: Developer_1

---
# Production
apiVersion: azure.platform.my-org.co/v1alpha1
kind: XIngress
metadata:
  name: prod-gateway
spec:
  name: prod-gateway
  location: UK
  apiManagement:
    skuName: Standard_1
```

### Regional Deployment

```yaml
# UK Region
apiVersion: azure.platform.my-org.co/v1alpha1
kind: XIngress
metadata:
  name: uk-gateway
spec:
  name: uk-gateway
  location: UK

---
# US Region
apiVersion: azure.platform.my-org.co/v1alpha1
kind: XIngress
metadata:
  name: us-gateway
spec:
  name: us-gateway
  location: USEast
```

### API Versioning

```yaml
# API v1
apiVersion: platform.my-org.co/v1alpha1
kind: ApiClaim
metadata:
  name: my-api-v1
spec:
  ingressRef: my-gateway
  apiName: "My API v1"
  apiPath: "myapi/v1"
  apiVersion: "v1"

---
# API v2
apiVersion: platform.my-org.co/v1alpha1
kind: ApiClaim
metadata:
  name: my-api-v2
spec:
  ingressRef: my-gateway
  apiName: "My API v2"
  apiPath: "myapi/v2"
  apiVersion: "v2"
```
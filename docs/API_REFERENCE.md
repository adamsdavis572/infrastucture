# API Reference

## XIngress API

### Resource Definition

```yaml
apiVersion: apiextensions.crossplane.io/v1
kind: CompositeResourceDefinition
metadata:
  name: xingresses.azure.platform.my-org.co
spec:
  group: azure.platform.my-org.co
  names:
    kind: XIngress
    plural: xingresses
  versions:
  - name: v1alpha1
    served: true
    referenceable: true
```

### XIngress Specification

#### Schema

```yaml
apiVersion: azure.platform.my-org.co/v1alpha1
kind: XIngress
metadata:
  name: string
spec:
  name: string                    # Required
  location: string               # Required
  apiManagement:
    publisherEmail: string       # Optional
    publisherName: string        # Optional
    skuName: string             # Optional
  networking:
    vnetAddressSpace: []string   # Optional
    subnetPrefix: []string       # Optional
```

#### Field Descriptions

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `metadata.name` | string | Yes | Kubernetes resource name |
| `spec.name` | string | Yes | Identifier for the XIngress instance |
| `spec.location` | string | Yes | Azure region (UK or USEast) |
| `spec.apiManagement.publisherEmail` | string | No | Email address of the API publisher |
| `spec.apiManagement.publisherName` | string | No | Name of the API publisher |
| `spec.apiManagement.skuName` | string | No | API Management service SKU |
| `spec.networking.vnetAddressSpace` | []string | No | Virtual network address spaces |
| `spec.networking.subnetPrefix` | []string | No | Subnet address prefixes |

#### Default Values

```yaml
spec:
  apiManagement:
    publisherEmail: "company@terraform.io"
    publisherName: "My Company"
    skuName: "Developer_1"
  networking:
    vnetAddressSpace: ["10.0.0.0/16"]
    subnetPrefix: ["10.0.1.0/24"]
```

#### Validation Rules

- `spec.name`: Must be a valid DNS label
- `spec.location`: Must be one of: "UK", "USEast"
- `spec.apiManagement.publisherEmail`: Must be a valid email address
- `spec.apiManagement.skuName`: Must be one of: "Consumption", "Developer_1", "Basic_1", "Standard_1", "Premium_1"
- `spec.networking.vnetAddressSpace`: Must be valid CIDR notation
- `spec.networking.subnetPrefix`: Must be valid CIDR notation

### XIngress Status

```yaml
status:
  conditions:
  - type: Ready
    status: "True"
    reason: Available
    message: "XIngress is ready"
  - type: Synced
    status: "True"
    reason: ReconcileSuccess
    message: "Successfully reconciled"
  connectionDetails:
    apiManagementEndpoint: "https://apim-my-gateway-uk.azure-api.net"
    gatewayEndpoint: "https://gateway-my-gateway-uk.azure-api.net"
```

## ApiClaim API

### Resource Definition

```yaml
apiVersion: apiextensions.crossplane.io/v1
kind: CompositeResourceDefinition
metadata:
  name: apiinstances.platform.my-org.co
spec:
  group: platform.my-org.co
  names:
    kind: ApiInstance
    plural: apiinstances
  claimNames:
    kind: ApiClaim
    plural: apiclaims
  versions:
  - name: v1alpha1
    served: true
    referenceable: true
```

### ApiClaim Specification

#### Schema

```yaml
apiVersion: platform.my-org.co/v1alpha1
kind: ApiClaim
metadata:
  name: string
spec:
  ingressRef: string             # Required
  apiName: string               # Required
  apiPath: string               # Required
  apiVersion: string            # Optional
  apiRevision: string           # Optional
  apiProtocols: []string        # Optional
  swaggerDefinition: string     # Required
  swaggerFormat: string         # Optional
  backendUrl: string            # Required
  serviceUrl: string            # Optional
```

#### Field Descriptions

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `metadata.name` | string | Yes | Kubernetes resource name |
| `spec.ingressRef` | string | Yes | Reference to XIngress resource |
| `spec.apiName` | string | Yes | Display name for the API |
| `spec.apiPath` | string | Yes | Base path for the API |
| `spec.apiVersion` | string | No | API version identifier |
| `spec.apiRevision` | string | No | API revision identifier |
| `spec.apiProtocols` | []string | No | Supported protocols |
| `spec.swaggerDefinition` | string | Yes | OpenAPI/Swagger definition |
| `spec.swaggerFormat` | string | No | Format of the API definition |
| `spec.backendUrl` | string | Yes | Backend service URL |
| `spec.serviceUrl` | string | No | Service URL (defaults to backendUrl) |

#### Default Values

```yaml
spec:
  apiVersion: "v1"
  apiRevision: "1"
  apiProtocols: ["https"]
  swaggerFormat: "openapi"
  serviceUrl: "<backendUrl>"
```

#### Validation Rules

- `spec.ingressRef`: Must reference an existing XIngress resource
- `spec.apiName`: Must be a non-empty string
- `spec.apiPath`: Must be a valid URL path segment (no leading slash)
- `spec.apiProtocols`: Must contain valid protocol values
- `spec.swaggerDefinition`: Must be valid JSON or YAML
- `spec.backendUrl`: Must be a valid HTTP/HTTPS URL
- `spec.serviceUrl`: Must be a valid HTTP/HTTPS URL

### ApiClaim Status

```yaml
status:
  conditions:
  - type: Ready
    status: "True"
    reason: Available
    message: "API is ready"
  - type: Synced
    status: "True"
    reason: ReconcileSuccess
    message: "Successfully reconciled"
  connectionDetails:
    apiEndpoint: "https://apim-my-gateway-uk.azure-api.net/myapi"
    gatewayApiEndpoint: "https://gateway-my-gateway-uk.azure-api.net/myapi"
```

## Managed Resource APIs

### ResourceGroup

```yaml
apiVersion: azure.upbound.io/v1beta1
kind: ResourceGroup
metadata:
  name: rg-my-gateway-uk
spec:
  forProvider:
    location: uksouth
    name: rg-my-gateway-uk
```

### VirtualNetwork

```yaml
apiVersion: network.azure.upbound.io/v1beta1
kind: VirtualNetwork
metadata:
  name: vnet-my-gateway-uk
spec:
  forProvider:
    addressSpace:
    - 10.0.0.0/16
    location: uksouth
    name: vnet-my-gateway-uk
    resourceGroupName: rg-my-gateway-uk
```

### Subnet

```yaml
apiVersion: network.azure.upbound.io/v1beta1
kind: Subnet
metadata:
  name: subnet-my-gateway-uk
spec:
  forProvider:
    addressPrefixes:
    - 10.0.1.0/24
    name: subnet-my-gateway-uk
    resourceGroupName: rg-my-gateway-uk
    virtualNetworkName: vnet-my-gateway-uk
```

### ManagementService

```yaml
apiVersion: apimanagement.azure.upbound.io/v1beta1
kind: ManagementService
metadata:
  name: apim-my-gateway-uk
spec:
  forProvider:
    location: uksouth
    name: apim-my-gateway-uk
    publisherEmail: company@terraform.io
    publisherName: My Company
    resourceGroupName: rg-my-gateway-uk
    skuName: Developer_1
```

### Gateway

```yaml
apiVersion: apimanagement.azure.upbound.io/v1beta1
kind: Gateway
metadata:
  name: gateway-my-gateway-uk
spec:
  forProvider:
    apiManagementId: /subscriptions/.../apim-my-gateway-uk
    locationData:
    - name: my-gateway
      region: uksouth
    name: gateway-my-gateway-uk
```

### API

```yaml
apiVersion: apimanagement.azure.upbound.io/v1beta1
kind: API
metadata:
  name: api-myapi-my-gateway
spec:
  forProvider:
    apiManagementName: apim-my-gateway-uk
    displayName: My API
    name: api-myapi-my-gateway
    path: myapi
    protocols:
    - https
    resourceGroupName: rg-my-gateway-uk
    revision: "1"
    serviceUrl: https://my-backend.azurewebsites.net/api
    import:
    - contentFormat: openapi
      contentValue: |
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

### GatewayAPI

```yaml
apiVersion: apimanagement.azure.upbound.io/v1beta1
kind: GatewayAPI
metadata:
  name: gateway-api-myapi-my-gateway
spec:
  forProvider:
    apiId: /subscriptions/.../apis/api-myapi-my-gateway
    gatewayId: /subscriptions/.../gateways/gateway-my-gateway-uk
```

## Status Conditions

### Condition Types

| Type | Description |
|------|-------------|
| `Ready` | Resource is ready for use |
| `Synced` | Resource is synchronized with external system |
| `Creating` | Resource is being created |
| `Deleting` | Resource is being deleted |
| `Failed` | Resource operation failed |

### Condition Status

| Status | Description |
|--------|-------------|
| `True` | Condition is true |
| `False` | Condition is false |
| `Unknown` | Condition status is unknown |

### Common Reasons

| Reason | Description |
|--------|-------------|
| `Available` | Resource is available and ready |
| `ReconcileSuccess` | Resource reconciliation succeeded |
| `ReconcileError` | Resource reconciliation failed |
| `Creating` | Resource is being created |
| `Deleting` | Resource is being deleted |

## Connection Details

### XIngress Connection Details

| Key | Description |
|-----|-------------|
| `apiManagementEndpoint` | API Management service endpoint |
| `gatewayEndpoint` | Gateway endpoint URL |
| `resourceGroupName` | Azure resource group name |
| `location` | Azure region |

### ApiClaim Connection Details

| Key | Description |
|-----|-------------|
| `apiEndpoint` | API endpoint URL |
| `gatewayApiEndpoint` | Gateway API endpoint URL |
| `apiId` | Azure API resource ID |
| `gatewayApiId` | Gateway API association ID |

## Error Codes

### Common Error Codes

| Code | Description | Solution |
|------|-------------|----------|
| `InvalidSpec` | Resource specification is invalid | Check field validation rules |
| `ResourceNotFound` | Referenced resource not found | Verify resource references |
| `AuthenticationFailed` | Azure authentication failed | Check service principal credentials |
| `InsufficientPermissions` | Insufficient Azure permissions | Verify role assignments |
| `ResourceConflict` | Resource name conflict | Use different resource name |
| `QuotaExceeded` | Azure quota exceeded | Request quota increase |

### Validation Errors

| Error | Description | Solution |
|-------|-------------|----------|
| `InvalidEmail` | Invalid email format | Use valid email address |
| `InvalidURL` | Invalid URL format | Use valid HTTP/HTTPS URL |
| `InvalidCIDR` | Invalid CIDR notation | Use valid network address |
| `InvalidProtocol` | Invalid protocol | Use supported protocol |
| `InvalidSKU` | Invalid SKU name | Use supported SKU |

## API Examples

### Minimal XIngress

```yaml
apiVersion: azure.platform.my-org.co/v1alpha1
kind: XIngress
metadata:
  name: minimal-gateway
spec:
  name: minimal-gateway
  location: UK
```

### Complete XIngress

```yaml
apiVersion: azure.platform.my-org.co/v1alpha1
kind: XIngress
metadata:
  name: complete-gateway
  labels:
    environment: production
    team: platform
spec:
  name: complete-gateway
  location: UK
  apiManagement:
    publisherEmail: platform@company.com
    publisherName: Platform Team
    skuName: Standard_1
  networking:
    vnetAddressSpace:
      - 10.0.0.0/16
    subnetPrefix:
      - 10.0.1.0/24
      - 10.0.2.0/24
```

### Minimal ApiClaim

```yaml
apiVersion: platform.my-org.co/v1alpha1
kind: ApiClaim
metadata:
  name: minimal-api
spec:
  ingressRef: minimal-gateway
  apiName: "Minimal API"
  apiPath: "minimal"
  backendUrl: "https://backend.example.com"
  swaggerDefinition: |
    openapi: 3.0.0
    info:
      title: Minimal API
      version: 1.0.0
    paths:
      /health:
        get:
          responses:
            '200':
              description: OK
```

### Complete ApiClaim

```yaml
apiVersion: platform.my-org.co/v1alpha1
kind: ApiClaim
metadata:
  name: complete-api
  labels:
    service: user-service
    version: v2
spec:
  ingressRef: complete-gateway
  apiName: "Complete API v2"
  apiPath: "complete/v2"
  apiVersion: "v2"
  apiRevision: "1"
  apiProtocols:
    - https
    - wss
  swaggerDefinition: |
    openapi: 3.0.0
    info:
      title: Complete API
      version: 2.0.0
      description: Complete API with all features
    paths:
      /users:
        get:
          summary: List users
          responses:
            '200':
              description: List of users
        post:
          summary: Create user
          responses:
            '201':
              description: User created
      /users/{id}:
        get:
          summary: Get user
          parameters:
          - name: id
            in: path
            required: true
            schema:
              type: integer
          responses:
            '200':
              description: User details
            '404':
              description: User not found
  swaggerFormat: openapi
  backendUrl: "https://backend.example.com/api/v2"
  serviceUrl: "https://backend.example.com/api/v2"
```
# Crossplane Azure Infrastructure Platform

A Crossplane-based infrastructure platform for managing Azure API Management gateways and APIs through composable resources.

## Overview

This project provides a set of Crossplane Composite Resource Definitions (XRDs) and Compositions that enable platform teams to provision and manage Azure API Management infrastructure declaratively. The platform abstracts complex Azure resources into simple, reusable components.

## Architecture

### Core Components

1. **XIngress** - Manages the shared API gateway infrastructure
2. **ApiInstance/ApiClaim** - Manages individual APIs within the gateway
3. **Crossplane Providers** - Azure resource providers for various services

### Resource Hierarchy

```
XIngress (Shared Gateway)
├── Azure Resource Group
├── Virtual Network
├── Subnet
├── API Management Service
└── API Management Gateway
    └── ApiClaim (Individual APIs)
        ├── API
        └── Gateway API Association
```

## Requirements

### Infrastructure Requirements

1. **Kubernetes Cluster** with Crossplane installed
2. **Azure Subscription** with appropriate permissions
3. **Azure Service Principal** with the custom role defined in [`crossplane-role-definition.json`](crossplane-role-definition.json)

### Azure Permissions

The platform requires a custom Azure role with the following permissions:
- Resource group management
- Virtual network operations
- API Management service operations
- CDN profile management
- DNS zone management
- Role assignment management

See [`crossplane-role-definition.json`](crossplane-role-definition.json) for the complete role definition.

### Crossplane Providers

The following providers must be installed:
- [`provider-azure-apimanagement`](crossplane-provider/provider-azure-apimanagement.yaml)
- [`provider-azure-network`](crossplane-provider/provider-azure-network.yaml)
- [`provider-azure-storage`](crossplane-provider/provider-azure-storage.yaml)
- [`provider-azure-authorization`](crossplane-provider/provider-azure-authorization.yaml)
- [`provider-azure-cdn`](crossplane-provider/provider-azure-cdn.yaml)

## Installation

### 1. Install Crossplane Providers

Apply all provider configurations:

```bash
kubectl apply -f crossplane-provider/
```

### 2. Configure Azure Credentials

1. Create an Azure Service Principal with the custom role
2. Create a Kubernetes secret with the credentials:

```bash
kubectl create secret generic azure-secret \
  --from-literal=creds='{"clientId":"<client-id>","clientSecret":"<client-secret>","subscriptionId":"<subscription-id>","tenantId":"<tenant-id>"}' \
  --namespace crossplane-system
```

3. Apply the provider configuration:

```bash
kubectl apply -f crossplane-provider/providerconfig-azure.yaml
```

### 3. Deploy Composite Resource Definitions

Deploy the XRDs and Compositions:

```bash
kubectl apply -f compositions/
```

## Usage

### Creating a Shared API Gateway (XIngress)

Use the [`XIngress`](compositions/xingress/xingress-definition.yaml) resource to create shared API Management infrastructure:

```yaml
apiVersion: azure.platform.my-org.co/v1alpha1
kind: XIngress
metadata:
  name: my-gateway
spec:
  name: my-gateway
  location: UK  # or USEast
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

See [`example-xingress.yaml`](components/example-xingress.yaml) for a complete example.

### Adding APIs to the Gateway

Use the [`ApiClaim`](compositions/xapi/xapi-definition.yaml) resource to add APIs to an existing gateway:

```yaml
apiVersion: platform.my-org.co/v1alpha1
kind: ApiClaim
metadata:
  name: my-api
spec:
  ingressRef: my-gateway  # Reference to XIngress
  apiName: "My API"
  apiPath: "myapi"
  apiProtocols: 
    - "https"
  swaggerDefinition: |
    openapi: 3.0.0
    info:
      title: My API
      version: 1.0.0
    # ... OpenAPI definition
  backendUrl: "https://my-backend.azurewebsites.net/api"
```

See [`example-api-claim.yaml`](components/example-api-claim.yaml) for a complete example.

## Configuration Options

### XIngress Configuration

| Field | Description | Required | Default |
|-------|-------------|----------|---------|
| `name` | Identifier for the XIngress instance | Yes | - |
| `location` | Azure region (UK or USEast) | Yes | - |
| `apiManagement.publisherEmail` | API Management publisher email | No | "company@terraform.io" |
| `apiManagement.publisherName` | API Management publisher name | No | "My Company" |
| `apiManagement.skuName` | API Management SKU | No | "Developer_1" |
| `networking.vnetAddressSpace` | Virtual network address space | No | ["10.0.0.0/16"] |
| `networking.subnetPrefix` | Subnet address prefix | No | ["10.0.1.0/24"] |

### ApiClaim Configuration

| Field | Description | Required | Default |
|-------|-------------|----------|---------|
| `ingressRef` | Reference to XIngress resource | Yes | - |
| `apiName` | Display name for the API | Yes | - |
| `apiPath` | Base path for the API | Yes | - |
| `apiProtocols` | Supported protocols | No | ["https"] |
| `swaggerDefinition` | OpenAPI/Swagger definition | Yes | - |
| `swaggerFormat` | Format of the definition | No | "openapi" |
| `backendUrl` | Backend service URL | Yes | - |

## Supported Azure Regions

The platform supports the following regions:
- `UK` → Maps to "UK South"
- `USEast` → Maps to "East US"

## API Management SKUs

Supported SKUs:
- `Consumption`
- `Developer_1`
- `Basic_1`
- `Standard_1`
- `Premium_1`

## Monitoring and Troubleshooting

### Check Resource Status

```bash
# Check XIngress status
kubectl get xingress

# Check ApiClaim status
kubectl get apiclaim

# Check underlying Azure resources
kubectl get managed
```

### Common Issues

1. **Provider not ready**: Ensure all providers are installed and healthy
2. **Azure permissions**: Verify the service principal has the required permissions
3. **Resource naming conflicts**: Check for existing resources with the same names

## File Structure

```
.
├── crossplane-role-definition.json     # Azure RBAC role definition
├── crossplane-provider/                # Crossplane provider configurations
│   ├── provider-azure-*.yaml          # Azure provider definitions
│   ├── providerconfig-azure.yaml      # Azure provider configuration
│   └── function-patch-and-transform.yaml
├── compositions/                       # Crossplane compositions
│   ├── xingress/                      # Shared gateway composition
│   │   ├── xingress-definition.yaml   # XIngress XRD
│   │   └── xingress-composition.yaml  # XIngress Composition
│   └── xapi/                          # API composition
│       ├── xapi-definition.yaml       # ApiInstance XRD
│       └── xapi-composition.yaml      # ApiInstance Composition
└── components/                         # Example resources
    ├── example-xingress.yaml          # Example XIngress
    ├── example-api-claim.yaml         # Example ApiClaim
    └── api-claim-README.md            # ApiClaim usage guide
```

## Contributing

1. Follow the existing naming conventions
2. Test changes in a development environment
3. Update documentation for any new features
4. Ensure all required fields are properly validated

## Security Considerations

- Use least-privilege access for Azure service principals
- Regularly rotate service principal credentials
- Monitor resource usage and costs
- Implement proper network security groups for the virtual networks

## Next Steps

Consider implementing:
- CDN integration for global API distribution
- WAF policies for API security
- Monitoring and alerting integration
- Custom policies for API governance
- Multi-region deployment support
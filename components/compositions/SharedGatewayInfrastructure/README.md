# Crossplane Shared API Gateway Platform on Azure

This project contains the Crossplane configurations to provision a sophisticated, shared API Gateway platform on Azure. It creates a foundational infrastructure and provides a self-service mechanism for application teams to deploy and manage their APIs.

## Overview

The platform is built on a two-stage model:

1.  **Shared Infrastructure (`SharedGatewayInfrastructure`):** A cluster-scoped resource that provisions the core, one-time components:
    * Azure API Management (APIM) in VNet "Internal" mode
    * Azure Front Door Premium
    * Azure WAF Policy with OWASP rules
    * A dedicated DMZ Virtual Network

2.  **Team "Landing Zone" (`APITeamWorkspaceClaim`):** A namespaced claim for application teams to request their own secure "landing zone". This provisions:
    * A dedicated APIM `Product`.
    * A custom DNS name and route on the shared Front Door.
    * Azure RBAC permissions for the team to manage APIs within their workspace.

3.  **API Deployment (`APIClaim`):** A secondary claim for teams to declaratively deploy their individual APIs (from an OpenAPI spec) and policies into their landing zone.

##  Prerequisites

* **Azure Subscription:** An active Azure subscription.
* **Tools:**
    * `kubectl` (v1.25+)
    * `helm` (v3+)
    * `az` (Azure CLI)

---

## 🚀 Installation and Setup Guide

Follow these steps to get the platform running.

### Step 1: Install Crossplane

First, install the core Crossplane controller into your Kubernetes cluster using Helm.

```bash
# Add the Crossplane Helm repository
helm repo add crossplane-stable https://charts.crossplane.io/stable
helm repo update

# Install Crossplane into its own namespace
helm install crossplane --namespace crossplane-system --create-namespace crossplane-stable/crossplane
```

### Step 2: Install Azure Provider

Apply the following manifest to install the official Upbound provider for Azure.

```yaml
# FILE: provider-azure.yaml
apiVersion: pkg.crossplane.io/v1
kind: Provider
metadata:
  name: provider-family-azure
spec:
  package: "xpkg.upbound.io/upbound/provider-family-azure:v1.13.0" # Or a more recent version
```

```bash
kubectl apply -f provider-azure.yaml
```

### Step 3: Prepare Azure Credentials (Least-Privilege Role)

We will create a custom role in Azure with the exact permissions Crossplane needs, then create a Service Principal and assign it this role.

**1. Define the Custom Role**

Save the following JSON content as `crossplane-role-definition.json`. Remember to replace `YOUR_SUBSCRIPTION_ID` with your actual subscription ID.

```json
{
  "Name": "Crossplane Shared Gateway Manager",
  "Description": "Grants permissions to manage the shared API gateway infrastructure via Crossplane.",
  "IsCustom": true,
  "AssignableScopes": [
    "/subscriptions/YOUR_SUBSCRIPTION_ID"
  ],
  "Actions": [
    "Microsoft.Resources/subscriptions/resourcegroups/read",
    "Microsoft.Resources/subscriptions/resourcegroups/write",
    "Microsoft.Resources/subscriptions/resourcegroups/delete",
    "Microsoft.Network/virtualNetworks/*",
    "Microsoft.ApiManagement/service/*",
    "Microsoft.Cdn/profiles/*",
    "Microsoft.Network/frontDoorWebApplicationFirewallPolicies/*",
    "Microsoft.Network/privateEndpoints/write",
    "Microsoft.Network/privateEndpoints/read",
    "Microsoft.Dns/dnszones/CNAME/write",
    "Microsoft.Dns/dnszones/A/write",
    "Microsoft.Dns/dnszones/read",
    "Microsoft.Authorization/roleAssignments/write",
    "Microsoft.Authorization/roleAssignments/delete",
    "Microsoft.Authorization/roleAssignments/read"
  ],
  "NotActions": [],
  "DataActions": [],
  "NotDataActions": []
}
```

**2. Create the Role and Service Principal**

Now, run the following `az` commands:

```bash
# Set your subscription ID
export AZURE_SUBSCRIPTION_ID=$(az account show --query id -o tsv)

# Update the placeholder in the JSON file
sed -i.bak "s|YOUR_SUBSCRIPTION_ID|$AZURE_SUBSCRIPTION_ID|g" crossplane-role-definition.json && rm crossplane-role-definition.json.bak

# Create the custom role in Azure
az role definition create --role-definition @crossplane-role-definition.json

# Create the Service Principal and assign the new custom role
az ad sp create-for-rbac --name "crossplane-gateway-sp" --role "Crossplane Shared Gateway Manager" --scopes "/subscriptions/$AZURE_SUBSCRIPTION_ID" --sdk-auth > azure-credentials.json
```

**3. Create the Kubernetes Secret**

Create the secret in the `crossplane-system` namespace to store these credentials.

```bash
kubectl create secret generic azure-creds -n crossplane-system --from-file=creds=./azure-credentials.json
```

### Step 4: Configure the Crossplane Provider

Apply the following `ProviderConfig` manifest. It tells Crossplane to use the secret you just created.

```yaml
# FILE: providerconfig-azure.yaml
apiVersion: azure.upbound.io/v1beta1
kind: ProviderConfig
metadata:
  name: default # This is the name referenced in the Compositions
spec:
  credentials:
    source: Secret
    secretRef:
      namespace: crossplane-system
      name: azure-creds
      key: creds
```

```bash
kubectl apply -f providerconfig-azure.yaml
```

### Step 5: Prepare Azure Resources

* **DNS Zone:** Ensure the public Azure DNS Zone you plan to use (e.g., `teamapis.yourcompany.com`) **already exists**. The Service Principal will automatically get permissions to manage records in it if it's in the same subscription.

### Step 6: Apply Platform Configurations

Apply all the XRDs and Compositions you have created to your cluster.

```bash
# Apply all your configuration files
kubectl apply -f compositeresourcedefinition-sharedinfra.yaml
kubectl apply -f composition-sharedinfra-azure.yaml
kubectl apply -f compositeresourcedefinition-teamworkspace.yaml
kubectl apply -f composition-teamworkspace-azure.yaml
kubectl apply -f compositeresourcedefinition-api.yaml
kubectl apply -f composition-api-azure.yaml
```

---

## 🛠️ Usage

### 1. Provision the Shared Infrastructure

A platform administrator creates an instance of `SharedGatewayInfrastructure`.

```yaml
# FILE: my-shared-gateway.yaml
apiVersion: platform.yourcompany.com/v1alpha1
kind: SharedGatewayInfrastructure
metadata:
  name: prod-gateway
spec:
  parameters:
    sharedInfraInstanceName: "prod-gateway"
    location: "UK South"
    apimInstanceName: "myorg-prod-apim"
    frontDoorProfileName: "myorg-prod-fd"
    apimSkuName: "Premium"
    apimPublisherName: "My Platform Team"
    apimPublisherEmail: "platform@myorg.com"
    dmzVnetAddressPrefix: "10.200.0.0/16"
    publicDnsZoneName: "teamapis.yourcompany.com"
```

```bash
kubectl apply -f my-shared-gateway.yaml
```

### 2. Team Claims a Workspace ("Landing Zone")

An application team creates an `APITeamWorkspaceClaim` in their own namespace.

```yaml
# FILE: my-app-workspace-claim.yaml
apiVersion: platform.yourcompany.com/v1alpha1
kind: APITeamWorkspaceClaim
metadata:
  name: payments-app-workspace
  namespace: payments-team
spec:
  parameters:
    teamIdentifier: "payments-app"
    customDnsHostnamePrefix: "payments"
    apimProductName: "Payments API Product"
    teamAadGroupId: "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" # Object ID of the team's AAD Group
    sharedInfraXRName: "prod-gateway"
```

```bash
kubectl apply -f my-app-workspace-claim.yaml
```

### 3. Team Deploys an API

The application team can now deploy their API into their workspace using an `APIClaim`.

```yaml
# FILE: my-app-api-claim.yaml
apiVersion: platform.yourcompany.com/v1alpha1
kind: APIClaim
metadata:
  name: orders-api
  namespace: payments-team
spec:
  parameters:
    targetProductName: "Payments API Product"
    apiName: "payments-orders-api-v1"
    apimBasePath: "/orders/v1"
    openapiSpec:
      url: "https://raw.githubusercontent.com/my-app/specs/main/orders-v1.yaml"
    backendServiceUrl: "https://my-payments-app.azurewebsites.net"
    sharedInfraXRName: "prod-gateway"
```

```bash
kubectl apply -f my-app-api-claim.yaml
```
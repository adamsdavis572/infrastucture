# Installation Guide

## Prerequisites

Before installing the Crossplane Azure Infrastructure Platform, ensure you have:

- Kubernetes cluster (v1.24+)
- `kubectl` configured to access your cluster
- Azure CLI installed and authenticated
- Crossplane installed on your cluster

## Step 1: Install Crossplane (if not already installed)

```bash
# Install Crossplane using Helm
helm repo add crossplane-stable https://charts.crossplane.io/stable
helm repo update
helm install crossplane crossplane-stable/crossplane \
  --namespace crossplane-system \
  --create-namespace
```

Wait for Crossplane to be ready:

```bash
kubectl wait --for=condition=ready pod -l app=crossplane --namespace crossplane-system
```

## Step 2: Create Azure Service Principal

### Create the Custom Role

```bash
# Create the custom role definition
az role definition create --role-definition crossplane-role-definition.json
```

### Create Service Principal

```bash
# Create service principal
az ad sp create-for-rbac --name crossplane-sp --role "Crossplane Azure Provider" --scopes /subscriptions/{subscription-id}

# Note the output - you'll need these values:
# - appId (clientId)
# - password (clientSecret)
# - tenant (tenantId)
```

## Step 3: Configure Azure Credentials

Create a Kubernetes secret with your Azure credentials:

```bash
kubectl create secret generic azure-secret \
  --from-literal=creds='{"clientId":"<app-id>","clientSecret":"<password>","subscriptionId":"<subscription-id>","tenantId":"<tenant-id>"}' \
  --namespace crossplane-system
```

## Step 4: Install Crossplane Providers

Install all required providers:

```bash
# Install all providers
kubectl apply -f crossplane-provider/

# Wait for providers to be ready
kubectl wait --for=condition=healthy provider.pkg.crossplane.io/provider-azure-apimanagement --timeout=300s
kubectl wait --for=condition=healthy provider.pkg.crossplane.io/provider-azure-network --timeout=300s
kubectl wait --for=condition=healthy provider.pkg.crossplane.io/provider-azure-storage --timeout=300s
kubectl wait --for=condition=healthy provider.pkg.crossplane.io/provider-azure-authorization --timeout=300s
kubectl wait --for=condition=healthy provider.pkg.crossplane.io/provider-azure-cdn --timeout=300s
```

## Step 5: Configure Provider Credentials

Apply the provider configuration:

```bash
kubectl apply -f crossplane-provider/providerconfig-azure.yaml
```

Verify the configuration:

```bash
kubectl get providerconfig
```

## Step 6: Deploy Composite Resource Definitions

Deploy the XRDs and Compositions:

```bash
# Deploy XRDs and Compositions
kubectl apply -f compositions/

# Verify XRDs are established
kubectl get xrd
```

## Step 7: Verify Installation

Check that all components are ready:

```bash
# Check providers
kubectl get providers

# Check XRDs
kubectl get xrd

# Check compositions
kubectl get composition
```

## Step 8: Test with Example Resources

Deploy the example resources to test the installation:

```bash
# Create an example XIngress
kubectl apply -f components/example-xingress.yaml

# Check status
kubectl get xingress -o wide

# Create an example API
kubectl apply -f components/example-api-claim.yaml

# Check status
kubectl get apiclaim -o wide
```

## Troubleshooting Installation

### Provider Issues

If providers fail to install:

```bash
# Check provider logs
kubectl logs -n crossplane-system -l pkg.crossplane.io/provider=provider-azure-apimanagement

# Check provider status
kubectl describe provider provider-azure-apimanagement
```

### Credential Issues

If authentication fails:

```bash
# Verify secret exists
kubectl get secret azure-secret -n crossplane-system

# Check provider config
kubectl describe providerconfig default
```

### Permission Issues

If resources fail to create due to permissions:

```bash
# Verify Azure role assignment
az role assignment list --assignee <service-principal-id>

# Check Azure permissions
az role definition show --name "Crossplane Azure Provider"
```

## Post-Installation Configuration

### Resource Quotas

Consider setting resource quotas for the crossplane-system namespace:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: crossplane-quota
  namespace: crossplane-system
spec:
  hard:
    requests.cpu: "2"
    requests.memory: 4Gi
    limits.cpu: "4"
    limits.memory: 8Gi
```

### Monitoring

Set up monitoring for Crossplane:

```bash
# Enable metrics
kubectl patch deployment crossplane -n crossplane-system -p '{"spec":{"template":{"spec":{"containers":[{"name":"crossplane","args":["--enable-metrics"]}]}}}}'
```

### Backup

Consider backing up your Crossplane configuration:

```bash
# Backup XRDs and Compositions
kubectl get xrd,composition -o yaml > crossplane-backup.yaml
```

## Next Steps

1. Review the [Configuration Reference](CONFIGURATION.md)
2. Check the [API Reference](API_REFERENCE.md)
3. Explore the example resources in the `components/` directory
4. Set up monitoring and alerting
5. Configure backup procedures

## Uninstallation

To remove the platform:

```bash
# Remove example resources
kubectl delete -f components/

# Remove compositions and XRDs
kubectl delete -f compositions/

# Remove providers
kubectl delete -f crossplane-provider/

# Remove Crossplane (optional)
helm uninstall crossplane -n crossplane-system
```
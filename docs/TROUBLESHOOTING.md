# Troubleshooting Guide

## Common Issues and Solutions

### Provider Issues

#### Provider Not Installing

**Problem**: Crossplane providers fail to install or become healthy

**Symptoms**:
```bash
kubectl get providers
NAME                               INSTALLED   HEALTHY   PACKAGE
provider-azure-apimanagement      True        False     ...
```

**Solutions**:

1. Check provider logs:
```bash
kubectl logs -n crossplane-system -l pkg.crossplane.io/provider=provider-azure-apimanagement
```

2. Verify Crossplane is healthy:
```bash
kubectl get pods -n crossplane-system
```

3. Check for resource constraints:
```bash
kubectl describe pod -n crossplane-system -l pkg.crossplane.io/provider=provider-azure-apimanagement
```

4. Restart the provider:
```bash
kubectl delete pod -n crossplane-system -l pkg.crossplane.io/provider=provider-azure-apimanagement
```

#### Provider Configuration Issues

**Problem**: ProviderConfig fails to authenticate with Azure

**Symptoms**:
```bash
kubectl describe providerconfig default
# Shows authentication errors
```

**Solutions**:

1. Verify Azure credentials secret:
```bash
kubectl get secret azure-secret -n crossplane-system -o yaml
```

2. Check service principal permissions:
```bash
az role assignment list --assignee <service-principal-id>
```

3. Test Azure CLI authentication:
```bash
az login --service-principal -u <client-id> -p <client-secret> --tenant <tenant-id>
az account show
```

4. Recreate the secret:
```bash
kubectl delete secret azure-secret -n crossplane-system
kubectl create secret generic azure-secret \
  --from-literal=creds='{"clientId":"<client-id>","clientSecret":"<client-secret>","subscriptionId":"<subscription-id>","tenantId":"<tenant-id>"}' \
  --namespace crossplane-system
```

### XIngress Issues

#### XIngress Not Ready

**Problem**: XIngress resource stays in "Not Ready" state

**Symptoms**:
```bash
kubectl get xingress
NAME         READY   SYNCED   AGE
my-gateway   False   False    5m
```

**Solutions**:

1. Check XIngress status:
```bash
kubectl describe xingress my-gateway
```

2. Check underlying managed resources:
```bash
kubectl get managed | grep my-gateway
```

3. Check resource group creation:
```bash
kubectl describe resourcegroup
```

4. Verify Azure permissions for resource group creation:
```bash
az provider show --namespace Microsoft.Resources
```

#### Resource Naming Conflicts

**Problem**: Azure resources fail to create due to naming conflicts

**Symptoms**:
```
Error: Resource with name 'apim-my-gateway-uk' already exists
```

**Solutions**:

1. Check existing Azure resources:
```bash
az resource list --resource-group rg-my-gateway-uk
```

2. Use a different name for the XIngress:
```yaml
spec:
  name: my-gateway-v2
```

3. Clean up existing resources if safe to do so:
```bash
az group delete --name rg-my-gateway-uk
```

### ApiClaim Issues

#### ApiClaim Not Creating API

**Problem**: ApiClaim resource doesn't create the API in API Management

**Symptoms**:
```bash
kubectl get apiclaim
NAME     READY   SYNCED   AGE
my-api   False   False    3m
```

**Solutions**:

1. Check ApiClaim status:
```bash
kubectl describe apiclaim my-api
```

2. Verify the referenced XIngress exists and is ready:
```bash
kubectl get xingress my-gateway
```

3. Check API Management service status:
```bash
kubectl get managementservice
```

4. Verify the Swagger definition is valid:
```bash
# Extract and validate the OpenAPI definition
kubectl get apiclaim my-api -o jsonpath='{.spec.swaggerDefinition}' | jq .
```

#### Invalid Swagger Definition

**Problem**: API fails to create due to invalid OpenAPI/Swagger definition

**Symptoms**:
```
Error: Invalid OpenAPI definition
```

**Solutions**:

1. Validate the OpenAPI definition:
```bash
# Use online validator or swagger-cli
npx swagger-cli validate swagger.yaml
```

2. Check common issues:
   - Missing `info` section
   - Invalid paths format
   - Missing required fields

3. Use a minimal valid definition:
```yaml
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

### Network Issues

#### Virtual Network Creation Fails

**Problem**: Virtual network or subnet creation fails

**Symptoms**:
```
Error: Address space conflicts with existing network
```

**Solutions**:

1. Check existing virtual networks:
```bash
az network vnet list --resource-group rg-my-gateway-uk
```

2. Use different address spaces:
```yaml
networking:
  vnetAddressSpace:
    - 10.1.0.0/16
  subnetPrefix:
    - 10.1.1.0/24
```

3. Verify network service provider registration:
```bash
az provider show --namespace Microsoft.Network
```

### Permission Issues

#### Insufficient Azure Permissions

**Problem**: Service principal lacks required permissions

**Symptoms**:
```
Error: The client does not have authorization to perform action 'Microsoft.ApiManagement/service/write'
```

**Solutions**:

1. Check current role assignments:
```bash
az role assignment list --assignee <service-principal-id>
```

2. Verify custom role exists:
```bash
az role definition show --name "Crossplane Azure Provider"
```

3. Recreate the custom role:
```bash
az role definition create --role-definition crossplane-role-definition.json
```

4. Assign the role to the service principal:
```bash
az role assignment create --assignee <service-principal-id> --role "Crossplane Azure Provider" --scope /subscriptions/<subscription-id>
```

#### RBAC Issues in Kubernetes

**Problem**: Crossplane lacks permissions in Kubernetes

**Symptoms**:
```
Error: User "system:serviceaccount:crossplane-system:crossplane" cannot create resource "managementservices"
```

**Solutions**:

1. Check Crossplane RBAC:
```bash
kubectl get clusterrolebinding | grep crossplane
```

2. Verify provider RBAC:
```bash
kubectl get clusterrole | grep provider-azure
```

3. Reinstall providers if RBAC is broken:
```bash
kubectl delete -f crossplane-provider/
kubectl apply -f crossplane-provider/
```

### Resource Quota Issues

#### Resource Limits Exceeded

**Problem**: Kubernetes resource limits prevent pod creation

**Symptoms**:
```
Error: exceeded quota: compute-resources
```

**Solutions**:

1. Check namespace resource quota:
```bash
kubectl describe quota -n crossplane-system
```

2. Check node resources:
```bash
kubectl describe nodes
```

3. Increase resource limits or add nodes:
```bash
kubectl patch resourcequota compute-resources -p '{"spec":{"hard":{"limits.cpu":"4","limits.memory":"8Gi"}}}'
```

### API Management Issues

#### API Management Service Not Responding

**Problem**: API Management service becomes unresponsive

**Symptoms**:
- APIs return 503 errors
- Management portal is inaccessible

**Solutions**:

1. Check API Management service status in Azure:
```bash
az apim show --name apim-my-gateway-uk --resource-group rg-my-gateway-uk
```

2. Check service health:
```bash
az apim check-name-availability --name apim-my-gateway-uk
```

3. Restart the service (if necessary):
```bash
az apim restart --name apim-my-gateway-uk --resource-group rg-my-gateway-uk
```

#### SKU Limitations

**Problem**: API Management SKU limits are exceeded

**Symptoms**:
```
Error: Maximum number of APIs exceeded for Developer SKU
```

**Solutions**:

1. Check current SKU limits:
```bash
az apim show --name apim-my-gateway-uk --resource-group rg-my-gateway-uk --query "sku"
```

2. Upgrade SKU:
```yaml
apiManagement:
  skuName: Standard_1
```

3. Or remove unused APIs:
```bash
kubectl delete apiclaim unused-api
```

## Diagnostic Commands

### General Health Check

```bash
# Check all Crossplane resources
kubectl get crossplane

# Check providers
kubectl get providers

# Check XRDs
kubectl get xrd

# Check compositions
kubectl get compositions

# Check XIngress resources
kubectl get xingress

# Check ApiClaim resources
kubectl get apiclaim

# Check managed resources
kubectl get managed
```

### Detailed Diagnostics

```bash
# Get detailed provider information
kubectl describe provider provider-azure-apimanagement

# Check provider logs
kubectl logs -n crossplane-system -l pkg.crossplane.io/provider=provider-azure-apimanagement

# Check Crossplane logs
kubectl logs -n crossplane-system -l app=crossplane

# Get events
kubectl get events --sort-by=.metadata.creationTimestamp
```

### Azure Resource Check

```bash
# List all resources in resource group
az resource list --resource-group rg-my-gateway-uk

# Check API Management service
az apim show --name apim-my-gateway-uk --resource-group rg-my-gateway-uk

# List APIs
az apim api list --service-name apim-my-gateway-uk --resource-group rg-my-gateway-uk

# Check virtual network
az network vnet show --name vnet-my-gateway-uk --resource-group rg-my-gateway-uk
```

## Performance Troubleshooting

### Slow Resource Creation

**Problem**: Resources take too long to create

**Solutions**:

1. Check Azure service limits:
```bash
az vm list-usage --location uksouth
```

2. Monitor resource creation progress:
```bash
kubectl get managed -w
```

3. Check Azure activity logs:
```bash
az monitor activity-log list --resource-group rg-my-gateway-uk
```

### High Resource Usage

**Problem**: Crossplane consuming too many resources

**Solutions**:

1. Check resource usage:
```bash
kubectl top pods -n crossplane-system
```

2. Adjust resource limits:
```bash
kubectl patch deployment crossplane -n crossplane-system -p '{"spec":{"template":{"spec":{"containers":[{"name":"crossplane","resources":{"limits":{"cpu":"500m","memory":"512Mi"}}}]}}}}'
```

## Getting Help

### Log Collection

When reporting issues, collect the following logs:

```bash
# Crossplane logs
kubectl logs -n crossplane-system -l app=crossplane > crossplane.log

# Provider logs
kubectl logs -n crossplane-system -l pkg.crossplane.io/provider=provider-azure-apimanagement > provider-apim.log

# Resource descriptions
kubectl describe xingress my-gateway > xingress-describe.log
kubectl describe apiclaim my-api > apiclaim-describe.log

# Events
kubectl get events --sort-by=.metadata.creationTimestamp > events.log
```

### Debug Mode

Enable debug logging for providers:

```bash
# Add debug annotation to provider
kubectl annotate provider provider-azure-apimanagement debug=true
```

### Community Resources

- [Crossplane Slack](https://slack.crossplane.io)
- [Crossplane GitHub](https://github.com/crossplane/crossplane)
- [Provider Azure GitHub](https://github.com/upbound/provider-azure)

### Azure Support

For Azure-specific issues:
- [Azure Support](https://azure.microsoft.com/support)
- [Azure CLI Issues](https://github.com/Azure/azure-cli)
- [Azure Documentation](https://docs.microsoft.com/azure)

## Prevention Best Practices

### Resource Monitoring

Set up monitoring for:
- Resource creation time
- Error rates
- Resource usage
- Cost tracking

### Regular Maintenance

- Update providers regularly
- Review and clean up unused resources
- Monitor Azure service limits
- Backup important configurations

### Testing

- Test changes in development environment first
- Use dry-run mode when possible
- Validate configurations before applying
- Monitor resource creation closely

### Documentation

- Keep runbooks updated
- Document custom configurations
- Track known issues and solutions
- Share knowledge with team members
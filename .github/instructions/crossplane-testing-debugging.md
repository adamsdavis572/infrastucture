---
applyTo: "**/*.yaml"
---
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

# Example of using ApiClaim

This example demonstrates how to use the ApiClaim resource to create an API in Azure API Management.

## Prerequisites

1. An existing XIngress resource that has provisioned the API Management gateway
2. The ApiInstance CompositeResourceDefinition and Composition deployed to your cluster

## Usage

1. Edit the `example-api-claim.yaml` file to reference your existing XIngress resource:
   - Update `ingressRef` to match the name of your XIngress resource
   - Customize `apiName` and `apiPath` as needed
   - Optionally add a base64-encoded Swagger/OpenAPI definition

2. Apply the claim to your cluster:
   ```
   kubectl apply -f example-api-claim.yaml
   ```

3. Check the status of your API:
   ```
   kubectl get apiclaim example-api
   ```

4. Once the API is provisioned, you can access it at the URL shown in the status:
   ```
   kubectl get apiclaim example-api -o jsonpath='{.status.apiUrl}'
   ```

## Swagger Definition

The commented `swaggerDefinition` field contains a base64-encoded OpenAPI/Swagger definition. 
To use your own definition:

1. Create a Swagger/OpenAPI JSON file
2. Base64 encode it:
   ```
   cat your-swagger-file.json | base64
   ```
3. Paste the result into the `swaggerDefinition` field

## Cleanup

To delete the API:
```
kubectl delete apiclaim example-api
```
# Basic Edge Access without OAuth2 Proxy

This example demonstrates edge-access configuration for public APIs that don't require authentication.

## Features

- OAuth2 authentication disabled
- Direct routing to backend services
- Multiple path-based routing rules
- Custom annotations and labels on HTTPRoute

## Configuration

The configuration includes:
- **HTTPRoute**: Creates routes for `api.example.com` with multiple path rules
  - `/api/v1` → api-v1-service:8080
  - `/health` → api-v1-service:8080 (exact match)
  - `/static` → static-content-service:80
- **OAuth2 Proxy**: Disabled - traffic goes directly to backend
- **No External Secrets**: Not needed without OAuth2 proxy

## Use Cases

This configuration is ideal for:
- Public REST APIs
- Health check endpoints
- Static content delivery
- Services with built-in authentication
- Internal services behind a VPN

## Prerequisites

- Gateway API CRDs installed
- A Gateway resource named `public-ingress-gateway` in namespace `infra-network`
- Backend services deployed and accessible

## Installation

```bash
# From this directory
helm dependency update
helm install public-api . -n my-namespace
```

## Verify Installation

```bash
# Check the HTTPRoute
kubectl get httproute public-api-route -n my-namespace -o yaml

# Verify routing rules
kubectl describe httproute public-api-route -n my-namespace
```

## Testing

```bash
# Test API endpoint
curl https://api.example.com/api/v1/users

# Test health endpoint
curl https://api.example.com/health

# Test static content
curl https://api.example.com/static/logo.png
```

## Cleanup

```bash
helm uninstall public-api -n my-namespace
```

## Notes

- No OAuth2 proxy pods will be created
- Traffic flows directly from the Gateway to your backend services
- Ensure your backend services handle any required authentication/authorization

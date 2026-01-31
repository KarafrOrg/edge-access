# Multiple Routes with OAuth2 Proxy

This example demonstrates an advanced production-ready setup with multiple hostnames, advanced OAuth2 configuration, and high availability.

## Features

- OAuth2 authentication with domain restrictions
- Multiple hostnames routing to the same backend
- Multiple path-based routing rules
- High availability with 2 replicas
- Pod disruption budget for reliability
- Resource limits and requests
- Security context configuration
- Selective authentication bypass for health checks
- Advanced cookie configuration
- TLS certificate automation with cert-manager

## Configuration

The configuration includes:
- **Multiple Hostnames**: 
  - `app.example.com`
  - `app.example.org`
  - `enterprise.example.com`
- **Multiple Routes**:
  - `/admin` → admin-backend-service:8080
  - `/api/v2` → api-v2-service:9000
  - `/` → frontend-service:3000
- **OAuth2 Proxy**: Production-grade configuration
  - Email domain restrictions
  - Skip auth for health/metrics endpoints
  - 7-day cookie expiration
  - User information passed to backend
- **High Availability**: 2 replicas with PDB

## Use Cases

This configuration is ideal for:
- Enterprise applications requiring authentication
- Multi-tenant applications
- Production environments requiring HA
- Applications with admin interfaces
- Services requiring user context in backend

## Prerequisites

- Gateway API CRDs installed
- A Gateway resource named `public-ingress-gateway` in namespace `infra-network`
- External Secrets Operator installed
- OAuth2 credentials configured with allowed domains
- Cert-manager installed (optional, for TLS)
- Backend services deployed

## OAuth2 Secret Format

The external secret should contain:
```json
{
  "client-id": "your-oauth2-client-id",
  "client-secret": "your-oauth2-client-secret",
  "cookie-secret": "your-random-cookie-secret"
}
```

Generate cookie secret:
```bash
python -c 'import os,base64; print(base64.urlsafe_b64encode(os.urandom(32)).decode())'
```

## Installation

```bash
# From this directory
helm dependency update
helm install enterprise-app . -n production
```

## Verify Installation

```bash
# Check all resources
kubectl get all -n production -l app=enterprise-app

# Check HTTPRoute
kubectl get httproute multi-domain-route -n production -o yaml

# Check OAuth2 proxy pods
kubectl get pods -n production -l app.kubernetes.io/name=oauth2-proxy

# Check external secret
kubectl get externalsecret -n production

# Check PDB
kubectl get pdb -n production
```

## Testing

```bash
# Test authentication flow
curl -v https://app.example.com/
# Should redirect to OAuth2 provider

# Test health endpoint (should skip auth)
curl https://app.example.com/health

# Test admin interface
curl https://app.example.com/admin
# Should redirect to OAuth2 provider

# Test API
curl -H "Authorization: Bearer <token>" https://app.example.com/api/v2/data
```

## Monitoring

```bash
# Check oauth2-proxy logs
kubectl logs -n production -l app.kubernetes.io/name=oauth2-proxy -f

# Check metrics
kubectl port-forward -n production svc/oauth2-proxy 4180:80
curl http://localhost:4180/metrics
```

## Scaling

```bash
# Scale oauth2-proxy replicas
helm upgrade enterprise-app . -n production \
  --set edge-access.oauth2-proxy.replicaCount=3
```

## Cleanup

```bash
helm uninstall enterprise-app -n production
```

## Security Notes

- Email domains are restricted to `example.com` and `example.org`
- Cookies are secure, httponly, and have 7-day expiration
- OAuth2 proxy runs as non-root user
- Health and metrics endpoints bypass authentication for monitoring
- All authentication events are logged

# Basic Edge Access with OAuth2 Proxy

This example demonstrates a basic setup of the edge-access Helm chart with OAuth2 proxy enabled for authentication.

## Features

- OAuth2 authentication enabled
- Single hostname routing
- External secret management for OAuth2 credentials
- Routes traffic through oauth2-proxy before reaching backend service

## Configuration

The configuration includes:
- **HTTPRoute**: Creates a route for `myapp.example.com`
- **OAuth2 Proxy**: Enabled with OIDC provider (Google)
- **External Secret**: Fetches OAuth2 credentials from external secret store
- **Backend Service**: Routes authenticated traffic to `my-backend-service:8080`

## Prerequisites

- Gateway API CRDs installed
- A Gateway resource named `public-ingress-gateway` in namespace `infra-network`
- External Secrets Operator installed (for external secret management)
- OAuth2 credentials stored in your external secret store

## Installation

```bash
# From this directory
helm dependency update
helm install my-app . -n my-namespace
```

## Verify Installation

```bash
# Check the HTTPRoute
kubectl get httproute -n my-namespace

# Check the OAuth2 proxy deployment
kubectl get deployment -n my-namespace

# Check the external secret
kubectl get externalsecret -n my-namespace
```

## Access

Once deployed, navigate to `https://myapp.example.com` and you'll be redirected to the OAuth2 provider for authentication.

## Cleanup

```bash
helm uninstall my-app -n my-namespace
```

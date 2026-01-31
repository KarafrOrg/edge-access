# Custom External Secret Configuration

This example demonstrates how to use edge-access with a custom external secret store (HashiCorp Vault) instead of the default GCP Secret Manager.

## Features

- OAuth2 authentication with custom OIDC provider
- Custom SecretStore for HashiCorp Vault
- Custom ServiceAccount with Vault annotations
- RBAC configuration for secret access
- Secure cookie configuration with strict same-site policy

## Configuration

The configuration includes:
- **HTTPRoute**: Routes `secure.example.com` to backend service
- **OAuth2 Proxy**: Enabled with custom OIDC provider
- **Custom Secret Store**: HashiCorp Vault backend
- **Custom ServiceAccount**: With Vault authentication
- **RBAC**: Permissions for secret access

## Use Cases

This configuration is ideal for:
- Organizations using HashiCorp Vault
- Custom OIDC providers (Keycloak, Auth0, Okta, etc.)
- Environments requiring strict security policies
- Multi-cloud deployments with centralized secrets
- Compliance-driven architectures

## Prerequisites

- Gateway API CRDs installed
- A Gateway resource named `public-ingress-gateway` in namespace `infra-network`
- External Secrets Operator installed with Vault support
- HashiCorp Vault server accessible
- Vault Kubernetes authentication configured
- OAuth2 credentials stored in Vault

## Vault Setup

1. **Enable Kubernetes Auth in Vault:**
```bash
vault auth enable kubernetes

vault write auth/kubernetes/config \
  kubernetes_host="https://kubernetes.default.svc:443" \
  kubernetes_ca_cert=@/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
```

2. **Create Vault Policy:**
```bash
vault policy write edge-access-policy - <<EOF
path "secret/data/oauth2-credentials-path" {
  capabilities = ["read"]
}
EOF
```

3. **Create Vault Role:**
```bash
vault write auth/kubernetes/role/edge-access-role \
  bound_service_account_names=edge-access-sa \
  bound_service_account_namespaces=production \
  policies=edge-access-policy \
  ttl=24h
```

4. **Store OAuth2 Credentials in Vault:**
```bash
vault kv put secret/oauth2-credentials-path \
  client-id="your-client-id" \
  client-secret="your-client-secret" \
  cookie-secret="$(openssl rand -base64 32)"
```

## Secret Structure in Vault

The secret at `secret/oauth2-credentials-path` should contain:
```json
{
  "client-id": "your-oauth2-client-id",
  "client-secret": "your-oauth2-client-secret",
  "cookie-secret": "your-random-cookie-secret"
}
```

## Installation

```bash
# From this directory
helm dependency update

# Install with custom values
helm install secure-app . -n production \
  --create-namespace
```

## Verify Installation

```bash
# Check all resources
kubectl get all -n production

# Check SecretStore (if you've deployed it separately)
kubectl get secretstore -n production

# Check ExternalSecret
kubectl get externalsecret -n production

# Verify the secret was created from Vault
kubectl get secret custom-oauth2-secret -n production

# Check ServiceAccount
kubectl get sa edge-access-sa -n production

# Check RBAC
kubectl get role,rolebinding -n production
```

## Testing External Secret Sync

```bash
# Watch the ExternalSecret status
kubectl get externalsecret -n production -w

# Check if secret is synced
kubectl describe externalsecret -n production

# Verify secret contents (be careful in production!)
kubectl get secret custom-oauth2-secret -n production -o yaml
```

## Troubleshooting

### ExternalSecret Not Syncing

```bash
# Check ExternalSecret status
kubectl describe externalsecret -n production

# Check external-secrets-operator logs
kubectl logs -n external-secrets-system \
  -l app.kubernetes.io/name=external-secrets
```

### Vault Authentication Issues

```bash
# Verify ServiceAccount exists
kubectl get sa edge-access-sa -n production

# Check Vault role configuration
vault read auth/kubernetes/role/edge-access-role

# Test Vault authentication manually
kubectl run vault-test --rm -it --image=vault:latest -- sh
# Inside the pod:
# vault login -method=kubernetes role=edge-access-role
```

### OAuth2 Proxy Issues

```bash
# Check oauth2-proxy logs
kubectl logs -n production -l app.kubernetes.io/name=oauth2-proxy

# Verify secret is mounted
kubectl describe pod -n production -l app.kubernetes.io/name=oauth2-proxy
```

## Alternative Secret Stores

This example uses Vault, but you can adapt it for other secret stores:

### AWS Secrets Manager
```yaml
customSecretStore:
  enabled: true
  name: aws-secrets
  spec:
    provider:
      aws:
        service: SecretsManager
        region: us-east-1
        auth:
          jwt:
            serviceAccountRef:
              name: edge-access-sa
```

### Azure Key Vault
```yaml
customSecretStore:
  enabled: true
  name: azure-keyvault
  spec:
    provider:
      azurekv:
        vaultUrl: "https://my-vault.vault.azure.net"
        authType: WorkloadIdentity
        serviceAccountRef:
          name: edge-access-sa
```

## Cleanup

```bash
helm uninstall secure-app -n production

# Clean up Vault configuration
vault delete auth/kubernetes/role/edge-access-role
vault policy delete edge-access-policy
```

## Security Notes

- Vault authentication uses Kubernetes service account tokens
- Secrets are never stored in Git
- Cookie same-site policy set to "strict" for maximum security
- Service account has minimal required permissions
- Vault tokens have limited TTL (24h)

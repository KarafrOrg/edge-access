# multiple-routes-with-oauth2

![Version: 0.1.0](https://img.shields.io/badge/Version-0.1.0-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square)

Example with multiple hostnames and advanced OAuth2 configuration

## Requirements

| Repository | Name | Version |
|------------|------|---------|
| file://../.. | edge-access | 0.1.0 |

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| edge-access.httpRoute.annotations."cert-manager.io/cluster-issuer" | string | `"letsencrypt-prod"` |  |
| edge-access.httpRoute.annotations.description | string | `"Multi-domain application with OAuth2"` |  |
| edge-access.httpRoute.enabled | bool | `true` |  |
| edge-access.httpRoute.hostnames[0] | string | `"app.example.com"` |  |
| edge-access.httpRoute.hostnames[1] | string | `"app.example.org"` |  |
| edge-access.httpRoute.hostnames[2] | string | `"enterprise.example.com"` |  |
| edge-access.httpRoute.labels.app | string | `"enterprise-app"` |  |
| edge-access.httpRoute.labels.environment | string | `"production"` |  |
| edge-access.httpRoute.labels.team | string | `"platform"` |  |
| edge-access.httpRoute.name | string | `"multi-domain-route"` |  |
| edge-access.httpRoute.parentRefs[0].group | string | `"gateway.networking.k8s.io"` |  |
| edge-access.httpRoute.parentRefs[0].kind | string | `"Gateway"` |  |
| edge-access.httpRoute.parentRefs[0].name | string | `"public-ingress-gateway"` |  |
| edge-access.httpRoute.parentRefs[0].namespace | string | `"infra-network"` |  |
| edge-access.httpRoute.rules[0].backendRefs[0].group | string | `""` |  |
| edge-access.httpRoute.rules[0].backendRefs[0].kind | string | `"Service"` |  |
| edge-access.httpRoute.rules[0].backendRefs[0].name | string | `"admin-backend-service"` |  |
| edge-access.httpRoute.rules[0].backendRefs[0].port | int | `8080` |  |
| edge-access.httpRoute.rules[0].backendRefs[0].weight | int | `1` |  |
| edge-access.httpRoute.rules[0].matches[0].path.type | string | `"PathPrefix"` |  |
| edge-access.httpRoute.rules[0].matches[0].path.value | string | `"/admin"` |  |
| edge-access.httpRoute.rules[1].backendRefs[0].group | string | `""` |  |
| edge-access.httpRoute.rules[1].backendRefs[0].kind | string | `"Service"` |  |
| edge-access.httpRoute.rules[1].backendRefs[0].name | string | `"api-v2-service"` |  |
| edge-access.httpRoute.rules[1].backendRefs[0].port | int | `9000` |  |
| edge-access.httpRoute.rules[1].backendRefs[0].weight | int | `1` |  |
| edge-access.httpRoute.rules[1].matches[0].path.type | string | `"PathPrefix"` |  |
| edge-access.httpRoute.rules[1].matches[0].path.value | string | `"/api/v2"` |  |
| edge-access.httpRoute.rules[2].backendRefs[0].group | string | `""` |  |
| edge-access.httpRoute.rules[2].backendRefs[0].kind | string | `"Service"` |  |
| edge-access.httpRoute.rules[2].backendRefs[0].name | string | `"frontend-service"` |  |
| edge-access.httpRoute.rules[2].backendRefs[0].port | int | `3000` |  |
| edge-access.httpRoute.rules[2].backendRefs[0].weight | int | `1` |  |
| edge-access.httpRoute.rules[2].matches[0].path.type | string | `"PathPrefix"` |  |
| edge-access.httpRoute.rules[2].matches[0].path.value | string | `"/"` |  |
| edge-access.oauth2-proxy.config.configFile | string | `"provider = \"oidc\"\noidc_issuer_url = \"https://accounts.google.com\"\n# Restrict to specific email domains\nemail_domains = [\"example.com\", \"example.org\"]\n# Whitelist specific users\nauthenticated_emails_file = \"/etc/oauth2-proxy/emails/authenticated-emails.txt\"\ncookie_secure = true\ncookie_httponly = true\ncookie_samesite = \"lax\"\ncookie_name = \"_oauth2_proxy_enterprise\"\ncookie_expire = \"168h\"  # 7 days\n# Pass user information to backend\npass_access_token = true\npass_authorization_header = true\npass_user_headers = true\nset_authorization_header = true\nset_xauthrequest = true\n# Skip authentication for specific paths\nskip_auth_routes = [\n  \"GET=/health\",\n  \"GET=/metrics\",\n  \"GET=/favicon.ico\"\n]\n# Logging\nrequest_logging = true\nauth_logging = true\nstandard_logging = true"` |  |
| edge-access.oauth2-proxy.config.existingSecret | string | `"enterprise-oauth2-secret"` |  |
| edge-access.oauth2-proxy.podDisruptionBudget.enabled | bool | `true` |  |
| edge-access.oauth2-proxy.podDisruptionBudget.minAvailable | int | `1` |  |
| edge-access.oauth2-proxy.replicaCount | int | `2` |  |
| edge-access.oauth2-proxy.resources.limits.cpu | string | `"500m"` |  |
| edge-access.oauth2-proxy.resources.limits.memory | string | `"512Mi"` |  |
| edge-access.oauth2-proxy.resources.requests.cpu | string | `"100m"` |  |
| edge-access.oauth2-proxy.resources.requests.memory | string | `"128Mi"` |  |
| edge-access.oauth2-proxy.securityContext.fsGroup | int | `2000` |  |
| edge-access.oauth2-proxy.securityContext.runAsNonRoot | bool | `true` |  |
| edge-access.oauth2-proxy.securityContext.runAsUser | int | `2000` |  |
| edge-access.oauth2Proxy.enabled | bool | `true` |  |
| edge-access.oauth2Proxy.externalSecret.enabled | bool | `true` |  |
| edge-access.oauth2Proxy.externalSecret.externalSecretKey | string | `"enterprise-oauth2-credentials"` |  |
| edge-access.oauth2Proxy.externalSecret.secretName | string | `"enterprise-oauth2-secret"` |  |
| edge-access.oauth2Proxy.service.port | int | `80` |  |

----------------------------------------------
Autogenerated from chart metadata using [helm-docs v1.14.2](https://github.com/norwoodj/helm-docs/releases/v1.14.2)

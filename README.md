# edge-access

![Version: 0.2.0](https://img.shields.io/badge/Version-0.2.0-informational?style=flat-square)

Publishes Kubernetes services using Gateway API, optional OAuth2, and an Envoy frontend proxy.

**Homepage:** <https://github.com/KarafrOrg/edge-access>

## Requirements

| Repository                               | Name         | Version |
|------------------------------------------|--------------|---------|
| https://oauth2-proxy.github.io/manifests | oauth2-proxy | 10.1.2  |

## Values

| Key                                                       | Type   | Default                       | Description |
|-----------------------------------------------------------|--------|-------------------------------|-------------|
| cloudflareGateway.className                               | string | `"cloudflare-tunnel"`         |             |
| cloudflareGateway.credentials.externalSecret.accountIdKey | string | `""`                          |             |
| cloudflareGateway.credentials.externalSecret.apiTokenKey  | string | `""`                          |             |
| cloudflareGateway.credentials.externalSecret.enabled      | bool   | `false`                       |             |
| cloudflareGateway.credentials.externalSecret.storeKind    | string | `"ClusterSecretStore"`        |             |
| cloudflareGateway.credentials.externalSecret.storeName    | string | `"gcp-cluster-secret-store"`  |             |
| cloudflareGateway.credentials.secretName                  | string | `""`                          |             |
| cloudflareGateway.enabled                                 | bool   | `false`                       |             |
| cloudflareGateway.name                                    | string | `""`                          |             |
| cloudflareGateway.tunnel.patches                          | list   | `[]`                          |             |
| cloudflareGateway.tunnel.replicas                         | list   | `[]`                          |             |
| frontendProxy.enabled                                     | bool   | `false`                       |             |
| frontendProxy.image                                       | string | `"envoyproxy/envoy:v1.36.2"`  |             |
| frontendProxy.name                                        | string | `""`                          |             |
| frontendProxy.nodeSelector                                | object | `{}`                          |             |
| frontendProxy.replicas                                    | int    | `2`                           |             |
| frontendProxy.resources.limits.memory                     | string | `"256Mi"`                     |             |
| frontendProxy.resources.requests.cpu                      | string | `"25m"`                       |             |
| frontendProxy.resources.requests.memory                   | string | `"64Mi"`                      |             |
| frontendProxy.telemetry.appName                           | string | `""`                          |             |
| frontendProxy.telemetry.environment                       | string | `"production"`                |             |
| frontendProxy.telemetry.host                              | string | `""`                          |             |
| frontendProxy.telemetry.path                              | string | `"/faro/collect"`             |             |
| frontendProxy.telemetry.port                              | int    | `12347`                       |             |
| frontendProxy.telemetry.version                           | string | `"2.13.0"`                    |             |
| frontendProxy.tolerations                                 | list   | `[]`                          |             |
| frontendProxy.topologySpreadConstraints                   | list   | `[]`                          |             |
| frontendProxy.upstream.host                               | string | `""`                          |             |
| frontendProxy.upstream.port                               | int    | `80`                          |             |
| httpRoute.annotations                                     | object | `{}`                          |             |
| httpRoute.enabled                                         | bool   | `true`                        |             |
| httpRoute.hostnames[0]                                    | string | `"seaweed.karafra.net"`       |             |
| httpRoute.labels                                          | object | `{}`                          |             |
| httpRoute.name                                            | string | `""`                          |             |
| httpRoute.parentRefs[0].group                             | string | `"gateway.networking.k8s.io"` |             |
| httpRoute.parentRefs[0].kind                              | string | `"Gateway"`                   |             |
| httpRoute.parentRefs[0].name                              | string | `"public-ingress-gateway"`    |             |
| httpRoute.parentRefs[0].namespace                         | string | `"infra-network"`             |             |
| httpRoute.rules[0].backendRefs[0].group                   | string | `""`                          |             |
| httpRoute.rules[0].backendRefs[0].kind                    | string | `"Service"`                   |             |
| httpRoute.rules[0].backendRefs[0].name                    | string | `"seaweed-admin"`             |             |
| httpRoute.rules[0].backendRefs[0].port                    | int    | `8080`                        |             |
| httpRoute.rules[0].backendRefs[0].weight                  | int    | `1`                           |             |
| httpRoute.rules[0].matches[0].path.type                   | string | `"PathPrefix"`                |             |
| httpRoute.rules[0].matches[0].path.value                  | string | `"/"`                         |             |
| oauth2-proxy.config.existingSecret                        | string | `"oauth2-proxy-secret"`       |             |
| oauth2-proxy.config.externalSecretKey                     | string | `"oauth2-proxy-credentials"`  |             |
| oauth2Proxy.enabled                                       | bool   | `false`                       |             |
| oauth2Proxy.externalSecret.enabled                        | bool   | `true`                        |             |
| oauth2Proxy.externalSecret.externalSecretKey              | string | `"oauth2-proxy-credentials"`  |             |
| oauth2Proxy.externalSecret.secretName                     | string | `"oauth2-proxy-secret"`       |             |
| oauth2Proxy.service.port                                  | int    | `80`                          |             |
| tests.enabled                                             | bool   | `true`                        |             |
| tests.image                                               | string | `"busybox:1.36"`              |             |
| tests.kubectlImage                                        | string | `"bitnami/kubectl:1.31"`      |             |

----------------------------------------------
Autogenerated from chart metadata using [helm-docs v1.14.2](https://github.com/norwoodj/helm-docs/releases/v1.14.2)

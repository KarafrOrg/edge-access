# edge-access

![Version: 0.2.0](https://img.shields.io/badge/Version-0.2.0-informational?style=flat-square)
[![Artifact Hub](https://img.shields.io/endpoint?url=https://artifacthub.io/badge/repository/edge-access)](https://artifacthub.io/packages/search?repo=edge-access)

Publishes Kubernetes services through Gateway API, with optional OAuth2 authentication,
Envoy frontend instrumentation, and Cloudflare Tunnel gateway provisioning.

**Homepage:** <https://github.com/KarafrOrg/edge-access>

## Requirements

| Repository                               | Name         | Version |
|------------------------------------------|--------------|---------|
| https://oauth2-proxy.github.io/manifests | oauth2-proxy | 10.1.2  |

The Gateway API CRDs and an appropriate Gateway controller must already be installed.
Cloudflare gateway provisioning also requires the Cloudflare Gateway Controller; creating
an ExternalSecret requires External Secrets Operator and the configured secret store.

## Frontend proxy

The frontend proxy is disabled by default. To expose an application through an existing
Gateway with Faro browser telemetry, use values such as:

```yaml
oauth2Proxy:
  enabled: false
httpRoute:
  parentRefs:
    - name: private-ingress-gateway
      namespace: infra-network
      sectionName: https
  hostnames: [grafana.example.com]
  annotations:
    external-dns.alpha.kubernetes.io/target: "10.96.0.200"
    external-dns.alpha.kubernetes.io/cloudflare-proxied: "false"
frontendProxy:
  enabled: true
  upstream:
    host: grafana.infra-monitoring.svc.cluster.local
    port: 80
  faro:
    host: alloy.infra-monitoring.svc.cluster.local
```

Each application rule forwards to the proxy Service instead of its `backendRefs`.
The proxy serves the configured same-origin Faro path by forwarding to `/collect` on
the Faro receiver, and injects the Faro SDK and tracing scripts into HTML responses.
An additional HTTPRoute rule exposes that Faro path even when application matches
are restricted to other paths.
It preserves WebSocket upgrades and disables upstream compression for HTML injection.
Compressed and partial responses are passed through without injection.
The deployment uses two replicas by default; `frontendProxy.nodeSelector`,
`tolerations`, `topologySpreadConstraints`, and `resources` control placement and
capacity. Set `frontendProxy.name` to override the release-based resource name.
Faro loads scripts from `unpkg.com`; browser CSP and access to that host must allow them.

When OAuth2 is also enabled, application traffic goes through Envoy **then** OAuth2
proxy; `/oauth2` callbacks go directly to OAuth2 proxy. Configure the `oauth2-proxy`
subchart's upstream to point to the application, not the frontend proxy. The Faro
collector path bypasses OAuth2, so the receiver must be appropriate for public ingestion.
`frontendProxy.upstream` is used only when OAuth2 is disabled.

## Dedicated Cloudflare Gateway

Set `cloudflareGateway.enabled: true` to create a Gateway and
CloudflareGatewayParameters, and attach the HTTPRoute to it instead of `parentRefs`.
By default the controller reads a pre-existing Secret named
`<release>-cloudflare-gateway-creds`. Set `credentials.secretName` to use an existing
shared Secret, or enable `credentials.externalSecret` and provide `accountIdKey` and
`apiTokenKey` to create it with External Secrets. The Secret must contain
`CLOUDFLARE_ACCOUNT_ID` and `CLOUDFLARE_API_TOKEN`.

`cloudflareGateway.tunnel.replicas` and `patches` are passed through to the controller;
they can configure named replicas, zone preferences, tolerations and node placement.
See `examples/frontend-proxy/values.yaml` for the private gateway configuration and
`examples/cloudflare-gateway/values.yaml` for a dedicated public tunnel.
These settings do not install or configure the GatewayClass, controller, DNS provider,
or secret store. HTTPRoute filters are controller-dependent: in particular, the
Cloudflare Gateway Controller does not support route rewrite filters.

Run `make test-render` to check the default, OAuth, frontend-proxy and Cloudflare
rendering paths. The checks require Python 3, Helm, yq v4, and restored chart dependencies.

## Additional values

| Key | Default | Description |
|---|---|---|
| `frontendProxy.enabled` | `false` | Create the Envoy ConfigMap, Deployment and Service. |
| `frontendProxy.name` | `<release>-frontend-proxy` | Override all three proxy resource names. |
| `frontendProxy.image` / `replicas` | `envoyproxy/envoy:v1.36.2` / `2` | Envoy image and replica count. |
| `frontendProxy.upstream.host` / `port` | `""` / `80` | Application DNS name and port; required when OAuth2 is disabled. |
| `frontendProxy.faro.host` / `port` | `""` / `12347` | Faro receiver DNS name and port; host is required. |
| `frontendProxy.faro.path` / `version` | `/faro/collect` / `2.13.0` | Same-origin endpoint and Faro SDK version. |
| `frontendProxy.faro.appName` / `environment` | `<release>` / `production` | Browser telemetry app name and environment. |
| `cloudflareGateway.enabled` | `false` | Create a dedicated Cloudflare Gateway. |
| `cloudflareGateway.name` / `className` | `<release>` / `cloudflare-tunnel` | Gateway name and existing GatewayClass. |
| `cloudflareGateway.credentials.secretName` | `<release>-cloudflare-gateway-creds` | Existing or generated credential Secret. |
| `cloudflareGateway.credentials.externalSecret` | disabled | Optional external secret store and account/token key configuration. |
| `cloudflareGateway.tunnel.replicas` / `patches` | `[]` / `[]` | Controller-specific tunnel replicas and deployment patches. |

## Values

| Key                                          | Type   | Default                       | Description |
|----------------------------------------------|--------|-------------------------------|-------------|
| httpRoute.annotations                        | object | `{}`                          |             |
| httpRoute.enabled                            | bool   | `true`                        |             |
| httpRoute.hostnames[0]                       | string | `"seaweed.karafra.net"`       |             |
| httpRoute.labels                             | object | `{}`                          |             |
| httpRoute.name                               | string | `""`                          |             |
| httpRoute.parentRefs[0].group                | string | `"gateway.networking.k8s.io"` |             |
| httpRoute.parentRefs[0].kind                 | string | `"Gateway"`                   |             |
| httpRoute.parentRefs[0].name                 | string | `"public-ingress-gateway"`    |             |
| httpRoute.parentRefs[0].namespace            | string | `"infra-network"`             |             |
| httpRoute.rules[0].backendRefs[0].group      | string | `""`                          |             |
| httpRoute.rules[0].backendRefs[0].kind       | string | `"Service"`                   |             |
| httpRoute.rules[0].backendRefs[0].name       | string | `"seaweed-admin"`             |             |
| httpRoute.rules[0].backendRefs[0].port       | int    | `8080`                        |             |
| httpRoute.rules[0].backendRefs[0].weight     | int    | `1`                           |             |
| httpRoute.rules[0].matches[0].path.type      | string | `"PathPrefix"`                |             |
| httpRoute.rules[0].matches[0].path.value     | string | `"/"`                         |             |
| oauth2-proxy.config.existingSecret           | string | `"oauth2-proxy-secret"`       |             |
| oauth2-proxy.config.externalSecretKey        | string | `"oauth2-proxy-credentials"`  |             |
| oauth2Proxy.enabled                          | bool   | `true`                        |             |
| oauth2Proxy.externalSecret.enabled           | bool   | `true`                        |             |
| oauth2Proxy.externalSecret.externalSecretKey | string | `"oauth2-proxy-credentials"`  |             |
| oauth2Proxy.externalSecret.secretName        | string | `"oauth2-proxy-secret"`       |             |
| oauth2Proxy.service.port                     | int    | `80`                          |             |
| tests.enabled                                | bool   | `true`                        |             |
| tests.image                                  | string | `"busybox:1.36"`              |             |
| tests.kubectlImage                           | string | `"bitnami/kubectl:1.31"`      |             |

----------------------------------------------
Autogenerated from chart metadata using [helm-docs v1.14.2](https://github.com/norwoodj/helm-docs/releases/v1.14.2)

---
title: Deploying Identity Analytics in an Istio service mesh
description: How to run Self-Managed Identity Analytics in an Istio service mesh
---

# Deploying Identity Analytics in an Istio service mesh

Self-Managed Identity Analytics runs in an [Istio](https://istio.io) service mesh **without any application change**. This page lists the Istio settings it needs.

The steps outlined in this document was validated with:

- Identity Analytics 3.8 (`ida-helm` 3.8.0 and `ida-shared-helm` 3.6.0);
- Istio 1.30.2 on Kubernetes 1.35;
- mTLS in `PERMISSIVE` mode.


## Architecture

Istio handles incoming traffic before forwarding it to the Shared Services API gateway, APISIX. It does not replace APISIX.

- APISIX stays the API gateway of Identity Analytics and the OIDC client towards Keycloak. Every service registers its routes there.
- Istio replaces only the entry point of external traffic, that is the Kubernetes `Ingress` of the charts, or the load balancer rule that pointed to APISIX.
- Traffic between pods of the meshed namespaces uses mTLS.

```
browser ─► [optional load balancer] ─► istio-ingressgateway ─► Gateway + VirtualService
                                                                   └─► APISIX (Shared Services) ─► Keycloak / Identity Analytics services
                                        [ pod-to-pod traffic inside the meshed namespaces: mTLS ]
```

## Requirements

1. **Kubernetes 1.29 or later, with Istio native sidecars.** Run istiod with `pilot.env.ENABLE_NATIVE_SIDECARS=true`.
   - The charts' provisioning Jobs need it: a classic `istio-proxy` sidecar never exits, so the Job never completes and `helm --wait` times out.
   - Set it on istiod, for the whole mesh: the Identity Analytics chart doesn't allow annotations on its Job pods.
2. **Label the namespaces before installing the charts**, the Shared Services namespace and each Identity Analytics namespace: `istio-injection=enabled`. Injection only applies to pods created after the label is set, pods that already exist need a restart.
3. **Set `global.pathPrefix`** in the Identity Analytics values, for example `/ida`, as in [Self-Managed Identity Analytics Deployment](../index.md). It's required: with an empty prefix, the portal isn't reachable (HTTP 403, `Access denied`).
4. **Set the port exclusions on istiod before creating the application pods** (see [Ports that must bypass the sidecar](#ports-that-must-bypass-the-sidecar)).
5. **Disable the ingress** that Istio replaces: `ingress.enabled: false` in the values of both charts (default value), or remove the load balancer rule that pointed to APISIX.

## Ports that must bypass the sidecar

The sidecar intercepts traffic and automatically detects protocols. This can disrupt database connections, cluster communication, and server-first protocols, such as SMTP, in which the server initiates communication.

Configure the following port exclusions mesh-wide, using the directions listed in the table. Pods remain in the mesh, but traffic on the excluded ports bypasses the sidecar.

| Port(s) | Direction | Component | Symptom if not excluded |
|---|---|---|---|
| `5432` | inbound + outbound | PostgreSQL (CloudNativePG) | Keycloak restarts in a loop |
| `2379`, `2380` | inbound + outbound | etcd (APISIX configuration store) | APISIX loses its routes: every URL returns 503 while the pods look healthy |
| `7800`, `57800` | inbound + outbound | Keycloak clustering (JGroups, failure detection) | Keycloak replicas can't form a cluster |
| SMTP port (`smtp.port`, `25` by default, and `keycloakSettings.smtp.port` if set) | outbound | Email (Identity Analytics and Keycloak) | Emails aren't sent |
| External database port (only with `database.type: external`) | outbound | External database | Same as `5432`. Not tested. |

All other Identity Analytics ports use HTTP and require no exclusions. These include the ports for the portal, workflow, configuration, controller, batch, extractor, Git server, operations manager, metrics, and AIDA / LiteLLM. 

> [!note] Connections from Identity Analytics to your own systems (collector sources such as LDAP directories or databases, a Git repository over SSH) may also require port exclusions. If a connection fails with sidecar interception enabled, try excluding its destination port from outbound interception.

To check the exclusions a pod actually got (ports after `-d` for inbound and after `-o` for outbound):

```
kubectl get pod <POD> -n <NAMESPACE> -o jsonpath='{.spec.initContainers[?(@.name=="istio-init")].args}'
```

To change the list on a running mesh:

1. Run `helm upgrade istiod` with the new list.
2. Restart istiod: `kubectl rollout restart deploy/istiod -n istio-system`.
3. Only **then** restart the application pods, otherwise they can still get the old list.

## Install Istio

```
helm repo add istio https://istio-release.storage.googleapis.com/charts
helm repo update

helm upgrade --install istio-base istio/base -n istio-system --create-namespace --version <ISTIO_VERSION> --wait

# Commas are escaped for --set. Replace 25 with your SMTP port.
helm upgrade --install istiod istio/istiod -n istio-system --version <ISTIO_VERSION> --wait \
  --set pilot.env.ENABLE_NATIVE_SIDECARS=true \
  --set meshConfig.enablePrometheusMerge=true \
  --set global.proxy.excludeInboundPorts="5432\,2379\,2380\,7800\,57800" \
  --set global.proxy.excludeOutboundPorts="5432\,2379\,2380\,7800\,57800\,25"

# The release name "istio-ingressgateway" gives the label istio: ingressgateway, selected by the Gateway below
helm upgrade --install istio-ingressgateway istio/gateway -n istio-ingress --create-namespace --version <ISTIO_VERSION> --wait
#   add  -f gateway-values.yaml  when a load balancer terminates TLS in front of Istio (option B below)

kubectl label namespace <SHARED_NAMESPACE> istio-injection=enabled --overwrite
kubectl label namespace <IDA_NAMESPACE> istio-injection=enabled --overwrite
```

Then install the Shared Services and Identity Analytics charts as described in [Self-Managed Identity Analytics Deployment](../index.md), with `ingress.enabled: false`. No other value is needed for the mesh.

## Edge TLS and the login redirect URI

Identity Analytics registers its OIDC callbacks in APISIX as relative paths (`{{PATH_PREFIX}}/callback`, `{{PATH_PREFIX}}/config/callback`..., where `{{PATH_PREFIX}}` is the value of `global.pathPrefix`, for example `/ida`). APISIX builds the scheme of the redirect URI from the request it receives: if this request looks like `http`, Keycloak rejects `redirect_uri=http://...` (HTTP 400, `invalid_redirect_uri`) and login fails.

This isn't specific to Istio: it applies to any component that terminates TLS before APISIX without passing `X-Forwarded-Proto: https`.

Choose one of these setups:

| Option | Configuration |
|---|---|
| A. Terminate TLS at the Istio gateway (**recommended**) | Configure an HTTPS server on Gateway port 443, with `credentialName` referencing a TLS secret in the `istio-ingress` namespace, such as one created by cert-manager. Enable `httpsRedirect` on port 80. If a load balancer sits in front of the gateway, configure it to pass TLS through or re-encrypt traffic to the gateway. |
| B. Terminate TLS at a load balancer | Configure the Gateway to listen only on HTTP port 80. Do not enable `httpsRedirect`, as this would cause a redirect loop. The load balancer must set `X-Forwarded-Proto: https`, and the gateway must trust this header, as described below. |

For option B, install the gateway with these values. The setting is read when the gateway pod starts:

```
# gateway-values.yaml
podAnnotations:
  proxy.istio.io/config: '{"gatewayTopology":{"numTrustedProxies":1}}'   # number of proxies in front of the gateway
```

Without this annotation, the gateway replaces the load balancer's `X-Forwarded-Proto: https` with `http`, and login fails.

## Routing

One host (`global.externalDns`) sends everything to APISIX. Example for option A (for option B, remove the port 443 server and the `httpsRedirect`):

```
apiVersion: networking.istio.io/v1
kind: Gateway
metadata:
  name: ida-gateway
  namespace: istio-ingress
spec:
  selector:
    istio: ingressgateway
  servers:
    - port: { number: 80, name: http, protocol: HTTP }
      hosts: ["{{HOSTNAME}}"]
      tls: { httpsRedirect: true }
    - port: { number: 443, name: https, protocol: HTTPS }
      hosts: ["{{HOSTNAME}}"]
      tls: { mode: SIMPLE, credentialName: <TLS_SECRET_NAME> }
---
apiVersion: networking.istio.io/v1
kind: VirtualService
metadata:
  name: ida
  namespace: istio-ingress
spec:
  hosts: ["{{HOSTNAME}}"]
  gateways: [ida-gateway]
  http:
    - route:
        - destination:
            host: apisix-gateway.<SHARED_NAMESPACE>.svc.cluster.local
            port: { number: 80 }
      timeout: 3600s
```

`{{HOSTNAME}}` is the value of `global.externalDns` and `{{PATH_PREFIX}}` the value of `global.pathPrefix`, with its leading slash (for example `/ida`).

## mTLS

Start with `PERMISSIVE`, one `PeerAuthentication` per meshed namespace:

```
apiVersion: security.istio.io/v1
kind: PeerAuthentication
metadata:
  name: default
  namespace: <NAMESPACE>
spec:
  mtls:
    mode: PERMISSIVE
```

Before switching to `STRICT` mode, every client outside the mesh that calls a meshed pod must either be in the mesh, or be allowed by a `portLevelMtls` exception. Examples include monitoring tools that scrape metrics ports and operators running in other namespaces.
 The ports excluded above bypass mTLS, so `STRICT` doesn't affect them.

## Verify the deployment

```
# Every pod has an istio-proxy sidecar (native sidecars are listed under initContainers)
kubectl get pods -n <IDA_NAMESPACE>

# Workload identity (mTLS certificate)
kubectl exec -n <IDA_NAMESPACE> <POD> -c istio-proxy -- pilot-agent request GET certs | grep spiffe

# The login redirect to Keycloak uses https (expect redirect_uri=https%3A%2F%2F...)
curl -s -o /dev/null -D- "https://{{HOSTNAME}}{{PATH_PREFIX}}/portal" | grep -i '^location:'
```

The redirect URI must be `https://{{HOSTNAME}}{{PATH_PREFIX}}/callback`. Verify that you can log in to the portal from a browser.

On a fresh install, a few restarts are expected while the services start: Keycloak waiting for its database, the batch server warming up, and the APISIX dashboard waiting for etcd (only when `apisixUI.enabled` is `true`, it's `false` by default). They resolve on their own.

## Other service meshes

This document covers only Istio, but some requirements also apply to other meshes and proxies:

- **The port exclusions** apply to any sidecar-based mesh, under another setting name (for example Linkerd `config.linkerd.io/skip-inbound-ports` and `config.linkerd.io/skip-outbound-ports`).
- **The Jobs** need a sidecar that exits when the Job ends, in any sidecar-based mesh. Enable it for the whole mesh, as the Identity Analytics chart doesn't allow annotations on its Job pods (for example Linkerd native sidecars, Helm value `proxy.nativeSidecar: true`). Sidecar-less meshes don't have this issue.
- **The `X-Forwarded-Proto: https` requirement** applies to any proxy that terminates TLS before APISIX.

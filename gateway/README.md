# Gateway API

Cilium-backed Gateway API resources. Replaces ingress-nginx.

## Out-of-band bootstrap: install Gateway API CRDs

CRDs are installed manually rather than via ArgoCD so a stray app sync
cannot delete them. Renovate keeps the pinned version below current —
when it opens a PR bumping the version, merge it, then re-run the
command on the cluster.

**Version is pinned to what Cilium supports**, not to "latest Gateway
API". Cilium ↔ Gateway API compatibility:

| Cilium | Gateway API |
| --- | --- |
| 1.18.x | v1.2.0 (experimental channel — operator needs TLSRoute v1alpha2) |
| 1.19.x | v1.4.1 |
| 1.20.x+ | v1.5.x |

When upgrading Cilium, bump this CRD version in the same PR.

The experimental channel is required at v1.2.0 because Cilium's operator
indexes `TLSRoute` at `v1alpha2`, which standard-install.yaml does not
ship at that version.

`--server-side` is required: the Gateway API HTTPRoute CRD spec exceeds
the 256 KB limit on the `kubectl.kubernetes.io/last-applied-configuration`
annotation that classic `kubectl apply` writes.

```bash
# renovate: datasource=github-releases depName=kubernetes-sigs/gateway-api
GATEWAY_API_VERSION=v1.4.1

kubectl apply --server-side -f "https://github.com/kubernetes-sigs/gateway-api/releases/download/${GATEWAY_API_VERSION}/experimental-install.yaml"
```

Cilium 1.19's installation docs split CRDs across standard + experimental
channels (TLSRoute is the only one from experimental). Applying
`experimental-install.yaml` is the superset and avoids piecemeal
kubectl-apply-per-CRD.

### Downgrading or version-skipping

`kubectl apply` cannot demote a CRD if it would remove a version listed
in the CRD's `status.storedVersions`. If you ever need to drop from a
newer Gateway API version to an older one (e.g. fresh-installed v1.5.x
when the controller only supports v1.4.1), delete the Gateway API CRDs
first, then apply the supported version. Only do this when there are no
Gateway/HTTPRoute resources in the cluster.

```bash
kubectl get gateway,httproute,tlsroute,grpcroute -A   # confirm empty

# v1.5+ ships a ValidatingAdmissionPolicy that blocks installing CRDs
# from before v1.5.0. If we are demoting from a v1.5+ install, drop it.
kubectl delete validatingadmissionpolicybinding safe-upgrades.gateway.networking.k8s.io --ignore-not-found
kubectl delete validatingadmissionpolicy        safe-upgrades.gateway.networking.k8s.io --ignore-not-found

kubectl get crd -o name | grep gateway.networking.k8s.io | xargs kubectl delete
kubectl apply --server-side -f "https://github.com/kubernetes-sigs/gateway-api/releases/download/${GATEWAY_API_VERSION}/experimental-install.yaml"
kubectl -n kube-system rollout restart deploy/cilium-operator
```

Verify:

```bash
kubectl get crd | grep gateway.networking.k8s.io
# expect 6 CRDs: gatewayclasses, gateways, httproutes, grpcroutes,
# tlsroutes, referencegrants
```

## What's in this directory

- `namespace.yaml` — `gateway-system` namespace
- `public-gateway.yaml` — public-facing Gateway (LB IP via Cilium BGP), wildcard TLS
- `private-gateway.yaml` — internal Gateway (LB IP via Cilium BGP), wildcard TLS
- `kustomization.yaml` — wires it together for ArgoCD

HTTPRoutes live in their owning app's directory (e.g.
`argocd/httproute.yaml`, `www/deploy/overlays/production/httproute.yaml`)
and attach to these Gateways via `parentRefs`.

## Upgrading the CRDs

1. Renovate opens a PR updating `GATEWAY_API_VERSION` in this file.
2. Read the [Gateway API release notes](https://github.com/kubernetes-sigs/gateway-api/releases)
   for breaking changes (rare for patch/minor on stable channel).
3. Merge the PR.
4. Run the `kubectl apply` command above on the cluster.
5. Confirm Cilium and existing Gateways still reconcile cleanly:
   ```bash
   kubectl get gatewayclass cilium
   kubectl -n gateway-system get gateway
   ```

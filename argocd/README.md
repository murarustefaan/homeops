# ArgoCD

## Installation

As per the [official documentation](https://argoproj.github.io/argo-cd/getting_started/):

```shell
kubectl create namespace argocd
kubectl apply -n argocd --server-side --force-conflicts -f https://raw.githubusercontent.com/argoproj/argo-cd/v3.3.2/manifests/ha/install.yaml
```

## Out-of-band ConfigMap settings

`argocd-cmd-params-cm` carries fields set manually (not via ArgoCD sync,
since the upstream manifest ships an empty `data` block and ArgoCD only
reverts fields present in its last-applied snapshot):

```shell
kubectl -n argocd patch cm argocd-cmd-params-cm --type=merge -p '{"data":{
  "redis.server": "argocd-redis-ha-haproxy:6379",
  "server.insecure": "true"
}}'
```

- `redis.server` points argocd-server at the HA Redis (HAProxy frontend).
- `server.insecure` makes argocd-server serve plain HTTP/h2c on 8080;
  TLS termination happens at the Cilium Gateway using the wildcard cert.
  Required for the `gitops.stefanmuraru.com` HTTPRoute to work.

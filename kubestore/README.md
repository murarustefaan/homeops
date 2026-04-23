## Installation

Replace installation name and values file with the correct one below.

```sh
helm upgrade \
    --install \
    --values hdd.values.yaml \
    --namespace democratic-csi --create-namespace \
    hdd-iscsi democratic-csi/democratic-csi
```

## hive (RWX, NFS — ArgoCD-managed)

Shared-access storage class. Use only where `ReadWriteMany` is mandatory —
everything else stays on `zoom` (iSCSI, RWO), which is faster for metadata
and small random I/O.

Driver config lives in 1Password (Infrastructure vault) and is projected into
the `democratic-csi` namespace by the `hive-driver-config` ExternalSecret.

TrueNAS prep (one-time):
1. Create datasets `zoom/compute/nfs-volumes` and `zoom/compute/nfs-snapshots`.
2. Services → NFS → enable, bind to the storage VLAN, NFSv4 on.

One ArgoCD Application with two sources backs this class: the kustomize path
contributes the ExternalSecret, the helm chart contributes the CSI driver.
`prune: false` — Argo will never garbage-collect the CSI driver or storage
class if git drifts. Deletion is a deliberate `helm uninstall` action.

Bootstrap once:

```sh
kubectl --context flipper apply -f kubestore/hive/deploy.app.yaml
```

Smoke-test with `kubestore/hive/pvc.yaml` (asks for `ReadWriteMany`).

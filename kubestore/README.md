## Installation

Replace installation name and values file with the correct one below.

```sh
helm upgrade \
    --install \
    --values hdd.values.yaml \
    --namespace democratic-csi --create-namespace \
    hdd-iscsi democratic-csi/democratic-csi
```
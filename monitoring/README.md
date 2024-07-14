# Monitoring setup

## Install

```shell
helm upgrade --install kube-prometheus-stack prometheus-community/kube-prometheus-stack --namespace monitoring -f values.yaml
```
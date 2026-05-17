# Kuvik ADC Distribution

Public distribution repository for **Kuvik ADC** (Application Delivery Controller).

This repository holds **release artifacts only** — Helm charts and container image references for installation on workload clusters. Source code lives in private repositories.

## Helm Chart: `kuvik-operator`

The Kuvik Operator runs on workload Kubernetes clusters and registers `LoadBalancer`-type Services with a Kuvik LB cluster via gRPC.

### Add the Helm repository

```bash
helm repo add kuvik https://kuvik-io.github.io/kuvik-adc/
helm repo update
```

### Install

```bash
helm upgrade --install kuvik-operator kuvik/kuvik-operator \
  --namespace kuvik-system --create-namespace \
  --set controllerGRPCAddress=<YOUR-CONTROLLER-NODE-IP>:30900 \
  --set clusterID=<CLUSTER-ID> \
  --set site=<SITE-LABEL> \
  --set grpc.operatorRegistrationToken=<TOKEN-FROM-UI> \
  --set-string grpc.caCert="<BASE64-CA-CERT>"
```

- `controllerGRPCAddress`: controller node IP and NodePort (default `30900`). If the operator runs in the same Kubernetes cluster as the controller, use `kuvik-controller.kuvik-system.svc:19000` instead.
- `clusterID`, `site`, `grpc.operatorRegistrationToken`, `grpc.caCert`: copy from the **Clusters → Add Cluster** wizard in the Kuvik Management UI.

### Uninstall

```bash
helm uninstall kuvik-operator --namespace kuvik-system
```

The chart ships a Helm `pre-delete` hook (`gracefulDeregister.enabled=true`) that calls `FullSync(empty)` against the controller, causing all `LBService` records for this cluster to be removed from the LB cluster automatically. If the controller is unreachable, the hook times out after 60 seconds and uninstall proceeds; orphaned records can then be cleaned up from the Management UI.

To skip the graceful deregister entirely (e.g. controller is permanently gone):

```bash
helm uninstall kuvik-operator --namespace kuvik-system --no-hooks
```

## Container Image

The operator image is published to GitHub Container Registry:

```
ghcr.io/kuvik-io/kuvik-adc/operator:<version>
```

## Versions

See [`index.yaml`](index.yaml) for the full chart catalog.

| Chart | Version | App Version |
|---|---|---|
| `kuvik-operator` | 0.6.1 | 0.6.1 |
